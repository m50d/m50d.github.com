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

This is a very concise notation that Scala doesn't have. I've opted to go the other way and produce a very explicit invariant-witness type.

````scala
  trait LoopInvariant[RunLen <: HList, StackSize <: Nat] {
    type StackSizeMinusFour <: Nat
    val witness: Diff[StackSize, _4] { type Out = StackSizeMinusFour }
    def at[I <: Nat](i: I)(lt: LT[I, StackSizeMinusFour]): {
      type RunLenI <: Nat
      val runLenIWitness: Selector[RunLen, I] { type Out = RunLenI }
      type RunLenIPlusOne <: Nat
      val runLenIPlusOneWitness: Selector[RunLen, Succ[I]] { type Out = RunLenIPlusOne }
      type RunLenIPlusTwo <: Nat
      val runLenIPlusTwoWitness: Selector[RunLen, Succ[Succ[I]]] { type Out = RunLenIPlusTwo }
      type RunLenIPlusOnePlusRunLenIPlusTwo <: Nat
      val runLenIPlusOnePlusRunLenIPlusTwoWitness: Sum[RunLenIPlusOne, RunLenIPlusTwo] { type Out = RunLenIPlusOnePlusRunLenIPlusTwo }

      val witness: LT[RunLenIPlusOnePlusRunLenIPlusTwo, RunLenI]
    }
  }
````

Please don't think every encoding of that invariant into a type system must look like this! Type-level functions are not a first-class feature in Scala, so we have to encode each one as a pair: a `type` showing the output value, and a `val` that's an instance of a suitable type which "witnesses" that our output value is the output of the function that we want it to be.
