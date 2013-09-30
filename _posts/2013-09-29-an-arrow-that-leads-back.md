---
published: false
title: An Arrow that points back to where we started
---

I used a Kleisli arrow for the first time at work yesterday, and it was good. (It would have been better if Scala made it easier to define object methods in pointfree style, but that's a topic for another post). But it forced me to consider anew my initial objection to monads - that they may be full of sound and fury, but signify very little.

The monad is a very simple and general abstraction, but at the moment it's most commonly used to enable purer functional programming, i.e. greater control over efects. At first glance this sounds reasonable: whether a given function performs I/O, or hands part of itself off to another thread, or could fail in a particular way ginving a particular (strongly-typed) error, is important information that it would be nice to make obvious in a function's signature.

(At this point a sliver of doubt is already creeping in. Didn't the same idea lead to Java's checked exceptions, now almost universally acknowledged (even by Gosling himself) as a mistake? But perhaps some mistake was made in their implementation, or perhaps Java's abstractions weren't powerful enough to work with them effectively - certainly few would dare implement monadic algorithms in Java. But it seems telling that despite its general emphasis on safety and improved functional abstractions, Scala chose to treat exceptions as unchecked)

So we have our functions, looking something like this:

    def fetchUser(id: Long): Future[TwitterUser]
    def fetchTweets(user: TwitterUser): Future[List[Tweet]]
    def interestingTweets(tweets: List[Tweet]): List[Tweet]
    def fetchAvatar(tweet: Tweet): Future[Avatar]
    def buildMosaic(avatars: List[Avatar]): Image

And it's very obvious which ones are making async calls and which ones aren't, and we can't possibly mistake an async call for a pure calculation or vice versa, which is nice. But immediately we find this is *horrific* to use directly:

    def getMosaic(id: Long) = fetchUser(id) flatMap {
        user => fetchTweets(user) flatMap {
            tweets =>
                val interesting = interestingTweets(tweets)
                (Future.successful(Nil) /: interesting) {
                    (futureAvatars, tweet) =>
                        futureAvatars flatMap {
                             avatars => fetchAvatar(tweet) flatMap {
                                 avatar => avatar :: avatars
                             }
                         }
                     } map {
                         avatars => buildMosaic(avatars)
        }
    }

We've got our clarity - the call to interestingTweets looks very different from the async calls - but at too high a price.

Of course, no-one would actually use a monad like this. We have scala's lovely yield notation instead:

    def getMosaic(id: Long) = for {
            user <- fetchUser(id)
            tweets <- fetchTweets(user)
            interesting = interestingTweets(tweets)
            avatars <- (interesting map fetchAvatar).sequence
        } yield buildMosaic(avatars)

This is clearly miles ahead - so much so that it takes a while to realize we've also lost something. The distinction between two types of call is still there, in the difference between = and <-, but it's subtle, easy to miss. While using <- where you should have used = is a syntax error, the converse will compile fine until you come to use the value. In practice this can lead to IO actions never taking place (we form a pure IO monad but never bind it), or, when using futures, to subtle concurrency bugs[1].

Using an arrow makes the code even prettier:

    val getMosaic = ☆(fetchUser) >=> fetchTweets >=> interestingTweets ∘ interesting ↦ avatars ∘ buildMosaic

And yet... and yet. Somewhere along the line those clear syntactic distinctions - the very reason we're using monads in the first place, remember - have become rather blurred. Oh, you can still tell exactly what kind of function each of the above is - assuming you've memorized the symbol table[2]. But there are an awful lot of them, and checking which is which can be slower than just reading the functions being composed. It makes me wonder how much better this approach really is than, say, a python implementation using some stack-slicing greenlet-style system, where only the function names would indicate that they were or weren't making async calls.

Most likely I'm overreacting. My IDE can display type information by hovering a function, so there's definitely some improvement over having to read the whole body. And there's value in having a compact, glyphic representation of a function - it makes it faster to scan for a particular section, even if I then have to mentally unpack what's going on around it. And unlike function names, a type signature is guaranteed to be updated if a function changes from async to non (or vice versa). Maybe in a week or two other code styles will look hopelessly verbose, and living without monadic isolation of I/O will be as unthinkable as living without immutable objects is now.



[1] Imagine a function that creates a collection, and fills it with objects, returning a future that indicates completion of this operation - but *doesn't* contain all the objects, only the metadata. So we create a List of Future[Unit]s, sequence them into a Future[List[Unit]], and then use "\_ =" where we meant "\_ <-", forgetting to compose it into the future we're returning. Yeah.

[2] Scalaz 7 appears to be doing away with a whole lot of symbols, something I largely support. But I'm not sure the following is any clearer:

    val getMosaic = Kleisli apply fetchUser >=> fetchTweets >=> interestingTweets map interesting traverse avatars map buildMosaic

The names are only slightly more self-explanatory, and the distinction between our methods and scalaz operators has disappeared.