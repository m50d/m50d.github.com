---
published: false
---

If we're living in a functional programming renaissance, no word emblemizes it more than "monad" - for fans and skeptics alike. Arguments rage over whether monads are indispensible tools or pointless sophistry; you'd wonder whether people were even talking about the same thing. I suspect that, largely, they aren't.

As Alan Perlis said, a language that doesn't affect the way you think about programming is not worth knowing. A programming language should be a thesis, or a manifesto: a controversial claim about programming (in the same way as facebook's "move fast and break things"). The Haskell thesis goes something like:

 1. Reading and debugging code is important
 1. Some very simple structures occur lots of times in lots of programs, and particular manipulations of those structures are done again and again; we can make programs simpler by providing libraries for operating on these structures
 1. Most of the time, when there is a bug in a program, it involves a certain effect. If we knew what effects every method had, we would be able to narrow down most bugs very quickly
 1. Using our structure libraries, it actually takes very little effort to track most of the effects we might be interested in
 1. Therefore, we should always write code that keeps track of all possible effects

## Reading and debugging

