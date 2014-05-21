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
While I would argue that internal classes should prefer public fields, there's a legitimate case for these methods in library classes that are exposed to downstream projects; in Java a public field cannot be replaced with a method (e.g. to add a log statement, or compute the value from some other internal state) and maintain even source compatibility. You may object that it's trivial to autogenerate these methods in an IDE, and indeed it is, but 