---
layout: default
title: Generic Contexts, or Don't Mention the M Word
---

I took a [Category Theory course](http://gopher.srcf.net:70/users/md401/pt3notes/category.pdf) at university, but in truth I never understood more than the basics. [Lampson's (or Wheeler's) Law](http://c2.com/cgi/wiki?ButlerLampson) applies just as strongly in mathematics, and I hit my personal indirection limit about a third of the way through.

Fortunately, in software we know how to avoid this: Agile. So let's talk through (an anonymized version of) a real-life problem I had at work, and the abstraction that can be used to solve it - no theory, just programming.

##Level 1: Flatmap that Shit

Suppose we have a series of functions that might fail - perhaps they're calls to remote webservices that could go down.

    val f1: {} => Option[Int] = ...;
    val f2: Int => Option[String] = ...;
    val f3: String => Option[Set[Int]] = ...;

We want to call them one after another, and return a failure if any of them fails, success if they're all succesful. The most naïve possible way to write our function looks something like:
    
    def doFunctionsInSequence1(): Option[Set[Int]] = {
      val r1 = f1(null)
      if(!r1.isEmpty) {
        val r2 = f2(r1.get)
        if(!r2.isEmpty)
          f3(r2.get)
        else None
        }
      else None
    }
    
Even typing this makes me feel ill - and replacing the ifs and gets with cases and pattern-matching, while more idiomatic, wouldn't really be any better. There are too many branches in this code (as would show up under measures like NPath or Cyclomatic complexity) when fundamentally it's just running through a flat sequence of functions:

    def doFunctionsInSequence2(): Option[Set[Int]] =
      f1(null) flatMap f2 flatMap f3

##Level 2: "for", not just for collections any more

Flatmap is great when each step is an existing function - but when each stage is a handful of statements that don't make sense as a function in their own right, we can easily end-up in callback hell with the so-called [pyramid of doom](http://raynos.github.com/presentation/shower/controlflow.htm?full#PyramidOfDoom)

    def doFunctionsInSequence3(): Option[Set[Int]] = {
      f1(null) flatMap {i: Int => f2(i + 3) flatMap {
        s:String => 
          val t = s.toUpperCase + i
          f3(t + s)
        }
      }
    }

This can get especially bad when using futures, as the point at which we want to perform an asynchronous step is often not a clean break in the logic of the code. Say we're using futures to make asynchronous calls to some web API (so that a small number of pooled threads can make our calls to lots of slow third-party web servers in parallel, rather than having to keep one blocked thread around for each call that hasn't completed yet).

    def webApi1(s: String): Future[Int] = ...
    def webApi2(i: Int): Future(Set[Int]) = ...
    def webApi3(i: Int): Future[String] = ...
    
    def fetchSomeData(username: String): Future[Set[Int]] =
      webApi1("hello") flatMap {
        userId: Int =>
          val alternateFormatId = reformat(userId)
          logSomething()
          webApi3(alternateFormatId) flatMap {
            api3Output: String =>
              val objectRepresentation = parseJson(api3Ouptut)
              val keyForApi2 = if(objectRepresentation.someFlag) objectRepresentation.field1(alternateFormatId)
                else objectRepresentation.wrapper2(userId).field1
              val ret = webApi2(keyForApi2)
              logSomethingElse()
              ret
            }
        }

