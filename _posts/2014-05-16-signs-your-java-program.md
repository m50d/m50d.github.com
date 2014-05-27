---
layout: default
title: Signs your Java program wants to be written in Scala
published: true
---

## The code is too verbose / noisy

This is perhaps the most trivial reason, but it's the reason I started using Scala. Generics inference in Java 7 and lambdas in Java 8 have closed some of the gap, but Scala is still a substantially more concise language. With full type inference and the ability to omit the visual clutter of braces or semicolons, it's possible to write something close to Python. With the "\_" syntax for very short lambdas, and the ability to omit brackets when calling single-argument methods, one can in fact do better than Python; in Scala we could have

    someList map someFunction fold {_ + _}

whereas in Python we would need to write

    reduce(lambda x,y: x + y, map(someFunction, someList))

In Java 8 this might be

    someList.map(this::someFunction).reduce(0, (x, y) -> x + y)

which is still a huge improvement over previous versions of Java.

Sadly we haven't seen a similar improvement in the bean conventions; a typical Java codebase still contains thousands of lines of methods like this:

    public String getName() {
        return name;
    }

While I would argue that internal classes should prefer public fields, there's a legitimate case for these methods in library classes that are exposed to downstream projects; in Java a public field cannot be replaced with a method (e.g. to add a log statement, or compute the value from some other internal state) and maintain even source compatibility.

You may object that it's trivial to autogenerate these methods in an IDE. True, but you still have to read the generated code. Worse, thousands of these simple getters hide the occasional more complex implementation - so you know that getter code is [almost, but not quite](http://c2.com/cgi/wiki?AlmostCorrect) always simple. You start to assume that when splitting up a large class you can delete all the getters and use your IDE to regenerate them - which works great, right until it doesn't.

Thanks to Scala's "uniform access principle", a simple property can be replaced by accessor methods without changing downstream code - meaning you don't need to add the methods until you actually need them. So when what looks like a simple property access is actually something more complex, the method that does this stands out, rather than being lost in a sea of getters.

C# has a syntactic shortcut for properties with trivial getters; I predict that Java will gain a similar feature in the not too distant future. But even if Java adopted the C# feature, data classes in Java would still more verbose than Scala's case classes - and the latter can be used today.

## You're using `instanceof` checks and casts - or using the "visitor pattern" just to avoid them

In classic OO tradition, suppose you have `Dog` and `Cat` classes that extend `Animal`; they conform to the same interface, but have different behaviour. E.g. we might have a `makeNoise()` method that would return "woof" for a `Dog` and "meow" for a `Cat`.

Which is fine for core functionality like this, but what about auxillary functionality like serialization, or persistence, or printing? The printer is its own subsystem (it might be maintained by a different team, or even a different company). The `Animal`s shouldn't have any direct knowledge of printing, it's not their responsibility. But the code to print them needs to do different things for different types of animal.

The most direct approach is to, well, check the type directly:

    if(animal instanceof Dog) {
        Dog d = (Dog) animal;
        // code to print a Dog
    } else if(animal instanceof Cat) {
        Cat c = (Cat) animal;
        //code to print a Cat
    }

The casts are repetitive and error-prone (though a good IDE can help with that), and can seem like a code smell even when they're correct (to the reader, a cast is usually a big red flag). But worse, when we add `Horse`s to our system, our code will silently start to fail.

There's a standard way to work around this in Java - the "visitor pattern" - which makes the code safer. But it's verbose and syntactically noisy, making this an unpleasant tradeoff:

    interface AnimalVisitor<T> {
        T visitDog(Dog d);
        T visitCat(Cat c);
    }
    
    animal.visit(new AnimalVisitor<String>(){
        @Override public String visitDog(Dog d) {
            //code to print a Dog
        }
        @Override public String visitCat(Cat c) {
            //code to print a Cat
        }
    });

The extra class is an annoying overhead if we only have one or two visitors; we also have to add the `<T> visit(AnimalVisitor<T> visitor);` method to the `Animal` interface, and implementations in `Dog` and `Cat`. But everything is typesafe, and when we add `Horse` we add a `visitHorse` method to `AnimalVisitor` and get compile errors until we define it, so this is often the preferred approach on large codebases where people need to be able to change one part of the code without knowing about every other part.

In Scala we can do better than either. If we write

    animal match {
        case d: Dog =>
            //code to print a Dog
        case c: Cat =>
            //code to print a Cat
    }

