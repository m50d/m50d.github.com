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
in an earlier constructor. ()

[1] It's aliased as `Disjunction`, but aliases have their own issues
e.g. they don't show up in type errors.
Worse is the inconsistency: the corresponding monad transformer is isn't `\/T` -
and it isn't `DisjunctionT` either.