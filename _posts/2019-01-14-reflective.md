---
layout: post
title: Reflective Consistency
---

Reflective consistency is a consistency model which exposes anomaly statistics
to the user. The anomalies are defined by some underlying consistency model.
For example, a system could run under eventual consistency but track anomalies
as defined by linearizability. Statistics on these anomalies can then be used
by the system or the user of such a system. One example of using the statistics
is for modifying the underlying consistency model.

## Example: Cassandra with Reflective Consistency

Cassandra offers many consistency levels[^0] under which every read and write
can be performed. A user could request Cassandra to perform all writes under a
strong consistency level, such as requiring a quorum of replicas to acknowledge
every write, and all reads under a relatively weaker consistency level.

Using the anomaly statistics, a user could modify their use of Cassandra based
on anomalies. A user could perform all writes under the same, weak consistency
level as reads, and when anomalies start to increase more than the user wants
they could direct Cassandra to use the stronger consistency level until
anomalies being to decrease.

Statistics could even be tracked per key, per replica, or per anything. This
gives the user fine or coarse grained control.

[^0]: [Cassandra Consistency](https://docs.datastax.com/en/cql/3.3/cql/cql_reference/cqlshConsistency.html)
