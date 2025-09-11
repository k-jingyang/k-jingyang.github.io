---
layout: post
title:  "something something similar ideas"
date:   2025-01-05 09:00:00 +0800
categories: ideas
excerpt: ""
---

Having spent some time reading tech articles and understanding various systems, I've come to appreciate how similar ideas pop up in different ways in these systems.

> systems here can refer to tools (e.g. git) and not necessarily servers that run 24/7 **OR AM I REFERRING TO SYSTEMS IN GENERAL**

This post is intended for me to internalise and pen down my observations and thoughts. I may update this as I discover more connecting ideas.

## The log

LSM Tree, filesystem, flash translation layer

## OS's page cache & DB's buffer pool (it's just caching!)

OS's [page cache](https://en.wikipedia.org/wiki/Page_cache) is a portion of RAM used as a cache for data loaded in (i.e. paged in) from disk. This reduces lookup costs when retrieving the same items repeatedly. It takes up a proportion of RAM which would otherwise be unused. Since this segment can be easily reclaimed, it is included into `available` in `free`.

Databases (e.g. InnoDB) has the concept of [buffer pools](https://www1.columbia.edu/sec/acis/db2/db2d0/db2d0122.htm) where data and indexes are cached after loading them from disk. Similarly, this is also used to reduce the cost of repeated access.

Hence, on a OS host that's running a database, the same piece of data could be cached twice; once in the OS's page cache, once in the db's buffer pool.

### The idea

The idea here is that disk data (or slower access medium) is cached in memory for quicker access. This is also not unlike caches frequently used in web applications. It's a simple idea in Computer Science that just happen to have different names in different systems.

## Cache eviction & Garbage collection

[2Q algorithm](https://www.vldb.org/conf/1994/P439.PDF) is a cache replacement algorithm which maintains 2 queues. The first queue is a FIFO queue meant for transient data, the other queue is a LRU queue for hot data. When a data is first accessed, it's first placed in the transient queue. When the data is accessed again, it's then placed in the hot queue. When we have to evict data, we evict from the transient data queue.

Generational GC like xxx and xxx 

### The idea

In both cases, we are trying to identify data items that have a higher probability of not being needed again.

- In the case of 2Q, we replace such data items so that we make space for newer items that could potentially be used more often
- In the case of generational GC, we focus our CPU cycles on these data items so that the yield for objects freed per CPU cycle spent is higher

The idea to solve this problem here is that data items that have been repeatedly accessed will be accessed again.

- In the case of 2Q, these items are promoted to the hot queue
- In the case of generational GC, objects that are not cleared in a few minor GCs (i.e. still referenced) are promoted to old space.

### Human's long-term short-term memory

2Q (page cache) vs generational GC vs Human memory (short term, long term memory)

## Git and Haystack

## Summary

While writing this, I felt like I'm just spouting simple ABC and stating the obvious. But I guess that's the cool things about first principles.


## Reference

https://en.wikipedia.org/wiki/Page_cache
https://pages.cs.wisc.edu/~remzi/OSTEP/
