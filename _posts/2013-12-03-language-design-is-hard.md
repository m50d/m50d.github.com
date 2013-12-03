---
published: false
title: "Language Design is Hard. Let's Go Shopping"
---

There's been a bit of fuss around Scala on Hacker News this week, which ultimately pointed me towards [some more useful criticism (video)](http://www.youtube.com/watch?v=TS1lpKBMkgg). Paul Phillips is not a great speaker, but what he's saying is important, and in many cases true. I agree with the criticism, but disagree with the proposed solutions[1]. So I want to talk about why Scala is my favourite language, and what I'd do to make a better one.

Where Philipps starts to go wrong, I think, is about 2/3 of the way through his video, where he says that for a typical project we have maybe four layers of languages involved: Ant XML, Java, and Scala. Leaving aside the questionable arithmetic, this is simply false for modern Scala projects: it's very much possible to write a modern project in pure scala, with no Java anywhere, and with SBT your build is configured in Scala too. And that's not all; things that would be another format in Java are handled in pure Scala. Squeryl's Table models are normal code, where Hibernate would be configured with annotations or XML. Spray-json serializers are again ordinary scala, as opposed to Jackson's annotation approach. Play does use a separate route-mapping file and I consider this a mark against it; at my work this was a substantial factor in our choice of Spray instead. In Spray, your endpoint declarations are, once again, simply code.

To a certain extent, you get this in Python (and Ruby, as I understand it). Your build system is python; your Django models are python classes, and your routes are configured with a mix of code and decorators - and unlike Java annotations, decorators have well-defined semantics in Python, so this is really just pure python code.

All this is true...ish. Django models are real python classes... sort of (they use custom metaclasses, which are approximately equivalent to macros in terms of their effect on the predictability of your code). Django's core config file ends up returning a bunch of strings, arranged in some maps - which class to use for a particular piece of functionality is determined by a string classname, not an object you pass around. The routing configuration is somewhat better - it boils down to a map of matcher -> callable. It's nice that we can at least build these objects up using real python code - your django config file can connect to a database to assemble its configuration if you want - but it doesn't hold a candle to the flexibility of spray, where a route is really just a function, any function, and you can compose two routes in the standard fashion, or in your own custom way if that's not good enough for you - e.g. randomly choosing between two subroutes.

Of course it's possible to Greenspun this into Django, but don't expect the existing routing infrastructure to work with you. Don't expect to be able to pass a Django route into a higher-order function defined in a different library and get something useful out. I mean, Python won't *stop* you - the "we're all consenting adults here" philosophy means you can call any method with any argument - and it might even work, for a while. But there's no way to know (short of asking the devs, or hoping their roadmap is accurate) which styles of calling routes are supported, and which will be broken in the next release.

This is the problem with dynamic syntax in, well, dynamic languages. As Philipps puts it, ignorance is strength, freedom is slavery; the *only* way to make maintainable software is by imposing non-leaky interfaces between layers. Sadly I have to keep repeating the same example, because I only know one library that has ever done this properly: [Wicket](http://wicket.apache.org)

[another critique](http://www.reddit.com/r/haskell/comments/1pjjy5/odersky_the_trouble_with_types_strange_loop_2013/cd3bgcu)

[javascript](http://acko.net/blog/on-asmjs/)

[1] I'm reminded of [Zed Shaw's complaints about the web and OOP](http://vimeo.com/43380467)