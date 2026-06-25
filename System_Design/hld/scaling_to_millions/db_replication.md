# Database Replication (Master-Slave)

Each master and each slave is a **separate physical/virtual server** with its own CPU, RAM, and disk.

## The Setup

```
        Master DB Server
        (handles WRITES)
        /      |       \
       ↓       ↓        ↓
  Slave 1   Slave 2   Slave 3
(READ only) (READ only) (READ only)
```

- **Master** — 1 server, accepts all writes (INSERT, UPDATE, DELETE)
- **Slaves** — N servers, each a full copy, handle only reads (SELECT)

## How Data Stays Consistent

Through **replication logs** — master logs every change, slaves replay that log.

### Step by Step

1. Write hits the **master** → master saves the change → writes to a **replication log (binlog)**
2. Each slave **continuously listens** to master's binlog
3. Slave pulls the log → **replays the same operations** on its own data
4. Slave now has the same data as master

```
Master:  INSERT user(Alice)  →  logs it  →  sends to slaves
Slave1:  receives log        →  replays INSERT user(Alice)  ✓
Slave2:  receives log        →  replays INSERT user(Alice)  ✓
```

## The Catch — Replication Lag

Slaves sync **asynchronously** by default — there's a tiny delay (milliseconds to seconds).

```
t=0ms  → Master writes Alice
t=50ms → Slave syncs Alice
```

If you read from a slave at `t=10ms` — Alice won't be there yet.

### How to Handle It

| Scenario | Solution |
|----------|----------|
| Read your own writes | Route that read to master |
| Critical reads | Always hit master |
| Analytics, feeds | Slaves are fine (slight delay ok) |

**TL;DR:** Master and each slave are unique independent servers. Data is consistent via replication logs — slaves replay every write the master does. There's a small replication lag to account for.
