# Twitter Snowflake — Datacenter ID vs Machine ID

Two separate fields in a Snowflake ID exist so that **no two machines anywhere can ever generate the same ID**.

## Snowflake 64-bit ID Layout

```
 1 bit        41 bits              5 bits        5 bits       12 bits
┌─────┬──────────────────────┬────────────┬────────────┬──────────────┐
│sign │     timestamp        │ datacenter │  machine   │   sequence   │
│  0  │  (ms since epoch)    │    ID      │    ID      │    number    │
└─────┴──────────────────────┴────────────┴────────────┴──────────────┘
```

- **1 bit** — sign bit, always 0 (keeps the number positive)
- **41 bits** — timestamp (ms) → ~69 years of unique timestamps
- **5 bits — datacenter ID** → 2⁵ = **32 datacenters**
- **5 bits — machine ID** → 2⁵ = **32 machines per datacenter**
- **12 bits** — sequence number → 2¹² = **4096 IDs per machine per millisecond**

## The Actual Difference

| | Datacenter ID | Machine ID |
|---|---|---|
| **What it identifies** | *Which* data center / region the server is in | *Which* specific server (node) **inside** that data center |
| **Scope** | Across all your data centers | Local to one data center |
| **Range** | 0–31 (32 possible) | 0–31 (32 possible) |
| **Example** | `us-east = 1`, `eu-west = 2` | server #7 within us-east |

They are **two levels of a hierarchy**:

```
Datacenter ID = 1 (US-East)
        ├── Machine ID = 0   (server 0)
        ├── Machine ID = 1   (server 1)
        └── Machine ID = 7   (server 7)   ← e.g. this exact box

Datacenter ID = 2 (EU-West)
        ├── Machine ID = 0   (server 0)   ← different box, SAME machine id as above
        └── Machine ID = 1   (server 1)
```

Notice: **Machine ID 0 exists in *both* datacenters** — that's fine, because the *combination* `(datacenter ID, machine ID)` is what's unique. Datacenter 1 + Machine 0 ≠ Datacenter 2 + Machine 0.

## Why Split Into Two Fields Instead of One 10-bit "Worker ID"?

You *could* use a single 10-bit field (1024 machines). Twitter split it 5+5 for **operational clarity**:

- **Easier assignment / management** — each data center independently manages its own 32 machine IDs (0–31) without global coordination. No central registry needed to avoid cross-region collisions.
- **Reflects real topology** — maps cleanly to "region → server", how infra is actually organized.
- **Together they form the unique worker identity** — `(datacenter, machine)` guarantees two servers in different regions never clash, even if their clocks tick the same millisecond and produce the same sequence number.

## How They Guarantee Global Uniqueness

An ID is unique because the combination of all parts can't repeat:

```
same timestamp (ms) + same datacenter + same machine + same sequence → impossible to collide

- Different machine?      → machine ID differs       ✓ unique
- Different datacenter?    → datacenter ID differs    ✓ unique
- Same machine, same ms?   → sequence number differs  ✓ unique (up to 4096/ms)
- Different ms?            → timestamp differs         ✓ unique
```

So **datacenter ID + machine ID** together answer *"which exact server made this ID,"* split into two levels (region, then box within region) for decentralized, collision-free assignment.

## One-Line Summary
**Datacenter ID says *which data center* the server is in; machine ID says *which server within that data center* — together (10 bits) they uniquely identify the generating node, so two machines in different regions never produce the same ID.**
