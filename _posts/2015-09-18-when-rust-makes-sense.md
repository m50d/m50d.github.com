---
published: false
---



Even as I've admired the design work going into Rust, I've been confused by its growing popularity. A language without garbage collection, in 2015? There are use cases where you need that, and a replacement for C++ in its remaining niches is a worthy goal, but for most developers managing one's own memory is an extravagance, akin to growing one's own food - something you might do on a hobby project, but never for a paying client.

It seemed bizzare that users of Python or Ruby looking for an ordinary, general-purpose language with a decent type system would fall on this systems language. Rust's spectacular engineering makes memory management far less of a chore than it's predecessors - but a chore it remains. The language puts an emphasis on memory usage that- well, saving memory is never unwelcome, but for general-purpose languages there are generally better places to spend your budget.

But then I tried to put myself in their position, and survey the landscape of mainstream functional languages - or rather, mainstream languages with decent type systems - with fresh eyes.

## Haskell

The 800lb gorilla - and for the most part it deserves its reputation - but very much a leap off the deep end. The syntax is alien enough, but the worst problem for a beginner is laziness, and the resulting need to explicitly sequence I/O operations. In some theoretical "strict Haskell" I might[1] tell beginners to use `unsafePerformIO` everywhere until they became comfortable with constructs like `sequence`, but in the Haskell we have you can't implement a simple unix utility like, I don't know, `sort` without understanding `do` notation and the difference between `<-` and `=`.

[1] No doubt to the horror of purists. But I think it's important to provide an incremental migration path; until someone has seen the difficulties with unmanaged side effects - or rather, the advantages of pure code *in a language that takes advantage of it* - for themselves, asking them to do a lot of bookkeeping for no visible benefit is not going to be persuasive.

## Scala

My own weapon of choice, with much to recommend it: familiar syntax (even `extends` inheritance), a corporate-branded ecosystem, and the shallowest migration path around. For an organization that's already committed to the JVM it offers an unbeatable proposition: use your existing IDE, libraries, and codebase; migrate one class at a time if you want to, and even code in Java style to start with, introducing safer or higher-level constructs only as you need them.

But for a solo programmer, or one outside the Java ecosystem? It's only then you realise how many compromises there are in Scala's design; how many "you can do that but you shouldn't, it's just there for Java compatibility" or "yeah there's a JVM restriction that means you can't do that".

## F#

I've played with F# but I don't know 
