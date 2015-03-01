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

I haven't covered all the paths through this branching code, but hopefully the principle is clear. Note the recursion in FirstCollapseCase - rather than a while loop, each `LoopStep` knows either that it's the termination of the iteration, or what the next `LoopStep` is. We can write a wrapper function that will actually call the loop cases:

````scala
def mergeCollapse[RunLen <: HList, StackSize <: Nat, StackSizeMinusOne <: Nat, StackSizeMinusTwo <: Nat](runLen: RunLen, stackSize: StackSize)(implicit
    stackSizeMinusOneWitness: Pred[StackSize]{type Out = StackSizeMinusOne}, stackSizeMinusTwoWitness: Pred[StackSizeMinusOne]{type Out = StackSizeMinusTwo},
    initialInvariant: LoopInvariant[RunLen, StackSize],
    loopStep: LoopStep[RunLen, StackSize, StackSizeMinusTwo]) = {
  val (newRunLen, newStackSize, invariant) = loopStep.doLoop(runLen, stackSize, initialInvariant)
  (newRunLen, newStackSize)
}
````

But in practice if we care about the invariants we would probably call the implicit `LoopStep` instances directly wherever we want to call `mergeCollapse`.

## Thoughts

This was a purely mechanical port - nothing "hard" about it - but it was more tedious than I was expecting. Mainly I put this down to the original Java code being very non-scala-idiomatic; it's full of arrays, if/else, and increment/decrement operations. In Scala it would be more natural to write immutable code, and use polymorphism for branches rather than conditionals; this would have removed the need to have separate implicit `LoopStep` instances as the cases could be moved into the polymorphic types.

But even if the code were more idiomatic, it's clear that Scala is not entirely suited to this kind of work. Lifting an operation as simple as `n = stacksize - 2` up to the type level requires adding two type parameters to the function and passing two implicit witness parameters to guide the inference of these to the correct types. At some point these extra values will add runtime overhead, of the kind that's acceptable in most code but not in the default JVM sort routine. And since we encode proven results as the existence of implicit parameters, it's impossible to talk about results being true "for all `N`". Had TimSort been originally implemented like this, we would have found out at compile time that the code was incorrect - but only when someone tried to compile a call to sort a sufficiently large array.

In a language with first-class support for type-level programming, a lot of these issues go away; I look forward to seeing an Idris implementation of provably-correct-TimSort, and would expect it to be of a similar length and performance to the Java/KeY implementation.

What's the fair comparison? I picked Scala because it's a production-ready language usable today, even in situations where the JVM is a hard requirement - in 99% of the cases where you could use Java, you can use Scala, which is not (yet) something you can say for Idris. While I believe there are some production users already, Idris remains experimental and academic; tooling support is quite limited, and the standards for even quite basic functionality like dependency management are in a state of flux.

But while Java is a production-ready language, the same can't really be said for KeY; do IDEs understand the KeY comment format correctly? Will they include the correct KeY comments when performing automated refactorings (e.g. "extract method")? If your proof is encoded as part of the type system then you get this for free: any Scala or Idris IDE would handle such a refactoring correctly, because at a minimum an "extract method" extracts a method of appropriate type.

If KeY is not part of the workflow, but merely an external step that's run once, then its results are ultimately of limited value. The paper has been published, and the specific bug fixed - but if KeY is not run as part of normal JDK development, then the proof is likely to rot, and similar bugs could easily be reintroduced. In my professional life I've twice introduced the [checker framework](http://types.cs.washington.edu/checker-framework/), only to see it abandoned, because not enough of the tools developers used supported it. And this is a framework that integrates through annotations rather than magic comments, one that offers Eclipse plugins and has at least some support in IntelliJ. If they haven't succeeded, I very much doubt that KeY will. Whereas when I've been able to introduce the same functionality as Checker offers, but as a "normal" library in standard Scala - e.g. [this units of measure library](https://github.com/KarolS/units) - that code has (to date) been a success.

I don't advise anyone to try to write full proofs 