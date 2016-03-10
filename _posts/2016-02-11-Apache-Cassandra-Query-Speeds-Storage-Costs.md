---
layout: post
title: Apache Cassandra for Analytics - Performance and Storage Cost Analysis
tags: [cassandra, big data, FiloDB, benchmarking]
date: 2016-02-11
---

Apache Cassandra is one of the most widely used distributed NoSQL databases in the modern open source data engineering repertoire.  While lots of posts exists on best practices, data modeling and specific features, there hasn't really been a comprehensive performance analysis done that compares the two key TCO metrics: storage cost and query efficiency.  This post that I just wrote should give users and architects deep insights into the key factors affecting Cassandra TCO when used for analytics.

https://www.oreilly.com/ideas/apache-cassandra-for-analytics-a-performance-and-storage-analysis?cmp=tw-data-na-article-lgen_beta_post_tweet

TL/DR; storage format and data modeling makes the biggest differences in terms of performance and cost.