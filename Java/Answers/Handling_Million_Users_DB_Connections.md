# Handling 1 Million Users — DB Connection Strategy

## First, Do the Math

```
1,000,000 users × 1000 requests / 10 minutes
= 1,000,000,000 requests / 600 seconds
= ~1,666,667 requests per second  (1.67 Million RPS)
```

A single PostgreSQL / MySQL instance handles **~5,000–10,000 queries/sec** at best.
A single DB connection handles **1 query at a time**.
A single HikariCP pool of 10 connections = **~10,000 queries/sec** (if each query is 1ms).

So hitting DB for every request at 1.67M RPS is **physically impossible** with just a pool.

**The answer is layered — you never let all 1.67M RPS hit the DB.**

```
1.67M RPS
    │
    ├── ~80% served from Cache (Redis)        → ~1.33M RPS absorbed
    ├── ~15% served from Read Replicas        → ~250K RPS absorbed
    └── ~5% actual DB writes / cache misses   → ~83K RPS hits DB
                                                (still needs sharding)
```

---

## Layer 1 — API Gateway + Rate Limiting (first line of defense)

Before requests even reach your service, throttle at the gateway.

```
1.67M RPS
    │
    ▼
[API Gateway — Kong / AWS API Gateway / Nginx]
    │
    ├── Rate limit per user: 100 req/sec
    ├── Rate limit per IP: 500 req/sec
    ├── Block burst attacks
    └── Route to correct service cluster
    │
    ▼
Legitimate traffic passes through (~1.67M RPS still, but controlled)
```

---

## Layer 2 — Horizontal Scaling of Application Servers

One service instance with 10 HikariCP connections handles ~10K queries/sec.
You need multiple instances behind a load balancer.

```
                    [Load Balancer]
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
  [Service Instance 1]  [Instance 2]  [Instance 3] ... [Instance N]
  HikariCP pool=10      pool=10       pool=10
  10K queries/sec       10K q/s       10K q/s
```

```
How many instances needed?
  Target: 1.67M RPS
  Per instance: ~10K q/s (with 10ms avg query time, pool=10)
  Instances needed: 1,670,000 / 10,000 = ~167 instances

  BUT — this assumes every request hits the DB.
  With caching, you need far fewer instances.
```

### HikariCP across instances

```
Each instance has its OWN HikariCP pool (pools are not shared across JVMs)

167 instances × pool size 10 = 1,670 total DB connections
DB server max connections: ~1,000 (Postgres default) → EXHAUSTED

Solution:
  Option A: Reduce pool size per instance (pool = 5 → 835 connections total)
  Option B: Use a connection pooler proxy (PgBouncer) — see Layer 4
  Option C: Read replicas to distribute connections
```

---

## Layer 3 — Caching (absorbs ~80% of traffic)

Most of 1.67M RPS is likely **reads** — same data read repeatedly.
Cache it. Never hit the DB for the same data twice.

```
Request flow with cache:
    │
    ▼
Check Redis cache
    │
    ├── HIT  (80% of requests) → return from Redis (~0.1ms)  ← NO DB touched
    │
    └── MISS (20% of requests) → query DB → store in Redis → return
```

### Cache strategies

```
Cache-Aside (most common):
    App checks Redis → miss → query DB → write to Redis → return

Write-Through:
    Every DB write also updates Redis → cache always fresh

Write-Behind (Write-Back):
    App writes to Redis only → async background job flushes to DB
    Risky but fast for write-heavy workloads
```

```java
@Service
public class UserService {

    @Autowired private RedisTemplate<String, User> redisTemplate;
    @Autowired private UserRepository userRepository;

    public User getUser(Long id) {
        String key = "user:" + id;

        // 1. Check cache
        User cached = redisTemplate.opsForValue().get(key);
        if (cached != null) return cached;   // cache hit — no DB call

        // 2. Cache miss — hit DB
        User user = userRepository.findById(id).orElseThrow();

        // 3. Store in cache (TTL = 10 minutes)
        redisTemplate.opsForValue().set(key, user, Duration.ofMinutes(10));
        return user;
    }
}
```

### Redis cluster for scale

```
Single Redis: ~100K ops/sec
Redis Cluster (6 nodes): ~600K ops/sec

For 1.67M RPS:
  Need ~3 Redis Clusters (sharded by key)
  or use Redis Enterprise / ElastiCache with auto-scaling
```

---

## Layer 4 — PgBouncer (Connection Pooler Proxy)

The real problem: 167 service instances × 10 connections = 1,670 connections to DB.
PostgreSQL forks a process per connection — 1,670 processes = huge memory + context switch overhead.

**PgBouncer sits between your app and DB and multiplexes connections:**

```
[Service Instance 1]  ──┐
[Service Instance 2]  ──┤
[Service Instance 3]  ──┤  →  [PgBouncer]  →  [PostgreSQL]
...                     │      500 app         50 real
[Service Instance 167]──┘      connections     DB connections
```

```
Your app thinks it has 1,670 connections.
PgBouncer actually maintains only 50-100 real connections to DB.
When a query needs to run, PgBouncer assigns a real connection.
When done, real connection returned to PgBouncer's pool.
```

### PgBouncer modes

```
Session pooling   — 1 real connection per app session (least sharing)
Transaction pooling — real connection held only for duration of transaction ← BEST
Statement pooling  — real connection held only for one statement (most sharing, risky)
```

