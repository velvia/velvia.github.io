---
layout: post
title: Two Simple Scala Futures Hints
date: 2016-04-14
---

What's wrong with this Scala for-comprehension, which attempts to carry out three asynchronous I/O operations?  (This comes from [FiloDB](http://github.com/filodb/FiloDB)'s actor code for creating a new dataset.  There are three operations that must be done:  First, creating a dataset, second, creating a whole bunch of column definitions, and third, initializing the actual data table.  The second future is not supposed to execute if the first one did not return `Success`.)

```scala
      (for { resp1 <- metaStore.newDataset(datasetObj)
             resp2 <- Future.sequence(columns.map(metaStore.newColumn(_, ref))) if resp1 == Success
             resp3 <- columnStore.initializeProjection(datasetObj.projections.head) }
      yield {
        originator ! DatasetCreated
      }).recover {
        case e: StorageEngineException => originator ! e
      }
```

What this code is intended to do is return a `Future` (hence the yield) if all futures complete without an exception, unless the first one did not return `Success`.  If an error occurs, the `recover` will handle exceptions and fire off an error message, in this case to the sending actor.

## All Errors Must be Handled by Recover

Let's say that I/O errors indeed do return `StorageEngineException`.  However, there may be other errors involved.  In this example, the line with `metaStore.newColumn` will fail with a different exception if one attempts to create the exact same column that already exists.  What you will find is that this code will actually hang if any exception other than `StorageEngineException` is thrown during one of the three futures!  This is definitely not what we want in good, reactive Scala code.

Thus, the first lesson is that a `recover` block must handle all exceptions.

## Predicates must be on the same line as the operation

The second problem is that the above code will actually still execute the `Future.sequence` even if the first `newDataset` code does not return Success.  Why is that?  This is really tricky, but any predicate on the outcome of a future in a for comprehension must be defined on the same line.

Furthermore, what happens if the predicate fails?  This causes a `NoSuchElementException` to be thrown, which we have not handled.

## The Corrected Code

```scala
      (for { resp1 <- metaStore.newDataset(datasetObj) if resp1 == Success
             resp2 <- Future.sequence(columns.map(metaStore.newColumn(_, ref)))
             resp3 <- columnStore.initializeProjection(datasetObj.projections.head) }
      yield {
        originator ! DatasetCreated
      }).recover {
        case e: NoSuchElementException => originator ! DatasetAlreadyExists
        case e: StorageEngineException => originator ! e
        case e: Exception => originator ! DatasetError(e.toString)
      }
```

## If you are interested in the code

Please see the `createDataset` method in FiloDB [NodeCoordinatorActor](https://github.com/filodb/FiloDB/blob/feature/automated-stress-testing/coordinator/src/main/scala/filodb.coordinator/NodeCoordinatorActor.scala).
