---
published: false
title: Infrastructure as code is easy. Stop using crappy tools
layout: default
---


A response to [Infrastructure as code might be literally impossible
](http://blog.packagecloud.io/eng/2015/09/15/automacon-infrastructure-as-code-might-be-literally-impossible/)

What if I told you:
 * Some languages are perceived as terribly difficult, but are easy
 * I'm not surprised MRI is buggy. It's written in C
 * I'm not surprised apt is buggy. It's written in C
 * You don't have to read every line to understand code if you:
   * Understand the *laws* the code obeys
   * Require [proof](https://gist.github.com/alanpog/3316784)
 * We *already know* better ways of building computer systems
 * Abstraction works
   * The "law" of leaky abstractions is false
   * Worse, it's convinced a generation of engineers that broken abstractions are acceptable
 * We need to be realistic about how much we care about correctness
   * If you're using ruby, you've already put a lower bound on your defect rate
   * Those of us in functional-land are equally dishonest.
     * We say "write the same number of tests and have fewer defects"
     * But any real business:
       * Knows what their acceptable defect rate is
       * Is already operating at it
     * So: defect rate homeostasis
     * The real value proposition is "write fewer tests and have the same number of defects"
     * But no-one wants to admit that a nonzero defect rate is acceptable to them
 * Big companies already adjust 
 * Thinking bigger is already happening
   * nix/nixos is the bigger solution to package management you were asking for
   * (but it's written in C++, so it's probably still buggy)
 * If reducing your defect rate is important to you:
   * Stop using Ruby
   * Stop using C
 * But you won't, because you don't really care about defect rate
 * Software will continue to be buggy as long as there are economic incentives for software to be buggy
 
 
