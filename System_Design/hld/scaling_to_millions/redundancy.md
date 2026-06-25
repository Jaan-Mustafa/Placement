# Redundancy

Redundancy = **having a backup/duplicate of something so if one fails, the other takes over.**

## Simple Example

**No Redundancy:**
```
User → Server A
       Server A dies → site is DOWN ✗
```

**With Redundancy:**
```
User → Server A
       Server A dies → Server B takes over → site still UP ✓
```

Server B was sitting there as a duplicate — the moment A failed, it jumped in.

## Real Life Analogy

- Car has a **spare tyre** — if one tyre bursts, you're not stranded
- Hospitals have **backup generators** — main power fails, generator kicks in instantly
- Document saved on **Google Drive + local disk** — one fails, other has the copy

## In System Design

Don't have only one of anything critical:

- 1 server → add another server
- 1 DB → add a replica
- 1 data center → add another data center
- 1 load balancer → add another load balancer

## Build Redundancy at Every Tier

```
[ Load Balancer 1 ]  [ Load Balancer 2 ]
          ↓
[ App Server 1 ]  [ App Server 2 ]  [ App Server 3 ]
          ↓
[ Cache 1 ]  [ Cache 2 ]
          ↓
[ Master DB ]  [ Slave DB 1 ]  [ Slave DB 2 ]
          ↓
[ Data Center 1 ]  [ Data Center 2 ]
```

Chain is only as strong as its weakest link — redundancy at only some tiers is useless if one unprotected tier fails.

**TL;DR:** Redundancy = duplicate of critical components so failure of one doesn't cause downtime.
