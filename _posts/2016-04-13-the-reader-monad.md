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
    def get(id: Int): Future[Book] = ...
}
class TreelogBookService extends BooksService[DescribedComputation] {
    def get(id: Int): DescribedComputation[Book] = ...
}
class DoobieBookService extends BooksService[ConnectionIO] {
    def get(id: Int): ConnectionIO[Book] = ...
}
class AuthorizedDoobieBookService extends BooksService[EitherT[ConnectionIO, Unauthorized, ?]] {
    def get(id: Int): EitherT[ConnectionIO, Unauthorized, Book] = ...
}
````

We can use all these implementations and more with the same client code, because all these things are monads:

````scala
class AggregateBooksService[F[_]: Monad](booksService: BooksService[F], authorService: AuthorService[F])
    def fetchAllBooksBySameAuthorAs(bookId: Int): F[Vector[Book]] =
    	for {
        	book <- booksService.get(bookId)
            author <- authorService.get(book.authorId)
            books <- author.books.traverse(booksService.get)
        } yield books
````

What if `booksService` and `authorService` use different effect stacks? No problem, we can use `MonadPartialOrder` to handle both:

````scala
class AggregateBooksService2[F[_]: Monad, G[_], H[_]](booksService: BooksService[G], 
authorService: AuthorService[H])(implicit fg: MonadPartialOrder[F, G], fh: MonadPartialOrder[F, G]) extends IAggregateBooksService[F] {
    def fetchAllBooksBySameAuthorAs(bookId: Int): F[Vector[Book]] =
    	for {
        	book <- fg(booksService.get(bookId))
            author <- fh(authorService.get(book.authorId))
            books <- fg(author.books.traverse(booksService.get))
        } yield books
````

The end result is that we make testing very easy: our unit tests can follow the exact same code paths as live code, but without having to worry about any effects.

````scala
class AggregateBooksService2Test {
    val stubBooksService: BooksService[Id] = ...
    val stubAuthorService: AuthorService[Id] = ...
    val aggregateBooksService2 = new AggregateBooksService2[Id, Id, Id](stubBooksService, stubAuthorService)
    ...
}
````

Our effects - dependencies via Reader, but also things like remoting, audit logging, database access or possible failures - are minimal overhead in day-to-day code (just the difference between `=` and `<-`). But they're not entirely invisible: we can look at the type of a function and see exactly what kind of effects it might have or depend on. If we're using the Reader monad, that means we can see exactly what dependencies any call has, just by mousing over it in our IDE - which in turn means we can easily see when something's not quite right ("why does logging in need the `BookService`?"). The end result is code that's very concise, but without any "magic" or reflection: everything's plain old code, which is the easiest kind to maintain.