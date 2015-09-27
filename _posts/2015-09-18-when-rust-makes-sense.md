---
published: false
---



Even as I've admired the design work going into Rust, I've been confused by its growing popularity. A language without garbage collection, in 2015? There are use cases where you need that, and a replacement for C++ in its remaining niches is a worthy goal, but for most developers managing one's own memory is an extravagance, akin to growing one's own food - something you might do on a hobby project, but never for a paying client.

It seemed bizzare that users of Python or Ruby looking for an ordinary, general-purpose language with a decent type system would fall on this systems language. Rust's spectacular engineering makes memory management far less of a chore than it's predecessors - but a chore it remains. The language puts an emphasis on memory usage that- well, saving memory is never unwelcome, but for general-purpose languages there are generally better places to spend your budget.

But then I tried to put myself in their position, and survey the landscape of mainstream functional languages - or rather, mainstream languages with decent type systems - with fresh eyes.

## Haskell

The 800lb gorilla - and for the most part it deserves its reputation - but very much a leap off the deep end. The syntax is alien enough, but the worst problem for a beginner is laziness, and the resulting need to explicitly sequence I/O operations. In some theoretical "strict Haskell" I might[1] tell beginners to use `unsafePerformIO` everywhere until they became comfortable with constructs like `sequence`, but in the Haskell we have it's 

[1] No doubt to the horror of purists. But I think it's important to provide an incremental migration path; until someone has seen the difficulties with unmanaged side effects - or rather, the advantages of pure code *in a language that takes advantage of it* - for themselves, asking them to do a lot of bookkeeping for no visible benefit is not going to be persuasive.

## Scala

My own weapon of choice.

## F#

I've played with F# but I don't know 
