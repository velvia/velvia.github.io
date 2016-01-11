---
layout: post
title: Scala Performance Hints
date: 2017-04-15
---

Writing high performance code is not easy in any language.  I'm hoping to share some tips learned from working on many high performance data projects, including [FiloDB](http://github.com/tuplejump/FiloDB), my high performance, Spark-based analytical database written all in Scala!

## Iterators

Beware of long iterator chains, particularly with `Iterator.flatMap`.  Each level adds some overhead.  Getting rid of one level of Iterator can easily shave 20-25% off of tight loops.  

Why is `flatMap` particularly bad?  It has to nest both the `hasNext` and `next` methods, and due to Scala's implementation, neither can be inlined, thus you pay the JVM virtual method call penalty.

Some times you can't get rid of a level, but you can still optimize `flatMap`.  I was able to shave 20% by rewriting the flatMap into a custom Iterator class.  Using final methods, and getting rid of a `hasNext` check within the `next` method, allows JVM to inline the call and shave some time off.
