---
published: false
layout: default
title: My Five Years of Scala
---

Scala is a broad church, as any general-purpose programming language ultimately must be.
But let me give my advice from my own 5.5 years of Scala,
as they have been rather different from [those described here](http://manuel.bernhardt.io/2015/11/13/5-years-of-scala-and-counting-debunking-some-myths-about-the-language-and-its-environment/).

## Avoid NPEs, avoid Exceptions

Avoiding NPEs is a great first step, but my Scala idiom applies the same reasoning
to all Exceptions. Even fans of exceptions advise against using them for control flow;
they make it much harder to reason about code you're reading, and even in a good IDE,
automated refactoring will often leave you with unwarranted `throws` declarations.
Using scalaz's unfortunately-named[1] `\/` type and `for`/`yield` is the perfect balance
between explicitness and conciseness; in a construct like:

    for {
      a <- method1()
      b = method2(a)
      c <- method3(b)
      _ <- if(c < 0) "Something's gone wrong".left else {}.right
      d <- (catching(classOf[SomeException]) either someJavaMethod(c)).disjunction
    } yield d
    
we can see clearly that `method1` and `method3` might error but `method2` never will,
but we can still write our code in "straight-through" style without polluting the
"happy path" with error handling.
Best of all our errors are just values, so they're guaranteed to refactor correctly -
and if not, the compiler will catch us.

I follow a similar set of rules regarding exceptions:

 1. Never `throw`
 2. Where possible, don't call a method that might throw
 (e.g. use `.headOption` on a `List` instead of `.head` - [wartremover](https://github.com/puffnfresh/wartremover) can enforce this)
 3. When using a library that might throw, wrap it with the `catching` construct

## One class should = one file

This is one case where I think Scala is a step backwards from the more opinionated Java.
The "one class = one file" rule makes it easy to find where any code is located.
Exceptions are `sealed` internal-only classes (similar to Java inner classes)
and `sealed trait`/`case class` patterns where the subclasses aren't intended to be
used explicitly.

## If mixing in traits, be very careful about initialization order

Scala avoids the [biggest pitfall of multiple inheritance](https://fuhm.net/super-harmful/) elegantly,
by only allowing one parent to have a constructor;
`class`es may only be the first parent, and `trait`s can't have constructors
that take arguments.
Unfortunately this doesn't mean they don't need to be initialized; in particular,
`val`s in a mixed-in `trait` will be `null` if you try to access them
in an earlier constructor.[2]
As with any `null`, in the worst cases you may not get an error until much later on.

I won't say to avoid multiple inheritance entirely, but use it sparingly,
and only when little initialization is required.
I find the "cake pattern" to be far more trouble than it's worth,
and while I still value composition over inheritance,
often I'll implement it the same way I would in Java - with composed objects as fields,
and explicit delegating method implementations.

## SBT really is an undocumented mess, but you don't have to use it

SBT offers precisely one compelling feature: cross-building for multiple Scala versions.
If you need this, there's sadly no alternative.
But for anything other than publicly-released libraries,
you can get away with supporting only one Scala major version,
and at that point you can stick with the wonderful Maven.
Just enable the maven scala plugin (and m2eclipse-scala if using eclipse)
and you're good to go, with declarative builds, well-documented configuration,
and perfect, instant IDE integration.

I'd say a similar thing about ScalaTest.
If you find Cucumber-like frameworks useful, then ScalaTest is a great one.
But if you don't like that kind of framework, it's fine to keep using JUnit.

## You don't need to learn much to get started...

If you want to approach Scala as a new language then you can.
But if you're not going to maintain an existing Scala codebase,
it's fine to just make a `.scala` file in your IDE and start typing Java there.
(I guess you need to learn `val` and `def`, but that's about it)
Save your semicolon finger, and then learn more as and when you need to.

Play is a fine framework.
But personally I prefer to stick with Wicket, and Scala makes that easy.
In my early days of Scala I kept using Spring,
and even today I still use Hibernate.

## ...but do adopt the idioms eventually

Writing Scala in one or two files is an easy first step, but getting a team onboard
is a big investment, particularly if you have coverage tools, runtime instrumentation
and the like. So you should get your money's (well, time's) worth,
and take advantage of the full power of Scala.
I'd point to things like [Treelog](http://typelevel.org/blog/2013/10/18/treelog.html) and [Doobie](https://www.youtube.com/watch?v=M5MF6M7FHPo)
that simply couldn't be implemented in any other mainstream language except Haskell[3].

The really wonderful part? Both of these things work with ordinary values.
So if I want to integrate them with e.g. Spray, I can make a routing directive
that says to perform the database operation that a given value represents.
And this will be a first-class directive, that I can use in a route definition
exactly like any of the built-in spray directives.
(Indeed the built-in spray directives are just code, and you can click through
to their definitions in your IDE and see exactly how they're implemented -
very different from working with JAX-RS annotations).
So I have database-session-in-view, but in a principled, refactor-safe way.
And I can tell whether I need to spin up an in-memory database for
one of my unit tests by simply reading it off from the type.
Similarly, I can write a spray `Marshaller` for a treelog-logged value
that performs content negotiation to decide whether to render the log
and this will integrate perfectly with the rest of spray's system e.g. `complete`.

I don't have to know anything about the internals of Treelog, Doobie or Spray. Everything is just an ordinary value, and I can tie them together using ordinary Scala.

## Pair-programming and code reviews

Even on teams where everyone was hired as a Scala developer,
there can be a wide range of skill levels,
and this goes triple when just adopting Scala.
I've found pair-programming the best way to learn, and probably the best way to teach.
And for an experienced dev chomping at the bit to use advanced libraries or features,
the most effective argument you can make is to look at some code under review,
and explain exactly how using an applicative functor (for example)
would save lines of code,
or how ordinary good engineering practices like factoring out repeated code
naturally lead to using a datatype that already exists in ScalaZ
(which happens extremely often).

## Notes

[1] It's aliased as `Disjunction`, but aliases have their own issues
e.g. they don't show up in type errors.
Worse is the inconsistency: the corresponding monad transformer is isn't `\/T` -
and it isn't `DisjunctionT` either.

[2] Really Scala's type system should be able to prevent this,
but I doubt that could be introduced in a backwards-compatible way.

[3] Should you adopt Haskell instead? Maybe. After five years' experience,
on a green field project,
you would probably be in a better place with Haskell than with Scala.
But it's very hard to get to there from here;
you can't write "Java in Haskell" to start with,
you can't reuse familiar frameworks,
and rightly or wrongly, many employers are less willing to try Haskell than Scala.
