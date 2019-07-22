---
layout: post
title: Applying Reflective Consistency to Software Transactional Memory
---

This post will describe adding
[reflective consistency]({% post_url 2019-01-14-reflective %})
to a
[software transactional memory](https://en.wikipedia.org/wiki/Software_transactional_memory)
(STM) implementation in
[go](https://golang.org).

The
[github.com/lukechampine/stm](https://github.com/lukechampine/stm)
package implements STM for go.
The important function is
[verify](https://github.com/lukechampine/stm/blob/master/stm.go#L115),
which checks that all of the read STM variables in the current transaction
still hold the same values
(it does this by tracking and comparing a version number).
This logic is triggered inside
[Atomically](https://github.com/lukechampine/stm/blob/master/stm.go#L193),
the 'runner' for atomic transactions.
Adding reflective consistency to Atomically
is simple:
```diff
@@ -189,6 +190,11 @@ func catchRetry(fn func(*Tx), tx *Tx) (retry bool) {
        return
 }

+var commit_attempts = 0
+var anomalies = 0
+var target_anomaly_rate = 0.05
+
 // Atomically executes the atomic function fn.
 func Atomically(fn func(*Tx)) {
 retry:
@@ -204,9 +210,18 @@ retry:
        }
        // verify the read log
        globalLock.Lock()
+       commit_attempts += 1
        if !tx.verify() {
-               globalLock.Unlock()
-               goto retry
+               anomaly_rate := float64(anomalies) / float64(commit_attempts)
+               if anomaly_rate >= target_anomaly_rate {
+                       globalLock.Unlock()
+                       goto retry
+               } else {
+                       anomalies += 1
+               }

```
Now, 5% of transactions which should not be allowed (due to stale versions) are
allowed to commit.

I have not found any great application for this code yet.
An ideal application would have expensive retries (meaning allowing 5% through
will speed things up) and not too high cost of stale data (meaning 5% anomalies
don't cause too much trouble for the program or its users).
