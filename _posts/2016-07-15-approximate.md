---
layout: post
title: The Difficulty in a General Approximate Computing Framework
jax: true
---

Approximate computing is sacrificing accuracy in the hopes of reducing
resource usage.

Application specific techniques in this space require too much domain
specific knowledge to be used as generic solutions for approximate
programming.

I will go through some common techniques in approximate computing, why
they don't work well as generic solutions, and then offer a potential
future direction.

## Statically Bounding Error

One commonly sought after goal in approximate computing is to
statically bound the absolute error of a computation.
The most applied approach for accomplishing this is to statically
recognize patterns of computation and use statistical techniques to
derive bounds[^1] [^2].

For example, we can use
[Hoeffding's Inequality](https://en.wikipedia.org/wiki/Hoeffding%27s_inequality)
to approximate the summation of variables.
Unfortunately it's use is constrained
in that the variables it sums must be independent
and bounded by some interval $[a,b]$.
If we meet these constraints then we can give
the following error bound,
with a probability of at least $1-\epsilon$,
on a summation of
only $m$ numbers out of an original $n$-number summation:

$$
(b-a) \sqrt{\frac{n(n-m)}{2m}\ln{\frac{2}{\epsilon}}}
$$

With further use of statistical techniques, it is possible to
derive static absolute error bounds when calculating mean,
argmin-sum, and ratio.
However, depending on which statistical
technique you use, you might have to prove or assume i.i.d.-ness of random
variables, that sequences of random variables are random walks,
the distribution of
random variables, or other properties.
These are all very domain dependent properties which don't fit well into
a generic programming framework.
Further, the patterns of computation these techniques work for (i.e. sum, mean, argmin-sum, and ratio)
are
very low level;
it's unclear whether these techniques can scale to high-level algorithms
without running into very advanced statistical techniques.
For example,
how would the error bounds on a sum help you give error bounds on a Fourier transform?

## Reasoning and Proving

There has been at least one effort in offering programmers the chance
to reason and prove properties of approximate programs[^3].
The dream is that you would be able to prove more complex properties
about more complex calculations than the pattern recognition tools
offer.

For example, the referenced paper gives an example of reasoning over a
program which skips locks to speed up execution time. The program is
14 lines. The reasoning, done in Coq, statically bounds
the error on the calculation.
The proof is 223 lines
(not counting comments,
blank lines,
or the almost 5k lines of Coq which make up the library which powers the proof).
I am not pointing out the line counts because I do not believe in this approach.
On the contrary, I _do_ believe in this approach—I just don't think it's
feasible to expect an average programmer today to reason about their approximate
programs using this framework.
Maybe in a few years; not now.


## A Hands on Approach

I do not believe that static analysis via pattern detection is a good
fit for generic approximate computing.
I _do_ believe that proving and reasoning could _become_ a good generic
solution if and when proof automation becomes more mainstream.
I believe the closest we can get today to a generic solution is to
offer programmers a toolbox which allows them to explore the effects
of approximate programming techniques in a hands on manner.

I've started writing a user-guided loop perforation system called aperf.
The code is on
[github](http://github.com/philipdexter/aperf).
Right now it only supports loop perforation.
Loop perforation is when some iterations of a process
are dropped in the name of approximate computing.
The goal is to drop as much as possible while still maintaining an
acceptable error/resource tradeoff.
aperf takes an annotated source program as input and automatically searches the error/time tradeoff space for a pareto curve.
A detailed report of the perforation options are returned the user.


[^1]: [Randomized Accuracy-Aware Program Transformations For Efficient Approximate Computations](http://dspace.mit.edu/openaccess-disseminate/1721.1/72439)

[^2]: [Probabilistically Accurate Program Transformations](http://dspace.mit.edu/openaccess-disseminate/1721.1/73897)

[^3]: [Proving Acceptability Properties of Relaxed and Nondeterministic Approximate Programs](http://people.csail.mit.edu/rinard/paper/pldi12.relaxed.pdf)
