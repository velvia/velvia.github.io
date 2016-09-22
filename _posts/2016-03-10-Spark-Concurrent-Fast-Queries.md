---
layout: post
title: 700 SQL Queries per Second in Apache Spark with FiloDB
tags: [bigdata, spark, sql, filodb]
date: 2016-03-09
---

[Apache Spark](http://spark-project.org) is increasingly thought of as the new jack-of-all-trades distributed platform for big data crunching -- what with everything from traditional MapReduce-like workloads, streaming, graph computation, statistics, and machine learning all in one package.  Except for Spark Streaming, with its micro-batches, Spark is focused for the most part on higher-latency, rich/complex analytics workloads.  What about using Spark as an embedded, web-speed / low-latency query engine?  This post will dive into using Apache Spark for low-latency, higher concurrency reporting / dashboard / SQL-like applications - up to hundreds of queries a second!

## Sharing a Spark Context with FAIR scheduler for Low Latency

Launching Spark applications on a cluster, or even on localhost, has a pretty high overhead.  Each SparkContext must launch many services - including a web UI, file/torrent server, and more.  It also has to wait to launch executors on workers in a cluster - and the larger your cluster is, the longer this takes.  So, the first step to low-latency queries is to start up a long-running SparkContext and run queries against it.   If you are running Spark on Mesos, you will want to use the coarse-grained execution mode, so that the workers are pre-allocated, instead of allocated for every task/query.  This is a common pattern used by Spark's own Hive Thrift Server, or in the [Spark Job Server](http://github.com/spark-jobserver/spark-jobserver).

By default, Spark schedules tasks to run using a FIFO scheduler.  If tasks do not take up all executor threads, it can actually schedule concurrent tasks.  However, for concurrency, it is best to configure Spark to use the FAIR scheduler (`spark.scheduler.mode = FAIR`), which is designed to run multiple Spark tasks at the same time.  This works even if Spark tasks take up most or all of the threads.

So here is the game plan:

- Start a persistent Spark Context (or use the Hive ThriftServer - we'll get to that below)
- Run it in FAIR scheduler mode
- Use fast in-memory storage
- Maximize concurrency by using as few partitions/threads as possible
- Host the data and run it on a single node - avoid expensive network shuffles

I brought up SQL specifically to make a point: let's say that your queries are more SQL-like, that is analytical, as opposed to simply key-value lookups.  You want to know the sums, averages, top-Ks, but you want some flexibility too.  Thus we want storage tailored for these queries, but that can help concurrency as well.

## Enabling Fast Queries with FiloDB Columnar Storage

You might not have heard of [FiloDB](http://www.github.com/filodb/FiloDB).  FiloDB is columnar storage technology that allows for two-dimensional filtering and updates, with an in-memory storage engine as well as a persistent one for Apache Cassandra.  We'll be looking at the in-memory option here.

Why would you want to use FiloDB for low-latency queries, instead of, say, Spark's built-in DataFrame caching?

- You can filter data in two dimensions, whereas the built-in dataframes are mostly meant for whole-table scans.  Filtering is crucial to low latency.
- FiloDB tables support insert and update by primary key.  This might be important for Spark Streaming, IoT, time series applications.

FiloDB has a [data model](https://github.com/filodb/FiloDB#introduction-to-filodb-data-modelling) similar to Cassandra's, enabling easy fine-grained partition key filtering.

## RDD collectAsync

We'll be using the [NYC Taxi Trip and Fare Info](http://www.andresmh.com/nyctaxitrips/) dataset, which contains records of every taxi trip, including pickup and dropoff location, date/time, passengers, etc.  Suppose we want to query on trip distance and passenger count statistics for different drivers within a certain time range.  We can model the data as a time-series:

* Using the first two characters of the medallion (an ID of the cab) as a partition key, we can shard the rate information into hundreds of shards
* Within each partition, we group by the pickup timestamp and the license

The code and instructions for running benchmarks can be viewed [here](https://github.com/filodb/FiloDB/tree/master/stress).

To support running concurrent queries better, we rely on a relatively unknown feature of Spark's RDD API, `collectAync`:

    sqlContext.sql(queryString).rdd.collectAsync

This returns a Scala `Future`, which can easily be composed using `Future.sequence` to launch a whole series of asynchronous RDD operations.  They will be executed with the help of a separate ForkJoin thread pool.  

After ingesting the first million rows of the NYC Taxi dataset, and running queries using the above methodology on my laptop (using Spark `local[8]`), I got a speed of around 50 queries per second.  This seems pretty good on first glance, but when examining using a profiler, it seems hardly any time was being spent on the query execution itself, but most of the time was spent in the main application loop - in the `sql()` method above.  Most of the time spent was in translating the SQL string into a Spark DataFrame / LogicalPlan!   Looking at the logs, the time spent executing the queries was completely trivial.

## Eliminating SQL parsing overhead

Here are two strategies (which can be combined) to combat the SQL parsing overhead:

1. Cache the SQL to DataFrame/LogicalPlan parsing.  This saves ~20ms per parse, which is not insignificant for low-latency apps
2. Distribute the SQL parsing away from the main thread so it's not gated by one thread

Actually, the 20ms is not just SQL parsing.  It also involves creating the `BaseRelation` for the `DataFrame` - basically the object instance which connects Spark to the FiloDB Data Source.  This has some overhead as well, such as reading the schema for the dataset being queried.

Trying out the first strategy resulted in a huge boost - we were now getting in the neighborhood of 700 queries per second!

## Scaling with more data

What happens if we work with a larger dataset?

I ingested 15 million rows - comprising an entire month of the NYC taxi data - all into memory, and ran the concurrency test again.  Now, we were getting roughly 200 queries per second.  Considering that each partition now had ~60k records, this is still a pretty good result.  Of course, more cores are being used.

(UPDATE: After changing the query to not be returning all 60k data points, the speed went back up to 600-700 queries per second - even for 15 million rows, but I'll have to investigate to make sure it's legit)

## What's Your Stack?

So, if this is exciting, where do you go from here?

I would say the highest performance way to take advantage of a Spark SQL - FiloDB stack is to embed Spark and FiloDB directly in your Scala/Java application stack.  Next best would be to use a custom API such as Spark Job Server, where you can implement the custom/concurrent SQL parsing and caching logic, or better yet, map an API call to the required LogicalPlan directly.

Some other possibilities are:

* BI client -> Hive Thrift Server -> Spark.  This is possible, but the standard thrift server does not have the SQL caching logic I speak of above, and in my experience the thrift server (at least when used from Spark-beeling) has a pretty high latency overhead
* Dashboard / JS -> Spark Job Server (REST) -> Spark.

## Current Status and What's Next

Currently, FiloDB's InMemoryColumnStore is intended only for single-node low-latency operation.  Adding distribution, or making the in-memory layer as a cache for Cassandra and other persistent stores, is a possible direction, so your feedback is welcome!

To find out more about FiloDB, come and check out our [O'Reilly Webcast](http://www.oreilly.com/pub/e/3652), or feel free to play with the [Github project](http://github.com/filodb/FiloDB).
