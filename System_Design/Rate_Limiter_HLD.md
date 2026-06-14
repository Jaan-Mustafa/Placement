# How to Design a Rate Limiter — HLD

## What is a Rate Limiter?

A rate limiter controls how many requests a client can make to a service in a given time window. Requests beyond the limit are rejected (HTTP 429) or queued.

```
Without rate limiter:
  Malicious user sends 1M req/sec → server crashes → everyone affected

With rate limiter:
  User allowed 100 req/min → 101st request → 429 Too Many Requests
  Server protected, legitimate users unaffected
```

**Why we need it:**
```
1. Protect against DDoS / abuse
2. Prevent resource starvation (one user shouldn't starve others)
3. Control costs (third-party API calls, DB queries)
4. Enforce business rules (free tier: 100 req/day, paid: unlimited)
5. Maintain SLA for all users under load
```

---

## Step 1 — Clarify Requirements (Always Ask First)

```
Functional:
  → Rate limit per user? Per IP? Per API key? Per endpoint?
  → What is the limit? (100 req/min, 1000 req/day, 10 req/sec)
  → Hard reject (429) or soft throttle (queue and delay)?
  → Different limits for different tiers? (free vs paid)

Non-functional:
  → How many users? (1M users, 10K req/sec globally)
  → Latency budget? (rate limiter must add < 5ms overhead)
  → Accuracy? (exact or approximate is OK?)
  → Availability? (what if rate limiter itself goes down?)
  → Distributed? (multiple data centers / regions)
```

---

## Step 2 — Where to Place the Rate Limiter

```
Option 1 — Client Side
  Pros:  no server load
  Cons:  client can bypass it, not trustworthy
  Use:   SDK-level throttling only, never for security

Option 2 — API Gateway (Recommended for most cases)
  Pros:  single enforcement point, before request hits your service
         handles auth + rate limiting together
         no code changes in downstream services
  Cons:  gateway becomes critical path, single point of failure
  Use:   Kong, AWS API Gateway, Nginx, Envoy

Option 3 — Application Middleware
  Pros:  fine-grained control, business-logic-aware limits
  Cons:  each service must implement it, duplication
  Use:   when limits depend on business logic (e.g., per subscription tier)

Option 4 — Dedicated Rate Limiter Service
  Pros:  centralized, reusable across all services
  Cons:  extra network hop, another service to maintain
  Use:   large scale, multiple services sharing one limit store

Best answer:
  API Gateway for coarse-grained limits (IP, API key, global)
  + Application-level for fine-grained business limits (per user tier)
```

---

## Step 3 — The 5 Algorithms (Know All, Recommend One)

### Algorithm 1 — Fixed Window Counter

```
Divide time into fixed windows. Count requests per window.

Window: [00:00 → 00:01] limit = 100 req/min

Request 1  at 00:00:01 → count = 1  → ALLOW
Request 50 at 00:00:30 → count = 50 → ALLOW
Request 100 at 00:00:59 → count = 100 → ALLOW
Request 101 at 00:00:59 → count = 101 → REJECT 429

Storage: key = "user123:2024061510:00", value = count
TTL = window size (1 min)
```

```
Redis implementation:
INCR user123:minute:202406151030      → atomically increment
EXPIRE user123:minute:202406151030 60 → TTL of 1 minute
```

**The fatal flaw — boundary burst:**
```
Limit = 100 req/min

00:00:59 → 100 requests  (end of window 1, all allowed)
00:01:01 → 100 requests  (start of window 2, all allowed)

In 2 seconds: 200 requests went through = 2x the rate limit
```

---

### Algorithm 2 — Sliding Window Log

```
Store timestamp of every request in a sorted set.
Count entries in last 60 seconds. Reject if over limit.

Request arrives at T=100s:
  ZREMRANGEBYSCORE key 0 (100-60) → remove entries older than 40s
  ZCARD key → count remaining
  If count < limit → ZADD key T member → ALLOW
  Else → REJECT
```

```
Redis:
ZREMRANGEBYSCORE user123:log 0 (now-60)  → evict old entries
ZCARD user123:log                         → count in window
ZADD user123:log now uuid                 → add this request
```

**Accurate, no boundary burst.**
**Problem:** Memory — each request stored. 1M users × 100 requests = 100M entries in Redis.

---

### Algorithm 3 — Sliding Window Counter (Best Balance)