then the code is shorter than the `instanceof` example, with no opportunity to cast to the wrong type since the instance check and the cast have been combined. And provided we declared the `Animal` trait as `sealed`, the compiler will warn us when we add a new case.

What about in a larger codebase, where `Animal`s are in an animal library, and the safari team who've created a `Leopard` in their own library have no idea that they should be talking to the team that makes the printing library? (they might even work in different companies). In Java you hit the limit here; if the animal library includes an AnimalVisitor there's no way for team safari to add a method to it. So the printing library has to use `instanceof`s, and even if the printing team are active in the animal community and add support for any popular new animals they see,
it's easy to miss a few. The only alternative would be something like Jackson's mapper/module system, which makes heavy use of reflection and is getting close to Greenspunning.

Scala has a better solution: the typeclass pattern. Our printing library might look something like:

    object Printer{
        implicit object DogPrinter extends Printer[Dog] {
            //code to print a Dog
        }
        implicit object CatPrinter extends Printer[Cat] {
            //code to print a Cat
        }
    }
    def print[A <: Animal: Printer](a: A) = doOtherPrintingStuff(implicitly[Printer[A]].print(a))

If we `print` a `Cat`, the library will use its own `CatPrinter`. If we try and print a `Leopard`, our code won't compile. But we can write our own `LeopardPrinter` to handle that case - and we can do this without needing any help from the printing team or the safari team.

I won't say it's painless - we have to pass the type information right through our system, so any methods that call `print` will also have the `[A <: Animal: Printer]` type, and what could be a `List[Animal]` in the `sealed trait` scenario will now have to be a `HList`, with a type like `Cat :: Dog :: Leopard :: Cat :: HNil`. But by having both patterns available Scala makes the easy thing easy and the hard thing possible.

## You're writing code that looks the same, but can't quite find a common interface

Class hierachies and generics can take you a long way in abstraction - further than many codebases need to go. But once you start to nest generic types it can be difficult to figure out the right `<? extends T<? super ...>>` incantation, and cumbersome even after you've got it right). In published libraries this kind of flexibility is vital, but in internal code it's often not worth the verbosity, so many Java programmers write code that's less general than it could be (e.g. most methods that take a `List<T>` should really take a `List<? extends T>`). The Scala style of covariance is easier to think about (variance annotations go on the type rather than on the method, so because List is defined as `List[+A]`, any method that accepts a `List[Animal]` will accept a `List[Cat]` without having to do anything), and means that a third-party library method that was written with no thought of variance can still be used covariantly.

(Writers of third-party library *types* have to pay attention to variance, but that's the right tradeoff - it's much more common to pass your own types to a third-party method than to pass a third-party type to your own method).

The other case where Java's type system starts to creak at the seams is when passing callbacks, or with more complex interleaving of control flow between library and application. A Java library can accept a callback in the form of an anonymous class or lambda expression implementing a certain interface (perhaps Guava's `Function`, perhaps that library's custom interface), and as long as this is always vanilla code then it will work. But if the callback wants to use something as simple as exceptions[1] then we have a problem - I've seen libraries that offer two copies of their callback interface, one that `throws Exception` and one that does not.

In the case of exceptions we can hack around this with RuntimeException, but what about if we wanted to make an async HTTP call in a callback? That's simply impossible in Java - the library makes a method call and even if we signal another thread to perform the HTTP operation, the library thread has to block until there's a response. Or from the other side, suppose we want our callback to participate in a database transaction; in Java the state associated with the transaction would be stored in a `ThreadLocal`, so if the library uses multiple threads and calls our callback from a different one, we're out of luck.

In Scala we can write a common "context" interface that's flexible enough to use for exception-like success/failure, async calls, database transactions and more - scalaz's `Monad` is the most popular. And since the interface uses the typeclass pattern, you can not only write your own `Monad`s, you can also write typeclass instances that allow third-party libraries to be used as `Monad`s.

This kind of thing might be theoretically possible in Java, but Scala makes it practical. The second part of the trick is Scala's `for`/`yield` syntax, available to any type that implements the right methods. If we want to write a series of async calls (or validating methods, or database operations, or...), we can express them like:

    for {
        a <- fetchTweet()
        b = computeRating(a)
        c <- postReply(a, b)
        } yield c.status

