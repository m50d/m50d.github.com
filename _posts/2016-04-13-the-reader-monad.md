---
published: false
title: The Reader Monad is a Monad
---


[This article about dependency injection](http://www.schibsted.pl/2016/04/dependency-injection-play-framework-scala/) is concerned about the reader monad compromising separation of interface and implementation. In fact, this simply reveals the need to use Scala's most powerful capability: higher-kinded types.

````scala
trait BooksService[F[_]] {
	def get(id: Int): F[Book]
}
class CacheingBooksService extends BooksService[Reader[CacheApi, ?]] {
	def get(id: Int): Reader[CacheApi, Book] = ...
}
````

This is a general pattern: there are many other "effects" that we might want to keep track of in different implementations of an interface:

````scala
class RemotedBooksService extends BooksService[Future] {
    def get(id: Int): 
}
````