```
Hybrid approach — uses counts from current + previous window.

Formula:
  rate = prev_window_count × (1 - elapsed_in_current_window / window_size)
       + current_window_count

Example (limit = 100 req/min):
  Previous window count: 80 requests
  Current window count:  30 requests
  30 seconds elapsed in current window (50% of 60s)

  rate = 80 × (1 - 0.5) + 30
       = 80 × 0.5 + 30
       = 40 + 30 = 70

  70 < 100 → ALLOW

Storage: only 2 counters per user (previous + current window)
```

**Why this works:**
```
The weighted previous window approximates the sliding window
without storing individual timestamps.
Cloudflare uses this approach.
~0.003% error rate — acceptable for rate limiting.
```

---

### Algorithm 4 — Token Bucket (Most Common in Practice)

```
Bucket holds up to N tokens.
Tokens refill at rate R per second.
Each request consumes 1 token.
No token → reject.

Bucket capacity: 10 tokens
Refill rate: 2 tokens/second

T=0:   bucket = 10, request → bucket = 9  ALLOW
T=0:   burst of 10 requests → bucket = 0  ALLOW (burst absorbed)
T=0:   11th request → bucket = -1         REJECT
T=1s:  bucket refills to 2               (2 tokens added)
T=1s:  request → bucket = 1              ALLOW
```

**Allows controlled bursting** — good for APIs where occasional bursts are OK.
Used by: **Stripe, AWS, Shopify**.

```
Redis implementation (Lua script for atomicity):

local tokens = tonumber(redis.call('GET', KEYS[1])) or ARGV[1]
local last_refill = tonumber(redis.call('GET', KEYS[2])) or ARGV[3]
local now = tonumber(ARGV[3])
local rate = tonumber(ARGV[2])
local capacity = tonumber(ARGV[1])

-- Refill tokens based on elapsed time
local elapsed = now - last_refill
local new_tokens = math.min(capacity, tokens + elapsed * rate)

if new_tokens >= 1 then
    redis.call('SET', KEYS[1], new_tokens - 1)
    redis.call('SET', KEYS[2], now)
    return 1  -- ALLOW
else
    return 0  -- REJECT
end
```

---

### Algorithm 5 — Leaky Bucket

```
Requests enter a queue (the bucket).
Queue drains at a fixed, constant rate.
If queue is full → reject.

Incoming:  ████████████ (bursty)
Queue:     [req][req][req][req] ← max size
Outgoing:  █ █ █ █ █ █ █       (smooth, fixed rate)

Guarantees smooth output — no bursts downstream.
Used for: traffic shaping, protecting downstream services.
Problem: queued requests experience latency — not great for real-time APIs.
```

---

### Algorithm Comparison

```
                Fixed    Sliding   Sliding   Token    Leaky
                Window   Log       Counter   Bucket   Bucket
─────────────────────────────────────────────────────────────
Burst allowed    No       No        No        YES      No
Boundary burst   YES      No        No        No       No
Memory use       Low      HIGH      Low       Low      Low
Accuracy         OK       Exact     ~99.99%   Exact    Exact
Implementation   Easy     Medium    Medium    Hard     Medium
Best for         Simple   Audit     Scale     APIs     Shaping

RECOMMEND: Token Bucket for APIs, Sliding Window Counter for scale
```

---

## Step 4 — High-Level Design

```
                        ┌──────────────────────────────┐
                        │         CLIENT               │
                        └──────────────┬───────────────┘
                                       │ HTTP Request
                                       ▼
                        ┌──────────────────────────────┐
                        │      API GATEWAY / LB        │
                        │  (Nginx / Kong / AWS AGW)    │
                        └──────────────┬───────────────┘
                                       │
                                       ▼
                        ┌──────────────────────────────┐
                        │     RATE LIMITER MIDDLEWARE  │
                        │                              │
                        │  1. Extract identity         │
                        │     (user ID / IP / API key) │
                        │  2. Check Redis counter      │
                        │  3. Allow or Reject          │
                        │  4. Set response headers     │
                        └──────────┬───────────────────┘
                                   │           │
                           ALLOW   │           │ REJECT
                                   ▼           ▼
                   ┌───────────────────┐  HTTP 429
                   │  YOUR SERVICE     │  Retry-After: 30
                   │  (Order, Payment) │  X-RateLimit-Limit: 100
                   └───────────────────┘  X-RateLimit-Remaining: 0
                                          X-RateLimit-Reset: 1718016000
                                   │
                         ┌─────────▼─────────┐
                         │   REDIS CLUSTER   │
                         │                   │
                         │ user123:counter=87│
                         │ user456:counter=12│
                         │ ip:1.2.3.4:cnt=99 │
                         └───────────────────┘
                                   │
                         ┌─────────▼─────────┐
                         │   CONFIG STORE    │
                         │  (Rules/Limits)   │
                         │                   │
                         │ free_tier: 100/hr │
                         │ paid_tier: 10K/hr │
                         │ /api/search: 10/s │
                         └───────────────────┘
```

