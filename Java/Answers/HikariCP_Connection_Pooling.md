# How HikariCP Connection Pooling Works

## The Problem — Why Connection Pooling Exists

Opening a database connection is expensive:

```
App → TCP handshake → DB server → authentication → session setup → ready
         ~1-3ms each step                              Total: 5-50ms
```

Without pooling, every DB operation pays this cost:

```java
// WITHOUT pooling — slow and dangerous
public User getUser(Long id) {
    Connection conn = DriverManager.getConnection(url, user, pass); // 20ms penalty
    PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id=?");
    ps.setLong(1, id);
    ResultSet rs = ps.executeQuery();
    // ...
    conn.close();  // connection destroyed — wasted all that setup time
}
```

Under 1000 req/sec, you'd be creating and destroying 1000 connections/sec — DB collapses.

**Connection pooling:** create connections once, reuse them, return them when done.

---

## What is HikariCP?

HikariCP is the **fastest, lightest JDBC connection pool** available for Java. It is the **default connection pool in Spring Boot** since version 2.0.

```
"Hikari" = light in Japanese — it is designed to be minimal overhead
```

It maintains a **pool of live, ready-to-use DB connections**. Your app borrows one, uses it, returns it — the connection is never actually closed.

---

## Big Picture — How the Pool Works

```
                    ┌─────────────────────────────┐
                    │        HikariCP Pool         │
                    │                              │
  App Thread 1 ───► │  [conn1] ← borrowed         │
  App Thread 2 ───► │  [conn2] ← borrowed         │
  App Thread 3 ───► │  [conn3] ← borrowed         │
  App Thread 4 ───► │  [conn4] ← IDLE (available) │ ◄── Thread 4 gets it instantly
  App Thread 5 ──── │  [conn5] ← IDLE (available) │
        waiting...  │                              │
                    └─────────────────────────────┘
                              │
                    All connections stay open to DB
                              │
                         [PostgreSQL]
```

---

## Core Internal Components

### 1. ConcurrentBag — The Pool Data Structure

HikariCP does NOT use a `BlockingQueue` like older pools (c3p0, DBCP).
It uses its own **ConcurrentBag** — a lock-free, thread-aware bag of connections.

```
ConcurrentBag internals:
├── sharedList     — CopyOnWriteArrayList of ALL connections
├── threadList     — ThreadLocal cache of connections last used by THIS thread
└── waiters        — count of threads waiting for a connection
```

**Thread-local borrow (the fast path):**
```
Thread asks for connection
    │
    ▼
Check ThreadLocal list first
    │
    ├── found idle connection used by this thread before?
    │       └── return it immediately — NO lock, NO contention
    │
    └── not found → scan sharedList → CAS to mark as BORROWED
                        └── still not found → wait on SynchronousQueue
```

This is why HikariCP is fast — **the common case (same thread reuses its connection) requires zero locking**.

### 2. Connection States

```java
// Each connection wrapper (PoolEntry) has one of these states
STATE_NOT_IN_USE   = 0   // idle, available to borrow
STATE_IN_USE       = 1   // borrowed by a thread
STATE_REMOVED      = -1  // evicted from pool (bad connection)
STATE_RESERVED     = -2  // reserved for maintenance
```

State transitions happen via **CAS (Compare-And-Swap)** — no locks.

```
getConnection():
    CAS(STATE_NOT_IN_USE → STATE_IN_USE)  // atomic, lock-free
    success → return connection
    fail    → try next connection in pool

close() / return to pool:
    CAS(STATE_IN_USE → STATE_NOT_IN_USE)
    notify waiting threads
```

---

## Connection Lifecycle in HikariCP

```
Pool starts up
    │
    ▼
minimumIdle connections created eagerly
    │
    ▼
App calls dataSource.getConnection()
    │
    ▼
HikariCP checks ThreadLocal → sharedList → wait
    │
    ▼
Returns ProxyConnection (wraps real Connection)
    │
    ▼
App uses connection (queries, updates)
    │
    ▼
App calls connection.close()           ← does NOT close real connection!
    │
    ▼
ProxyConnection.close() → returns to pool (state = NOT_IN_USE)
    │
    ▼
Next thread waiting gets notified → borrows it
```

