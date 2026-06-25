# Conflict Resolution & Reconciliation in Distributed Key-Value Stores

When two different servers (replicas) update the **same key** concurrently — typically during a network partition — both sides end up with different values. **Reconciliation** = deciding which value wins, or merging them, once they sync back up. This is the core problem in AP systems (Dynamo, Cassandra).

## The Problem

```
Partition happens. Client writes to BOTH sides:
   N1 [ cart = {apple} ]   <--- X --->   N2 [ cart = {banana} ]

Link heals. Now N1 says {apple}, N2 says {banana}.
Which is correct? → this is the CONFLICT we must reconcile.
```

---

## Strategy 1: Last Write Wins (LWW)

Attach a **timestamp** to every write. On conflict, keep the one with the latest timestamp, discard the other.

```
N1: cart = {apple},  timestamp = 10:00:05
N2: cart = {banana}, timestamp = 10:00:07   ← newer → wins
Result: cart = {banana}.  {apple} is discarded.
```

- ✅ Dead simple. Used by **Cassandra** by default.
- ❌ **Silently loses data** (the loser vanishes). Relies on synchronized clocks → clock skew can pick the wrong winner.
- Fine for: "last known status", a profile field. Bad for: shopping carts, counters.

---

## Strategy 2: Vector Clocks (detect conflicts accurately)

**Dynamo's** approach. Instead of one timestamp, attach a **version vector** — a counter per node — to track causality (which version descended from which).

A vector clock is a list: `[N1:count, N2:count, ...]`

```
Start:  cart = {}, version = []

Client writes via N1:   cart = {apple},  version = [N1:1]
N1 replicates to N2.    Both at [N1:1].

PARTITION. Two clients write:
  via N1: cart = {apple, milk},   version = [N1:2]
  via N2: cart = {apple, banana}, version = [N1:1, N2:1]
```

Compare the two vectors after heal:
- `[N1:2]` vs `[N1:1, N2:1]` → neither is ≥ the other on all entries → **concurrent → real conflict.**

| Comparison | Meaning | Action |
|---|---|---|
| `[N1:1]` vs `[N1:2]` | First is ancestor of second | Auto-pick newer (no conflict) |
| `[N1:2]` vs `[N1:1, N2:1]` | Concurrent — neither descends from other | **Genuine conflict** — must reconcile |

> Vector clocks don't *resolve* the conflict — they **accurately detect** it (unlike timestamps, no false positives/negatives). Then you resolve it.

---

## Strategy 3: Application-Level Merge (semantic reconciliation)

When a true conflict is flagged, hand **both versions** back to the application and let it merge by meaning.

Dynamo's classic **shopping cart** example:

```
Conflict: N1 cart = {apple, milk},  N2 cart = {apple, banana}

Merge rule for a cart = UNION of items:
Result = {apple, milk, banana}
```

User never loses an item — worst case a deleted item reappears (acceptable for a cart). The *meaning* of the data decides the merge rule. This is why Amazon chose AP for carts: never block "add to cart", reconcile later.

---

## Strategy 4: CRDTs (auto-merging data types)

**Conflict-free Replicated Data Types** — structures mathematically designed so concurrent updates **always merge automatically**.

```
G-Counter (grow-only counter), e.g. "likes":
  N1 counts +3 → [N1:3]
  N2 counts +5 → [N2:5]
  Merge = sum each node's count = 3 + 5 = 8   ✓ always correct
```

Used for counters, sets, collaborative editors (Riak, Redis CRDTs). No manual merge — the type guarantees convergence.

---

## Strategy 5: Quorum (prevent conflicts upfront)

Reduce conflicts in the first place with **W + R > N**:
- N = number of replicas, W = nodes that must ack a write, R = nodes read from.
- If W + R > N, every read set overlaps the latest write set → you read fresh data.

```
N=3, W=2, R=2 → W+R=4 > 3 → read set always overlaps write set → strong-ish consistency
```

Shifts you toward CP. Doesn't help during a full partition (can't reach quorum → unavailable), but reduces conflict frequency in normal operation.

---

## How It Fits Together (Dynamo-style flow)

```
1. Quorum (W+R>N)      → avoid most conflicts during normal operation
2. Vector clocks       → accurately DETECT real conflicts when they slip through
3. App merge / CRDTs   → RESOLVE the conflict (union cart, sum counter, etc.)
4. LWW                 → fallback when you don't care about losing the loser
```

## Summary

| Technique | What it does | Trade-off |
|---|---|---|
| **Last Write Wins** | Newest timestamp wins | Simple, but silently drops data + clock skew |
| **Vector clocks** | *Detect* concurrent vs causal writes | No data loss, but more metadata + needs resolution step |
| **App-level merge** | Merge by meaning (cart union) | Correct, but you must write merge logic |
| **CRDTs** | Auto-merge by data-type math | No conflict ever, but limited to certain types |
| **Quorum (W+R>N)** | Prevent conflicts upfront | Less availability during partitions |

**Headline:** detect conflicts properly with **vector clocks**, then resolve via **application merge or CRDTs**; use **LWW** only when losing the loser is acceptable; use **quorums** to keep conflicts rare in the first place.
