---
published: false
title: Becoming More Functional
---
People on [/r/scala](https://www.reddit.com/r/scala/) sometimes ask how to make their Scala more functional, or about what "advanced" techniques they should learn. This is a list aimed at people who already follow the [twitter Scala style guide](http://twitter.github.io/effectivescala/), and want to know where to go from there. I'll assume [ScalaZ](https://github.com/scalaz/scalaz) is in scope (because I can't find the scaladocs for Cats); [learning ScalaZ](http://eed3si9n.com/learning-scalaz/7.0/) may be a useful reference for some things. I will also use [kind-projector](https://github.com/non/kind-projector) syntax as this tends to be more readable even if one isn't actually using kind-projector.

# Use types to represent data, avoid branching

Types help you keep track of distinctions in your code - if a value has two different states, make them two different types. e.g. a few months ago I had a bug where I passed a graph to a function that expected that graph to have been filtered by another function first. Solution: make the filtered graph and the unfiltered graph different types.

 * `if`/`else` is generally a sign that you want an "ADT" (`sealed trait`). So is a datastructure full of `Option`s or `Either`s, especially if the state of one field affects that of another (e.g. "if `a` is `Some` then `b` is `Left`")
 * `match` constructs are [easy to write unsafely](http://typelevel.org/blog/2014/11/10/why_is_adt_pattern_matching_allowed.html#a-selector-subtlety) and can often be replaced with `fold` (e.g. never `match` an `Option` or an `Either` (unless you need to for `@tailrec`))
  * It might be worth defining your own `fold` methods on any custom `sealed trait`s.
 * If you find yourself writing a `fold` with `identity` or `{}` in one of the branches, see whether the datatype defines a more specialized method for that use case (e.g. `Option#getOrElse`).
 * Use shapeless-based typeclass derivation to avoid having to write boilerplate for custom datatypes
  * Particularly applicable to "walk the object graph"-like problems e.g. JSON serialization.
  * This is much safer than reflection (and higher-performance too) since it happens at compile time rather than run time, and can give you an error if you try to e.g. include a `File` in your JSON output.
 * Tentative: Use [matryoshka](https://github.com/slamdata/matryoshka) to avoid boilerplate for custom tree-like datatypes where you want to enable traversal.

# Replace general `foldLeft` and friends with more specific operations

`foldLeft` is [a very general/powerful method which makes it hard to reason about](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html) - it's more or less as powerful as a general imperative `for` loop - though at least it reduces the scope of the "variables" and makes it explicit exactly what is being threaded through each iteration. So while one of the first steps in making a codebase more functional is replacing looping constructs with `foldLeft` calls, a next step is often to replace that with a more specific/constrained construct:

 * Sometimes there is simply a method for your case e.g. `find`, `forall` `exists`, `groupBy`, `min`, `max` and `partition`. ScalaZ `separate` also sees some use.
 * `reduce` should often be `suml` - you may need to define a `Monoid` instance for your type, or use [shapeless-scalaz](https://github.com/typelevel/shapeless-contrib) to derive one
 * `map` followed by `suml` is `foldMap`
 * `foldLeft` where the body includes a `flatMap` should usually be `traverse` (or `sequence`). This can sometimes be simplified further:
  * `traverse` followed by `map(_.suml)` is `foldMapM`
  * `traverse` followed by `map(_.flatten)` is `traverseM`
 * If a `foldLeft` accumulates or modifies a "secondary" parameter along with its primary operation, often the clearest way to express this is using `Writer` or `State` (see the next section). In which case you can then often rewrite the `foldLeft` as a `traverse` as above.
 
# Use standard `for`/`yield`-enabled types for "secondary" parameters and concerns

Often code has to manage a secondary concern as well as the primary thing it does. Often these are application-wide "cross-cutting concerns", e.g. audit logging, database transaction management, async I/O, or error handling - we can think of these as "effects". There is a tension between making these things explicit enough that the reader understands what the code is "actually" doing (and isn't [confused by "magic"](http://thecodelesscode.com/case/211) or subtle differences in the handling of different effects as in AOP approaches), and ensuring that the "happy path" and primary concern of the code remains clear (a difficulty in "handle errors where they happen" approaches).

Scala's `for`/`yield` offers a useful "third way": one can write a chain of `for { a <- f(); b = g(); c <- h() } yield ...` where the reader can clearly see where the secondary concerns are happening (the `<-` calls) but they don't obscure the straight-through control flow (and the function can remain single-entry/single-exit). We can shift seamlessly between the "value perspective" (where the full effectful value is an ordinary value that we can reason about like any other value, and if necessary compose "manually" with `flatMap` - remember that `for`/`yield` is just a different syntax for `flatMap` chains) and the "happy path perspective" (where we write our code in "straight through style" and trust (in a compiler-verified way) that the secondary effects will be handled somewhere) as appropriate for a given piece of code.

Better still, there are well-known libraries of these types that have already been written, covering many of the common cases and making it easy for your colleagues to know exactly what any given effectful value represents. There are also library functions for managing effects. E.g. to sequence effectful operations on collections, use operations like `traverse` mentioned in the previous section. If you've defined a treelike datatype using matyroshka, the standard traversal operations on it will come with in "monadic" versions that work like `traverse` (i.e. they will perform the traversal using `flatMap` to compose the effects at each stage). Note that these library functions are also usable for any custom effects that conform to the standard interfaces.

 * Code that produces a value and accumulate a secondary value (often a list) should be represented as ScalaZ `Writer`
  * This is particularly useful for structured logging, possibly with the [treelog](https://github.com/lancewalton/treelog) library
 * If you want to thread a secondary value through a series of function calls that also need to change that secondary value, use ScalaZ `State`
  * One especially clear sign of this is if you're passing the secondary value into functions and getting a tuple of (primary result, new secondary value) back.
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
 * Need to pass a read-only "context" value down through your business-logic layers even though it's only going to be used at low level? ScalaZ `Reader` might be appropriate.
  * This can be used as a replacement for global constants / `object`s / singletons or even as a form of dependency injection, but beware of overusing it. If an `object` contains no business logic I would leave it as a "global static" `object`, and only move towards a `Reader` style if you actually need to pass different values on occasion (e.g. test stubs).
  * IMO conventional dependency injection can be a relatively benign form of "magic" provided you:
   * Use constructor injection (rather than field injection), so that object constructors and fields still behave as expected. In Scala this is if anything more concise, and it ensures that objects can still easily be constructed "normally" e.g. for unit testing.
   * If there is the possibility of tooling (e.g. "find references") not knowing about the DI mechanism, ensure that there is some visible marker on classes that are constructed through DI so that a reader can immediately see this class is instantiated in a non-standard way.
   * Spring example: `@Component class MyService @Autowired (someDependency: SomeDependency)`
   * For a "green field" project, [MacWire](https://github.com/adamw/macwire) is a good pure-Scala option.
  * That said, manual object construction is lightweight enough in Scala that I generally prefer to do my DI "by hand".
  * I appreciate the theoretical elegance of the "cake pattern", but I find it's too much (code-level) overhead to use in practice.
 * Have a piece of effectful code that you can't or won't model in detail, but still want to be able to pass around as a value (i.e. control when the effects happen)? Use ScalaZ `Task`.
 * Want to do async I/O? Use ScalaZ `Task`.
  * This is usually better than using akka actors, since `Task` is typesafe and you can keep reasoning about functions rather than having to think about messages.
   * If your async tasks need to access isolated pieces of state concurrently, they can safely use traditional Java tools e.g. `AtomicInteger`, `LongAdder`, `ConcurrentHashMap`, `AtomicReference`.
   * Actors are only useful if you have two or more pieces of state that you need to access concurrently but also always keep in sync, IME. (Or if you need akka's distribution functionality)
  * You can also use (standard library) `Future`, but beware that it *doesn't* control when the effects happen
   * `Future`s with effects inside them aren't generally values you can pass around and control when they actually happen - rather the effects (e.g. a web request) start immediately when the `Future` is instantiated
   * `Future` would make sense for pure computations. But async in general probably has more overhead than it's worth for cases where you're [working simultaneously rather than waiting simultaneously](http://yosefk.com/blog/working-simultaneously-vs-waiting-simultaneously.html) - where async shines is things like external web requests - and in those cases you usually want to control when the I/O happens.
 * Have operations that need to happen in some kind of "block" or "context"? (e.g. a database transaction) Represent the operations as a value that you pass into a single method that does the open/close, so that you can't have a path where you forget to match them up.
  * This is often a good replacement for "magic" proxies/interceptors (based on method annotations, XML pointcuts or similar)
  * At its simplest the value could just be a function (or a `Task` created using `Task.delay`)
   * In that case you have to be careful not to allow the context to escape (e.g. a file handle that will be closed at the end of the block)
    * Tentative: there is a [theoretical technique for avoiding this](https://apocalisp.wordpress.com/2010/07/02/higher-rank-polymorphism-in-scala/), but I don't think there's a practical library for it yet.
  * If you want a more declarative/introspectable/testable way to express your commands, define a custom ADT (`sealed trait`).
  * If you want to allow "composite" commands connected by functions (so that you can pass around e.g. a series of database operations to be executed in a single transaction), the Free monad is a way to do this without any boilerplate. You define the "primitive" operations in your ADT (e.g. `Load`/`Save`), and then building composite operations out of them (using `for`/`yield` syntax) is supported without any further code.
   * Tentative: If you want to allow composite commands but ensure they can be executed in parallel, consider `FreeAp`.
    * Good ways of writing "partially parallel" command structures are an open research area. [Fetch](http://www.47deg.com/blog/fetch-scala-library/) is probably the current state of the art; I'm hoping to explore some of the possibilities in my coming [tierney](https://github.com/m50d/tierney) project.
    
# Tentative: Combining multiple `for`/`yield`-oriented types

A theory I'm considering lately is that effects are only ever problematic when two or more effects interact. E.g. implicit, pervasive, unmanaged state mutation is fine on its own. Implicit, pervasive, unmanaged asynchronicity is fine on its own. But [the interaction of both is extremely difficult to debug](https://glyph.twistedmatrix.com/2014/02/unyielding.html). So traditional imperative programming allows working with one effect at a time, but no more.

The techniques in the previous section provide a huge advance over this, because they make it practical to work with two effects at once: one effect that you're managing via `for`/`yield`, and one implicit, pervasive, global effect. This is probably enough for most programs and even for many libraries (which potentially have to deal with user-defined effects if they ever accept callbacks or similar), as demonstrated by the fact that the techniques for dealing with more have only really been developed in the last few years, at least in Scala. (Haskell handles I/O as an explicitly managed effect, so without these techniques Haskell programs would expend their `for`/`yield`-equivalent managing I/O and have difficulty expressing explicit management/sequencing of other effects, at least in code that was also performing I/O). Certainly everything in this section is tentative/experimental. However, a limit of two effects seems decidedly inelegant, and can become a practical issue as a codebase gets large enough or wants to manage many effects explicitly. So:

 * Double-`flatMap` (`flatMap { _.flatMap {... }}`, or similar constructs involving `map` e.g. `flatMap { _.map { ... } }`) is often a sign that you should be using a monad transformer.
 * The simplest way to work with monad transformers is usually to define a single consistent "stack" that you can use throughout your application.
  * E.g. `EitherT[WriterT[Task, Vector[AuditEvent], ?], ValidationError, ?]` for an application that needs to record audit events, report validation errors, and perform async I/O.
  * You can define type aliases for your stack, and helper methods for "lifting" single effects into a complete stack:
   * `type Action[A] = EitherT[WriterT[Task, Vector[AuditEvent], ?], ValidationError, A]`
   * `def log(ae: AuditEvent): Action[Unit] = EitherT.rightU[ValidationError](WriterT.put(Task.now({}))(Vector(ae)))`
 * If you need to write code that can be reused in different effect stacks (in different parts of your application which have different effect stacks e.g. with/without database access, or because you're writing a library that will be used with a user-provided effect stack), you can write it in a "stack-generic" form using a typeclass constraint:
  * `def log[F[_]](ae: AuditEvent)(implicit mt: MonadTell[F, Vector[AuditEvent]]): F[Unit] = mt.tell(Vector(ae))`
  * You can also put the `F[_]` type parameter on a service class.
   * Accept dependencies parameterized by the same `F`: `class MyService[F[_]: MonadTell[?, AuditEvent](dependentService: DependentService[F])`
   * This also lets you instantiate with a minimal `F` when testing (e.g. just `Writer[Vector[AuditEvent], ?]` in this case), and then the full `F` for "live". 
