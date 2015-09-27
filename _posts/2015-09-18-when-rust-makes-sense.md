---
published: false
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

## F#

.net is a parallel world, not just a different language and VM but a foreign operating system. I've heard less criticism of it than the rest of my list, but I'm not sure whether that's a testament to its elegant design or an indication that it's one of Stroustup's second kind of languages. Scala struggles to combine `extends` inheritance with type inference, particularly in the presence of covariance - but if F# had solved this properly I'd expect to hear more about it.

Writing this I'm realising just how little I know about F#; the grass is greener over there, 

## OCaml

[1] No doubt to the horror of purists. But I think it's important to provide an incremental migration path; until someone has seen the difficulties with unmanaged side effects - or rather, the advantages of pure code *in a language that takes advantage of it* - for themselves, asking them to do a lot of bookkeeping for no visible benefit is not going to be persuasive.

[2] "Social issues" being a euphemism for one particular individual. But that's another topic.
