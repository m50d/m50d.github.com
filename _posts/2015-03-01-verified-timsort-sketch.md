---
published: false
---

Some commentors on [this lobste.rs story](https://lobste.rs/s/blokto/an_haskell_implementation_of_timsort_exhibits_the_same_bug_as_python_and_java_implementations) were interested in what an implementation of TimSort that was verified at the (language) type level would look like; as a Scala programmer I volunteered to provide one. This is *not* a full proof like the [published paper](http://www.envisage-project.eu/timsort-specification-and-verification/); rather it's a sketch of what the critical example might look like in Scala.

Recall that the loop invariant is given in KeY as

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

On to the implementation, which in Java looks like:

````java
private void newMergeCollapse() {
  while (stackSize > 1) {
    int n = stackSize - 2;
    if (n > 0   && runLen[n-1] <= runLen[n] + runLen[n+1] || 
        n-1 > 0 && runLen[n-2] <= runLen[n] + runLen[n-1]) {
      if (runLen[n - 1] < runLen[n + 1])
        n--;
    } else if (n<0 || runLen[n] > runLen[n + 1]) {
      break; // Invariant is established
    }
    mergeAt(n);
  }
}
````

The if-else branches are another piece that's a poor fit for Scala. To know which branch is taken at the type level, we have to move each branch into its own implicit instance of a new trait:

````scala
trait LoopStep[RunLen <: HList, StackSize <: Nat, N <: Nat] {
  type NewRunLen <: HList
  type NewStackSize <: Nat
  def doLoop(runLen: RunLen, stackSize: StackSize, currentInvariant: LoopInvariant[RunLen, StackSize]): (NewRunLen, NewStackSize, LoopInvariant[NewRunLen, NewStackSize])
}

implicit def ZeroCase[RunLen <: HList, StackSize <: Nat] = new LoopStep[RunLen, StackSize, _0] {
  type NewRunLen = RunLen
  type NewStackSize = StackSize
  def doLoop(runLen: RunLen, stackSize: StackSize, currentInvariant: LoopInvariant[RunLen, StackSize]) =
    (runLen, stackSize, currentInvariant)
}

implicit def InvariantEstablishedCase[RunLen <: HList, StackSize <: Nat, N <: Nat, RunLenN <: Nat, RunLenNPlusOne <: Nat](
  implicit runLenNWitness: Selector[RunLen, N] { type Out = RunLenN }, runLenNPlusOneWitness: Selector[RunLen, Succ[N]] { type Out = RunLenNPlusOne },
  ltWitness: LT[RunLenNPlusOne, RunLenN]) = new LoopStep[RunLen, StackSize, N] {
  type NewRunLen = RunLen
  type NewStackSize = StackSize
  def doLoop(runLen: RunLen, stackSize: StackSize, currentInvariant: LoopInvariant[RunLen, StackSize]) =
    (runLen, stackSize, currentInvariant)
}

implicit def FirstCollapseCase[RunLen <: HList, StackSize <: Nat, N <: Nat, NMinusOne <: Nat, NMinusTwo <: Nat, RunLenNMinusOne <: Nat, RunLenN <: Nat, RunLenNPlusOne <: Nat,
  NextRunLen <: HList, NextStackSize <: Nat](
  implicit nMinusOneWitness: Pred[N] { type Out = NMinusOne },
  nMinusTwoWitness: Pred[NMinusOne]{ type Out = NMinusTwo },
  runLenNMinusOneWitness: Selector[RunLen, NMinusOne] { type Out = RunLenNMinusOne },
  runLenNWitness: Selector[RunLen, N] { type Out = RunLenN },
  runLenNPlusOneWitness: Selector[RunLen, Succ[N]] { type Out = RunLenNPlusOne },
  mergeAtCase: MergeAtCase[RunLen, StackSize, N] {
    type NewRunLen = NextRunLen
    type NewStackSize = NextStackSize
  },
  nextLoopStep: LoopStep[NextRunLen, NextStackSize, NMinusTwo]
  ) = new LoopStep[RunLen, StackSize, N]{
  type NewRunLen = nextLoopStep.NewRunLen
  type NewStackSize = nextLoopStep.NewStackSize
    
  def doLoop(runLen: RunLen, stackSize: StackSize, currentInvariant: LoopInvariant[RunLen, StackSize]) = {
    val (nextRunLen, nextStackSize, nextInvariant) = mergeAtCase.mergeAt(runLen, stackSize, null.asInstanceOf[N])
    nextLoopStep.doLoop(nextRunLen, nextStackSize, nextInvariant)
  }
}

...
````

I haven't covered all the paths through this branching code, but hopefully the principle is clear. Note the recursion in FirstCollapseCase - rather than a while loop, each `LoopStep` knows either that it's the termination of the iteration, or how to 