---
layout: post
title: Checking, but not enforcing, Transaction Semantics
---

Checking against transaction semantics is much
easier than enforcing transaction semantics.

One way to check is to track versions for every value, for every replica.
A derivation of
[version vectors](https://en.wikipedia.org/wiki/Version_vector)
can be used.

We'll assume key-value API of get and put over single keys.
The get will return, in addition to the value, its version.
A put will contain, in addition to the updated value, the name and versions of
all of its dependent keys.
For example, the operation `y = x + y` where `x` and `y` are backed
by the key-value store would look like the following:

```python
x, version_x = get('x')
y, version_y = get('y')
put('y', x + y, {'x': version_x, 'y': version_y})
```

The server can then check the versions of the keys to see if stale values were
used.

What does this buy us? This can be used to implement
[reflective consistency]({% post_url 2019-01-14-reflective %}).
