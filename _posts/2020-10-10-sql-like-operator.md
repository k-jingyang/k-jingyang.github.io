---
layout: post
title:  "How does the SQL LIKE operator work with strings?"
date:   2020-10-10 14:00:00 +0800
categories: sql
excerpt: "Understanding how string search is done in databases."
---

### How are strings ordered?

First, we have to understand how strings are usually ordered.

Here is an example of how strings are ordered: \
JOE < JOES < JOEY < JOF < JON

This is also known as the lexicographic order or alphabetical order.

### The SQL LIKE operator

Imagine issuing an SQL query such as:

{% highlight sql %}
SELECT * FROM tbl_name WHERE key_col LIKE 'JOE%';
{% endhighlight %}

`%` is a wildcard character for zero or more characters. Any string that results from appending any amount of characters to `JOE` will still be less than `JOF`. Therefore, the database has to return any records which satisfies the condition of `JOE <= key_col < JOF`  

### How fast is it?

How expensive is this? To return records `WHERE key_col LIKE 'JOE%'`, the database will have to scan through the entire table, check every record against the condition `JOE <= key_col < JOF`. This will result in an O(n) query where n is the size of the table.

What if the column `key_col` is indexed? Remember that if you create an index for a column in a SQL database, the database will generate a sorted [B Tree/B+ Tree](https://www.youtube.com/watch?v=aZjYr87r1b8) data structure to hold the indexes.

One approach to do a range query of `JOE <= key_col < JOF`:

1. Look for the first  `key_col` value *V* that satisfies the condition of `JOE <= key_col`
2. Check every `key_col` value including and after *V* against the condition of
  `key_col < JOF` until the first value that does not satisfy it

The implementation of this approach greatly depends on whether a B Tree or B+ Tree is used.

Step 1 takes O(logn) while Step 2 takes O(m) where m is the size of records that are within the range of `JOE <= key_col < JOF`

This query will be in the order of:

1. O(m) if logn < m < n
1. O(logn) if m < logn
1. O(n) if m ~= n

### Caveats

There are some caveats regarding the placement of the wildcard character that will significantly impact the search. Look at [this answer](https://stackoverflow.com/a/25171340) in stackoverflow for more information.
