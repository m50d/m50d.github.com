---
published: false
title: Becoming More Functional
---
People on [/r/scala](https://www.reddit.com/r/scala/) sometimes ask how to make their Scala more functional, or about what "advanced" techniques they should learn. This is a list aimed at people who already follow the [twitter Scala style guide](http://twitter.github.io/effectivescala/), and want to know where to go from there. I'll assume [ScalaZ](https://github.com/scalaz/scalaz) is in scope (because I can't find the scaladocs for Cats); [learning ScalaZ](http://eed3si9n.com/learning-scalaz/7.0/) may be a useful reference for some things.

# Replace general `foldLeft` and friends with more specific operations

`foldLeft` is [a very general/powerful method which makes it hard to reason about](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html) - it's more or less as powerful as a general imperative `for` loop - though at least it reduces the scope of the "variables" and makes it explicit exactly what is being threaded through each iteration. Still, it's best to use more specific constructs when you can:

 * Sometimes there is simply a method for your case e.g. `find`, `forall` `exists`, `groupBy`, `min`, `max` and `partition`. ScalaZ `separate` also sees some use.
 * `reduce` should often be `suml` - you may need to define a `Monoid` instance for your type, or use [shapeless-scalaz](https://github.com/typelevel/shapeless-contrib) to derive one
 * `map` followed by `suml` is `foldMap`
 * `foldLeft` where the body includes a `flatMap` should usually be `traverse` (or `sequence`). This can sometimes be simplified further:
  * `traverse` followed by `map(_.suml)` is `foldMapM`
  * `traverse` followed by `map(_.flatten)` is `traverseM`
 
# Use standard `for`/`yield`-enabled types for "secondary" parameters and concerns

Often code has to manage a secondary concern as well as the primary thing it does. Often these are application-wide "cross-cutting concerns", e.g. audit logging, database transaction management, async I/O, or error handling - we can think of these as "effects". There is a tension between making these things explicit enough that the reader understands what the code is "actually" doing (and isn't [confused by "magic"](http://thecodelesscode.com/case/211) as in AOP approaches), and ensuring that the "happy path" and primary concern of the code remains clear (a difficulty in "handle errors where they happen" approaches).

Scala's `for`/`yield` offers a useful "third way": one can write a chain of `for { a <- f(); b = g(); c <- h() } yield ...` where the reader can clearly see where the secondary concerns are happening (the `<-` calls) but they don't obscure the straight-through control flow (and the function can remain single-entry/single-exit). To sequence effectful operations on collections, use the operations from the pervious section.

 * Code that produces a value and accumulate a secondary value (often a list) should be represented as ScalaZ `Writer`
  * This is particularly useful for structured logging, possibly with the [treelog](https://github.com/lancewalton/treelog) library
 * If you want to thread a secondary value through a series of function calls that also need to change that secondary value, use ScalaZ `State`
 * For validation-like code:
  * Want fail-fast? Use `Either` (or in pre-2.12 Scala, ScalaZ `\/` or Cats `Xor`)
   * If you need to integrate with a library that uses exceptions for failures, you can convert these into `Either` values using the constructs in `scala.util.control.Exception._`:
    * `catching(classOf[SomeSpecificException]) either someLibraryMethod` (returns `Either[SomeSpecificException, ...]`)
    * `nonFatalCatch either someLibraryMethod` (catches all the exceptions that are sensible to retry - everything except fatal system errors)
    * `catching(classOf[SomeSpecificException]) opt someLibraryMethod` (returns `Option[...]`)
  * Want to accumulate all failures? Use ScalaZ `Validation` and accept that you won't be able to use `for`/`yield`
   * `for`/`yield` can't accumulate all errors, because later validations are allowed to depend on the results of earlier ones, but if an earlier validation fails there's no input value for the later validation.
   * Look at applicative chaining (using `*>`) or "applicative builder syntax" (using `|@|`/`⊛`) instead.
  * Want to accumulate failures but still return a result value even if there are failures? Use ScalaZ `Writer`
   * `Writer` *can* use `for`/`yield` and accumulate all failures, because earlier validations always return a value even when there's a failure.
 * Have a piece of effectful code that you can't or won't model in detail, but still want to be able to pass around as a value (i.e. control when the effects happen)? Use ScalaZ `Task`.
 * Want to do async I/O? Use ScalaZ `Task`.
  * You can also use (standard library) `Future`, but beware that it *doesn't* control when the effects happen
   * `Future`s with effects inside them aren't generally values you can pass around and control when they actually happen - rather the effects (e.g. a web request) start immediately when the `Future` is instantiated
   * `Future` would make sense for pure computations. But async in general probably has more overhead than it's worth for cases where you're [working simultaneously rather than waiting simultaneously](http://yosefk.com/blog/working-simultaneously-vs-waiting-simultaneously.html) - where async shines is things like external web requests - and in those cases you usually want to control when the I/O happens.
   
# Use types to represent data, avoid branching

A If a value has two different states, make them two different types
  * e.g. a few months ago I had a bug where I passed a graph to a function that expected that graph to have been filtered by another function first. Solution: make the filtered graph and the unfiltered graph different types
 * Use shapeless-based typeclass derivation to avoid having to write any boilerplate for custom datatypes, freeing you to use as many types as you 
 * Avoid reflection. Things that use reflection to walk the object graph (e.g. serialization) are usually best replaced with typeclasses; use shapeless-based typeclass derivation to avoid the overhead of writing them by hand.
 * `match` constructs are [easy to write unsafely](http://typelevel.org/blog/2014/11/10/why_is_adt_pattern_matching_allowed.html#a-selector-subtlety) and can often be replaced with `fold` (e.g. never `match` an `Option` or an `Either` (unless you need to for `@tailrec`))
 * `if`/`else` is generally a sign that you want an ADT (`sealed trait`). So is a datastructure full of `Option`s or `Either`s, especially if there are invariants that relate them (e.g. "if `a` is `Some` then `b` is `Left`"). I recommend defining a `fold` method on your ADT as in the previous point.

 * Proxies/interceptors should be avoided. Any kind of "block" or "context" construct (e.g. a database transaction) should probably be represented as a value that you pass into a single method that does the open/close, so that you can't have a path where you forget to match them up. The free monad can give you a more lightweight way to represent your commands
 * double-`flatMap` (`flatMap { _.flatMap {... }}` or `flatMap { _.map { ... } }`) is sometimes a sign you should be using a monad transformer. Alternatively, if you're struggling to combine stacks of effects and nest `flatMap`s correctly, consider using a free coproduct approach instead.
 * if you're defining a lot of tree-like structures and find yourself repeating a lot of traversal code boilerplate, consider recursion-schemes style with fixed-point types.
 * `map(_.map(...))` (or similarly with flatMap) probably indicates you should be using a monad transformer
 * if a parameter is passed down untouched through several layers of function calls it might be better for those functions to return Reader
 * if you pass a "secondary" parameter in and out (by returning tuples) of a series of functions it might be better for those functions to return State
 * if a bunch of functions have "secondary" return values that are merged to be the "secondary" return value of their parent it might be better for them to return Writer
 * if you're foldLefting with a secondary state value that functions like any of the above types that's probably better expressed as foldMapA
 * anything that iterates over a collection can be expressed as foldLeft (and then apply the above)
 * anything that pattern matches on a sealed trait can be expressed as fold
 * if you do this and end up with `identity` or `{}` anywhere there might well be a more specialized method (e.g. `option.fold(...)(identity)` is just `option.getOrElse(...)`)
 * if you have a custom recursive data structure (e.g. a specialized tree) and traverse it in a way that ends up as a lot of nested flatmap calls it might be better to express the data structure as a fixed point and then use a standard traversal on it (recursion-schemes style). There's a lot of code overhead to using those techniques in Scala though, so it's probably not worth doing for a simple catamorphism etc.
 * if you have some construct that needs to only be "executed" in a particular context (i.e. certain things need to happen before and after) consider introducing a monad
 * if you have a sealed set of "commands" but also want to allow arbitrary pure functions that should be part of the same "execution", consider a free monad
 * if you're stacking a lot of "unrelated" monads maybe you want a free coproduct (kind of an open "research" area in Scala as in there are a bunch of libraries being written in search of the right way to do this - freek sounds promising)
 
 # Things to not bother with
 
 * Don't bother with sbt - use maven (until you need to cross-build libraries)
 * Don't bother with scalatest, or any fancy "tests in plain english" library - use plain junit
 * Don't use the cake pattern or reader monad. If you're using Spring or similar, keep using it for now - just make sure you use constructor injection rather than field injection.
 * Do make sure that you complete migrations of components, and have a clear border between Scala and Java. Jumping back and forth repeatedly in the same project seems fine to start with, but brings trouble in the long run
 * At the very start, keep all your existing libraries - once you have a component that's solidly on Scala you can start looking for better replacements for libraries that component uses.
 * The most useful libraries to change are anything you're currently using annotations for - you'll see an immediate return by removing the amount of "magic" in your codebase. Web backends and database access are good candidates.
 * Spark is great but only if you have so much data that you can't use anything else.
 * IMO there are no good web UI libraries for Scala - I stick to Wicket. There are excellent web API libraries though - my favourite is Spray.
 * I wouldn't use play, akka actors (except as an implementation detail that you never directly touch), or anything that uses typesafe-config if you can help it. These libraries have very poor typesafety which destroys the main advantage of using Scala. More generally any library designed to be used from Java is probably not worth bothering with.
 * If you think you want Actors you probably just want Futures.
 * `case class`es and type inference are common-sense and you can use them basically immediately.
 * pattern matching and multiple inheritance promote bad habits and may be worth just avoiding entirely.
 * fancier functions/types found in cats/scalaz need care. I wouldn't say ban them; I would say require code review (and maybe an explicit example of what the code would look like without that function/type) for each new concept so that you're only introducing new constructs where they pay their way.
 * This is doubly true of streaming libraries or anything else very advanced. They're very much worthwhile where needed, but make sure you're justifying them rather than cargo-culting.