I've seen worse code in my time, but this is pretty nasty. We can make it clearer using scala's for/yield syntax (misleadingly called [sequence comprehensions](http://www.scala-lang.org/node/111) in the official tour, but useful for far more than sequences).

    def fetchSomeData(username: String): Future[Set[Int]] =
      for {
        userId <- webApi1("hello")
        alternateFormatId = reformat(userId)
        _ = logSomething()
        api3Output <- webApi3(alternateFormatId)
        objectRepresentation = parseJson(api3Ouptut)
        keyForApi2 = if(objectRepresentation.someFlag) objectRepresentation.field1(alternateFormatId)
           else objectRepresentation.wrapper2(userId).field1
        ret <- webApi2(keyForApi2)
        _ = logSomethingElse()
      } yield(ret)

Still not the nicest function in the world, but it's a definite improvement.

##Level 3: Let's traverse a collection

I left the Set\[Int\] 'til last in the previous example, because using it presents a problem. Often we want to fetch a set (or list, or map... but let's not get ahead of ourselves) from one web service, and then call another service for each element - imagine a twitter-like service where we fetch all the tweets for a given user by calling one service to get their IDs, and then call another service with each ID to fetch the tweets themselves. So we want to write something like this:

    def mysteryFunction[T](futureSet: Set[Future[T]]): Future[Set[T]] = ...
    
    def fetchTweets(username: String): Future[(UserProfile, Set[TweetData])] =
      for {
        userId <- getUserId(username)
        tweetIds <- getTweetIds(userId)
        tweetDataFutureSet = for { //We can still use for on sequences
          tweetId <- tweetIds
          tweetData <- getTweetData(tweetId)
        } yield(tweetData)
        tweetDataSet <- mysteryFunction(tweetDataFutureSet)
        userProfile <- getUserProfile(userId)
      } yield((UserProfile, tweetData))
      
mysteryFunction is some kind of "gather" function; it takes a Set of Futures (results that will be available at some point in the future), and returns a Future for the set of all the results (so this Set will be available at some point in the future - namely, when all the individual results are). Without mysteryFunction, this code won't compile; tweetDataFutures is a Set\[Future\[TweetData\]\] where we need a single Future; we can't just write "tweetDataSet <- tweetDataFutureSet" (that would make tweetDataSet a Future, when we want it to be a Set).

Thinking about it, we can easily see how to combine a Future\[Set\[T]]\] and a Future\[T\] to add the T to the set:

    def addToFutureSet[T](futureSet: Future[Set[T]], futureElementToAdd: Future[T])
        (implicit ec: ExecutionContext): Future[Set[T]] =
      for {
        set <- futureSet
        elementToAdd <- futureElementToAdd
      } yield(set + elementToAdd)

If we were imperative-minded we might implement mysteryFunction like this:

    def mysteryFunction[T](futureSet: Set[Future[T]])(implicit ec: ExecutionContext): Future[Set[T]] = {
      var workingSetFuture: Future[Set[T]] = future(Set())
      for {future <- futureSet} {workingSetFuture = addToFutureSet(workingSetFuture, future)}
      workingSetFuture
    }

But hopefully we can recognize this as a fold:

    def mysteryFunction[T](futureSet: Set[Future[T]])(implicit ec: ExecutionContext): Future[Set[T]] =
      futureSet.foldLeft(future(Set[T]()))(addToFutureSet)

As you may have guessed, our mysteryFunction is generally known as "traverse" (or rather, this is the special case "sequence").

When working with several APIs of this form, we might find ourselves wanting to do the same thing with List instead of Set - and perhaps other containers as well. It turns out it's quite easy to rewrite our method to work for most scala collections, though the type signature is a bit intimidating:

    def sequence[TL[X] <: TraversableLike[X, TL[X]] with GenTraversable[X]
        with GenericTraversableTemplate[X, TL], T]
        (futureTraversableLike: TL[Future[T]])
        (implicit ec: ExecutionContext): Future[TL[T]] = {
      def addToFutureTL(futureTL: Future[TL[T]], futureElementToAdd: Future[T]): Future[TL[T]] =
        for {
          tl <- futureTL
          elementToAdd <- futureElementToAdd
        } yield {
          val builder = tl.genericBuilder[T]
          builder ++= tl
          builder += elementToAdd
          builder.result
        }
      futureTraversableLike.foldLeft(
          future{futureTraversableLike.genericBuilder[T].result})(
          addToFutureTL)
    }

##Level 4: Genericity in the context itself

