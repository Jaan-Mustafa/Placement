# Message Flow: 1-to-1 vs Group

Fundamental difference: **1-to-1 has exactly one recipient, while group chat must fan out one message to N recipients.**

## Common Building Blocks (both flows share these)

```
- Chat Servers       → hold the live WebSocket connections to online users
- Message Queue      → decouples sending from delivery
- Message Store (DB) → persists every message (Cassandra — high write volume)
- Presence Service   → knows who's online and which chat server they're on
- Push (FCM/APNs)    → delivers to offline users' devices
```

Each online user holds a **WebSocket** to a chat server (a server-initiated pipe). The whole job is routing a message to the right pipe(s).

---

## 1-to-1 Message Flow

User A sends to User B:
```
1. User A → sends message over its WebSocket to Chat Server 1
2. Chat Server 1 → generates message_id, puts message on the queue
3. Message PERSISTED to DB (survives; B can fetch later)
4. Look up B in Presence Service:
       ├─ B ONLINE  → find B's chat server (Server 2) → forward → Server 2 pushes
       │              down B's WebSocket → B sees it ✅
       └─ B OFFLINE → push notification via APNs/FCM; message waits in DB,
                      B pulls it on next login
```

One message → one recipient → one lookup → one delivery (or one push).
```
A ──► Chat Server 1 ──► Queue ──► DB
                          │
                          ▼ (presence lookup for B)
                   Chat Server 2 ──► WebSocket ──► B
```

---

## Group Message Flow

User A sends to a group of e.g. 50 members. Big difference is **fanout** — one message must reach everyone.
```
1. User A → sends message to Chat Server 1
2. Chat Server 1 → persists message + looks up the group's member list
       group_id → [member1, ... member50]
3. FANOUT: for EACH member (except A):
       - put a copy into that member's message queue
       - check presence:
            ├─ online  → route to their chat server → push down WebSocket
            └─ offline → push notification (FCM/APNs), store for later pull
```

One send becomes N deliveries:
```
                       ┌──► member1's queue ──► (online)  WebSocket ──► member1
A ──► Chat Server 1 ──►├──► member2's queue ──► (offline) push notif ──► member2
                       ├──► member3's queue ──► ...
                       └──► member50's queue ──► ...
```

### Message queue per recipient (inbox model)
Common design: **each user has their own message queue/inbox.** A group message is just written into every member's inbox:
```
Group [B, C, D], A sends "hi" →  copy into B's inbox
                                 copy into C's inbox
                                 copy into D's inbox
Each user reads from their own inbox.
```
This is **fanout-on-write** (same idea as the news feed push model) — do the copying at send time so each user's read is simple.

---

## Key Differences

| | 1-to-1 | Group |
|---|---|---|
| **Recipients** | Exactly 1 | N members |
| **Core operation** | Direct route to one user | **Fanout** — copy to all members |
| **Lookups** | One presence lookup | Member-list lookup + N presence lookups |
| **Deliveries per message** | 1 | N (one per member) |
| **Cost** | Cheap | Scales with group size |

## The Group-Size Problem (interview gotcha)

Fanout-on-write is great for **small groups** (hundreds). For **huge groups** (100k members), copying one message into 100k inboxes per message is expensive — same **celebrity problem** as the news feed.
```
Small group  → fanout-on-write (copy to everyone) ✅ simple, fast reads
Huge group   → too many copies per message ❌
            → fanout-on-read: store ONE copy, members PULL it when they open the chat
```
Large-scale chat uses a **hybrid**: push (copy) for normal groups, pull for very large ones. WhatsApp limits group size (~1024) partly to keep fanout-on-write feasible.

## One-Line Summary
**1-to-1 routes a single message to one recipient via a presence lookup (deliver if online, push if offline). Group messaging does fanout — looks up the member list and copies the message to each member's queue/inbox, delivering N times. Small groups use fanout-on-write (copy to all); very large groups switch to fanout-on-read (one copy, members pull) to avoid the copy explosion.**
