# Internal Working of ConcurrentHashMap

## What is it?

`ConcurrentHashMap` is a thread-safe version of `HashMap` introduced in Java 5, redesigned completely in **Java 8**. It allows **concurrent reads and limited concurrent writes** without locking the entire map — unlike `Hashtable` which locks the whole map on every operation.

---

## Java 7 vs Java 8 — Two Different Internals

### Java 7: Segment-based locking (Striped Locking)

```
ConcurrentHashMap
├── Segment[0]  (ReentrantLock)  → array of buckets
├── Segment[1]  (ReentrantLock)  → array of buckets
├── Segment[2]  (ReentrantLock)  → array of buckets
└── ...up to 16 segments by default
```

- The map was divided into **16 segments** (configurable via `concurrencyLevel`).
- Each segment was an independent `ReentrantLock`-protected `HashMap`.
- Two threads writing to **different segments** never blocked each other.
- Two threads writing to the **same segment** — one blocked.

**Limitation:** Segment overhead, complex code, lock still coarse-grained at segment level.

---

### Java 8: Node array + CAS + synchronized block (current default)

Java 8 threw away segments entirely. Internally it's now very close to `HashMap` structure but with fine-grained synchronization.

```
ConcurrentHashMap (Java 8+)
│
├── Node<K,V>[] table   (volatile array)
│     │
│     ├── bucket[0]  → null
│     ├── bucket[1]  → Node → Node → Node   (linked list, len < 8)
│     ├── bucket[2]  → TreeBin (Red-Black Tree, len >= 8)
│     └── ...
│
└── Lock granularity: ONE synchronized block per BUCKET HEAD
```

**Lock is taken only on the first node (head) of each bucket** — so two threads writing to different buckets never block each other.

---

## Core Internal Structure

```java
// Simplified internal Node
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;       // volatile — reads never need a lock
    volatile Node<K,V> next;
}
```

```java
// The table itself
transient volatile Node<K,V>[] table;
```

Both `val` and `next` are `volatile` — this is how **reads are always lock-free**.

---

## How `put()` Works Internally

```java
map.put("key", "value");
```

**Step-by-step:**

```
1. hash = spread(key.hashCode())    // extra bit-mixing to reduce collisions

2. index = hash & (table.length - 1)

3. Read table[index]

   ├── CASE A: bucket is null
   │     └── CAS (Compare-And-Swap) to insert new Node atomically
   │           if CAS fails (race) → retry loop
   │
   ├── CASE B: bucket head has hash == MOVED (-1)
   │     └── another thread is resizing → help with resize, then retry
   │
   └── CASE C: bucket has existing nodes
               └── synchronized(bucketHead) {
                       walk the linked list / tree
                       if key exists → update val
                       else → append new Node at tail
                   }
```

```java
// CAS path (Case A) — no lock needed
casTabAt(tab, i, null, new Node<>(hash, key, value));

// Synchronized path (Case C) — lock only this one bucket
synchronized (f) {   // f = head node of bucket
    // insert or update
}
```

---

## How `get()` Works Internally

**Completely lock-free.**

```java
map.get("key");
```

```
1. hash = spread(key.hashCode())
2. index = hash & (table.length - 1)
3. Read table[index]  ← volatile read, always sees latest value
4. Walk the chain / tree comparing keys
5. Return val  ← volatile field, always current
```

No lock, no CAS — just volatile reads. This is why `ConcurrentHashMap` reads are extremely fast even under heavy write contention.

---

## How `size()` Works — LongAdder trick

`size()` does **not** lock the whole map. Internally it uses a distributed counter:

```java
// Simplified
private transient volatile long baseCount;          // uncontended path
private transient volatile CounterCell[] counterCells;  // contended path
```

- On low contention: increments `baseCount` via CAS.
- On high contention (CAS fails): increments a random slot in `counterCells` array.
- `size()` = `baseCount` + sum of all `counterCells`.

This is the same idea as `LongAdder` — stripe the counter to reduce CAS contention.

> `size()` is **approximate** in a live concurrent map. The value is accurate only if no writes happen during the call.

---

## Resizing — Cooperative / Assisted

When load factor exceeds threshold (default 0.75), the table resizes to 2x.

```
Old table (16 buckets) → New table (32 buckets)
```

**Key insight:** Multiple threads help with the resize simultaneously.

- The table is split into **stripes**.
- Each thread claims a stripe by CAS-setting `transferIndex`.
- Threads move buckets from old → new table in parallel.
- Buckets already moved are marked with a **ForwardingNode** (hash = MOVED = -1).
- A `put()` or `get()` that hits a ForwardingNode follows it to the new table.

This avoids a single thread doing all the work during resize — the whole map doesn't stall.

---

## Treeification — Same as HashMap

When a bucket's linked list length hits **8** and total map size >= **64**:

```
Linked list → Red-Black Tree (TreeBin)
```

When elements are removed and count drops to **6**:

```
Red-Black Tree → Linked list (untreeify)
```

Tree operations are also protected by `synchronized(bucketHead)`.

---

## Null Keys/Values — Not Allowed

```java
map.put(null, "value");   // NullPointerException
map.put("key", null);     // NullPointerException
```

**Why?** In a concurrent context, `get("key") == null` is ambiguous:
- Does the key not exist?
- Or was `null` explicitly stored?

`HashMap` tolerates this ambiguity (you can call `containsKey` after). In `ConcurrentHashMap`, by the time you call `containsKey`, another thread may have inserted or removed the key — so the two-step check is inherently racy. Banning null removes the ambiguity entirely.

---

## Comparison Table

| Feature | HashMap | Hashtable | ConcurrentHashMap |
|---|---|---|---|
| Thread-safe | No | Yes (whole map lock) | Yes (bucket-level lock) |
| Null keys/values | Yes | No | No |
| Read lock | No lock | Full lock | No lock (volatile) |
| Write lock | No lock | Full lock | Per-bucket lock |
| Java 8 structure | Array + List/Tree | Array + List | Array + List/Tree |
| Concurrent reads | Unsafe | Blocking | Always safe |
| Resize | Single thread | Single thread | Multi-thread cooperative |
| `size()` accuracy | Exact | Exact | Approximate (live) |

---

## Key Interview Points

1. **Reads are always lock-free** — `volatile` on `val` and `next` ensures visibility without locking.

2. **Write lock granularity = one bucket head** — not the whole map, not a segment.

3. **CAS is used for empty buckets** — no lock at all when inserting into an empty slot.

4. **Resize is cooperative** — threads help each other; ForwardingNode signals "bucket moved."

5. **`size()` is approximate** — uses `LongAdder`-style striped counter, not a single locked variable.

6. **Java 7 vs Java 8** — Segment/ReentrantLock vs Node/CAS/synchronized. Java 8 is finer-grained and faster.

---

## One-Line Summary for Interviews

> "In Java 8, `ConcurrentHashMap` uses a volatile Node array where reads are always lock-free via volatile fields, empty-bucket writes use CAS, and non-empty bucket writes take a synchronized lock only on that bucket's head node — giving per-bucket write granularity instead of locking the entire map."
