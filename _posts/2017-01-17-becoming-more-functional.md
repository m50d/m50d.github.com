---
published: false
title: Becoming More Functional
---
I've been doing Scala for 7 years now and I'm still figuring out new things. I think it's important to realise that a codebase is never "done", and that this year's code will always be better than last year's code - rather than trying to do a bunch of up-front refactorings it's better to adopt an idea of continuous improvement where as and when you work in a given area you bring it up to the current standards.

Your list is pretty good, and I think the best practices you link is good. Some next steps I'd say are:

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