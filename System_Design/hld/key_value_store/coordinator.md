# Coordinator (in a Distributed Key-Value Store)

In a Dynamo-style key-value store, the **coordinator** is the node that **handles a particular client request** — it orchestrates the read or write for that key on behalf of the client. It is **not** a fixed "boss" node; the role is assigned **per-request**.

## What It Does

When a client sends `get(key)` or `put(key, value)`, the request lands on *some* node. That node becomes the **coordinator for that request**. Its job:

```
1. Figure out which nodes own this key
      → hash the key on the consistent-hashing ring
      → the key maps to a position; the next N nodes clockwise are the replicas
2. Forward the request to those N replica nodes
3. Wait for a quorum of responses (W acks for write, R for read)
4. Apply conflict resolution if reads disagree (version / vector clocks)
5. Return the result to the client
```

So the coordinator is the **orchestrator / middleman** between the client and the replica set.

## Picture

```
Client ──put(x, 10)──► [N3]  ◄── this node is the COORDINATOR for this request
                         │
       key x hashes to replicas {N4, N5, N6} on the ring
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
          [N4]         [N5]         [N6]      ← replica nodes that store x
            │            │            │
            └──── acks ──┴──── acks ──┘
                         │
       coordinator waits for W=2 acks → tells client "SUCCESS"
```

Here N3 (the coordinator) isn't even one of the replicas — it just *coordinates*. Often the coordinator **is** the first replica (first node clockwise from the key), the typical Dynamo setup.

## Key Properties

**1. Per-request, not a fixed master.** Any node can coordinate any request. The next request for a different key might be coordinated by a different node → no single point of failure, no special "coordinator server."

**2. How a node becomes coordinator:**
- Client hits a **load balancer** → routes to any node → that node coordinates (forwards to the correct replicas). Simple, but adds a hop if it's not a replica.
- Or a **smart client** that knows the ring picks a replica node directly → coordinates with no extra hop.

**3. It enforces quorum + consistency logic.** All the W/R quorum waiting, reading multiple versions, and conflict detection happens *at the coordinator*. Replicas just store/return data.

## Contrast: Coordinator vs Leader/Master

Don't confuse Dynamo's coordinator with a **leader** in leader-based systems (Raft / primary-replica DBs):

| | Dynamo coordinator | Leader / master (Raft, primary-replica) |
|---|---|---|
| Who | Any node, chosen per-request | One elected node, fixed until it fails |
| Role | Orchestrate one request's quorum | All writes must go through it |
| Failure | No big deal — next request uses another node | Triggers a leader election, brief unavailability |
| Style | Leaderless / masterless (AP) | Leader-based (often CP) |

Dynamo, Cassandra, Riak are **leaderless** — the coordinator is just "whoever took the request." A deliberate AP/availability choice.

## One-Line Summary
**The coordinator is the node handling a given client request — it locates the key's replica nodes on the ring, forwards the read/write to them, waits for the quorum, resolves conflicts, and replies; the role is assigned per-request, so there is no single master.**
