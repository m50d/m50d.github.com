---
published: false
layout: default
title: My Five Years of Scala
---

Scala is a broad church, as any general-purpose programming language ultimately must be.
But let me give my views from my own 5.5 years of Scala, as they have been rather different from [those described here](http://manuel.bernhardt.io/2015/11/13/5-years-of-scala-and-counting-debunking-some-myths-about-the-language-and-its-environment/).

# Myths, or not

## True Tradeoffs

### Four or more ways to write the same thing

    obj.doSomething(_.value)
    obj.doSomething({_.value})
    obj doSomething {_.value}
    obj.doSomething({x => x.value})
