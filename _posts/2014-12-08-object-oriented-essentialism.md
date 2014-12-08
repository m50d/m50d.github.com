---
title: Object Oriented Essentialism
published: false
---

Many mathematics students have disproportionate trouble learning non-standard analysis. The concepts are not difficult; indeed, they align with many freshmen's naive intuitions about calculus: there's a new kind of number, infinitesimals; any number of infinitesimals will never add up to an ordinary positive number; an infinitesimal squared is doubly infinitesimal, and so on.

But paradoxically it's *because* these ideas are so obvious that they're so difficult to accept - to get to this stage, the student has spent months beating the intuitive notion of infinitesimals out of himself, in favour of the rigorous Weierstraussian notion of limits of epsilons and deltas. He scoffs at the notion of infinitesimals - the very idea! - with the righteous zeal of the fresh convert.

So too, I think, it is with essentialism - the misguided philosophical notion that a thing's identity is defined by its attributes, or even that the very concept of identity is well-defined. To the naive student, it seems obvious what is and isn't a [sandwich](https://medium.com/@kmikeym/is-this-a-sandwich-50b1317eb3f5) - and even if we haven't thought about it explicitly, we assume there's some Platonic definition of a sandwich "out there", declaring once and for all precisely what is and isn't a sandwich.

The slightly more experienced student realises that this is nonsense; there's no sharp line between a sandwich and a wrap, or mammal and bird, or even alive and dead. Instead, *operational* definitions take precedence: if you want to eat it, an open sandwich is a perfectly cromulent sandwich. If you want to put it on a shop shelf, maybe not so much. If you're looking at evolutionary history, a bat is emphatically not a bird - but if you want to know whether it flies, maybe the Aristotlean definition is appropriate. Earlier this year, Richard Dawkins [suggested](http://www.theguardian.com/science/2014/jan/12/what-scientific-idea-is-ready-for-retirement-edge-org) essentialism was the scientific idea most in need of retirement.

I speak of "operational" definitions, but I could just as well have said "functional". And although the etymology is (I assume) coincidence, there's a lot of similarity to the programming style; indeed, a newcomer's questions to a functional programming expert can resemble the "sandwich argument":

> Student: Is this taco a sandwich?

> Teacher: Well, there's an Edible typeclass instance for it if you tag it as BigEndEdibleTaco or LittleEndEdibleTaco

> Student: But neither of those things is the same as Taco

> Teacher: What's this "is"? There are prism typeclass instances for them, what more do you want?

But like the mathematicians who pour scorn on the notion of infinitesimals, many functional programmers miss that with object-oriented programming methods, the essentialist fallacy can be made rigorous, even *true*.

What's a `Bat`, in an OO program? If we declare it `extends Bird`, then it *is* a `Bird`. In the functional world there could be functions that work with `Bat`s just about anywhere - but in the OO world they're all on the `Bat` itself, as *methods*. Then knowing what the `Bat` *is* - its very essence - gives us an enormous amount of power and control; we know it must implement `fly()` and the rest of the `Bird` interface, because, in a fully rigorous sense, *it really is a `Bird`*.

Which makes it rather curious that OOP advocates tend to point to the ability to model the real world. Sure, you can have a population of `Car`s and `Bus`es and `Bicycle`s that are gradually mutated through their interactions with other things, as happens in reality. But in my experience the biggest failings of OOP tend to happen when reality - where essentialism is false - imposes itself. Perhaps the `Bat` really does need to be both a `Bird` and a `Mammal`, because we want to use implementation from both. Perhaps our `Taco extends Sandwich` because it shares most of the implementation, but has to `throw MethodNotSupported` when we try to `getTopSlice()`. Perhaps our `Garage` is a place to `park(Vehicle v)`, but you better not try with a `Bus`. We face, in short, all the problems of taxonomy - but we can't solve this with cladistics.

Which brings me to [this piece on ontology](http://www.shirky.com/writings/ontology_overrated.html). The difference between object-oriented and functional style is, almost exactly, the difference between categorization and tagging, which suggests the same reasons to pick one or the other should apply. An object-oriented approach should be appropriate when working with a small amount of data, with stable entities and clear edges between them; when programmers have domain knowledge and access to expert judgement. A functional approach should be appropriate for larger, less stable corpora, or when parts of a software project are written by less coordinated or experienced programmers.

Is this borne out in practice? Maybe; in the functional world I notice a wider range of "micro-libs", a handful of classes bundled up as a project like `libfoo-bar`, with a limited scope ("extends `libfoo` to support `libbar`"). In OO-land it's more likely that `libfoo`, `libbar` and `libfoo-bar` would be released together as a single framework; there are OO microframeworks too, but I've never heard of a functional "macroframework" with the all-encompassing scope of Spring or Rails.

And the "big data" world seems to gravitate inevitably to functional ideas like MapReduce - partly because of the execution model, but also because big data is rarely perfectly schematized. More often we find malformed or overlapping entries, and one of the hardest parts of a big data pipeline is handling these outliers, ensuring that these ontological arguments - when `Supercell` performed as `Egoist`, were they the same band or a new one?[1] - can be handled or at least contained. Many of Shirky's predictions are already coming true: when working with this kind of data, we *have* to treat users and time as first-class; we have to make merges probabilistically rather than binarilly. And a lot more of our categorization is becoming post-hoc, emergent, being driven by our tags rather than the other way around[2].

But we should not be too quick to cast aside our OO tools. For an internal business tool, we can be the American Psychiatric Association - forcing users to agree on a single unambiguous classifaction, to draw lines in the sand that are arbitrary ("5 months of grief is normal, 6 months is depression") but nonetheless very useful. In some organizations I've found that the very process of codifying the rules - forcing business users to agree once and for all on whether the tripartite division of a Hong Kong contract is the same thing as the bipartite split of a three-part London contract or a different kind of thing - is more useful than the working software we produce.

Fortunately, using Scala lets me have both OO and functional styles in my toolbox. When deciding which approach is best for a particular business problem, I've found it helpful to think about Shirky's criteria. Maybe they'll be useful for you too.
<br/>
<p/>
<br/>
[1] This was one of my favourite examples from my brief stint at last.fm, because there are (at least) two other unrelated artists called `Egoist` and `Supercell`. Our SQL schema didn't allow us to distinguish between multiple artists with the same name (there were ad-hoc hacks such as tagging one `ｓｕｐｅｒｃｅｌｌ`); it was hoped that a more service-oriented structure would allow us to make that distinction, though I left before the project came to fruition.  
[2]Indeed this was last.fm's original mission