---
layout: post
title:  "something something similar ideas"
date:   2025-01-05 09:00:00 +0800
categories: ideas
excerpt: ""
---

Having spent some time reading tech articles and understanding various systems, I've come to appreciate how similar ideas pop up in different ways in these systems.

> systems here can refer to tools (e.g. git) and not necessarily servers that run 24/7

This post is intended for me to internalise and pen down my observations and thoughts. I may update this as I discover more connecting ideas.

## The log

LSM Tree, filesystem, flash translation layer

## OS's page cache & DB's buffer pool (it's just cachingj)

OS's [page cache](https://en.wikipedia.org/wiki/Page_cache) is a portion of RAM used as a cache for data loaded in (i.e. paged in) from disk. This reduces lookup costs when retrieving the same items repeatedly. It takes up a proportion of RAM which would otherwise be unused. Since this segment can be easily reclaimed, it is included into `available` in `free`.

Databases (e.g. InnoDB) has the concept of [buffer pools](https://www1.columbia.edu/sec/acis/db2/db2d0/db2d0122.htm) where data and indexes are cached are loading them from disk. Similarly, this is also used to reduce the cost of repeated access.

Hence, on a OS host that's running a database, the same piece of data could be cached twice; once in the OS's page cache, once in the db's buffer pool.

Regardless, the idea here is that disk data (or slower access medium) is cached in memory for quicker access. This is also not unlike caches frequently used in web applications. It's a simple idea in Computer Science that just happen to have different names in different systems.

## Eviction (Computer memory vs Human memory)

Speaking about caches, there will never be enough space for everything that we want to cache. Hence, we always have to consider eviction.

Finite memory

2Q (page cache) vs generational GC vs Human memory (short term, long term memory)

## Git and Haystack

## Reference

https://en.wikipedia.org/wiki/Page_cache
https://pages.cs.wisc.edu/~remzi/OSTEP/
