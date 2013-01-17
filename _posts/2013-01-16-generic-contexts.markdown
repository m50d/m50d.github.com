---
layout: default
published: false
title: Generic Contexts, or: Don't Mention the M Word
---

I took a [Category Theory course](http://gopher.srcf.net:70/users/md401/pt3notes/category.pdf) at university, but in truth I never understood more than the basics. [Lampson's (or Wheeler's) Law](http://c2.com/cgi/wiki?ButlerLampson) applies just as strongly in mathematics, and I hit my personal indirection limit about a third of the way through.

Fortunately, in software we know how to avoid this: Agile. So let's talk through (an anonymized version of) a real-life problem I had at work, and the abstraction that can be used to solve it - no theory, just programming.

##Level 1: Flatmap that Shit

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
    
Even typing this makes me feel ill - and replacing the ifs and gets with cases and pattern-matching, while more idiomatic, would be no better. There are too many branches in this code (as would show up under measures like NPath or Cyclomatic complexity) when fundamentally it's just running through a flat sequence of functions:

    def doFunctionsInSequence2(): Option[Set[Int]] =
      f1(null) flatMap f2 flatMap f3

##Level 2: "for", not just for collections any more

Flatmap is great when each step is an existing function - but when each stage is a handful of statements that don't make sense as a function in their own right, we can easily end-up in callback hell with the so-called [pyramid of doom](http://raynos.github.com/presentation/shower/controlflow.htm?full#PyramidOfDoom)

    def doFunctionsInSequence3(): Option[Set[Int]] = {
      f1(null) flatMap {i: Int => f2(i + 3) flatMap {
        s:String => 
          val t = s.toUpperCase + i
          f3(t + s)
        }
      }
    }

This can get especially bad when using futures, as the point at which we want to perform an asynchronous step is often not a clean break in the logic of the code.

    def webApi1(s: String): Future[Int] = ...
    def webApi2(i: Int): Future(Set[Int]) = ...
    def webApi3(i: Int): Future[String] = ...
    
    def fetchSomeData(username: String): Future[Set[Int] =
      webApi1("hello") flatMap {
        userId: Int =>
          val alternateFormatId = reformat(userId)
          logSomething()
          val ret = webApi3(alternateFormatId) flatMap {
            api3Output: String =>
              val objectRepresentation = parseJson(api3Ouptut)
              val keyForApi2 = if(objectRepresentation.someFlag) objectRepresentation.field1(alternateFormatId)
                else objectRepresentation.wrapper2(userId).field1
              val ret = webApi2(keyForApi2)
              logSomethingElse()
              ret
            }
        }