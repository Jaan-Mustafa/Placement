# Sharding (Horizontal DB Scaling)

Split data across multiple DB servers based on a **shard key**.

```
Users 1-10M    → Shard 1 (DB Server 1)
Users 10M-20M  → Shard 2 (DB Server 2)
Users 20M-30M  → Shard 3 (DB Server 3)
```

## Limits of Sharding

### 1. Cross-Shard Queries (Joins)
```sql
-- Easy on single DB
SELECT users.name, orders.total
FROM users JOIN orders ON users.id = orders.user_id

-- With sharding, user is on Shard 1, order might be on Shard 2
-- No direct JOIN possible across servers
```

### 2. Hotspot / Uneven Distribution
```
Shard 1 (A-F users) → 80% of traffic  ← overloaded
Shard 2 (G-Z users) → 20% of traffic  ← underloaded
```
Bad shard key choice causes one shard to get hammered.

### 3. Resharding Problem
You start with 4 shards, now need 8.
- Have to **move massive amounts of data** around
- Downtime risk
- Very painful operation

### 4. No Cross-Shard Transactions
```
Transfer $100 from User A (Shard 1) → User B (Shard 2)
-- Atomic transaction across two DB servers is very hard
```

## How We Handle These Limits

| Problem | Solution |
|---|---|
| Cross-shard JOINs | Denormalize data — store redundant data together in same shard |
| Hotspots | Choose a better shard key (e.g. hash-based instead of range-based) |
| Resharding | **Consistent Hashing** — minimizes data movement when adding shards |
| Cross-shard transactions | Avoid them by design, or use distributed transaction protocols (2PC) |

## Consistent Hashing (Key Solution)

Normal sharding — add 1 shard, **everything remaps**:
```
Before: 4 shards → 75% of data moves
After consistent hashing: add 1 shard → only ~20% of data moves
```
Places shards on a ring — only the adjacent shard's data moves.

## When Sharding Itself Hits a Limit

- **Archive old data** — move cold data to cheap storage (S3)
- **Switch to NoSQL** — Cassandra, DynamoDB shard natively by design
- **CQRS** — separate read and write databases entirely

**TL;DR:** Sharding limits are cross-shard JOINs, hotspots, resharding pain, and no atomic transactions. Main solutions are good shard key design, consistent hashing, and denormalization.
