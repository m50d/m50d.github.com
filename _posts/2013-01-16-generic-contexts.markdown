---
layout: default
published: false
title: Generic Contexts, or: Don't Mention the M Word
---

I took a [Category Theory course](http://gopher.srcf.net:70/users/md401/pt3notes/category.pdf) at university, but in truth I never understood more than the basics. [Lampson's (or Wheeler's) Law](http://c2.com/cgi/wiki?ButlerLampson) applies just as strongly in mathematics, and I hit my personal indirection limit about a third of the way through.

Fortunately, in software we know how to avoid this: Agile. So let's talk through (an anonymized version of) a real-life problem I had at work, and the abstraction that can be used to solve it - no theory, just programming.

Suppose we have a series of functions that might fail - perhaps they're calls to remote webservices that could go down.

    val f1: {} => Option[Int] = ...;
    val f2: Int => Option[String] = ...;
    val f3: String => Option[Set[Int]] = ...;

We want to call them one after another, and return a failure if any of them fails, success if they're all succesful. The most naïve possible way to write our function looks something like:
    
    def doFunctionsInSequence1(): Option[Set[Int]] = {
      val r1 = f1(null)
      if(!r1.isEmpty) {
        val r2 = f2(r1.get)
        if(!r2.isEmpty)
          f3(r2.get)
        else None
        }
      else None
    }
    
Even typing this makes me feel ill (and it's nothing to do with the ifs and gets, as bad as they are - cases and pattern-matching would be fundamentally no better). As Scala programmers we should at least know to "flatmap that shit":

    def doFunctionsInSequence2(): Option[Set[Int]] =
      f1(null) flatMap f2 flatMap f3