This is the right balance; unlike a sequence of chained callbacks, the logic is clear and the call sequence is straightforward to follow. But unlike the stack-slicing async extensions we see in Python (and I believe Qasar does the same thing in Java?), you won't be surprised by a change of threads in the middle of an innocent-looking method call. Instead there's an explicit distinction between `fetchTweet`, which makes an async call, and `computeRating`, which doesn't, and this distinction is visible both in their method signatures and in the code that calls them.

Making it easy to work with these kind of contexts doesn't just make existing code simpler, it makes new approaches possible. Possibly Scala's greatest advantage is that it gives you the power to handle cross-cutting concerns, by expressing them in the type system. At my previous job we used monad techniques to record an audit trail of when certain actions occurred, without visually cluttering the business logic. A checked approach to error handling or validation becomes practical when only the code that throws and the code that catches need to know the explicit exception types (to the intervening methods, it's just a generic context). I've even seen [this style suggested for dependency injection](http://blog.originate.com/blog/2013/10/21/reader-monad-for-dependency-injection/) - something where traditional Java takes a very different approach...

##You're putting parts of your logic in config files, or embedded languages, or annotations

What makes Java compelling isn't so much the language itself as the ecosystem around it. Not just libraries and frameworks, but also tooling: first-class IDEs, debuggers, profilers, code analyzers, monitoring tools. The language makes a claim to a certain kind of simplicity (at least if we avoid primitive types and arrays) - everything is an object, all code reuse is through inheritance, all concurrency is ultimately using the same thread primitives - and hiring programmers is easier than for many languages because there's a large pool of programmers available (at various levels of experience, ability and salary).

Many enterprise codebases try to solve the problems of the previous section by externalizing part of the program into annotations or XML - Spring or similar for dependency injection, JDBC annotations for managing database transactions, JAXB or Jackson annotations for serialization, some kind of reflection-based AOP system for audit trails or security. And while all these approaches work, they spoil the very thing that makes Java great. Once you use a DI framework, your IDE becomes much less dependable, as things like "find usages" of a given class/method can no longer be relied upon (admittedly with Spring most IDEs now have at least some support). Debugging code that uses annotations for transactions is an exercise in frustration, as the stack frames no longer match up to what you see in the code, and I've seen coverage tools break when faced with tests that used such code.

Even the best annotation-driven extensions make it harder to reason about the code by reading it, and present a barrier to developers joining the project. (Of course, a skilled developer should be able to pick up a fresh extension fairly quickly - but the same could be said of picking up a new language). Worse, they're circumscribed; there's a huge divide between the annotations used to configure the behaviour and the code used to implement it, making it hard to make the leap from user to developer. Look at something like Jersey - a well-designed, well-regarded framework for REST APIs in Java, and a great improvement over what has gone before. But going beyond using its annotations to define your own represents a massive step in difficulty.

Contrast this with Scala's spray-routing. A route definition looks rather like a config file (as we might use in, say, [Play](http://www.playframework.com/documentation/1.0/routes)), with a layout that corresponds to the route layout:

    pathPrefix("orders") {
      path(IntNumber) {
        orderId =>
          authorize(userHasAccessTo(orderId)) {
            complete(getOrder(orderId))
          }
      } ~
      path("create") {
        ...
      } ~
      pathPrefix("custom") {
        ...
      }
    }

But in fact the route definition is just Scala, and rather than having some reflective code that reads annotations, if you click on a term like "path" (Spray calls these things "directives") you're taken directly to the implementation. Spray is mostly implemented in itself (with a few small primitives), so we can immediately see that, for example, `authorize` is implemented (approximately) as:

    def authorize(x: Boolean) = if(x) pass else reject(AuthorizationFailedRejection)

and it's easy to see how we might write our own custom directive. Better still, we have the full power of Scala while doing so. So e.g. we can use the `for`/`yield` syntax to construct a custom directive:

    def fooCorpAuth(companyId: Long) = for {
      authToken <- parameter('token)
      user <- onSuccess(loadUserFromDatabase(authToken))
      _ <- authorize(user.companies.contains(companyId)) & provide({})
    } yield user

which we can then use in our routes, just as though it were one of Spray's builtin directives. There's no spray-specific logic to learn, no need for explicit IDE support, no awkward boundary between the two languages - it's all just Scala.

Your IDE's Scala support is probably less mature, stable and complete than its Java support. But it may well be more mature than your IDE's Spring or Drools or Jersey support.

## You're extending the language

The extensions in the previous section may be configured through annotations or XML, but they're implemented mostly in Java, with perhaps a little reflection or similar at the boundaries. Some extensions go further than that, adding stages to the compiler or rewriting bytecode on the fly. I'm thinking of Project Lombok, or AspectJ, or Tapestry[2]. Extensive use of AOP can have much the same effect.

This is probably the clearest signal that Java is inadequate for what you're trying to do; everything I've said in the previous section applies triply. At the point where you're running different bytecode from what the Java compiler would generate from your source, you're already taking on most of the costs and risks associated with using Scala. (In fact you're probably taking a higher risk - these days application servers, debuggers, profilers and so on will likely have been tested against Scala code, and the Scala compiler is likely to be more reliable than these extensions simply because it's more mature and has more users). Is the productivity increase from whatever extension you're using as big as that from Scala? By its nature a special-case Java extension can be better suited to its specific use case, so if you *only* care about e.g. better concurrency handling, such a specialized extension may be the way to go. But as a general-purpose language, Scala can replace many such extensions, and the knowledge and techniques are generalizable to other problems - which is the whole point of programming.

# So, should I switch to Scala?

Scala is my go-to language nowadays; I still write Java (and Python, and other languages) for projects where there are specific reasons to, but for general-purpose programming I default to Scala. The case for Scala in general is probably too long for a blog post (it would have to include the case for using static typing at all), but I hope I've managed to describe the advantages I've found for Scala over Java specificly.

If the problems I've described resonate with you then Scala is well worth a look; you can start small, even with a single class, and you can start by writing code that's very similar to what you'd write in Java - so much so that both major IDEs can convert Java to Scala automatically[3].

If you're using vanilla, unextended Java, and not running into difficulty with the type system, then Scala is probably not worth it (at least if you're happy with the level of Java's verbosity). A good indicator might be how you felt about the last few Java releases - many of their new features had been previously found in Scala (or in Generics Java, its spiritual predecessor). If you were chomping at the bit for lambdas, fork/join, try-with-resources or the <> syntax, Scala gives you much more in the same vein (and if you loved JSR308 then you should switch to Scala right away); if you were indifferent to these changes or worried that they might make code more complicated then Scala may not be for you.

I feel obliged to say a few words on complexity, since most Scala discussions seem to end up there. Scala is a consistent language (more so than Java - e.g. primitives, arrays and operators are much less of a special case in Scala) with a relatively short specification, but it contains many very general features that can interact in complex ways; often something which first glance seems like a complex language feature is actually just a clever use of an existing feature ("ohhhh, so the rule that operators ending in : bind to the right *also* applies to *type-level* operators...").

My view is that complexity largely comes from the domain, and the much-maligned Java programmers manage much more complexity than they're given credit for; at one stage or another inner classes, generics, and lambdas were all seen as too complex for Java programmers. When a problem is complex, building the solution out of simple pieces doesn't make it any less complex, and it's better to have a standard solution available than to reinvent one every time. Like many Java programmers I was using "the visitor pattern" well before I knew it by that name; in the same way, I can retrospectively recognize monad behaviour in previous Java systems I've worked on. I'd written `traverse` for many types before I saw the commonality that let me write a generic version; even today I sometimes write out a method in full before recognizing a pattern that can be found in scalaz. Is that wasteful? In some sense, yes - but if the end result is more readable, more maintainable code, that uses a standard library rather than my own implementation of the same thing, that's got to be a win.

So Scala may be complex - but it can let you eliminate more complexity elsewhere. Is Scala code more complex than pure, vanilla Java code? Usually yes. Is Scala code more complex than real-world Java code, with the annotations, frameworks and configurations that are needed to address cross-cutting concerns? In my experience, no.
<br/>
<br/>
<br/>
[1] In my opinion checked exceptions were not an inherently bad idea - keeping track of which errors can occur is valuable, and handling them where they occur is a legitimate strategy in some cases, as we're seeing with the recent popularity of Go. But Java's type system doesn't offer enough power to use them effectively - it's too hard to make a function that's "generic in what exceptions it throws". To be successful again exceptions will need to be a library feature first and syntactic sugar second - compare the `async`/`await` feature introduced in Scala 2.11.  
[2] And perhaps Quasar, though I don't have the direct experience to comment there  
[3] I wouldn't recommend doing this on a large scale (at least the first few years), as there is no way to convert back, but converting a class or two can give you a feel for the basic syntax.