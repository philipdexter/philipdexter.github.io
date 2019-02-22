---
layout: post
title: Reading 'A Generalised Solution to Distributed Consensus'
---

[A Generalised Solution to Distributed Consensus](https://arxiv.org/abs/1902.06776)

This paper generalizes Paxos and Fast Paxos into a model of write-once registers
and provides three consensus algorithms for this model.

What caught my eye in this paper is this quote from the first page

> This paper aims to improve performance by offering a generalised solution
> allowing engineers the flexibility to choose their own trade-offs according
> to the needs of their particular application and deployment environment.

This jibes with the goal of [reflective consistency](#Reflective Consistency).
Developers using reflective consistency can choose their own tradeoffs
on the correctness/time curve.
They can sacrifice the consistency of their system
to improve its reactiveness.

Instead of the correctness/time curve,
each of the alternative algorithms proposed in this paper
fall somewhere on a curve
balancing tradeoffs between Paxos and Fast Paxos.

## Generalized Distributed Consensus

The authors reframe distributed consensus
into the processing of write-once registers.
They separate processes into servers, storing values,
and clients, getting and putting values.
They state that solving consensus means ensuring all of the following:

> Non-triviality - All output values must have been the input value of a client.
>
> Agreement - All clients that output a value must output the same value.
>
> Progress - All clients must eventually output a value if the system is
> reliable and synchronous for a sufficient period.

Their general solution organizes write-once registers into the following taxonomy

- Quorums: non-empty sets of servers.
- Decided register values: when a quorum has identical, non-nil values for any register.
- Register sets: the sets containing a copy of a single register from each server.

They then go on to define a general algorithm to establish consensus.
I could not keep up with the discussion of the algorithm, but it seemed
well-grounded.
In the end they offer proofs for their algorithms but passingly mention
that one goal of this work is to create a general enough model
for distributed consensus so that the correctness
of its algorithms become obvious.
Sounds tough to me, but what a noble goal!

## Questions

Modern distributed consensus often means [Raft](https://raft.github.io/).
I'm curious if this could generalize raft in addition to Paxos and Fast Paxos.

It'd be interesting to see a benchmark comparing the different example
consensus algorithms proposed in section 6.
