---
published: true
title: "When Rust Makes Sense, or The State of Typed Languages"
layout: default
---


Even as I've admired the design work going into Rust, I've been confused by its growing popularity. A language without garbage collection, in 2015? There are use cases where you need that, and a replacement for C++ in its remaining niches is a worthy goal, but for most developers managing one's own memory is an extravagance, akin to growing one's own food - something you might do on a hobby project, but never for a paying client.

It seemed bizzare that users of Python or Ruby looking for an ordinary, general-purpose language with a decent type system would fall on this systems language. Rust's spectacular engineering makes memory management far less of a chore than it's predecessors - but a chore it remains. The language puts an emphasis on memory usage that- well, saving memory is never unwelcome, but for general-purpose languages there are generally better places to spend your budget.

But then I tried to put myself in their position, and survey the landscape of mainstream functional languages - or rather, mainstream languages with decent type systems - with fresh eyes.

## Haskell

The 800lb gorilla - and for the most part it deserves its reputation - but very much a leap off the deep end. The syntax is alien, even by functional standards. Worse for a beginner is laziness, and the resulting need to explicitly sequence I/O operations. In some theoretical "strict Haskell" I might[1] tell beginners to use `unsafePerformIO` everywhere until they became comfortable with constructs like `sequence`, but in the Haskell we have you can't implement a simple unix utility like, I don't know, `sort` without understanding `do` notation and the difference between `<-` and `=`.

Finally, performance - especially memory and "space leaks". A lot of these concerns feel overblown, FUDish even - I've never heard of serious performance issues with a Haskell program or a failure of profiling in practice, certainly not to the extent of an organization abandoning Haskell for performance reasons. But the narrative is unsatisfactory; 

## Scala

My own weapon of choice, with much to recommend it: familiar syntax (even `extends` inheritance), a corporate-branded ecosystem, a great performance and concurrency story and the shallowest migration path around. For an organization that's already committed to the JVM it offers an unbeatable proposition: use your existing IDE, libraries, and codebase; migrate one class at a time if you want to, and even code in Java style to start with, introducing safer or higher-level constructs only as you need them.

But for a solo programmer, or one outside the Java ecosystem? It's only then you realise how many compromises there are in Scala's design; how many times you say "you can do that but you shouldn't, it's just there for Java compatibility" or "yeah there's a JVM restriction that means you can't do that". Many tutorials explain Scala constructs in terms of Java ones - a pragmatic approach, but one that can leave a new programmer feeling like they're learning two languages rather than one.

(There are more specific issues, technical and social[2], that make me cautious about recommending Scala to newcomers, even as my favourite language. But I don't know the rest of these languages well enough to compare - I could put a section about SBT here, but I don't know enough do the same for Cabal or Crate)

## F# ##

.net is a parallel world, not just a different language and VM but a foreign operating system. I've heard less criticism of it than the rest of my list, but I'm not sure whether that's a testament to its elegant design or an indication that it's one of Stroustup's second kind of languages. Scala struggles to combine `extends` inheritance with type inference, particularly in the presence of covariance - but if F# had solved this properly I'd expect to hear more about it.

Writing this I'm realising just how little I know about F# in practice; the grass is greener over there, but it's hard, not to mention hypocritical, to recommend such a dramatic shift in ecosystem.

## OCaml

Perhaps the closest to what Rust offers - it's no coincidence the first Rust compiler was written in it. There's a lot to like about OCaml - decent[3] typing but a simpler implementation and better performance-analysis story than Haskell, and none of the compromises that come with running on someone else's VM and ecosystem. One downside is the lack of a good concurrency story - something I see as more a matter of perception than reality (if you're doing linear algebra you're doing so via bindings that can release the "GIL"; if you're not number crunching then async I/O is probably more important than true parallelism). Still, not ideal.

Complaints about the syntax are legitimate, even if shallow - likewise the fact that much of the early documentation was in French. Of late this has been addressed, and the language has put together a much better packaging/library story - having used it in the mid '00s, the language's recent popularity feels like that rarest of things in the programming world, a comeback.

## So where are we?

A new language is a huge undertaking, and popularity depends on many factors, so on some level it shouldn't be surprising when no (mainstream-ish) language has some precise combination of features. Even so, the way each of these languages manages to get something wrong feels like an indictment of the functional[4] community.

It would be facile to propose new language - there are thousands out there, and no doubt one of them already ticks all my boxes. I have hope for all these languages, and I see some encouraging trends:

 * Tooling and performance analysis in Haskell are starting to get some attention. I'm not sure anyone has a solution to space leaks yet, but at least we're starting to admit that there's a problem.
 * From the other side, we're seeing Haskell-like languages with strict evaluation. Idris (my pet hope) is in many ways an even stricter, purer Haskell. But it could actually prove a more accessible environment for `unsafePerformIO`-style, imperative programming because of strict evaluation.
 * Scala.js implies that Java interoperability is less vital to Scala than it was (partly because of the maturity of the scala-native libary ecosystem). Dotty represents at least some interest in a Scala that sacrifices some compatability in the interest of simplicity/clarity/consistency.
 * The opening up of .net is extremely exciting - I can start to imagine using F# in a unix-based project, initially for standalone tools like [Flow](https://code.facebook.com/posts/1505962329687926/flow-a-new-static-type-checker-for-javascript/).
 * OCaml... I'm not sure. While I think in many ways it's the best option today, syntax is impractical to change, and writing a first-rate concurrency-friendly runtime would require massive investment that's hard to justify when F# and Haskell exist already. It's lead many of us towards the promised land, but I fear it may not live to see it.
 * Rust developers still seem interested in making some form of automatic memory management available, though it's not their primary focus. It's still a young language; not every language ends up serving the niche its creators intended.
 * From the other side, [this](http://okmij.org/ftp/Haskell/extensible/more.pdf) result reads like we've solved how to represent effects, including "region" effects like those needed for Rust-style manual-but-checked memory management. It raises the prospect of implementing Rust-like memory management as an ordinary library (built on `Unsafe` buffers in Scala, or equivalent).

Maybe Rust can and should become the best language in this space. Evidently for some people it already is.

I take a rather Whiggish view - I genuinely believe we're converging on the best design for a general-purpose programming language, and I expect these five languages and their successors to grow closer as time goes by. But right now each offers something that the others don't, and there's something to be learned from all of them. It's an exciting time to be programming.

<br />
<p />

[1] No doubt to the horror of purists. But I think it's important to provide an incremental migration path; until someone has seen the difficulties with unmanaged side effects - or rather, the advantages of pure code *in a language that takes advantage of it* - for themselves, asking them to do a lot of bookkeeping for no visible benefit is not going to be persuasive.

[2] "Social issues" being a euphemism for one particular individual. But that's another topic.

[3] Ish - I couldn't live without higher-kinded types these days, so I find it hard to recommend a language without them, even for a beginner dipping their first toe in the functional waters.

[4] That is to say, typed-and-immutable - the lisp tradition has every right to the name but is a quite different cluster of languages from the one I'm talking about. I have a post on this subject in the pipeline.
