# Storage Estimation — Back of the Envelope

## Step 1 — Know Your Data Types & Sizes

| Data Type | Approximate Size |
|---|---|
| Integer / ID | 4 bytes |
| Long / Timestamp | 8 bytes |
| UUID | 16 bytes |
| Char (1 character) | 1 byte |
| Average tweet/post | 300 bytes |
| Average email | 50 KB |
| Profile photo | 200 KB |
| HD photo | 2 MB |
| 1 min video (compressed) | 6 MB |
| 1 hour video (720p) | 1 GB |

## Step 2 — Formula

```
Total Storage = (data size per entry) × (entries per day) × (retention days)
```

## Real Example — Design Twitter

**Assumptions:**
- 100M new tweets/day
- Each tweet = 300 bytes (text) + image on 10% of tweets (200KB)
- Store data for 5 years

**Text storage:**
```
100M tweets × 300 bytes = 30 GB/day
30 GB × 365 × 5 = ~54 TB for text
```

**Image storage:**
```
10% tweets have image = 10M images/day
10M × 200 KB = 2 TB/day
2 TB × 365 × 5 = ~3.5 PB for images
```

**Total ≈ 3.5 PB over 5 years**

## Real Example — Design WhatsApp

**Assumptions:**
- 50M users, each sends 20 messages/day
- Each message = 100 bytes
- 10% messages have media (500 KB avg)
- Retain for 1 year

**Message storage:**
```
50M × 20 = 1B messages/day
1B × 100 bytes = 100 GB/day
100 GB × 365 = ~36 TB/year
```

**Media storage:**
```
10% of 1B = 100M media/day
100M × 500 KB = 50 TB/day
50 TB × 365 = ~18 PB/year
```

## Step 3 — Convert Units (Must Know)

```
1 KB  = 1,000 bytes      (10^3)
1 MB  = 1,000 KB         (10^6)
1 GB  = 1,000 MB         (10^9)
1 TB  = 1,000 GB         (10^12)
1 PB  = 1,000 TB         (10^15)
```

## Step 4 — Add Replication Factor

Raw storage is never enough — always replicate data:
```
Total storage × replication factor (usually 3)

3.5 PB × 3 = 10.5 PB actual storage needed
```

## The Process (Summary)

```
1. Estimate entries per day (DAU × actions per user)
2. Estimate size per entry (text + media separately)
3. Multiply by retention period
4. Multiply by replication factor (3x)
```

**TL;DR:** entries/day × size per entry × retention days × 3 (replication). Always separate text and media — media dominates.
