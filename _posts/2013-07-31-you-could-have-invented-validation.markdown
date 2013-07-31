---
published: false
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

    val file1OrError = readFile("filename1")
    if(file1OrError.isInstanceOf[Error]) return file1OrError
    val file2OrError = 