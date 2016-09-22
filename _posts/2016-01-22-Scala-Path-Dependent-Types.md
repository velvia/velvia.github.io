---
layout: post
title: Scala Path Dependent Types
tags: [scala]
date: 2016-02-02
---

In the course of working on [FiloDB](http://github.com/filodb/FiloDB) and other Scala data apps, we often have to work with data whose types are not known at compile time.  For FiloDB, for example, a user can ingest data with any schema.  At runtime, the schema tells us what type each column of data is.  What are some approaches we can take to work with data like that?

## Type classes

One approach is to use type classes.  Suppose we have a trait meant for comparing two items of a given type.  Actually, Scala has two such traits, Ordered and Ordering, but let's suppose we wanted to implement our own such trait, just as a thought exercise:

```scala
trait Comparator[T] {
    def ordering: Ordering[T]
    def compare(a: T, b: T): Int = ordering.compare(a, b)
}

object IntComparator extends Comparator[Int] {
    def ordering: Ordering[Int] = Ordering.Int
}
```

In order to use the above `Comparator` trait as a type class, we might have some class or function that uses implicits or the type class pattern:

```scala
def process[T: Comparator](items: Seq[T]): Int
```

However, the above function can only deduce the right `Comparator` statically, at compile time.  This means for our use case, where we don't know the types ahead of time, we might have to do dynamic calling of the `process` function depending on schema:

```scala
schema.dataType match {
    case IntData    => process[Int](items)
    case DoubleData => process[Double](items)
    etc.
}
```

Can we do better?

## Type Members and Path Dependent Types to the rescue

What if there was a way of capturing the needed type in the schema itself, as well as the comparator?  There is.  Let's rewrite the Comparator trait using type members:

```scala
trait Comparator {
    type T     // abstract type member, to be filled in by concrete classes
    def ordering: Ordering[T]
    def compare(a: T, b: T): Int = ordering.compare(a, b)
}

object IntComparator extends Comparator {
    type T = Int
    def ordering: Ordering[Int] = Ordering.Int
}
```

The type T in `Comparator` is called a type member.  Unlike Java, Scala allows not just concrete variables as trait members, but types as well.  The difference between the earlier `Comparator[T]` which uses Java-style type parameters and this latest version is that we are able to reference the type T within a `Comparator` and pass it in as a parameter to a function like `process`.

However, if you enter the above code and then enter in an implementation for process, you will find the Scala compiler will complain:

```scala
scala> def process[T: Comparator](items: Seq[T]): Int = { ...; 0 }
<console>:11: error: Comparator does not take type parameters
       def process[T: Comparator](items: Seq[T]): Int = {
```

This is because the Scala syntax `[T: Comparator]` is really shorthand for having a second parameter list which is similar to `(implicit c: Comparator[T])`, and `Comparator` no longer has a type parameter.

We need a different way of passing in the comparator and declaring the T type.  While we could manually declare our own implicit comparator without the type parameter, we would have a problem of enforcing that the comparator is of the same type as T.  Instead, let's look at a different approach:

```scala
def process(c: Comparator)(items: Seq[c.T]): Int = {
  c.compare(items(0), items(1))
}
```

What is happening here?  First, we notice that there are no more type parameters to this function.  Instead, the type of the items list is declared to be `c.T`, the type member of the `Comparator` trait.  In other words, the type of a parameter in the second list is dependent upon the `Comparator` value passed in in the first parameter list.  Is this some kind of magic?  It seems Scala is allowing types to be dynamically determined.  Instead of a static type like `Int`, or a parameterized type like `T`, `c.T` is called a path-dependent type, and there have been various blog posts introducing the concept ([here](http://www.shiftforward.eu/techtalks/2013/02/scala-path-dependent-types-a-real-world-example/), and [here](http://danielwestheide.com/blog/2013/02/13/the-neophytes-guide-to-scala-part-13-path-dependent-types.html)). The nice thing is that we are now able to process items of different types, without knowing the exact static type at compile time, but still have some guarantees that the type of items is the same as the `Comparator` instance.

If you pass inputs to the above function (eg `process(IntComparator)(Seq(0, 1))`), you will find that it does indeed work.

By the way, declaring `c.T` in a method parameter list must not occur in the same parameter list as `c` itself; the type members of a parameter value must be referred to in a subsequent parameter list.

## Limits of Path-Dependent Types

What if I want to pass `Comparator` around, to say a processing class that holds some state and returns results?  For FiloDB, this is very useful in type classes for different data types. What if we wanted to build some state around `Comparator`s, use them in say a `HashMap`?  Let's see how path-dependent types does in a more complicated situation and write a simple `Processor` class:

```scala
class Processor(c: Comparator) {
  def process(items: Seq[c.T]): Int = {
    c.compare(items(0), items(1))
  }
}
```

All we did was move the `Comparator` to be a class parameter so that we don't have to pass it when we call process.  You could imagine the class above could be used to hold state typed `c.T` and do much more complicated processing.  What happens if we call `process` with the same `Seq(0, 1)` input as earlier?

```scala
scala> p.process(Seq(0, 1))
<console>:15: error: type mismatch;
 found   : Int(0)
 required: p.c.T
              p.process(Seq(0, 1))
                            ^
```

What happened here?  It seems Scala is no longer able to figure out that `Int` is the same type as `c.T`.  Actually, to Scala, the *path* of the type taken by the `process` method of `Processor` is no longer just `c.T`, but it includes the instance of Processor itself, so now it is considered `p.c.T`.  This is truly what is meant by "path-dependent": the type for one instance of `Processor` is different than another instance of `Processor`.  This can be useful in some instances, but in this case it makes it difficult to pass around and abstract away `Comparator` without doing type-casts, which defeats one of the original purposes of the type safety of path-dependent types.  Just to be clear, each different instance of `Comparator` could hold a different type, but we want a way to tell the compiler when they are really the same instance.

## One possible workaround

I don't have any great solutions here.  Ideally, the Scala compiler would recognize that `p.c.T` is really the same type as `c.T` as it should know that we passed in an immutable reference to the original `Comparator` when creating `Processor`.  It might be possible to go back to using type parameters in some cases, like this:

```scala
class Processor[K](c: Comparator { type T = K }) {
  def process(items: Seq[K]): Int = {
    c.compare(items(0), items(1))
  }
}
```

This seems to work well:

```scala
scala> val p = new Processor(IntComparator)
p: Processor[Int] = Processor@3caaed78

scala> p.process(Seq(0, 1))
res1: Int = -1
```

However, I have seen that with more complex cases, you quickly run into nightmarish type errors, so YMMV.

I hope you enjoyed this introduction to Scala's path-dependent types, and some of its limitations!