---

## Step 5 — Deep Dive: Redis as the Counter Store

### Why Redis?

```
Requirements for rate limiter store:
  ✓ Sub-millisecond reads/writes       → Redis: ~0.1ms
  ✓ Atomic increment (no race)         → Redis: INCR is atomic
  ✓ Automatic expiry                   → Redis: TTL per key
  ✓ Survive high concurrency           → Redis: single-threaded, no locks
  ✓ Distributed access                 → Redis Cluster
```

### The race condition problem

```java
// WRONG — not atomic, race condition
int count = redis.get("user123:count");     // Thread A reads: 99
                                            // Thread B reads: 99
count++;                                    // Thread A: 100
redis.set("user123:count", count);          // Thread A writes: 100
                                            // Thread B: 100
                                            // Thread B writes: 100
// Both threads allowed! Count should be 101, not 100.
```

```
FIX 1 — Redis INCR (atomic):
  INCR user123:count   → always atomic, no race
  Returns new value after increment

FIX 2 — Lua script (multiple operations atomically):
  Redis executes Lua scripts atomically — no other command runs between steps
  Use for Token Bucket (check + decrement + update must be atomic)
```

### Key schema design

```
Per-user per-window:
  key: rl:{userId}:{windowTimestamp}
  value: request count
  TTL: window size

  rl:user123:202406151030    = 87   TTL=23s
  rl:user456:202406151030    = 12   TTL=23s

Per-IP:
  key: rl:ip:{hashedIP}:{windowTimestamp}
  value: count

Per-endpoint per-user:
  key: rl:{userId}:{endpoint}:{window}
  e.g.: rl:user123:/api/search:202406151030 = 5

Sliding window log:
  key: rl:log:{userId}
  type: Sorted Set
  members: request UUIDs
  scores: timestamps
```

---

## Step 6 — Distributed Rate Limiting (Multi-Region)

```
Problem:
  User sends 60 req to US server + 60 req to EU server = 120 req
  Each server sees only 60 → both allow → limit of 100 bypassed

Solutions:

Option A — Centralized Redis Cluster (single source of truth)
  All regions check same Redis cluster
  US Server ──┐
  EU Server ──┼──► Global Redis Cluster ◄── count = 120 → REJECT
  AP Server ──┘
  Problem: cross-region latency (US → EU Redis = 100ms+), defeats purpose

Option B — Sticky Routing (route user always to same region)
  Consistent hashing on user ID → always hits same region
  Each region has its own Redis
  No cross-region calls
  Problem: if region goes down, user loses their count

Option C — Local Redis + Async Sync (eventual consistency)
  Each region has local Redis counter
  Counters sync every 1 second
  Approximate but fast
  Acceptable for most rate limiting use cases
  Used by: Cloudflare, Fastly

Option D — Redis + Token Bucket with sync
  Each region gets a quota slice (region has 30% of tokens)
  Periodically rebalances with central counter
  Complex but accurate
```

---

## Step 7 — Rate Limiter Rules & Config

```
Rules should be configurable without code deploy:

{
  "rules": [
    {
      "name": "free-tier-global",
      "match": { "tier": "free" },
      "limit": 100,
      "window": "1h",
      "key": "userId"
    },
    {
      "name": "paid-tier-global",
      "match": { "tier": "paid" },
      "limit": 10000,
      "window": "1h",
      "key": "userId"
    },
    {
      "name": "search-endpoint",
      "match": { "endpoint": "/api/search" },
      "limit": 10,
      "window": "1s",
      "key": "userId"
    },
    {
      "name": "auth-endpoint",
      "match": { "endpoint": "/api/login" },
      "limit": 5,
      "window": "1m",
      "key": "ip"
    }
  ]
}
```

