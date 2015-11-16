---
published: false
layout: default
title: My Five Years of Scala
---

Scala is a broad church, as any general-purpose programming language ultimately must be.
But let me give my views from my own 5.5 years of Scala,
as they have been rather different from [those described here](http://manuel.bernhardt.io/2015/11/13/5-years-of-scala-and-counting-debunking-some-myths-about-the-language-and-its-environment/).

# Myths, or not

## True Tradeoffs

### Four or more ways to write the same thing

    obj.doSomething(_.value)
    obj.doSomething({_.value})
    obj doSomething {_.value}
    obj.doSomething({x => x.value})

There really are many syntactic ways to write the same expression in Scala.
But these are fundamentally superficial and syntactic.
Flexibility in bracketing exists in many languages:

    obj.doSomething(a, b);
    (obj).doSomething(a, b);
    (obj).doSomething((a), (b));
    
and few find this an issue.
Scala goes further than most (braces and brackets offer overlapping semantics, and the `_` syntax for lambdas is unique to Scala).
I find the value of `_` far outweighs the inconsistency it induces;
I miss it when working without it
(e.g. Python's `lambda x: x.value`, or Javascript's `function(x){ return x.value;}`).
Some libraries offer the same functionality through libraries,
of lesser or greater hackishness
(the reflection-based FunctionalJava is thankfully less necessary post-Java-8,
but many Python or Javascript options operate in similar fashion).
