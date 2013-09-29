---
published: false
title: An Arrow that leads back to where we started
---

I used a Kleisli arrow for the first time at work yesterday, and it was good. (It would have been better if Scala made it easier to define object methods in pointfree style, but that's a topic for another post). But it forced me to consider anew my initial objection to monads - that they may be full of sound and fury, but signify very little.

The monad is a very simple and general abstraction, but at the moment it's most commonly used to enable purer functional programming, i.e. greater control over efects. At first glance this sounds reasonable: whether a given function performs I/O, or hands part of itself off to another thread, or could fail in a particular way ginving a particular (strongly-typed) error, is important information that it would be nice to make obvious in a function's signature.