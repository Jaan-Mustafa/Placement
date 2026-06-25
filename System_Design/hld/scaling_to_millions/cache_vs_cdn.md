# Cache vs CDN

## Mental Model

```
User → CDN → Server → Cache → DB
```

- **CDN** — sits between **user and server** (user hits it directly)
- **Cache** — sits between **server and DB** (server hits it internally)

## Cache

- Stores frequently used **DB query results** in RAM
- Server checks cache before hitting DB
- Example: Redis, Memcached

```
Server: "give me user 123's profile"
  → check Redis (cache) first
  → if found (cache hit) → return instantly
  → if not found (cache miss) → hit DB → store in Redis → return
```

**What it solves:** Reduces DB load — same query doesn't hit DB every time.

## CDN (Content Delivery Network)

- Stores **static files** (images, videos, JS, CSS) on servers **geographically close to users**
- User never reaches your origin server for those files
- Example: Cloudflare, AWS CloudFront

```
User in Mumbai requests image.png
  → hits CDN server in Mumbai (not your server in US)
  → gets file instantly from nearby CDN node
```

**What it solves:** Reduces latency + reduces load on your origin server.

## Key Differences

| | Cache | CDN |
|---|---|---|
| Who hits it | Server | User directly |
| Stores | DB query results, computed data | Static files (images, JS, CSS, videos) |
| Lives | Inside your infrastructure | Geographically distributed globally |
| Purpose | Reduce DB load | Reduce latency + origin server load |
| Example | Redis, Memcached | Cloudflare, CloudFront |

**TL;DR:** CDN is user-facing, cache is server-facing.
