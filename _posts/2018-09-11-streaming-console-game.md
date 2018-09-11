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

The original code relies on a global `AtomicBoolean` called `isGameOn`, which other threads then check to see if they should terminate. Again it's unclear what the flow of logic and data is here.