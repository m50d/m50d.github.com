---
published: false
layout: default
title: Streaming Console Game
---
I was pleased to see the library introduction in [eed3si9n's post about console games in Scala](http://eed3si9n.com/console-games-in-scala) - but horrified by the global mutable state used in the example code. Even with only tens of lines of code, it was difficult to follow the data flow and logic. So I forked the code at [https://github.com/m50d/console-game](https://github.com/m50d/console-game) and had a go at using [fs2](https://github.com/functional-streams-for-scala/fs2) to pass the state through explicitly, and I think the resulting code is simpler and clearer as well.

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

Then we can examine and test the `ticks` stream alone. That said, we do want to combine it with the input stream for use - let's form a merged stream of `Left`s and `Right`s:

````scala
val inputAndTicks = inputHandling map Left.apply mergeHaltL (ticks map Right.apply)
````

(Note the `mergeHaltL`: we want the combined stream to halt as soon as the input halts i.e. when the user presses `q`)

