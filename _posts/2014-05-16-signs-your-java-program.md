---
published: false
---

## The code is too verbose / noisy

This is perhaps the most trivial reason, but it's the reason I started using Scala. Generics inference in Java 7 and lambdas in Java 8 have closed some of the gap, but Scala is still a substantially more concise language. With full type inference and the ability to omit the visual clutter of braces or semicolons, it's possible to write something close to Python. With the "\_" syntax for very short lambdas, and the ability to omit brackets when calling single-argument methods, one can in fact do better than Python; in Scala we could have
````scala
someList map someFunction fold {_ + _}
````
whereas in Python we would need to write
````python
reduce(lambda x,y: x + y, map(someFunction, someList))
````
In Java 8 this might be
````java
someList.map(this::someFunction).reduce(0, (x, y) -> x + y)
````
which is still a huge improvement over previous versions of Java.

Sadly we haven't seen a similar improvement in the bean conventions; a typical Java codebase still contains thousands of lines of methods like this:
````java
public String getName() {
    return name;
}
````
While I would argue that internal classes should prefer public fields, there's a legitimate case for these methods in library classes that are exposed to downstream projects; in Java a public field cannot be replaced with a method (e.g. to add a log statement, or compute the value from some other internal state) and maintain even source compatibility.

You may object that it's trivial to autogenerate these methods in an IDE. True, but you still have to read the generated code. Worse, thousands of these simple getters hide the occasional more complex implementation - so you know that getter code is [almost, but not quite](http://c2.com/cgi/wiki?AlmostCorrect) always simple. You start to assume that when splitting up a large class you can delete all the getters and use your IDE to regenerate them - which works great, right until it doesn't.

Thanks to Scala's "uniform access principle", a simple property can be replaced by accessor methods without changing downstream code - meaning you don't need to add the methods until you actually need them. So when what looks like a simple property access is actually something more complex, the method that does this stands out, rather than being lost in a sea of getters.

C# has a syntactic shortcut for properties with trivial getters; I predict that Java will gain a similar feature in the not too distant future. But even if Java adopted the C# feature, data classes in Java would still more verbose than Scala's case classes - and the latter can be used today.

## You're using `instanceof` checks and casts - or using the "visitor pattern" just to avoid them

In classic OO tradition, suppose you have `Dog` and `Cat` classes that extend `Animal`; they conform to the same interface, but have different behaviour. E.g. we might have a `makeNoise()` method that would return "woof" for a `Dog` and "meow" for a `Cat`.

Which is fine for core functionality like this, but what about auxillary functionality like serialization, or persistence, or printing? The printer is its own subsystem (it might even have its own specializations, a `UnicodePrinter` and an `AsciiPrinter` and a `IBMDrawingCharactersPrinter`). The `Animal`s shouldn't have any direct knowledge of printing, it's not their responsibility. But the code to print them needs to do different things for different types of animal.

The most direct approach is to, well, check the type directly:
````java
if(animal instanceof Dog) {
    Dog d = (Dog) animal;
    // code to print a Dog
} else if(animal instanceof Cat) {
    Cat c = (Cat) animal;
    //code to print a Cat
}
````
The casts are repetitive and error-prone, though a good IDE can help with that. But worse, when we add `Horse`s to our system, our code will silently start to fail.

There's a standard way to work around this in Java - the "visitor pattern" - which makes the code safer. But it's verbose and syntactically noisy, making this an unpleasant tradeoff in either direction.