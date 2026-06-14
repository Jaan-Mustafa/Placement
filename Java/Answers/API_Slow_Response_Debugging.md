# Your API is Working Fine, but Users Report Slow Responses — Where Would You Start?

## The Mindset — Measure Before You Fix

Never guess. Slow responses have many causes — DB, network, GC, thread starvation, N+1 queries, missing index, downstream service, memory pressure. Each fix is different. **Instrument first, fix second.**

```
Slow response = time is being spent SOMEWHERE
Your job      = find exactly WHERE before touching code
```

---

## Step 1 — Establish the Baseline (What is "slow"?)

Before debugging, define the problem precisely.

```
Questions to ask:
  - Is it ALL endpoints or ONE specific endpoint?
  - Is it ALL users or specific users (region, plan, data size)?
  - Is it constant slowness or spikes at certain times?
  - Since when? Did a deployment happen recently?
  - p50 slow or p99 slow? (median vs tail latency)
```

```bash
# Check recent deployments
git log --oneline -10

# Check if it correlates with traffic spikes
# Check your APM / metrics dashboard first
```

---

## Step 2 — Check Metrics Dashboard (30 seconds, not 30 minutes)

Go to your observability stack (Grafana, Datadog, New Relic, CloudWatch) immediately.

```
What to look at:

API Layer:
  ├── p50 / p95 / p99 response time  ← is tail latency spiking?
  ├── Request rate (RPS)             ← sudden traffic increase?
  ├── Error rate                     ← 500s alongside slowness?
  └── Active threads / thread pool   ← thread starvation?

Database:
  ├── Query execution time           ← slow queries?
  ├── Connection pool wait time      ← pool exhausted?
  ├── Active connections             ← connection leak?
  └── Lock wait / deadlocks          ← contention?

Infrastructure:
  ├── CPU usage                      ← app server or DB server?
  ├── Memory / GC pause time         ← GC causing stop-the-world?
  ├── Network latency / packet loss  ← infrastructure issue?
  └── Disk I/O                       ← DB doing full table scans?
```

---

## Step 3 — Trace a Single Slow Request (Distributed Tracing)

Use distributed tracing (Jaeger, Zipkin, AWS X-Ray) to see exactly where time is spent inside one request.

```
Trace for GET /orders/123  →  total: 2300ms

  OrderController.getOrder()         2300ms  ← total
  ├── AuthFilter.doFilter()            12ms
  ├── OrderService.getOrder()        2280ms  ← 99% of time here
  │     ├── OrderRepository.findById()  45ms
  │     ├── UserService.getUser()     2100ms  ← THIS IS THE PROBLEM
  │     │     └── HTTP call to user-service (timeout 3s, actual: 2100ms)
  │     └── mapToResponse()            135ms  ← suspicious, why so long?
  └── serialize to JSON                 8ms
```

Now you know exactly what to fix — not guessing.

### Spring Boot — Enable Tracing

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 1.0    # trace 100% of requests (reduce in prod: 0.1)
spring:
  zipkin:
    base-url: http://zipkin:9411
```

---

## Step 4 — Check Slow Query Log (Most Common Cause)

Database is the #1 culprit for slow APIs. Check slow query log first.

```sql
-- PostgreSQL: find slow queries
SELECT query,
       calls,
       mean_exec_time,
       total_exec_time,
       rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;
```

```sql
-- MySQL: enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;   -- log queries > 1 second
-- then check /var/log/mysql/slow.log
```

### Check EXPLAIN on suspect query

```sql
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'PENDING'
AND o.created_at > NOW() - INTERVAL '7 days';

-- Look for:
--   Seq Scan        ← full table scan, missing index
--   Nested Loop     ← N+1 problem
--   rows=1000000    ← estimating too many rows
--   actual time=2300 ← slow node identified
```

---

## Step 5 — Check for N+1 Query Problem

The most common "invisible" performance killer. API looks fine in dev (10 records), terrible in prod (10,000 records).

```java
// N+1 PROBLEM — 1 query for orders + 1 query PER order for user
List<Order> orders = orderRepository.findAll();    // 1 query → 1000 orders

