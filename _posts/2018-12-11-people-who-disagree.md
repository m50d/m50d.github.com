---
published: false
layout: default
title: >-
  People who disagree with you aren't trying to make things complex. Maybe
  you're wrong.
---
I recently saw Daniel Westheide's [The Complexity Trap: Think Before You Leap](https://danielwestheide.com/blog/2018/12/07/the-complexity-trap.html), and the more I think about it the more I disagree.

## Explicit JSON formatters are not simpler than typeclass derivation. Nor are DTOs

Westheide advocates writing large amounts of boilerplate code to map your domain objects to and from JSON. He claims that deriving equivalent code "almost always leads to strongly coupled systems that are very difficult to evolve."

But in fact deriving code in no way limits your ability to replace that derived code with custom code in the future. Adding an explicit transformation layer *when a transformation is actually needed* is essential complexity; adding an explicit transformation layer that does nothing in case you need a transformation in the future is the worst kind of accidental complexity, cargo cult decoupling. It may not take a lot of effort to write, but it takes a lot of effort to read and understand, and code is read more than it's written.

And what happens when you *do* introduce some transformations? Say one out of ten domain objects turns out to need a different representation on the server and the client. If you've been scrapping you boilerplate, this one object will be an obvious special case: there'll be a bunch of extra code that no other domain object has, a warning that "here be dragons". If you used manual formats for every case, that special case code will be buried under mountains of almost-but-not-quite-the-same code for every other domain object. A reader will learn that domain objects don't get transformed between the server and the client even though there's a lot of code to skip over, and this will be [almost](http://wiki.c2.com/?AlmostConsideredHarmful) true.

## Functional core, imperative shell is not simpler than Free monads or tagless final

A "functional core, imperative shell" model is great if you really can implement in "rather small, pure, and effect-free functions". But often there is essential complexity in the business domain that makes that impossible. Having business requirements that interleave pure operations with effectful ones - "if the request exceeds one of our free account limits then check the user's subscription is still active in this external service" - is normal, like it or not. If your functional core computes that you need to interact with an external service and then take one of several different computed actions based on the response, you have to figure out some kind of "command object" to represent that - which will usually end up being an ad-hoc, informally-specified implementation of half of the free monad.

In fact there's no tension between using Free monads or tagless final style and the "functional core, imperative shell" model - rather, the former is a technique for achieving the latter, without having to contort your program 