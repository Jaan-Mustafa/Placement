# Consistent Hashing

## Foundations: Key, Hash, and Why We Distribute

### What is a "key"?
A **key** is the unique identifier used to look something up — the "name" of a piece of data.

```
Like a dictionary:
  key   = the word        value = the definition
  key   = person's name   value = phone number

Real examples:
  user_id = 12345      → key for that user's profile
  "session_abc123"     → key for a login session
  "product:998"        → key for product details
```

Whenever you store or fetch something, you ask: "give me the value for *this key*."

### What is a "hash"?
A **hash function** turns any input (a key) into a big fixed-size number.

```
hash("user_12345")  →  3829104857
hash("user_99999")  →  1024857263
hash("hello")       →  9982134
```

Properties:
- **Deterministic** — same input always gives the same number.
- **Spreads evenly** — similar inputs (`user_1` vs `user_2`) scatter to very different numbers.
- **Fast** to compute.

Why turn a key into a number? A number is easy to do math on — like deciding *which server* should hold it. `"user_12345"` is hard to route; `3829104857` you can `% N` or place on a ring.

> Hashing = a meat grinder: put anything in, get a consistent scrambled number out. Same input → same number, every time.

### Why distribute among servers?
One server can't do everything:

1. **Storage** — limited disk/RAM. 1 billion users won't fit on one machine → split across many, each holds a slice.
2. **Traffic / load** — one server handles ~10k req/s; 100k req/s melts it → spread across 10 servers.
3. **Reliability** — if everything is on one box and it dies, *everything* goes down → spread it out (and keep copies).

Splitting data across servers is called **sharding** / **partitioning**.

### How it connects
```
key  ──hash──►  number  ──pick server──►  server holds the value
"user_12345"     3829104857    % 10 = 7      → Server #7
```

---

## Why Consistent Hashing Is Used

### The problem with naive hashing
Assign each key with:
```
server = hash(key) % N
```
Works fine — until N changes. Add or remove one server and N changes, so `hash(key) % N` changes for **almost every key**.

- In a distributed cache → near-total cache-miss storm.
- In a sharded DB → nearly all data must move.

Remapping ~100% of keys for a 1-server change is catastrophic.

### What consistent hashing does
Map **both keys and servers onto the same circular hash space** (a "ring", e.g. 0 to 2³²−1).

- Each server is hashed to one or more points on the ring.
- Each key is hashed to a point on the ring.
- A key belongs to the **first server clockwise** from the key's position.

```
Ring:   ...→ [key] → Server B → [key] → [key] → Server C → ...
         each key handled by the next server clockwise
```

Now:
- **Add a server** → it only steals keys in the arc between it and the previous server (~K/N keys move).
- **Remove a server** → only its keys move to the next server clockwise. Everyone else untouched.

> Key property: adding/removing a node remaps only ~1/N of keys, instead of ~all of them.

---

## Issues in Consistent Hashing & How We Fix Them

### Issue 1: Uneven load distribution (the big one)
With one random point per server, the gaps between servers are unequal — some own a huge arc, some a tiny one → some servers overloaded, others idle.

**Fix: Virtual nodes (vnodes)** — place each physical server at **many** points on the ring (e.g. 100–200 virtual copies). Many small arcs per server, scattered around → total load averages out to roughly equal. *(Used by Cassandra, DynamoDB.)*

### Issue 2: Cascading failure when a node dies
In a plain ring, a dead server dumps **all** its keys onto the *single* next server clockwise → that neighbor gets double load → domino collapse.

**Fix: Virtual nodes again** — a server's keys are spread across many arcs, so on death they redistribute to **many** servers, not one. No single neighbor gets crushed.

### Issue 3: Hot keys (skewed access)
Even with even key distribution, one key can be wildly popular (celebrity profile, viral product). It lives on one server → that server gets slammed. Consistent hashing balances *where keys live*, not *how often each is accessed*.

**Fixes:**
- **Replication** — store hot keys on multiple servers, spread reads.
- **Caching** — cache layer in front of the hottest keys.
- **Bounded loads** — cap how much any one server holds; overflow spills to the next.

### Issue 4: Heterogeneous servers (different sizes)
Basic ring assumes equal servers, but a 64GB box shouldn't get the same load as a 16GB box.

**Fix: Weighted virtual nodes** — bigger servers get more vnodes (64GB → 4× the vnodes of 16GB → ~4× the keys).

### Issue 5: Maintaining the ring / lookups
Need fast "next server clockwise" lookup and all clients agreeing on the ring state.

**Fixes:**
- **Sorted structure** (sorted map / TreeMap / binary search over vnode positions) → `O(log n)` lookup.
- **Gossip protocol** (Cassandra) or **ZooKeeper/etcd** keeps every node's view of the ring in sync.

### Summary table

| Issue | Cause | Fix |
|---|---|---|
| Uneven load | One point per server → unequal arcs | **Virtual nodes** |
| Cascading failure | Dead node dumps all keys on one neighbor | **Virtual nodes** (spread load) |
| Hot keys | One key super popular | **Replication + caching + bounded loads** |
| Unequal server sizes | Ring assumes equal capacity | **Weighted vnodes** |
| Lookup / sync | Find next node, agree on ring | **Sorted map + gossip / ZooKeeper** |

