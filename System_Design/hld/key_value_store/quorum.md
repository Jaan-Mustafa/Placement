# Quorum

A **quorum** is the *minimum number of nodes that must agree (respond) before an operation is considered successful.* It's how distributed systems get consistency without needing *all* replicas to be up.

## The Basic Idea

Data is copied on **N replicas**. Instead of requiring all N to respond (too fragile — one dead node blocks everything), you require a majority-ish subset.

- **N** = total number of replicas of the data
- **W** = write quorum — how many nodes must **acknowledge a write** before it's "successful"
- **R** = read quorum — how many nodes you must **read from** before returning an answer

```
N = 3 replicas:  [N1] [N2] [N3]

Write x=10 with W=2:
   send to all 3, but only WAIT for 2 acks
   N1 ✓  N2 ✓  (N3 slow/down — don't care) → write SUCCESS

Read x with R=2:
   read from 2 nodes, take the freshest value (by version/timestamp)
```

## The Magic Rule: W + R > N

If **W + R > N**, the set of nodes you wrote to and the set you read from are **guaranteed to overlap by at least one node** → any read is guaranteed to see the latest write.

```
N=3, W=2, R=2  →  W+R = 4 > 3  ✓

Write went to {N1, N2}
Read comes from {N2, N3}
            ↑ overlap! N2 has the fresh value → read returns latest
```

Why overlap is guaranteed: with N=3, the write touched 2 nodes and the read touched 2 nodes. By pigeonhole, two sets of size 2 out of 3 *must* share ≥1 node. That shared node carries the newest value.

> **W + R > N → strong consistency.** The read set and write set always intersect.

## Tuning the Knobs

Choose W and R based on read-heavy vs write-heavy workload:

| Config (N=3) | W+R>N? | Behavior |
|---|---|---|
| W=3, R=1 | 4>3 ✓ | **Fast reads, slow writes.** Read 1 node (latest always present since all 3 hold it). Read-heavy. |
| W=1, R=3 | 4>3 ✓ | **Fast writes, slow reads.** Write to 1, must read all 3 to be sure. Write-heavy. |
| W=2, R=2 | 4>3 ✓ | **Balanced** — common default. |
| W=1, R=1 | 2>3 ✗ | **Fastest, but NO consistency guarantee** — read may miss the write. Pure AP / eventual consistency. |

## Why Not Just Require All N?

```
W = N = 3:  write must wait for ALL 3 nodes.
If even ONE node is down or slow → the write blocks/fails.
```

That kills availability. Quorums tolerate failures: with N=3, W=2 you can lose **1 node** and still write.
Fault tolerance: you can lose `N - W` nodes for writes, `N - R` for reads.

## Connecting to CAP

- **W + R > N** → pushes toward **CP** (strong consistency, but during a partition you may not reach quorum → unavailable).
- **W + R ≤ N** (e.g. W=1, R=1) → pushes toward **AP** (always answer, but possibly stale).

Quorum is the **tunable dial** between consistency and availability. Dynamo/Cassandra let you set N, W, R *per query* so you pick the trade-off case by case.

## One-Line Summary
**A quorum is the minimum number of nodes that must respond for an operation to count; setting W + R > N guarantees read and write sets overlap, giving strong consistency while still tolerating some node failures.**
