---
published: false
---

Many mathematics students have disproportionate trouble with non-standard analysis. The concepts are not difficult; indeed, they align with many freshmen's naive intuitions about calculus: there's a new kind of number, infinitesimals; any number of infinitesimals will never add up to an ordinary positive number; an infinitesimal squared is doubly infinitesimal, and so on. But paradoxically it's *because* these ideas are so obvious that they're so difficult to accept - to get to this stage, the student has spent months beating them out of himself, in favour of the rigorous Weierstraussian notion of limits of epsilons and deltas. He scoffs at the very notion of infinitesimals - the very idea! - with the righteous zeal of the fresh convert. Small wonder that he has difficulty taking these freshly-crushed ideas seriously.

So too, I think, it is with essentialism - the misguided philosophical notion that a thing's identity is defined by its attributes, or even that the very concept of identity is well-defined. To the naive student, it seems obvious what is and isn't a [sandwich](https://medium.com/@kmikeym/is-this-a-sandwich-50b1317eb3f5) - and even if we haven't thought about it explicitly, we assume there's some Platonic definition of a sandwich "out there", declaring once and for all precisely what is and isn't a sandwich. And of course the slightly more experienced student realises that this is nonsense; there's no sharp line between a sandwich and a wrap, or mammal and bird, or even alive and dead. Instead, operational definitions take precedence: if you want to eat it, an open sandwich is a perfectly cromulent sandwich. If you want to put it on a shop shelf, maybe not so much. If you're looking at evolutionary history, a bat is emphatically not a bird - but if you want to know whether it flies, maybe the Aristotlean definition is appropriate. Earlier this year, Richard Dawkins (suggested)[http://www.theguardian.com/science/2014/jan/12/what-scientific-idea-is-ready-for-retirement-edge-org] essentialism was the scientific idea most in need of retirement.

In the last paragraph I mentioned "operational" definitions, but I could just as well have said "functional". And although the etymology is (probably) coincidence, there's a lot of similarity to the programming style; indeed, a newcomer's first questions to an expert can resemble the "sandwich argument":

> Student: Is this taco a sandwich?

> Teacher: Well, there's an Edible typeclass instance for it if you tag it as BigEndEdibleTaco or LittleEndEdibleTaco

> Student: But neither of those things is the same as Taco

> Teacher: There are prism typeclass instances for them, what more do you want?

> Student: Is a loaf of bread a monoid?

But like the mathematicians who pour scorn on the notion of infinitesimals, functional programmers have lost sight of the fact that with object-oriented programming methods, the essentialist fallacy can be made rigorous, even *true*.

What's a `Bat`? It's a `Bird`, because I've declared it one. We can see from its class what methods it has; it implements `.fly()` and the rest of the `Bird` interface. How do I know what I can do with a `Taco`? A function that accepts it could be just about anywhere - but if all my important `Taco`-manipulation functions are on the `Taco` itself, as *methods*, then knowing what something *is* - its very essence - gives me an enormous amount of power and control.

OOP advocates tend to point to its ability to model the real world - to manipulate `Car`s and `Bus`es and `Bicycle`s 

http://www.shirky.com/writings/ontology_overrated.html