### The Proxy Connection trick

```java
// What your code sees:
Connection conn = dataSource.getConnection();
conn.close();   // you think you're closing it

// What actually happens:
// conn is a HikariProxyConnection wrapping the real connection
// .close() just returns it to the pool
// real connection stays open to the database
```

---

## Configuration — Key Properties

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10        # max connections in pool (DEFAULT: 10)
      minimum-idle: 5              # min idle connections maintained
      connection-timeout: 30000    # ms to wait for connection (default: 30s)
      idle-timeout: 600000         # ms before idle connection is removed (10 min)
      max-lifetime: 1800000        # ms max connection lifetime (30 min)
      keepalive-time: 60000        # ms between keepalive pings (1 min)
      pool-name: MyHikariPool      # name shown in metrics/logs
      connection-test-query: SELECT 1  # validation query (JDBC4 drivers don't need this)
```

### How to size the pool

**Common mistake:** setting `maximum-pool-size` very high (100, 200).

HikariCP's own guidance (the "pool size" formula):

```
pool size = Tn × (Cm - 1) + 1

Tn = number of threads that can execute DB queries concurrently
Cm = max time for a single query / time for a context switch

Practical rule for most apps:
  pool size = (number of CPU cores * 2) + number of disk spindles

For a 4-core machine: ~10 connections is often optimal
```

**More connections ≠ faster.** DB has its own CPU limits. Too many connections cause context switching overhead on the DB side.

---

## What Happens When Pool is Exhausted

```
All 10 connections are borrowed
    │
Thread 11 calls dataSource.getConnection()
    │
    ▼
HikariCP waits up to connectionTimeout (30s default)
    │
    ├── A connection returned within 30s → Thread 11 gets it
    │
    └── 30s elapsed, no connection returned
            │
            ▼
    SQLTransactionRollbackException / SQLTimeoutException:
    "Connection is not available, request timed out after 30000ms"
```

This is the **connection pool exhaustion** problem — a major production issue.

**Common causes:**
- Connections borrowed but never returned (missing `close()`, exception path skips finally)
- `maximum-pool-size` too small for traffic
- Slow queries holding connections too long
- Downstream DB is slow → queries pile up → pool drains

---

## Connection Validation — Keeping Connections Fresh

DB connections can go stale (DB server restart, firewall timeout, network blip).

HikariCP validates connections:

```
1. keepalive-time (default 2 min)
   └── HikariCP sends a lightweight ping to idle connections
       to keep them alive through firewall/NAT timeouts

2. max-lifetime (default 30 min)
   └── Connections retired and replaced after 30 min
       Prevents "too many connections" at DB level from stale sessions

3. connectionTestQuery / JDBC4 isValid()
   └── Before giving a connection to your code, HikariCP
       optionally validates it with SELECT 1
       (JDBC4 drivers use isValid() which is faster — no round trip)
