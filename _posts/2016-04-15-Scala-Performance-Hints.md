---
layout: post
title: Scala Lessons from FiloDB
date: 2017-04-15
---

Writing high performance code is not easy in any language.  I'm hoping to share some tips learned from working on many high performance data projects, including [FiloDB](http://github.com/tuplejump/FiloDB), my high performance, Spark-based analytical database written all in Scala!  Also, general tips on concurrency, actors, and other relevant topics will be included.

## Iterators

Beware of long iterator chains, particularly with `Iterator.flatMap`.  Each level adds some overhead.  Getting rid of one level of Iterator can easily shave 20-25% off of tight loops.  

Why is `flatMap` particularly bad?  It has to nest both the `hasNext` and `next` methods, and due to Scala's implementation, neither can be inlined, thus you pay the JVM virtual method call penalty.

Some times you can't get rid of a level, but you can still optimize `flatMap`.  I was able to shave 20% by rewriting the flatMap into a custom Iterator class.  Using final methods, and getting rid of a `hasNext` check within the `next` method, allows JVM to inline the call and shave some time off.

## Path Dependent Types
This is a separate blog post.

## Functional Validation using Scalactic
This is a separate blog post.

## Futures

### Debugging Futures Deadlock

TL/DR; Don't create backpressure on your futures by employing an ArrayBlockingQueue with an executor and have futures block when the queue is full.  This created deadlocks in FiloDB, as we (and likely you) have nested futures (fetching most data in FiloDB consists of at least two separate reads, which are tied together using either for-comprehensions, `Future.sequence`s, or both).  Instead, fail fast when the queue is full, or better yet, follow Akka-style backpressure and send messages back to slow down your pipeline.

In short, don't ever block!!

### Be Careful of `Future.Sequence` esp with long lists

In order for Future.Sequence to execute, it has to actually have a list of Futures, which means that all of those Futures must have been added to a thread pool, or be sitting in a queue in memory somewhere.  Make sure that the list is bounded, or better yet throttled (do small chunks at a time), or else you will end up possibly running out of memory, or deadlocked (see above).

How the deadlock happened:
- There were two ExecutionContexts used.  One for writes, global one used for reads.
- Iterator[Segment] -> Iterator[Future[String]]; Future.sequence(iter.toList) forced all segments in MemTable to be appended at once
- The huge flood of futures starts flooding the readSegment method, huge queue in global EC.  Remember that reads have multiple parts, so as queue builds up, later parts have to wait at end of line
- As first reads come in, next part of appendSegment, the writeChunks, futures starts hitting the blocking writing EC.  
- Enough reads come in, converted to write chunk futures, to fill up the blocking writing EC.  No more writes can happen.
- First segment writes cannot complete because the last part, updating the segment cache, is done in the read EC, which is way backed up.
- Deadlock.  Read EC cannot drain because blocked by write EC.

### The winning recipe for dealing with Futures

1. Don't rely primarily on ExecutionContexts to throttle (the old Java way), especially with blocking.  Scala futures compose and mapping and other callbacks cause too many dependencies.  Instead, throttle in application-layer logic.
2. Follow-on to above: Use the Futiles library (https://github.com/johanandren/futiles), especially `traverseSequentially` and `foldLeftSequentially`.  They are gold (you can batch futures using `Future.sequence` within each sequential operation.  See FiloDB's reprojector for an example).  They give you a way to easily rate-limit.
3. If you want to use ExecutionContexts to control thread pool size, etc., use the CallerRunsPolicy RejectionExecutionHandler, so that instead of blocking, extra work gets queued in original caller's thread as a natural way of slowing down calls.

## Avoid lazy vals

Lazy vals are really slow in performance critical sections.  Sometimes us Scala programmers use lazy vals to get around initialization issues and NullPointerExceptions, especially in traits.  Stuff like this:

```scala
trait MyTrait {
    def someUnknownThing: A
    lazy val derivedVar = fooFunc(someUnknownThing)
}
```

My advice to you is, just don't go there.  You will keep being haunted by NPEs and other issues if you play with initialization in traits.  Instead, stick to `abstract class`es and single inheritance if you need initialization.  Your sanity and your performance will both thank you.

Here is how Filo's mask reader got changed:  <link to commit>.   The result is much easier to reason about, and MUCH faster.  The JVM can inline nested final method calls.