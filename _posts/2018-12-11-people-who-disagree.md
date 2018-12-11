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

But in fact deriving code in no way limits your ability to replace that derived code with custom code in the future. Adding an explicit transformation layer *when a transformation is actually needed* is 

