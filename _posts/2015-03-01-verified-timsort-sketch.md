---
published: false
---

Some commentors on [this lobste.rs story](https://lobste.rs/s/blokto/an_haskell_implementation_of_timsort_exhibits_the_same_bug_as_python_and_java_implementations) were interested in what an implementation of TimSort that was verified at the (language) type level would look like; as a Scala programmer I volunteered to provide one. This is *not* a full proof like the [published paper](http://www.envisage-project.eu/timsort-specification-and-verification/); rather it's a sketch of what the critical example might look like in Scala.

Recall that the loop invariant is given as
````java
/*@ private invariant 
  @   (\forall int i; 0<=i && i<stackSize-4; 
  @                      runLen[i] > runLen[i+1] + runLen[i+2]))
  @*/
````
