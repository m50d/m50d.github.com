---
published: false
layout: default
title: Streaming Console Game
---
I was pleased to see the library introduction in [eed3si9n's post about console games in Scala](http://eed3si9n.com/console-games-in-scala) - but concerned by the global mutable state used in the example code. Even with only tens of lines of code, it was difficult to follow the data flow and logic. So I forked the code at [https://github.com/m50d/console-game](https://github.com/m50d/console-game) and had a go at using [fs2](https://github.com/functional-streams-for-scala/fs2) to pass the state through explicitly, and I think the resulting code is simpler and clearer as well.

It's the nature of a refactoring post to criticise the existing code, but please don't take this as attack on the original post. I think Scala has a lot of unexplored potential for this kind of application and I'm very grateful to eed3si9n for starting the conversation and showing us how to make this kind of program.

## Input stream rather than a stateful thread

eed3si9n's original code looked like this.

````scala
  // inside a background thread
  val inputHandling = Future {
    val km = KeyMap.keyMaps().get("vi-insert")
    while (isGameOn.get) {
      blocking {
        val c = reader.readBinding(km)
        val k: Either[Operation, String] =
          if (c == Operation.SELF_INSERT) Right(reader.getLastBinding)
          else Left(c match { case op: Operation => op })
        keyPressses.add(k)
      }
    }
  }
````

I was confused by the use of a single `Future` that never actually completes, and by the global `keyPressses` (sic) queue. Looking it up it turned out to be `val keyPressses = new ArrayBlockingQueue[Either[Operation, String]](128)`, which could then be read from anywhere, bypassing the usual function call input/output relationship. I rewrote this as a stream that simply emits the `Either` values:

````scala
    val km = KeyMap.keyMaps().get("vi-insert")
    Stream.repeatEval(IO {
      val c = reader.readBinding(km)
      if (c == Operation.SELF_INSERT) Right(reader.getLastBinding)
      else Left(c match { case op: Operation => op })
    })
````

We've now got a standalone value that we could test in isolation.

I'm normally skeptical about the value of the `IO` monad - I find most of the effects that one cares about sequencing are more specific than I/O, and in a lot of typical programming contexts one doesn't particularly care about the sequencing of I/O (e.g. one usually doesn't care about the order in which logging or reading from a "static" filesystem happens). But a console game is exactly the environment where `IO` shines: we have a lot of operations that interact with stdout/stdin, and we care deeply about interleaving them in the correct order.

## Terminating the stream rather than global `isGameOn`

The original code relies on a global `AtomicBoolean` called `isGameOn`, which other threads then check to see if they should terminate. Again it's unclear what the flow of logic and data is here. The difficulty in factoring this more cleanly is that the logic for determining whether it's time to quit is entangled with the logic for determining a change to the game state. I resolved this by making `handleKeypress` return an `Option`:

````scala
  def handleKeypress(k: Either[Operation, String]): Option[GameState => GameState] =
    k match {
      case Right("q") | Left(Operation.VI_EOF_MAYBE) =>
        None
      // Left arrow
      case Left(Operation.BACKWARD_CHAR) =>
        Some { g =>
          val pos0 = g.pos
          g.copy(pos = (pos0._1 - 1, pos0._2))
        }
      ...
````

This does add a bit of boilerplate to the `case`s, but it means the concerns are clearly separated, and reveals an important property about the data flow: we can tell whether a keypress is a "quit" or not before we've even begun to look at the game state.

Our input stream can now be transformed to terminate appropriately:

````scala
  .map(handleKeypress)
    .unNoneTerminate
````

and again this becomes something we can test in isolation, without even needing access to the game state. Conversely we can test whether a keypress translates into the correct operation on a game state without involving the global `isGameOn` variable.

## Pass `Ansi` values around until we're ready to print them

The original code has some nicely structured logic to build up a `drawing` value. Sadly there's no way to access that value, as the function immediately calls a global `println` function:

````scala
  def drawGame(g: GameState): Unit = {
    val drawing: BuilderHelper[Ansi, Unit] =
      for {
        _ <- Draw.drawBox(2, 2, 20, 10)
        _ <- Draw.drawBlock(g.pos._1, g.pos._2)
        _ <- Draw.drawText(2, 12, "press 'q' to quit")
      } yield ()
    val result = drawing.run(Ansi.ansi())._1
    AnsiConsole.out.println(result)
  }
````

In general there are parts of the code that pass `Ansi` values and parts that print immediately, mixed more or less indiscriminately. Let's rewrite this function to return the value, making it more testable, and we'll look to have a single controlled place where we use that global `AnsiConsole`:

````scala
  def drawGame(g: GameState): Ansi =
    (for {
      _ <- Draw.drawBox(2, 6, 20, 6)
      _ <- Draw.drawBlock(g.pos._1, g.pos._2)
      _ <- Draw.drawText(2, 12, "press 'q' to quit")
    } yield ()).run(Ansi.ansi()).value._1
````

(Note also a slight change as I replaced the custom `BuilderHelper` type with standard `State`).

Similarly, the `info` function can return an `Ansi` value rather than doing an uncontrolled `println`.


## Unpicking the main loop

The original code combines several concerns - updating the game state, displaying the current state, and waiting for the next tick (via a blocking `Thread.sleep`!) - in a single `while` loop:

````scala
  while (isGameOn.get) {
    while (!keyPressses.isEmpty) {
      Option(keyPressses.poll) foreach { k =>
        gameState = handleKeypress(k, gameState)
      }
    }
    tick += 1
    if (tick % 10 == 0) {
      info("something ".concat(tick.toString))
    }
    drawGame(gameState)
    Thread.sleep(100)
  }
````

Better to pull these out into individual functions and values that can be inspected and tested separately. We can form the ticks into a stream, eliminating the `tick` global variable:

````scala
val ticks = Stream.unfoldEval[IO, Int, Int](0) { tick => IO.sleep(100 milliseconds).map(_ => Some((tick + 1, tick + 1))) }
````

(unfortunately it seems like we have to repeat the `tick + 1`, as there's no variant of `unfold` that uses the emitted value as the next input).

Then we can examine and test the `ticks` stream alone. That said, we do want to combine it with the input stream for use - let's form a merged stream of `Left`s and `Right`s (I'm surprised the `merge` functions don't do this wrapping for us):

````scala
val inputAndTicks = inputHandling map Left.apply mergeHaltL (ticks map Right.apply)
````

(Note the `mergeHaltL`: we want the combined stream to halt as soon as the input halts i.e. when the user presses `q`)

Now the actual game loop can be very simple: we start from the starting game state, if we have an input step we update the game step, if we have a tick then we (sometimes) build an info message and draw the game. Building the info message should probably be pushed into its own function - in this example I'd move it into the `ticks` stream, but in a real codebase I'm assuming we'd want to have access to the `gameState` to build the `info` message.

````scala
val gameAnsis = inputAndTicks.mapAccumulate(GameState(pos = (6, 7))) {
    case (gameState, Left(step)) =>
      (step(gameState), Seq.empty)
    case (gameState, Right(tick)) =>
      (gameState, (if (tick % 10 == 0)
        Seq(info("something ".concat(tick.toString)))
      else Seq.empty) :+ drawGame(gameState))
  }
````

Even this far out into the "shell" of our program, we're still mostly in a world of functions and values: this is a stream of `IO` effects that can yield sequences of `Ansi`s. I found it easiest to emit sequences here and then have a tiny tidy-up step to turn this into a stream of `Ansi`s:

````scala
  .flatMap {
    case (_, ansis) => Stream.emits(ansis)
  }
````

We put off actually running our effects until the last possible moment, just a couple of lines of plumbing:

````scala
  Stream.emit(eraseScreen).append(gameAnsis)
    .observe1(ansi => IO {
    AnsiConsole.out.println(ansi)
  }).compile.drain.unsafeRunSync()
````

## Final thoughts

I was expecting to have to add a lot of boilerplate to sequence effects carefully, and justify the value in terms of maintainability. While that's definitely true of the `handleKeypress` changes, for the most part I was pleasantly surprised: removing global variables and factoring separate concerns into their own functions often clarified the code and made the logic clearer. The fs2 version of the app is a couple of lines shorter than eed3si9n's, but far more importantly the functions are shorter and I've eliminated the global variables entirely. If this had meant a few more lines of code I would consider it a price worth paying; to have also saved on overall code length means these changes really are win-win.

On a separate note, fs2 is underdocumented and still in a state of flux. Even in this small a codebase, a couple of the functions I was using have changed name in between the latest released version and the code currently on GitHub - and I was constantly referring to GitHub since there was often no other documentation for functions I wanted to use. Having a solid understanding of the iteratee abstraction helps up to a point, but often it's not clear whether a given operation is available or what it would be called if it was. The selection of operations available on `Stream` is simultaneously overwhelming and with obvious holes, because there's often a combinatorial explosion of aspects that you'd expect to be possible in the same transformation step: e.g. there is `flatMap` but no `flatMapAccumulate` or `evalFlatMapAccumulate`; there is `scanSegmentsOpt` (renamed in the next version) but no `scanOpt` or `mapOpt`.

I can't really see a way to fix this - defining the most general versions and having the less general versions as delegates could help a little, but defining all the possible restricted variants would still be overwhelming. At the same time, the currently available selection seems like a random grab-bag from the available possibilities, in need of a more principled organisation. I suspect the current implementation focuses on those operations which can be implemented efficiently, which makes sense for some use cases but not for a case like this - here we're far more concerned about control and code clarity than raw performance. 