---
published: false
---
There's [a good explanation of the equivalence between monads and monoids in the category of endofunctors](http://blog.sigfpe.com/2008/11/from-monoids-to-monads.html), but it's in Haskell, and leaves the (important) reverse mapping as an exercise. So let's do this in Scala, with proof. We'll need some definitions:

    trait ~>[F[_], G[_]] {
      def apply[A](fa: F[A]): G[A]
    }
    type Id[A] = A
    trait Monoid[A] {
      def zero: A
      def plus(as: (A, A)): A
    }
    trait MonoidE[F[_]] {
      def zero: Id ~> F
      def plus(f1: Id ~> F, f2: Id ~> F): Id ~> F
    }
