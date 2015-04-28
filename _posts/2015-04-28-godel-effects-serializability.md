---
published: false
---

Programming in a pure language can look a lot like writing mathematical proofs in [Robinson Arithmetic](http://en.wikipedia.org/wiki/Robinson_arithmetic). Ordinary values are, well, ordinary values, but there is no way to be sure: a parameter passed to your function might secretly be a "nonstandard" value like `IO[A]`.

This is one of the things that makes passing values around so hard. In a language that makes a clear distinction between code and data, the solution is simple: data is serializable and code is not. You can then build structured logging, 