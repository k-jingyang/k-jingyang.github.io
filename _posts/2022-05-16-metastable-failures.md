---
layout: post
title:  "Metastable Failures"
date:   2022-05-16 20:47:00 +0800
categories: distributed-systems
---

As referred to by [Marc Brooker's blogpost][2] on [Metastability][1], a metastable failure

> is a kind of *stable down state*, where the system is stable but not doing useful work, even though it's only being offered a load that it successfully handled sometime in the past

A system can enter this state when it's in a *metastable vulnerable state* (e.g. being at peak utilization) and is given a trigger (e.g. unexpected spike in load). Once in this state, there is a self sustaining feedback loop that keeps it in this state, even when the trigger is removed. 

### An example
Given [here][3] where it started with a bulk of your web application's database queries being slow to reach your database (the trigger). As such, the web application retries, now you have 2x the load waiting to reach the database. 

If the 2x load (say 560 QPS) exceeds what the database can handle **in time** (say 300 QPS), the database will keep handling a backlog of queries that will be timed-out and discarded by the web application (i.e. useless work). This feedback loop sustains itself when the web application keeps retrying queries after its previous queries timed out, adding to the backlog of queries while the database is processing them.

We can see that even if the trigger is removed (i.e. database queries reach the database quickly now), the system sustains itself in the *stable down state*, a metastable failure.

### What to do?  

The root cause of the metastable failure is the sustaining feedback loop, not the trigger.
  - Think about how can we prevent this sustaining feedback loop from occuring? How can we stop it if it does occur? 
  - Some improvements that we make (e.g. retries) could actually be detrimental if it's not well thought out or implemented correctly (e.g. adding randomized jitter to retries reduces the risk of creating the feedback loop)  

There are many others stated in the [original paper][1] but the aforementioned is what I want to take back the most.

### Personal thoughts
Glad to come across across the [original paper][1] while reading [Slack's incident on 2-22-22](https://slack.engineering/slacks-incident-on-2-22-22/). 

This written entry was purely based off what I learnt from the cited links, nothing original. 

Although I haven't had the experience of using this in practice yet, but I'm sure this mental model will come in handy when that day comes. 

## References
[Metastable Failures in Distributed Systems][1] \
[Marc Brooker's blogpost on the paper][2] \
[A summary of the paper by Aleksey Charapko, one of the paper's authors][3] \

[1]: <https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-bronson.pdf>
[2]: <https://brooker.co.za/blog/2021/05/24/metastable.html>
[3]: <http://charap.co/metastable-failures-in-distributed-systems/>