```

---

## HikariCP vs Other Pools

| Feature | HikariCP | c3p0 | Apache DBCP2 | Tomcat JDBC |
|---|---|---|---|---|
| Default in Spring Boot | Yes (v2+) | No | No | No |
| Lock-free borrow | Yes (CAS) | No | No | No |
| ThreadLocal fast path | Yes | No | No | No |
| Bytecode instrumentation | Yes | No | No | No |
| Lines of code | ~2500 | ~50k | ~30k | ~25k |
| Overhead per borrow | ~microseconds | ~milliseconds | ~milliseconds | ~milliseconds |

---

## Spring Boot Auto-configuration

Spring Boot auto-configures HikariCP from `application.properties`/`yml`:

```java
// What Spring Boot does internally (simplified)
@Bean
@ConditionalOnMissingBean(DataSource.class)
public DataSource dataSource(HikariConfig config) {
    return new HikariDataSource(config);
}
```

```java
// Programmatic config (when you need custom setup)
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("user");
        config.setPassword("pass");
        config.setMaximumPoolSize(10);
        config.setConnectionTimeout(30_000);
        config.setMaxLifetime(1_800_000);
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        return new HikariDataSource(config);
    }
}
```

---

## Monitoring — Metrics to Watch in Production

HikariCP exposes metrics via Micrometer (auto-wired in Spring Boot Actuator):

```
hikaricp.connections.active     — currently borrowed connections
hikaricp.connections.idle       — available connections
hikaricp.connections.pending    — threads waiting for a connection  ← alert on this
hikaricp.connections.timeout    — total timeouts (pool exhaustion)  ← alert on this
hikaricp.connections.acquire    — time taken to acquire a connection
hikaricp.connections.creation   — time to create a new connection
```

```yaml
# Enable metrics
management:
  endpoints:
    web:
      exposure:
        include: health, metrics, prometheus
```

**Alert triggers:**
- `pending > 0` for sustained period → pool too small or queries too slow
- `timeout > 0` → connections being exhausted, immediate investigation needed
- `acquire time > 100ms` → pool under heavy contention

---

## Common Production Problem — Connection Leak

```java
// LEAK — exception thrown before close()
public void badCode() throws SQLException {
    Connection conn = dataSource.getConnection();
    Statement stmt = conn.createStatement();
    stmt.executeQuery("SELECT ...");    // exception thrown here
    conn.close();                       // NEVER REACHED — connection leaked
}

// FIX — try-with-resources guarantees close()
public void goodCode() throws SQLException {
    try (Connection conn = dataSource.getConnection();
         Statement stmt = conn.createStatement()) {
        stmt.executeQuery("SELECT ...");
    }  // conn.close() called automatically — returns to pool
}
```

HikariCP leak detection:

```yaml
hikari:
  leak-detection-threshold: 2000   # log warning if connection held > 2s
```

```
WARN  HikariPool - Connection leak detection triggered for connection
      com.zaxxer.hikari.pool.ProxyConnection, stack trace follows
      at com.myapp.OrderRepository.findById(OrderRepository.java:42)
```

---

## Full Flow Summary

```
Spring Boot starts
    │
    ▼
HikariCP creates pool (minimumIdle connections opened to DB)
    │
    ▼
HTTP request arrives → Spring calls repository method
    │
    ▼
@Transactional → DataSourceTransactionManager.getConnection()
    │
    ▼
HikariCP: check ThreadLocal → CAS borrow from sharedList
    │
    ▼
ProxyConnection returned to Spring
    │
    ▼
SQL executed → result returned
    │
    ▼
@Transactional commits/rolls back
    │
    ▼
ProxyConnection.close() → connection returned to pool
    │
    ▼
Next waiting thread (if any) gets notified
```

---

## Key Interview Points

1. **HikariCP is the default pool in Spring Boot 2+** — no setup needed, auto-configured.

2. **ConcurrentBag + ThreadLocal** — the fast path avoids all locking when the same thread reuses its last connection.

3. **`connection.close()` does not close the real connection** — it returns a `ProxyConnection` to the pool.

4. **Pool exhaustion = `connectionTimeout` exceeded** — all connections borrowed, no room for new requests.

5. **Pool size sweet spot is small** — `(cores * 2) + 1`, not hundreds. More connections hurt DB performance.

6. **`max-lifetime` prevents stale connections** — forces rotation before DB-side timeouts or firewalls kill them silently.

7. **Always use try-with-resources** — guarantees connection return even on exceptions.

---

## One-Line Summary for Interviews

> "HikariCP maintains a fixed pool of live DB connections using a lock-free ConcurrentBag with a ThreadLocal fast path — your code borrows a ProxyConnection, uses it, and `close()` silently returns it to the pool rather than closing the real connection, eliminating the per-request TCP handshake and authentication cost entirely."
