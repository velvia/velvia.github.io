---
layout: page
title: About Me
permalink: /about/
---

I'm an experienced software developer / data engineer with more than 20 years of experience, leading and building data systems and databases at massive scale, some of them productionized at companies such as Apple.
Hobbyist [photographer](https://www.instagram.com/platypus.arts/).  Community and diversity, including gender diversity, are important to me.
Tackling global challenges is also very important.

This page summarizes my professional credentials, including major open source projects.  You will find code, documentation, talks, and interviews below.

Keywords: Spark, Kafka, SQL, OLAP, data warehouse, database, time series, Prometheus, distributed systems, data engineering

## Software Projects and Experience

Most of these are all projects I created or co-created :)

### FiloDB (2015-)

[FiloDB](http://github.com/filodb/FiloDB) and [filo](http://github.com/velvia/filo) - high performance distributed Prometheus time series database.

I created FiloDB from scratch and lead its development from before when Apple acquired my company, all the way
to its productionization as the core time series database technology at Apple handling billions of time series
metrics and Prometheus PromQL queries for the last several years (since 2019).

- Heavily involved in the design, architecture, and development of virtually all components, including high speed data processing, columnar format, recovery, indexing, and the PromQL query engine which we wrote from scratch
- Got FiloDB productionized at Apple since 2019
- Scales up to process hundreds of thousands of Prometheus data samples per second per node
- Innovative histogram support, way more scalable than what is in Prometheus
- Created many innovative custom data structures, many of them zero-copy and with minimal allocations, super high performance
- Besides tech lead, also lead related open source efforts

FiloDB started its life as an Apache Spark-compatible OLAP layer and datasource on top of Apache Cassandra.  This means traditional data warehouse applications can be supported on top of Cassandra thanks to a custom columnar data format which sped up queries by orders of magnitude.
- Worked with Enterprise customers on integrating FiloDB into enterprises as a data warehouse
- Worked on data modeling and custom Apache Spark query optimizers

### Rust / Columnar Compression

I am a columnar compression and OLAP expert, and have created multiple columnar formats, including the one used in FiloDB.

* [ying-profiler](https://github.com/velvia/ying-profiler) - I wrote my own sampling (designed for production) Rust memory profiler which can track allocation length
* [compressed-vec](https://github.com/velvia/compressed-vec) - Rust SIMD columnar compression library, with innovative format designed for super fast reads esp of sparse columnar data
* [telemetry-subscribers](https://github.com/velvia/telemetry-subscribers) - A bunch of tracing subscribers for common app telemetry, such as Jaeger, span instrumentation, Tokio console, etc.

### Other Data Projects I Created

* [Spark Job Server](http://github.com/spark-jobserver/spark-jobserver) - REST API for Apache Spark job submission, logging, control
    * Job Server is being included in Datastax Enterprise!
* [Scala links](http://github.com/velvia/links) - useful links for learning Scala and Scala projects
* [ScalaStorm](http://github.com/velvia/ScalaStorm) - Scala API for Apache Storm real-time stream processor
* [msgpack4s](http://github.com/velvia/msgpack4s) - a fast, streaming-friendly, type-safe MessagePack library for Scala

ScalaStorm was used for Ooyala's real time analytics application, which started its life as my special project, and which I lead through to productionization.

### Other Projects I Have Contributions To

* https://github.com/MystenLabs/sui - Layer1 Blockchain
* Apache Spark

## Presentations and Conferences

See [SlideShare](http://www.slideshare.net/evanchan2) and my [presentations site](http://velvia.github.io/presentations)

SBTB 2021 - [Location-Based Data Engineering for Good](https://www.youtube.com/watch?v=dzNDrxVNjLk) - [slides](http://velvia.github.io/presentations/2021-lbs-data-eng-for-good-pyspark/index.html#1)

CNCF 2021 Rust Day - [Allocating Less: Really Thin Rust Cloud Apps](http://velvia.github.io/presentations/2021-cncf-rustday-alloc-less/index.html)

Reactive Summit 2020 - [Designing Stateful Apps for Cloud and Kubernetes](https://www.slideshare.net/EvanChan2/designing-stateful-apps-for-cloud-and-kubernetes)

SBTB 2019 - [Rust and Scala, Sitting in a Tree](http://velvia.github.io/presentations/2019-sbtb-rust-scala/#1)
    * [Youtube video](https://www.youtube.com/watch?v=bKfkGYdg6zE)

Monitorama PDX 2019 - [Rich Histograms at Scale: A New Hope](https://www.slideshare.net/EvanChan2/histograms-at-scale-monitorama-2019)

SBTB 2018 - [FiloDB: Real-time, In-Memory Time Series at Massive SMACK Scale](https://www.youtube.com/watch?v=EkIZPZbMoNE) (video)

KEYNOTE - Reactive Summit 2018 - [FiloDB: Reactive, Real-time, In-Memory Time Series at Scale](https://www.slideshare.net/EvanChan2/filodb-reactive-realtime-inmemory-time-series-at-scale)

Scale by the Bay 2017 - [2017 High Performance Database with Scala, Akka, Spark](https://www.slideshare.net/EvanChan2/2017-high-performance-database-with-scala-akka-spark)

Scala by the Bay 2016 - [Building a High-Performance Database in Scala, Akka, and Spark](http://www.slideshare.net/EvanChan2/building-a-highperformance-database-with-scala-akka-and-spark)

Spark Summit 2016 - [700 QUERIES PER SECOND WITH UPDATES: SPARK AS A REAL-TIME WEB SERVICE](http://www.slideshare.net/SparkSummit/700-queries-per-second-with-updates-spark-as-a-realtime-web-service) and [video](https://youtu.be/nAX53vQy9AQ)

Strata San Jose 2016 - [NoLambda: A new architecture combining streaming, ad hoc, machine-learning, and batch analytics](http://conferences.oreilly.com/strata/hadoop-big-data-ca/public/schedule/detail/46818)

Strata Singapore 2015 - [Breakthrough OLAP on Cassandra and Spark](http://velvia.github.io/presentations/2015-breakthrough-olap-cass-spark) - sorry video doesn't seem to be up yet but here is [synopsis](http://conferences.oreilly.com/strata/big-data-conference-sg-2015/public/schedule/detail/44794).

Spark Summit EU 2015 talk - Productionizing Spark and the Spark Job Server [video](https://www.youtube.com/watch?v=kQGS_6TxfTk&list=PL-x35fyliRwi8TqkQ_dZjoNSkUWkcl01e&index=6) and [slides](https://t.co/bhKKfWgopt)

Big Data Scala 2015 - [End to End Pipeline Training](http://bit.ly/pipeline-slides) and [Breakthrough OLAP on Cassandra and Spark](http://velvia.github.io/presentations/2015-breakthrough-olap-cass-spark)

SF Spark and Friends - Nov 2015 - [FiloDB: Combining Spark Streaming and Ad-Hoc Analytics](http://velvia.github.io/presentations/2015-filodb-spark-streaming)

Scala Days 2015 talk on [Productionizing Akka](https://www.parleys.com/tutorial/akka-production-why-how)

SF Spark and Friends .. Cassandra South Bay Meetup .. Scala Days 2015 SF .. FOSS4G-NA 2015 .. Cassandra Summit 13 14 .. Spark Summit 13 14 ..

## Interviews, WebCasts, Blogs

Reactive Foundation blog post - [Decoupling Space: Create Flexibility by Embracing the Network](https://www.reactive.foundation/post/decouple-space-the-reactive-principles-explained) - my blog post on how Actors and message passing decouples space and allows for flexible app architecture

O'Reilly Webcast - [Fast and Simplified Streaming, Ad-Hoc and Batch Analytics with FiloDB and Spark Streaming](http://www.oreilly.com/pub/e/3652)

Typesafe blog/interview - [Fast Forward With Fast Data, Scala and Akka: Q/A with Spark Job Server creator](https://t.co/YUCdpUTqyg)

O'Reilly blog: [Apache Cassandra for Analytics: A Performance and Storage Cost Analysis](https://www.oreilly.com/ideas/apache-cassandra-for-analytics-a-performance-and-storage-analysis)

O'Reilly Podcast Interview - [Building a Scalable Platform for Streaming Updates and Analytics](https://www.oreilly.com/ideas/building-a-scalable-platform-for-streaming-updates-and-analytics)

## Where you can find me

* [Twitter](https://twitter.com/Evanfchan)
* Discord - @tahoe_fp
* [Instagram](https://instagram.com/platypus.arts)
* [Gitter/spark-jobserver](https://gitter.im/spark-jobserver/spark-jobserver)


