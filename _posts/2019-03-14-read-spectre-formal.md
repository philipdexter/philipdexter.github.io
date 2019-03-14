---
layout: post
title: Reading 'Spectre is here to stay'
---

[Spectre is here to stay: An analysis of side-channels and speculative execution](https://arxiv.org/pdf/1902.05178.pdf)

This paper addresses three open problems made more relevant since
[Spectre](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)):

- finding side-channels
- understanding speculative vulnerabilities
- mitigating them

This paper begins by introducing a formal model for CPU architectures
in order to study spectre.
They distinguish the observable state of an architecture
(what a developer might see from its API)
from the micro architecture
(extra state kept by the CPU).
They then study the model and a class of attacks known as
[side-channel attacks](https://en.wikipedia.org/wiki/Side-channel_attack).
Whenever state is exposed from the micro architecture, there is a possibility
of an attacker to gain information they should not have.
For example, speculative execution can cause data to be read into a cache.
Based on timing, an attacker might gain some information about the, perhaps
confidential, data.

A main result from their study is that in most programming languages
with timers, speculative vulnerabilities on most modern CPUs allow for
the construction of a (well-typed) function with signature
`read(address: int, bit: int) → bit`. Frightening.

The paper then describes possible mitigations and their runtime impact.
The authors experimented with mitigations in the V8 JavaScript engine.
For instance, one mitigation was to augment every branch with an `LFENCE`
instruction (as suggested by Intel). This led to a 2.8x slowdown on
the Octane JavaScript benchmark.
Another mitigation,
[retpoline](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)#Retpoline),
led to a 1.52x slowdown on Octane.
Unfortunately,
none of the mitigations completely protect against Spectre.