> Headline: most problems trace back to "one point per server is too lumpy" — the master fix is **virtual nodes**. The rest (replication, weighting, bounded loads) handle access skew and uneven hardware.

---

## Where It's Used
- **Distributed caches** — Memcached clients, Redis Cluster (related hash-slot scheme)
- **Databases** — Amazon DynamoDB, Apache Cassandra, Riak
- **Load balancers / CDNs** — route a client consistently to the same backend
- **Sharding** in general — any time you partition across nodes that scale up/down

## One-Line Summary
**Use consistent hashing whenever you partition data/requests across a dynamic set of nodes and you want adding/removing a node to disturb as few keys as possible.**

---

## Worked Example: Virtual Nodes Step by Step

Using a small ring of **0–99** (instead of 0 to 2³²) just to keep the math readable.

### Step 1: 3 servers, the NAIVE way (one point each)

Hash each server name to a position:
```
hash("ServerA") = 10
hash("ServerB") = 40
hash("ServerC") = 45
```

Place on the ring (0 → 99, wrapping):
```
0 ........ 10 ............... 40 .. 45 ................. 99 (→back to 0)
           A                  B    C
```

**Rule:** a key belongs to the first server clockwise (higher numbers, wrapping past 99 to 0).
Each server owns the arc *ending at its position*:

| Server | Owns positions | Arc size |
|---|---|---|
| **A** (10) | 46 → 99 → 0 → 10 | **65 slots** 😱 |
| **B** (40) | 11 → 40 | 30 slots |
| **C** (45) | 41 → 45 | **5 slots** |

**Problem:** A owns 65% of the ring (overloaded), C owns 5% (idle) — purely by luck of where hashes landed (B and C hashed close together → big empty gap A swallowed). This is the **uneven load** issue.
Worse: if **A dies**, its 65 slots all dump onto **B** → B now owns 95 → it collapses too = **cascading failure**.

### Step 2: The FIX — Virtual nodes

Hash each server **multiple times** with a suffix → several scattered points per physical server.
Give each server **3 virtual nodes** (real systems use 100–200; 3 just for the example):
```
hash("ServerA#1") = 5      hash("ServerB#1") = 15     hash("ServerC#1") = 25
hash("ServerA#2") = 50     hash("ServerB#2") = 60     hash("ServerC#2") = 70
hash("ServerA#3") = 85     hash("ServerB#3") = 90     hash("ServerC#3") = 35
```

All 9 points on the ring, sorted:
```
Pos:  5    15   25   35   50   60   70   85   90
Node: A#1  B#1  C#1  C#2  A#2  B#2  C#3  A#3  B#3
       A    B    C    C    A    B    C    A    B
```

> Key insight: A#1, A#2, A#3 are the **same physical Server A** — just 3 different doorways into it.

Who owns each arc (arc ends at each point, clockwise):

| Point (pos) | Server | Owns arc | Size |
|---|---|---|---|
| A#1 (5) | A | 91→5 | 14 |
| B#1 (15) | B | 6→15 | 10 |
| C#1 (25) | C | 16→25 | 10 |
| C#2 (35) | C | 26→35 | 10 |
| A#2 (50) | A | 36→50 | 15 |
| B#2 (60) | B | 51→60 | 10 |
| C#3 (70) | C | 61→70 | 10 |
| A#3 (85) | A | 71→85 | 15 |
| B#3 (90) | B | 86→90 | 5 |

Totals **per physical server**:

| Server | Arcs | Total slots |
|---|---|---|
| **A** | 14 + 15 + 15 | **44** |
| **B** | 10 + 10 + 5 | **25** |
| **C** | 10 + 10 + 10 | **30** |

Compare: **44 / 25 / 30** (with vnodes) vs **65 / 30 / 5** (without) — far more balanced. With 150 vnodes each it becomes nearly perfectly even (law of averages). More scattered points → smoother distribution.

### Step 3: Why it fixes cascading failure

If **Server A dies**, its points (5, 50, 85) vanish; each arc goes to the next point clockwise. With realistic vnode counts (100+), A's points are scattered *everywhere*, so its load splits across **all** remaining servers — a little to each — instead of one neighbor inheriting everything.

### Mental model
```
WITHOUT vnodes:  each server = 1 big territory
                 → wildly different sizes
                 → one dies → 1 neighbor inherits ALL of it

WITH vnodes:     each server = 100 tiny scattered plots
                 → total land per server evens out
                 → one dies → plots shared by everyone nearby
```

Same ring — you just register each physical server at **many positions** instead of one. Lookup is unchanged: hash the key, walk clockwise to the next point, that point names the physical server (`A#2` → Server A).

### Bonus: weighting via vnode count
Give bigger servers more vnodes → they own more of the ring:
```
Server A (64 GB)  → 200 vnodes  → owns ~4× the ring
Server B (16 GB)  → 50  vnodes  → owns ~1× the ring
```
More plots = more total land = more keys.
