---
layout: post
title: Configuring and Deploying Apache Spark
---

I gave this talk at the inaugural [SF Spark and Friends Meetup](http://www.meetup.com/SF-Spark-and-Friends/) group in San Francisco during the week of the Spark Summit this year.  While researching this talk, I realized there is very little material out there giving an overview of the many rich options for deploying and configuring [Apache Spark](http://spark-project.org).  There are some specific articles by vendors - targeting YARN, or DSE, etc., but I think what developers really want is a broad overview.  So, this post will give you that, but you will have to look through the [slides here](http://www.slideshare.net/EvanChan2/productionizing-spark-and-the-spark-job-server) to dig through the meat of it.

I presume you know about Apache Spark, but you might not know who I am.  I started championing, working with, and contributing to Apache Spark at version 0.8, more than two years ago, at Ooyala.  The temptation of fast in-memory analytics with a REPL and native Scala DSL was just too much.  The growth in the Spark ecosystem and community since then has been unbelievable.

### Standalone, Mesos, or YARN?

Perhaps the most overarching question when it comes to Spark deployment is which clustering mode to choose - standalone, Mesos, or YARN?  Before talking about their differences, let's talk about what they all have in common, as of version 1.4 of Spark:

* HA cluster managers via Zookeeper
* Running the Spark driver app in cluster mode
* Restarts of the driver app upon failure
* UI to examine state of workers and apps

Standalone mode means you run your Spark cluster by itself with a Spark Master app, or more likely, two Masters which are leader-elected via Zookeeper.  It is the easiest to deploy -- make sure Spark distro is on all nodes, and start the master and slaves -- and the one deployed with the built-in Spark EC2 scripts. However, you must dedicate your entire cluster to Spark; it has had more reliability issues in the past; and it's rarely used in production.

[YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html) is the Hadoop 2 resource manager, so it is a natural choice for those with existing Hadoop installations.  There might already be tons of data on HDFS, and a shop might have Hadoop users and groups and other things set up already.  Furthermore, such shops will probably be using a distribution from Cloudera, HortonWorks, etc., so it makes sense to continue relying on such support.  Spark leverages the YARN resource manager to allocate jobs on Hadoop nodes, and can run Spark apps themselves on the cluster in cluster mode.

That leaves [Apache Mesos](https://mesos.apache.org/).  Mesos is more of a distributed kernel, intended to oversee many different cooperating frameworks. Highlights of differences with YARN:

* Mesos can manage resources for your entire enterprise -- microservices, databases, Spark and Hadoop, and more -- not just compute jobs.  It has great support for Docker containers for example.
* You can run YARN on Mesos (Project [Myriad](https://github.com/mesos/myriad)), but not the other way around.  Mesos supports multiple schedulers through a two-layer scheduling system.
* Mesos uses a C++ shared library, and can support non-JVM workloads such as MPI.  On the other hand, you need to install a JNI lib.

Mesos is heavily used at AirBNB, Twitter, Apple, etc., and commercially supported by [Mesosphere](https://mesosphere.com/).  They have a new product that makes using Spark on Mesos in the cloud very easy - more on that later.

If you don't have heavy investment in Hadoop infrastructure, I would seriously look at Mesos - to run not just Spark, but everything else.

### Datastax Enterprise

If you are running Spark on Cassandra, you are probably aware of [Datastax Enterprise](http://docs.datastax.com/en/datastax_enterprise/4.7/datastax_enterprise/newFeatures.html), the commercial offering that integrates Spark into Cassandra.  The HA is done via Cassandra gossip instead of Zookeeper, and you would store data in Cassandra tables, or in CFS, which is HDFS on Cassandra. This is a great option for easily installing and monitoring both Spark and Cassandra.  Plus you get great support.

### Spark in the Cloud

These days, everyone is looking to offload ops work to cloud providers, and many are rushing to provide support for Apache Spark, so this section is almost certainly going to be outdated by the time you read this.  Of course, there is the [Databricks Cloud](http://databricks.com/) product, which includes an interactive notebook as well as a way to run your jars.  Not to be left behind, Amazon EMR now has [first class Spark support](http://aws.amazon.com/elasticmapreduce/details/spark/) on YARN.

One compelling up-and-coming solution is Mesosphere's [DCOS](https://mesosphere.com/), or DataCenter Operating System.  DCOS makes installing Spark, Cassandra, Kafka, and other distributed systems on Amazon EC2, GCE, Azure, or private datacenters as easy as running `dcos package install spark`.  The service is still in the early days -- for example, there is no spark-shell support right now -- but I was able to create a 6-node Spark and Cassandra cluster in about 15 minutes on EC2.  It seems to be a great middle-of-the-road solution to deploy Spark and other systems in the cloud, take advantage of Mesos, with more control than you can get with Databricks, and without the overhead of doing all the deploys yourself.

### Spark Job Server

Finally, I'm going to give my own project, the [Spark Job Server](http://github.com/spark-jobserver/spark-jobserver), a plug  :)   Spark Job Server offers a REST interface to manage your Spark jobs, job history saved in a database, and named RDD caching and sharing between jobs.  It is particularly compelling for implementing low-latency REST-based query services based on in-memory datasets, and has built in support for streaming, SQL, and Hive jobs.  Future directions of development includes authentication and authorization services.

### Go see the slides

The above is but a fraction of the material I have in the slides [here](http://www.slideshare.net/EvanChan2/productionizing-spark-and-the-spark-job-server) -- for example I have two sections devoted to specific Spark configuration options and tips on running Spark apps you may find helpful.  Check out the slides!