Now we know how to handle collections when we're working with Futures. But we might hit the same problem when working with Options as before:

    def doFunctionsInSequence4() =
     for {
       ids <- f3("hello")
       optionalStrings = for {
         id <- ids
       } yield f2(id)
       strings <- sequence(optionalStrings)
       ...

Of course, we can implement this sequence in the same way - but writing almost exactly the same code twice should set your programmer spidey-sense tingling:

    def sequence[TL[X] <: TraversableLike[X, TL[X]] with GenTraversable[X]
        with GenericTraversableTemplate[X, TL], T]
        (optionTraversableLike: TL[Option[T]]): Option[TL[T]] = {
      def addToOptionTL(optionTL: Option[TL[T]], optionElementToAdd: Option[T]): Option[TL[T]] =
        for {
          tl <- optionTL
          elementToAdd <- optionElementToAdd
        } yield {
          val builder = tl.genericBuilder[T]
          builder ++= tl
          builder += elementToAdd
          builder.result
        }
      optionTraversableLike.foldLeft(
          Some(optionTraversableLike.genericBuilder[T].result): Option[TL[T]])(
          addToOptionTL)
    }

So, what did we actually need to know about our "context" type (Future or Option)? We needed to be able to "wrap up" our initial empty value, and we needed to be able to "combine" two things "inside" our context. There are several signatures we could use for this, but a popular definition is to say we know how to combine a value and a function that takes that value:

    trait ComposeableContext[C[_]] {
        def wrap[A](a: A): C[A]
        def combine[A, B](a: C[A], f: C[A => B]): C[B]
    }
    
This is generic enough to be used for many other operations besides our "sequence" (e.g. it's almost trivial to implement "map" for a generic C\[_\]:ComposeableContext), but it's fairly easy to use: we'll write a "contextless" addToTL function, and then use a ComposeableContext and the .curried builtin to turn it into one whose parameters and result are inside our context C:
    
    def sequence[C[_]: ComposeableContext, TL[X] <: TraversableLike[X, TL[X]] with GenTraversable[X]
        with GenericTraversableTemplate[X, TL], T]
        (contextTraversableLike: TL[C[T]]): C[TL[T]] = {
      def addToTL(tl: TL[T], elementToAdd: T): TL[T] = {
        val builder = tl.genericBuilder[T]
        builder ++= tl
        builder += elementToAdd
        builder.result
      }
      def addToContextTL(contextTl: C[TL[T]], elementToAdd: C[T]): C[TL[T]] = {
        val composer = implicitly[ComposeableContext[C]]
        composer.combine(elementToAdd,
            composer.combine(contextTl, composer.wrap((addToTL _).curried)))
      }
      contextTraversableLike.foldLeft(
         implicitly[ComposeableContext[C]].wrap(
             contextTraversableLike.genericBuilder[T].result))(addToContextTL)
    }

And let's check we can use it:

    implicit object ComposeOption extends ComposeableContext[Option] {
      def wrap[A](a: A) = Some(a)
      def combine[A, B](a: Option[A], f: Option[A => B]) =
        for {
          sa <- a
          sf <- f
        } yield (sf(sa))
    }

    scala> sequence(Set[Option[Int]](Some(1), Some(2), Some(3)))
    res0: Option[scala.collection.immutable.Set[Int]] = Some(Set(1, 2, 3))
    scala> sequence(Set(Some(1), None, Some(3)))
    res1: Option[scala.collection.immutable.Set[Int]] = None

At this point I'm pretty sure we've used the "M word", but who cares what we call it? We didn't use any category theory - we just wrote a function to solve our problem, and made it gradually more generic.

One place names *are* important is for compatibility, in the interests of which I'll tell you that our ComposeableContext is essentially scalaz.Applicative, and scalaz provides an implementation of sequence - although I believe only for Lists, not for generic TraversableLikes. Scalaz also separates the operation of traversal from the traversable object itself, in the same way that we've separated the operation of composing inside a context from the context itself, but I don't have a use case to demonstrate why this is helpful yet.

## Level 5? Weaving contexts through each other

Suppose we're composing some actions in our context, and we want to do several contextful operations on a set of elements:
   
    for {
      user <- fetchUser(userId)
      tweets <- sequence(for {tweetId <- user.tweetIds} yield fetchTweet(tweetId))
      tweetPhotos <- sequence(for {tweet <- tweets} yield fetchTweetPhoto(tweet))
      tweetPhotoMetadatas <- sequence(for {tweetPhoto <- tweetPhotos} yield fetchTweetPhotoMetadata(tweetPhoto))
      photoMetadataAnalytics <- performAnalytics(tweetPhotoMetadatas)
      ...

Clearly this isn't ideal - we're writing the same thing over and over again. It would be nice to "weave" seamlessly between mapping through the context and mapping through collections. The best approach I've managed so far lets you write code that looks like:

    for {
      user <- fetchUser(userId)
      tweetPhotoMetadatas <- (
        for {
          tweetId <- stepInsideCollection[Future](user.tweetIds)
          tweet <- fetchTweet(tweetId)
          tweetPhoto <- fetchTweetPhoto(tweet)
          tweetPhotoMetadata <- fetchTweetPhotoMetadata(tweetPhoto)
        } yield (tweetPhotoMetadata)).stepOutsideCollection
      photoMetadataAnalytics <- performAnalytics(tweetPhotoMetadatas)
      ...

This is an improvement, but it still feels a bit cumbersome. Can you do better? (I don't even know a way to do it in scalaz or Haskell, so examples you've found on the internet are fine). Answers on a pull request!