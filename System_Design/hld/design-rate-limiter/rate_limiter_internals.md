# Rate Limiter Internals — Rules, Workers, Redis

## Why Rules Stored on Disk?

Rules are **configuration** — they change rarely but must persist permanently.

```
Rule example:
- /login endpoint → max 5 requests/min per IP
- /api endpoint → max 100 requests/min per user
- Premium users → max 1000 requests/min
```

- If stored only in memory → rules lost on server restart
- Disk = permanent storage → survives restarts, deployments
- Rules are written by admins/engineers → saved to config files or DB (which lives on disk)

---

## Why Workers Pull Rules from Disk → Cache?

Because **disk is slow, cache is fast.**

```
Every request checks rules:
  → hit disk every time = 10-50ms per request ✗ (too slow)
  → hit cache (Redis) = <1ms per request ✓
```

Flow:
```
Admin updates rule on disk
     ↓
Worker (background process) periodically pulls latest rules from disk
     ↓
Worker stores rules in cache (Redis/memory)
     ↓
Rate limiter reads rules from cache (fast)
```

Worker exists because:
- Rate limiter shouldn't be responsible for polling disk — **separation of concerns**
- Worker runs every few seconds/minutes, keeps cache fresh
- Rate limiter just reads from cache — stays fast and simple

---

## Why Redis for Counter? Why Not Do Everything in Rate Limiter?

### Problem — Multiple Rate Limiter Servers

```
User sends 100 requests
        ↓
    Load Balancer
   /      |      \
  RL1    RL2    RL3
```

If each RL keeps its own counter in memory:
```
RL1 sees 34 requests from User 123  → thinks: still under limit ✓
RL2 sees 33 requests from User 123  → thinks: still under limit ✓
RL3 sees 33 requests from User 123  → thinks: still under limit ✓
Total actual = 100 requests → should be blocked ✗
```
Each instance only sees its slice — counter is split across servers.

### Redis Solves This — Shared Counter

```
RL1 → Redis: increment User123 counter → 34
RL2 → Redis: increment User123 counter → 35
RL3 → Redis: increment User123 counter → 36

All rate limiters see the SAME counter ✓
```

Redis is a **centralized shared store** — all RL instances read/write the same counter atomically.

### Why Not Store Counter Inside Rate Limiter?

| | In-Memory (inside RL) | Redis |
|---|---|---|
| Single server | Works fine | Overkill |
| Multiple servers | Broken (split counters) | Works perfectly |
| Server restart | Counter resets to 0 | Counter persists |
| Atomic increment | Race conditions possible | Built-in atomic ops |

Redis also has built-in **TTL (expiry)** — counter for last 60 seconds auto-expires. You'd have to build that yourself otherwise.

---

## Full Picture

```
Admin → updates rules → Disk
              ↓
         Worker pulls rules every 30s
              ↓
         Stores rules in Redis/Cache
              ↓
User Request → Rate Limiter
                  → reads RULES from cache (fast)
                  → reads/writes COUNTER in Redis (shared, atomic)
                  → allow or block
```

**TL;DR:**
- Rules on disk = permanence
- Worker + cache = speed (avoid slow disk on every request)
- Redis for counter = shared state across multiple rate limiter instances — in-memory counter breaks with multiple servers