for (Order order : orders) {
    User user = userRepository.findById(order.getUserId());  // 1000 queries!
    // ...
}
// Total: 1001 DB round trips
```

```
How to spot it:
  - Trace shows hundreds of tiny DB calls (1ms each, but 500 of them = 500ms)
  - DB query count spikes proportional to data size
  - Hibernate "show_sql" logs repeating the same query with different IDs
```

```yaml
# Enable SQL logging to spot N+1
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql: TRACE
```

```java
// FIX — JOIN FETCH in one query
@Query("SELECT o FROM Order o JOIN FETCH o.user WHERE o.status = :status")
List<Order> findWithUsers(@Param("status") String status);  // 1 query, done
```

---

## Step 6 — Check Thread Pool / Connection Pool Exhaustion

If response times are fine for first N requests then suddenly spike — pool exhaustion.

```
Symptoms:
  - Latency spikes at high traffic, fine at low traffic
  - HikariCP logs: "Connection is not available, request timed out after 30000ms"
  - Thread pool logs: "pool-1-thread-10 is blocked waiting"
  - /actuator/metrics shows hikaricp.connections.pending > 0
```

```bash
# Check HikariCP metrics
GET /actuator/metrics/hikaricp.connections.pending
GET /actuator/metrics/hikaricp.connections.active
GET /actuator/metrics/hikaricp.connections.acquire   # time waiting for connection

# Check thread pool
GET /actuator/metrics/executor.active
GET /actuator/metrics/executor.queued
```

```java
// If you find connections are exhausted — look for connection leaks
// Most common: connection not returned in exception path
public void badCode() {
    Connection conn = dataSource.getConnection();
    // exception thrown here → conn never closed → leaked
    conn.close();
}

// Fix: always try-with-resources
try (Connection conn = dataSource.getConnection()) {
    // even if exception thrown, conn.close() called automatically
}
```

---

## Step 7 — Check GC Pauses (Stop-The-World)

If slowness is intermittent and spiky (e.g. every 5–10 minutes, response time jumps to 500ms+ for a few seconds), GC is suspect.

```bash
# Enable GC logging
-Xlog:gc*:file=/var/log/app/gc.log:time,level,tags

# Or check via JVM metrics
GET /actuator/metrics/jvm.gc.pause
GET /actuator/metrics/jvm.memory.used
```

```
GC Pause pattern in traces:
  Normal request: 45ms
  GC pause happens:
    Thread frozen for 300ms (stop-the-world)
    Request appears to take 345ms — but code itself was fine

Signs:
  - Latency spikes correlate with GC metrics
  - jvm.memory.used approaches jvm.memory.max before spike
  - GC log shows Full GC events
```

```yaml
# Fix: tune GC or increase heap
# application startup flags
-Xms1g -Xmx4g
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

---

## Step 8 — Check External / Downstream Service Calls

If your API calls another service (HTTP, gRPC, Kafka consumer), slowness might not be yours.

```
Distributed trace shows:
  Your service:     15ms  ← fast
  user-service:   2100ms  ← THEM, not you

Steps:
  1. Check if downstream has its own latency spike
  2. Check if you have a timeout set (never call without timeout)
  3. Add Circuit Breaker — if downstream is slow, fail fast
  4. Cache downstream response if data doesn't change often
```

```java
// Always set timeouts on HTTP clients
RestTemplate restTemplate = new RestTemplateBuilder()
    .connectTimeout(Duration.ofSeconds(2))
    .readTimeout(Duration.ofSeconds(3))    // never wait forever
    .build();

// Or with WebClient (reactive)
WebClient.builder()
    .baseUrl("http://user-service")
    .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
    .build()
    .get()
    .uri("/users/{id}", userId)
    .retrieve()
    .bodyToMono(User.class)
    .timeout(Duration.ofSeconds(3));       // fail fast if slow
```

---

