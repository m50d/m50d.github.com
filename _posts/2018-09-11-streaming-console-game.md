---
published: false
layout: default
title: Streaming Console Game
---
I was pleased to see the library introduction in [eed3si9n's post about console games in Scala](http://eed3si9n.com/console-games-in-scala) - but horrified by the global mutable state used in the example code. Even with only tens of lines of code, it was difficult to follow the data flow and logic. So I forked the code at [https://github.com/m50d/console-game](https://github.com/m50d/console-game) and had a go at using [fs2](https://github.com/functional-streams-for-scala/fs2) to pass the state through explicitly, and I think the resulting code is simpler and clearer as well.