```ini
# pgbouncer.ini
[pgbouncer]
pool_mode = transaction
max_client_conn = 10000     # app connections PgBouncer accepts
default_pool_size = 50      # real connections to PostgreSQL
server_pool_size = 100      # per-database real connection cap
```

---

## Layer 5 — Read Replicas (split read/write traffic)

In most apps, 90% of traffic is reads. Route reads to replicas.

```
Writes (10%) → Primary DB  (1 server)
Reads  (90%) → Replica 1
               Replica 2    (N servers — scale reads horizontally)
               Replica 3
```

```yaml
# Spring Boot — separate DataSources
spring:
  datasource:
    primary:
      url: jdbc:postgresql://primary-db:5432/mydb
      hikari:
        maximum-pool-size: 10
    replica:
      url: jdbc:postgresql://replica-db:5432/mydb
      hikari:
        maximum-pool-size: 30    # more connections for heavy reads
```

```java
@Service
public class OrderService {

    @Autowired @Qualifier("primaryJdbc")   private JdbcTemplate writeDb;
    @Autowired @Qualifier("replicaJdbc")   private JdbcTemplate readDb;

    public void createOrder(Order order) {
        writeDb.update("INSERT INTO orders ...");    // goes to primary
    }

    public List<Order> getOrders(Long userId) {
        return readDb.query("SELECT * FROM orders WHERE user_id=?", ...);  // replica
    }
}
```

With 5 read replicas each handling 50K q/s → **250K read q/s** capacity just from replicas.

---

## Layer 6 — Async Processing + Message Queue (for writes)

Don't make users wait for heavy write operations. Decouple with a queue.

```
User creates order
    │
    ▼
POST /orders → validate → publish to Kafka → return 202 Accepted  (fast, ~5ms)
                                │
                                ▼
                        Kafka Consumer Service
                                │
                         ┌──────┴──────┐
                         ▼             ▼
                    Write to DB    Send email
                    (in batches)   notification
```

**Batch DB writes — massive throughput gain:**

```java
// Instead of 1000 individual INSERTs:
INSERT INTO orders VALUES (...);   // × 1000 = 1000 round trips

// Batch insert:
INSERT INTO orders VALUES (...), (...), (...);  // 1 round trip, 1000 rows
// ~100x faster
```

---

## Layer 7 — DB Sharding (for extreme write scale)

When even primary DB can't handle write load, shard the data.

```
User ID hash % 4 → determine shard

User IDs 1–25M      → Shard 0 (DB Server 0)
User IDs 25M–50M    → Shard 1 (DB Server 1)
User IDs 50M–75M    → Shard 2 (DB Server 2)
User IDs 75M–100M   → Shard 3 (DB Server 3)
```

Each shard has its own primary + replicas + PgBouncer + HikariCP pools.

---

## Complete Architecture for 1.67M RPS

```
1,670,000 req/sec
        │
        ▼
[CDN — static content]  ← serve static assets, ~50% traffic never reaches backend
        │
        ▼ ~800K RPS
[API Gateway + Rate Limiter]
        │
        ▼ ~800K RPS
[Load Balancer]
        │
   ┌────┼────┐
   ▼    ▼    ▼
[App]  [App]  [App]  × N instances  (auto-scaled)
   │
   ▼
[Redis Cluster]  ←── Cache HIT (~80%) → return immediately, no DB
   │
   │ Cache MISS (~20%) = ~160K RPS
   ▼
[PgBouncer]      ←── multiplexes 1000s of app connections → 100 real DB connections
   │
   ├─────────────────────────────────────┐
   ▼                                     ▼
[Primary DB]  ← writes only (~10%)   [Read Replica × 5]  ← reads (~90%)
   │
   ▼ async
[Kafka]  ←── heavy writes queued, processed in batches
```

---

## Numbers Summary

| Layer | RPS Absorbed | How |
|---|---|---|
| CDN | ~500K | Static assets never hit app |
| Rate Limiting | throttles abuse | Gateway blocks excess |
| Redis Cache | ~1.1M | 80% cache hit rate |
| Read Replicas | ~100K | 5 replicas × 20K q/s |
| PgBouncer | multiplexes | 1000 app → 100 real DB connections |
| Primary DB | ~67K writes | Async batch via Kafka |
| **DB sees** | **~167K RPS** | **manageable** |

---

## Key Interview Points

1. **Never let all traffic hit the DB** — cache is your first and biggest weapon (Redis absorbs 80%).

2. **HikariCP pool size stays small per instance (10–20)** — more instances, not bigger pools.

3. **PgBouncer is essential at scale** — 167 instances × 10 connections = 1,670 would kill PostgreSQL. PgBouncer collapses that to 50–100 real connections.

4. **Read replicas split the read/write bottleneck** — 90% of queries are reads; scale reads independently.

5. **Async writes via Kafka** — don't hold a DB connection open waiting for downstream processing; publish event, return fast, process in background.

6. **Sharding is a last resort** — it adds enormous operational complexity; exhaust caching, read replicas, and vertical scaling first.

---

## One-Line Summary for Interviews

> "At 1.67M RPS you never hit the DB directly — Redis absorbs 80% of reads with cache hits, read replicas handle the remaining reads, PgBouncer collapses thousands of app connections into ~100 real DB connections, Kafka queues and batches heavy writes, and HikariCP on each app instance stays small (pool=10) while you scale out the number of instances horizontally."
