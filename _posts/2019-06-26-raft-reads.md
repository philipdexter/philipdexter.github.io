---
layout: post
title: Reads in Raft
---

Writes are inserted into the raft log; there is a total order of
writes.
But what about reads?

Let's look at
[Consul](https://www.consul.io/api/features/consistency.html).
Consul offers three consistency levels:

+ consistent: leaders respond to reads but first confirm their leader status; and
+ default: leaders respond to reads but do not confirm their status as leader;
+ stale: followers can respond to reads.

In all cases,
reads are not inserted into the log.
Several problem can arise.

Problem 1: there could be uncommitted writes which were received before the
read.
These writes arrived before the read but haven't made it through
the raft protocol yet.
This means we get stale reads.
If reads went through the raft protocol we'd avoid this issue
(this is a trade-off, the read would be correct but it'd
be much slower).
This applies to the consistent consistency level.

Problem 2: there could be committed writes that the current node doesn't know
about.
Problem 2a: a node assumes it's the leader, but it's not, and services reads.
This applies to the default consistency level.
Problem 2b: the follower hasn't learned about a write yet. This applies to the
stale consistency level.

There are potential problems in each consistency level of Consul.
I believe there is room for a
[reflective consistency]({% post_url 2019-01-14-reflective %})
solution for users that can accept some level of anomalies.
They would then be able to describe their ideal anomaly rate
and the system would then adapt its consistency level to
accommodate.
