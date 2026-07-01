# News Feed System: Post Storage + Push vs Pull (Fanout)

## Part 1: How a Post Is Stored

When you publish a post, the data is split across stores, each suited to a job.

### 1. Post content → Database
The actual post (text, author, timestamp, media links) goes into a DB table.
```
posts table:
post_id | user_id | content        | media_url        | created_at
9981    | 42      | "Hello world"  | s3://.../img.jpg | 2026-06-29 10:00
```
- Relational DB or NoSQL — often a wide-column store (Cassandra) for scale, sharded by post_id or user_id.
- **Media (images/videos)** is NOT in the DB → goes to **object storage (S3/blob)** + served via **CDN**; DB keeps only the URL.

### 2. The "feed" itself → Cache (the key idea)
Your feed is NOT rebuilt from the DB every time. Each user has a precomputed **feed list** in a fast cache (Redis):
```
Redis — feed cache (per user):
user:42:feed → [post_9981, post_9975, post_9960, ...]   (post IDs, newest first)
```
- Stores **only post IDs**, not full content (tiny + fast).
- Render: read ID list from cache → fetch post details (also cached) → assemble.

### 3. Social graph → who follows whom
A separate store (graph DB or table) keeps follower/following relationships, needed to know whose posts go into whose feed.
```
follows table:  follower_id | followee_id
```

So storage = **Post DB** (content) + **Object store/CDN** (media) + **Feed cache** (per-user list of post IDs) + **Graph** (relationships).

---

## Part 2: Push vs Pull (the Fanout Models)

Big question: when user A posts, how does it reach all their followers' feeds? This is **fanout**. Two approaches.

### Push model (Fanout-on-Write)
When you post, **immediately push** that post ID into **every follower's feed cache**.
```
User A posts (500 followers)
   │  at WRITE time:
   ▼
Insert post_id into all 500 followers' feed lists in Redis
Later, follower opens app → feed ALREADY built → just read it (super fast)
```

| Pros | Cons |
|---|---|
| **Reads instant** — feed precomputed | **Writes expensive** — one post = many writes |
| Great real-time feel | **Celebrity problem** — 50M followers = 50M writes/post |
| Good for users with few followers | Wastes work on inactive followers |

### Pull model (Fanout-on-Read)
Do nothing special at post time. When a follower **opens their feed**, **pull** latest posts from everyone they follow and assemble **on the fly**.
```
User A posts → just save it once (no fanout)
Later, follower opens app → at READ time:
   get everyone they follow → fetch recent posts → merge + sort by time → show
```

| Pros | Cons |
|---|---|
| **Writes cheap** — store post once | **Reads slow** — assemble feed every time |
| No celebrity problem | Heavy DB load on every feed open |
| No wasted work for inactive users | Higher read latency |

### Trade-off in one line
```
PUSH = work at write time  → fast reads, expensive writes (bad for celebrities)
PULL = work at read time   → cheap writes, slow reads (bad for active users)
```

---

## Part 3: The Hybrid Model (what real systems use)

Neither pure model wins, so Instagram/Twitter **combine both**:
```
NORMAL users (few followers):
   → PUSH (fanout-on-write) → followers get fast precomputed feeds

CELEBRITIES (millions of followers):
   → PULL (fanout-on-read) → don't fan out to millions; followers pull the
     celebrity's posts at read time and merge into their pushed feed
```

So a follower's feed = **(precomputed pushed feed)** + **(pulled-in posts from the few celebrities they follow)**, merged at read time. Avoids the celebrity write-explosion while keeping normal feeds fast.

---

## Summary

| | Push (write) | Pull (read) | Hybrid |
|---|---|---|---|
| When work happens | At post time | At feed-open time | Both |
| Read speed | Fast ✅ | Slow ❌ | Fast ✅ |
| Write cost | High ❌ | Low ✅ | Balanced |
| Celebrity-safe? | No ❌ | Yes ✅ | Yes ✅ |
| Used for | Normal users | — | Real systems |

**One-line:** A post's content lives in a DB (media in S3/CDN), while each user's feed is a precomputed list of post IDs in a cache. **Push** fans out a new post into followers' feeds at write time (fast reads, bad for celebrities); **Pull** assembles the feed at read time (cheap writes, slow reads); real systems use a **hybrid** — push for normal users, pull for celebrities.
