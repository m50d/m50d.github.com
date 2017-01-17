---
published: false
title: Becoming More Functional
---
People on [/r/scala](https://www.reddit.com/r/scala/) sometimes ask how to make their Scala more functional, or about what "advanced" techniques they should learn. This is a list aimed at people who already follow the [twitter Scala style guide](http://twitter.github.io/effectivescala/), and want to know where to go from there. I'll assume the use of [cats](http://typelevel.org/cats/).

 * reduce should often be sum (introducing a monoid instance for that type)
 * map followed by sum is foldMap
 * foldLeft where the body includes a flatMap should be written as traverse
 * map(_.map(...)) (or similarly with flatMap) probably indicates you should be using a monad transformer
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
 * reduce is often a sign that you should define a monoid and use monoid sum.
 * `map` followed by sum is `foldMap`
 * folds that use `flatMap`s can usually be replaced with `traverse`/`sequence`
 * other folds can sometimes be replaced by specific methods e.g. `find`, `exists`, `groupBy`. Basically `foldLeft` is a very general/powerful method which makes it hard to reason about, it's best to use more specific things whenever you can.
 * `match` constructs are easy to write unsafely and can often be replaced with `fold` (e.g. never `match` an `Option` or an `Either` (unless you need to for `@tailrec`))
 * `if`/`else` is generally a sign that you want an ADT (`sealed trait`). So is a datastructure full of `Option`s or `Either`s, especially if there are invariants that relate them (e.g. "if `a` is `Some` then `b` is `Left`"). Define a `fold` method on your ADT
 * if there are two different states, make them two different types (e.g. a few months ago I had a bug where I passed a graph to a function that expected that graph to have been filtered by another function first. Solution: make the filtered graph and the unfiltered graph different types).
 *  folds (or just regular code flow) that produce a value and accumulate a secondary value (e.g. a list) should be represented as `Writer`
 * Any construction that threads a secondary "state" value through should be represented as `State`
 * Cheat sheet for validation-like code:
   * Want fail-fast? Use `Either` (or in pre-2.12 Scala, `\/` or `Xor`)
   * Want to accumulate all failures? Use `Validation` and accept that you won't be able to use `for`/`yield`
   * Want to accumulate failures but still return a value in the failure case? Use `Writer`
 * Avoid reflection. Things that use reflection to walk the object graph (e.g. serialization) are usually best replaced with typeclasses; use shapeless-based typeclass derivation to avoid the overhead of writing them by hand.
 * Proxies/interceptors should be avoided. Any kind of "block" or "context" construct (e.g. a database transaction) should probably be represented as a value that you pass into a single method that does the open/close, so that you can't have a path where you forget to match them up. The free monad can give you a more lightweight way to represent your commands
 * `foreach` should usually be replaced by `traverse`/`sequence` using a value to represent the effect (e.g. `Task`)
 * double-`flatMap` (`flatMap { _.flatMap {... }}` or `flatMap { _.map { ... } }`) is sometimes a sign you should be using a monad transformer. Alternatively, if you're struggling to combine stacks of effects and nest `flatMap`s correctly, consider using a free coproduct approach instead.
 * if you're defining a lot of tree-like structures and find yourself repeating a lot of traversal code boilerplate, consider recursion-schemes style with fixed-point types.
 
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
   * Shapeless is mostly hacks around stuff that should have been in the language; when you need it you need it, and it's relatively "non-infectious", so don't worry much about using it.