## Step 9 — Profile the Application (If Code is the Issue)

If traces show time in your own code (not DB, not downstream), profile it.

```bash
# Async profiler — attach to running JVM, no restart needed
./profiler.sh -d 30 -f flamegraph.html <PID>

# Or use JFR (Java Flight Recorder)
jcmd <PID> JFR.start duration=60s filename=profile.jfr
# Open in JDK Mission Control
```

```
Flamegraph tells you:
  - Which method consumes the most CPU time
  - Wide bars = hot code paths
  - Deep stacks = chain of calls leading to the hot method

Common findings:
  - JSON serialization of huge objects (Jackson)
  - Regex pattern matching on every request (compile once, reuse)
  - String concatenation in loops (use StringBuilder)
  - Reflection calls in tight loops
```

---

## Step 10 — Check Caching Opportunities

If the same data is fetched from DB repeatedly and it rarely changes — cache it.

```java
// Before — hits DB every request
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

// After — hits DB only on cache miss
@Cacheable(value = "products", key = "#id")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}
```

```
Check cache hit rate:
  GET /actuator/metrics/cache.gets?tag=result:hit
  GET /actuator/metrics/cache.gets?tag=result:miss

  If hit rate < 80% → either cache TTL too short or data too varied to cache
```

---

## Decision Tree — Full Debugging Flow

```
Users report slow API
        │
        ▼
Is it all endpoints or one?
  ├── One endpoint → go to distributed trace for that endpoint
  └── All endpoints → infrastructure problem (CPU, network, DB overloaded)
        │
        ▼
Check distributed trace — where is time spent?
  ├── In DB queries → check slow query log → EXPLAIN → add index / fix N+1
  ├── In downstream HTTP call → set timeout / add circuit breaker / cache
  ├── Waiting for DB connection → pool exhaustion → fix leak / increase pool
  ├── In your own code → profile with async-profiler / JFR
  └── Intermittent spikes → check GC pause metrics
        │
        ▼
Fix identified issue
        │
        ▼
Deploy → watch metrics → confirm p99 latency drops
        │
        ▼
Add alert so this is caught earlier next time
```

---

## Common Root Causes — Ranked by Frequency

```
1. Missing DB index              — full table scan, EXPLAIN shows Seq Scan
2. N+1 query problem             — 100 orders = 101 DB calls
3. No timeout on downstream call — one slow service hangs your threads
4. Connection pool exhaustion    — all connections borrowed, new requests wait
5. GC pressure                   — too many short-lived objects, frequent Full GC
6. No caching on read-heavy data — same DB query repeated on every request
7. Slow JSON serialization       — huge nested objects serialized on every call
8. Unindexed sort / ORDER BY     — DB sorts millions of rows in memory
9. Thread pool too small         — requests queue up waiting for thread
10. Memory leak → GC frequency   — heap fills up → GC runs constantly
```

---

## Key Interview Points

1. **Never guess — measure first.** Check metrics dashboard and distributed traces before touching code.

2. **Database is the #1 culprit** — slow query log + `EXPLAIN ANALYZE` should be your second stop (after metrics).

3. **N+1 is invisible in dev** — only shows up at scale. Enable `show-sql: true` in dev to catch it early.

4. **Distributed tracing shows exact hotspot** — which layer (DB, downstream, serialization, your code) consumed the time.

5. **Intermittent spikes = GC or pool exhaustion** — constant slowness = missing index, N+1, or downstream.

6. **Always set timeouts on outbound HTTP calls** — no timeout means one slow downstream holds your thread indefinitely.

7. **p99 matters more than p50** — median can look fine while 1% of users are waiting 10 seconds.

---

## One-Line Summary for Interviews

> "I'd start by checking the metrics dashboard for which endpoint is slow and when, then pull a distributed trace to pinpoint which layer (DB, downstream service, or application code) is consuming the time — the trace almost always reveals whether it's a missing index, an N+1 query, a slow downstream call, pool exhaustion, or GC pressure, and from there the fix is straightforward."
