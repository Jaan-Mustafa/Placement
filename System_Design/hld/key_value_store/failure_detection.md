# Failure Detection — How to Detect a Node Is Not Active

## The Core Challenge

```
Node A sends a request to Node B... no reply.
Is B dead?  Or is the network just slow/partitioned?  Or is B busy (GC pause)?
You CANNOT know for certain — you can only SUSPECT based on timeouts.
```

Failure detection is always about *probability and timeouts*, never certainty. That's why real systems use **suspicion levels** rather than a hard dead/alive flag.

---

## Method 1: Heartbeats (the basic building block)

Each node periodically sends a small "I'm alive" message (a **heartbeat**).

```
Every 1 second:   B ──"alive"──► A
A keeps a timer for B.

If A hears nothing from B for ~3× the interval (3s)
→ A marks B as DOWN.
```

- ✅ Simple.
- ❌ Fixed timeout is fragile: too short → false alarms on a slow network; too long → slow to detect real death.

Two flavors:
- **Push** — B actively sends heartbeats to A ("I'm alive").
- **Pull** — A periodically pings B and waits for a reply ("are you alive?").

---

## Method 2: Centralized Monitor (and why it's weak)

One coordinator pings everyone.

```
        ┌──ping──► N1
Monitor ┼──ping──► N2
        └──ping──► N3
```

- ❌ **Single point of failure** — if the monitor dies, detection stops.
- ❌ Doesn't scale — one node pinging thousands.
- Used in small clusters or with a reliable coordinator (Zookeeper-based).

---

## Method 3: Gossip Protocol (what Dynamo / Cassandra use)

Decentralized. Every node keeps a list of all nodes with a **heartbeat counter + last-updated time**, and **randomly gossips** its list to a few peers periodically.

```
Each node's membership table:
  Node   Heartbeat   Last seen
  N1     1034        10:00:05
  N2     2090        10:00:04
  N3     0871        10:00:01   ← counter stopped advancing

Every second, a node picks random peers and exchanges tables.
They merge: keep the highest heartbeat counter seen for each node.
```

**Detection rule:** if a node's heartbeat counter hasn't increased anywhere in the cluster beyond a threshold → mark it **down**, and gossip *that* around too.

- ✅ No single point of failure — detection spread across all nodes.
- ✅ Scales to thousands (each node talks to only a few peers).
- ✅ Info propagates fast (exponential spread, like a rumor).
- Used by **Cassandra, DynamoDB, Consul**.

---

## Method 4: Phi Accrual Failure Detector (the smart timeout)

Instead of a hard yes/no, output a **suspicion level (φ)** that rises the longer a node is silent — adapting to *actual* network conditions.

```
Learns the normal heartbeat-arrival pattern (mean + variance).
Computes φ = "how surprising is this silence, given history?"

φ low   → probably fine (silence within normal jitter)
φ high  → very likely dead
Set a threshold (e.g. act when φ > 8).
```

- ✅ **Adapts automatically** — tolerates more delay on a flaky network, reacts fast on a clean LAN. No magic fixed timeout.
- Used by **Cassandra and Akka** (on top of gossip).

---

## Avoiding False Positives

Because "slow ≠ dead," real systems add safety nets:

- **Multiple confirmations** — mark down only when *several* nodes independently report it unreachable (quorum of suspicions).
- **Indirect pings (SWIM protocol)** — if A can't reach B, A asks other nodes to ping B too.

```
A ──ping──► B   (timeout)
A asks C, D: "ping B for me"
   C ──ping──► B   (also timeout)
   D ──ping──► B   (also timeout)
→ Strong evidence B is down (not just A's link)
```

If one peer succeeds → it was just A's path that was bad, B is fine. (Used by **Consul/Serf, Hashicorp**.)

---

## Summary

| Method | How | Used by |
|---|---|---|
| **Heartbeat** | Periodic "I'm alive" + timeout | Building block of everything |
| **Centralized monitor** | One node pings all | Small clusters, Zookeeper-style |
| **Gossip** | Nodes randomly exchange membership tables | Cassandra, Dynamo, Consul |
| **Phi Accrual** | Adaptive suspicion score, not fixed timeout | Cassandra, Akka |
| **Indirect ping (SWIM)** | Ask peers to confirm before declaring dead | Consul/Serf |

**Headline:** detect a dead node via **heartbeats**, but since "slow" looks like "dead," scalable systems use **gossip** (no single point of failure), an **adaptive phi-accrual** detector (no fixed timeout), and **multiple/indirect confirmations** to avoid falsely killing a healthy-but-slow node.
