# Rate Limiter Placement — Client vs API Server vs Middleware

## 1. Client Side

```
[Client App] ←— rate limiter lives here
     ↓
[Server]
```

- Client itself limits how many requests it sends
- Example: mobile app waits 1 second between API calls

**Problem: Never trust the client**
```
Normal user   → respects limit ✓
Malicious user → bypasses client code, sends 10,000 requests ✗
```
Client-side code can be modified, reverse engineered, or bypassed completely. Useless for security.

**Only use for:** UX purposes (prevent accidental double clicks, debouncing search input)

---

## 2. API Server Level

```
[Client]
   ↓
[API Server] ←— rate limiter logic lives inside server code
   ↓
[DB]
```

- Rate limiting logic written directly inside server code
- Example: inside your Express/Spring controller

```js
if (requestCount > 100) {
    return res.status(429).send("Too many requests")
}
// actual business logic below
```

**Problem:**
- Mixes rate limiting logic with business logic
- Hard to reuse across multiple services
- 10 microservices = same logic written 10 times

**Use when:** Small app, single server, quick solution needed

---

## 3. Middleware / API Gateway

```
[Client]
   ↓
[Rate Limiter Middleware] ←— sits in front of everything
   ↓
[API Server 1]
[API Server 2]
[API Server 3]
```

- Separate layer that intercepts ALL requests before they hit any server
- Examples: **Nginx, AWS API Gateway, Kong, Cloudflare**

```
Request comes in
  → middleware checks: has this IP sent > 100 req/min?
  → YES → block, return 429
  → NO  → forward to actual server
```

**Best approach because:**
- Business logic stays clean
- One place to manage limits for all services
- Blocks request before reaching server (saves resources)
- Update rules without touching server code

---

## Summary Table

| | Client Side | API Server | Middleware |
|---|---|---|---|
| Location | User's device | Inside server code | Separate layer before server |
| Security | None (bypassable) | Good | Best |
| Reusability | N/A | Per service | Covers all services |
| Code separation | N/A | Mixed with logic | Fully separate |
| Use case | UX only | Small/simple apps | Production systems |

**TL;DR:**
- **Client side** — easily bypassed, only for UX
- **API server** — works but pollutes business logic
- **Middleware** — best practice, intercepts before server, clean separation
