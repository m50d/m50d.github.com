---
published: false
title: Infrastructure as code would be easy if we actually cared
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
   * Worse, it's convinced a generation of engineers that broken abstractions are ok
 * We need to be realistic about how much we care about correctness
   * If you're using ruby, you've already put a lower bound on your defect rate
   * Functional-language marketing is dishonest:
     * We say "write the same number of tests and have fewer defects"
     * But any real business:
       * Knows what their acceptable defect rate is
       * Is already operating at it
     * So: defect rate homeostasis
     * The real value proposition is "write fewer tests and have the same number of defects"
     * But we treat a nonzero defect rate as a moral failing
 * Big companies already adjust e.g. twitter:
   * Initially written in Ruby
   * Fail whale
   * Rewritten in Scala as correctness became more important
 * Thinking bigger is already happening
   * nix/nixos is the bigger solution to package management you were asking for
   * (but it's written in C++, so it's probably still buggy)
 * If reducing your defect rate is important to you:
   * Stop using Ruby
   * Stop using C tools
   * Idris if you really care
   * Haskell/F#/OCaml/Scala if you care a bit
 * It's not impossible to do it right. It's not even hard
   * We just don't care enough
     * And there's nothing wrong with that!
     * Users value features more than correctness
     * The free market works
   * But let's be honest about it
 