Rules evaluated in order. First match wins.
Store in Redis or a Config Service (Consul, etcd) with change notification.

---

## Step 8 — Response Headers (Always Include)

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 100          ← max requests in window
X-RateLimit-Remaining: 13       ← requests left in current window
X-RateLimit-Reset: 1718016000   ← Unix timestamp when window resets
Retry-After: 47                 ← seconds until they can retry (on 429)

HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1718016000
Retry-After: 47
Content-Type: application/json

{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Retry after 47 seconds.",
  "retryAfter": 47
}
```

---

## Step 9 — Edge Cases to Discuss

```
1. Rate limiter itself goes down
   → Fail open (allow all requests) or fail closed (block all)?
   → Usually fail open — better to let traffic through than block everyone
   → Alert immediately, fix fast

2. Clock skew across servers
   → Distributed systems have slightly different clocks
   → Use Redis server time (TIME command) not application server time
   → Or use NTP-synced clocks + small tolerance window

3. User distributes requests across IPs (using proxies)
   → Rate limit by user ID (auth token) not just IP
   → Both: per-IP limit + per-user limit

4. Thundering herd at window reset
   → 1000 users all hit limit at :00 and retry at :01 simultaneously
   → Jitter: add random 0-5s delay recommendation in Retry-After
   → Or use sliding window (no hard reset boundary)

5. Whitelisting
   → Internal services should bypass rate limits
   → Trusted IPs / service tokens skip the check
   → Implement as rule with limit = ∞ or bypass flag

6. Idempotency
   → Retried requests with same idempotency key shouldn't count twice
   → Check idempotency store before rate limit check
```

---

## Capacity Estimation

```
Assumptions:
  10M users, each makes avg 10 req/min peak
  = 100M req/min = ~1.67M req/sec

Rate limiter overhead per request:
  1 Redis INCR + 1 Redis EXPIRE = 2 Redis ops per request
  Redis: 100K-500K ops/sec per node
  Need: 1.67M × 2 = 3.34M Redis ops/sec
  Redis nodes needed: ~10 nodes in cluster

Storage:
  Key per user per window: user123:202406151030
  Key size: ~30 bytes, value: 8 bytes (counter)
  10M active users: 10M × 38 bytes = ~380MB
  Very small — easily fits in Redis
```

---

## Final Architecture Diagram

```
Request
  │
  ▼
[API Gateway]
  │
  ├─► Extract Key (userId from JWT / IP from header)
  │
  ├─► Fetch rule from Config Cache (in-memory, refreshed every 30s)
  │
  ├─► Call Redis:
  │     MULTI
  │     INCR rl:{key}:{window}
  │     EXPIRE rl:{key}:{window} {windowSeconds}
  │     EXEC
  │
  ├─► count > limit?
  │     │
  │     ├── YES → HTTP 429 + headers
  │     └── NO  → set X-RateLimit-* headers → forward to service
  │
  └─► [Your Microservice]
            │
            ▼
       [Response + RateLimit headers]
```

---

## How to Present This in an Interview (10-Minute Structure)

```
Min 0–1:  Requirements clarification
  "Before I design, let me clarify — are we rate limiting per user, IP,
   or API key? Hard reject or queue? Single region or global?"

Min 1–2:  High-level where to place it
  "I'd put it at the API Gateway level — single enforcement point
   before requests hit any service."

Min 2–5:  Algorithm choice + why
  "I'll use Token Bucket for the core algorithm — it handles burst
   traffic naturally which most real APIs need, and it's what Stripe
   and AWS use. Let me walk through how it works..."

Min 5–7:  Redis deep dive
  "The counters live in Redis — INCR is atomic so no race conditions.
   For Token Bucket the check+decrement needs to be in a Lua script
   to be atomic..."

Min 7–9:  Distributed problem
  "The hard part is multi-region. I'd use local Redis per region with
   async sync — approximate but the latency tradeoff is worth it..."

Min 9–10: Edge cases
  "A few things to handle: fail open if Redis goes down, clock skew,
   whitelisting internal services, thundering herd at window reset..."
```

---

## One-Line Summary for Interviews

> "A rate limiter sits at the API Gateway, uses Redis INCR for atomic
> counting with TTL-based window expiry, implements Token Bucket or
> Sliding Window Counter for the algorithm, returns 429 with Retry-After
> headers on rejection, and handles distributed scenarios with per-region
> Redis and async counter sync for low latency."
