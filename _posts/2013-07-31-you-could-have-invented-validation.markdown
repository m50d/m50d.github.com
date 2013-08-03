---
layout: default
title: You Could Have Invented the Validation Monad
---

(This is following on from a discussion about Go error handling)

Imagine you have a bunch of functions that might return success or error. Go has a special syntax for expressing this, but effectively we're talking about something like:

    case class Error(e: Exception)
    case class Success[A](result: A)
    
    def readFile(filename: String): Success[File] | Error

The | is made up syntax, unfortunately (at least for now) - to declare a function like this we need an inheritance hierarchy:

    sealed trait SuccessOrError[A]
    case class Error[A](e: Exception) extends SuccessOrError[A]
    case class Success[A](result: A) extends SuccessOrError[A]
    
    def readFile(filename: String): SuccessOrError[A]

And we use it something like this:

    readFile("filename1") match {
    	case Success(file1) =>
            val filename2 = computeFilenameFromFile(file1)
            readFile(filename2) match {
                case Success(file2) =>
                    computeFinalResultFromFiles(file1, file2)
                case Error(e) =>
                    e
                }
       case Error(e) =>
           e
   }
            
Even with only two uses, we're clearly heading towards a pyramid of doom. Some Go advocates suggest moving the error branches as early as possible, using short-circuit return to leave the function in the error case, leaving the rest of the body to handle only the success path. This improves the superficial appearance of the code, but the fundamental problem remains: this function has a far too high branching factor, too many code paths for what it's doing, making it harder to test, harder to reason about, and harder to refactor.

Let's take another look at those pattern matches. We're matching two subtypes of a common interface, and doing something fundamental to that common type - "do this if it's a success, if it's an error do nothing". Like many naive uses of pattern matching, it's a better fit for polymorphism:

    sealed trait SuccessOrError[A] {
        def runIfSuccess(f: A => Unit): Unit
    }
    case class Error[A](e: Exception) extends SuccessOrError[A] {
        def runIfSuccess(f: A => Unit) = {}
    }
    case class Success[A](result: A) extends SuccessOrError[A]{
        def runIfSuccess(f: A => Unit) = f(result)
    }

Actually we want to do a little more than just "run" f for side effects - we want to be able to return a value.

    sealed trait SuccessOrError[A] {
        //Want to return B | Exception - that's what our SuccessOrError type is for
        def runIfSuccess[B](f: A => B): SuccessOrError[B]
    }
    case class Error[A](e: Exception) extends SuccessOrError[A] {
        def runIfSuccess[B](f: A => B) = Error(e)
    }
    case class Success[A](result: A) extends SuccessOrError[A]{
        def runIfSuccess[B](f: A => B) = Success(f(result))
    }
    
    readFile("filename1") runIfSuccess {
    	file1 =>
            val filename2 = computeFilenameFromFile(file1)
            readFile(filename2) runIfSuccess {
                file2 =>
                    computeFinalResultFromFiles(file1, file2)
            }
    }

Unfortunately this gives us a SuccesOrError[SuccessOrError[Result]]. We need to be able to handle the "inner" method returning a SuccessOrError, so we need two methods:

    sealed trait SuccessOrError[A] {
        def map[B](f: A => B): SuccessOrError[B]
        def flatMap[B](f: A => SuccessOrError[B]): SuccessOrError[B]
    }
    case class Error[A](e: Exception) extends SuccessOrError[A] {
        def map[B](f: A => B) = Error(e)
        def flatMap[B](f: A => SuccessOrError[B]) = Error(e)
    }
    case class Success[A](result: A) extends SuccessOrError[A]{
        def map[B](f: A => B) = Success(f(result))
        def flatMap[B](f: A => SuccessOrError[B]) = f(result)
    }

Regular readers will know that these method names are special to scala, allowing us to use these types in a for comprehension:

    for {
       file1 <- readFile("filename1")
       filename2 = computeFilenameFromFile(file1)
       file2 <- readFile(filename2)
   } yield computeFinalResultFromFiles(file1, file2)

And there we have it. You'll find this type in Scalaz as "Validation".
