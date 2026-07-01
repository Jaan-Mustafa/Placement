# Group Fanout — Worked Example (per-user inbox, scaling to 500)

Concrete walk-through of how a group message reaches every member when users are spread across different chat servers (and some are offline).

## Setup: a group of 5 (then scale to 500)

```
Group "TeamChat" members: A, B, C, D, E

Where is everyone connected right now?
   A → Chat Server 1   (A is the sender)
   B → Chat Server 1   (same server as A, by chance)
   C → Chat Server 2
   D → Chat Server 3
   E → OFFLINE
```
Members are scattered across **different chat servers**, and one is offline — the system handles all of it.

## Step-by-step: A sends "Hello team"

### Step 1 — A sends to its chat server
```
A → (over A's WebSocket) → Chat Server 1: "send 'Hello team' to group TeamChat"
```

### Step 2 — Look up the group members
```
group_id "TeamChat" → members = [A, B, C, D, E]
```

### Step 3 — Persist the message once
```
Save to DB:  message_id=777, group=TeamChat, from=A, text="Hello team", time=...
```

### Step 4 — FANOUT: copy into each member's inbox
**Each user has their own message inbox/queue** (a personal mailbox keyed by user_id):
```
inbox:B ← message 777
inbox:C ← message 777
inbox:D ← message 777
inbox:E ← message 777
(A doesn't need it — A sent it)
```
So: each user DOES have their own queue/inbox. A 5-member group message = copied into 4 inboxes.

### Step 5 — Deliver from each inbox to the right place
For each member, check presence and deliver:
```
B: online on Server 1 → Server 1 pushes msg 777 down B's WebSocket  ✅
C: online on Server 2 → route to Server 2 → push down C's WebSocket ✅
D: online on Server 3 → route to Server 3 → push down D's WebSocket ✅
E: OFFLINE → push notification (FCM/APNs); msg stays in inbox:E, E pulls on login ✅
```

## Full picture
```
                                    ┌─ inbox:B ─► Server 1 ─► WebSocket ─► B
A ─► Chat Server 1 ─► [fanout] ─────┼─ inbox:C ─► Server 2 ─► WebSocket ─► C
       │ (lookup members,           ├─ inbox:D ─► Server 3 ─► WebSocket ─► D
       │  persist to DB,            └─ inbox:E ─► (offline) ─► push notif + wait
       │  copy to each inbox)
       ▼
      DB (one stored copy: message 777)
```

## Clearing up the common confusion

It is NOT "one queue that B's server grabs from." It is:
```
WRONG:  one queue → B's server extracts
RIGHT:  copy to inbox:B, inbox:C, ... → each routed to that user's CURRENT
        server via a presence lookup → that server pushes down their WebSocket
```
- Message is **copied into each recipient's personal inbox**.
- For **each inbox**, find which server that user is on (presence lookup) and route the copy there.
- That server pushes it down the user's WebSocket.

## Scale to 500 members

Same flow, 499 copies instead of 4:
```
A sends 1 message → fanout copies into 499 inboxes (everyone except A)
                  → for each: presence lookup → route to that user's server → push
                  → offline members get push notifications + message waits in inbox
```
```
A ─► Server 1 ─► fanout ─► inbox:user2   ─► (their server) ─► push
                           inbox:user3   ─► (their server) ─► push
                           ...
                           inbox:user500 ─► (their server) ─► push
```
500 members spread across maybe ~50 chat servers → fanout routes each copy to the correct server. **One send = 499 deliveries** → why group chat costs more than 1-to-1.

## Is it really 499 physical copies? (optimization)

```
Small/medium group (500)  → fanout-on-write: copy to each inbox ✅ simple, fast reads
Huge group (100k)         → fanout-on-read: store ONE copy, members PULL on open
                            (avoids 100k copies per message)
```
500 is comfortably in the "copy to each inbox" range — manageable.

## One-Line Answer
**Yes — each user has their own inbox/queue. A group message is copied into every member's inbox (499 copies for 500 members); then for each inbox the system does a presence lookup to find which chat server that specific user is currently on, routes the copy there, and that server pushes it down the user's WebSocket — offline users get a push notification and pull it later.**
