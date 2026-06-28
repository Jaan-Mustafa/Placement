# Preventing Data Loss in a Notification System

Notifications often carry important info (OTPs, payment alerts), so losing one is a real problem.

## The Core Principle

```
Requirement: a notification must NEVER be lost.
Acceptable trade-off: it MAY occasionally be delivered more than once.
```

Notification systems prioritize **"no data loss" over "no duplicates"** — better to send twice than never. The design is built to **persist and retry**, accepting rare duplicates as the lesser evil.

## Where Data Can Be Lost & How We Prevent It

Flow — loss can happen at any hop, so protect each one:
```
Client → Notification Server → Message Queue → Worker → 3rd-party (FCM/APNs/email)
```

### 1. Persist before acknowledging (the main technique)
Save the notification to a DB the moment it's accepted, **before** doing anything else.
```
Request comes in → write to DB (status: PENDING) → THEN put on queue
```
- Server crash mid-processing → record still in DB → reprocess.
- Nothing lives "in memory only" where a crash would vaporize it.

### 2. Durable message queue
The queue (Kafka, RabbitMQ, SQS) is **persistent** — messages on disk, not just RAM.
- Worker crashes while processing → message **isn't removed until the worker acks success**.
- No ack → queue **re-delivers** to another worker.
```
Worker pulls → processes → sends to FCM → SUCCESS → ack (queue deletes it)
Worker crashes before ack → message stays → redelivered to another worker
```

### 3. Retry mechanism for third-party failures
Third parties (FCM/APNs/SendGrid) can fail/time out. On failure:
- Worker **retries** with exponential backoff.
- Still failing after N retries → move to a **Dead Letter Queue (DLQ)** for inspection/manual retry, instead of silently dropping.
```
Send fails → retry 1 → retry 2 → retry 3 → still failing → Dead Letter Queue (not lost!)
```

### 4. Track delivery status in the DB
Each notification has a status updated as it progresses:
```
PENDING → SENT → DELIVERED   (or → FAILED → retry)
```
- Background job scans for notifications **stuck in PENDING/FAILED** and reprocesses them.
- Safety net: even if the queue lost something, the DB record shows it was never delivered.

## The Duplicate Problem (the flip side)

Aggressive retries can send the same notification **twice** (e.g. delivered, but the ack was lost → retried). To minimize:

### Deduplication with an idempotency / event ID
- Each notification carries a **unique event ID**.
- Before sending, check: "have I already sent this event ID?" (cache/DB lookup).
- Yes → skip. No → send and record the ID.
```
event_id = abc123
  → already sent abc123?  → yes → skip (dedup)
                          → no  → send + record abc123
```
Best-effort (not a guarantee), but combined with at-least-once delivery gives **"never lost, rarely duplicated."**

## Putting It Together

```
Client
  │
  ▼
Notification Server ──► (1) WRITE to DB (PENDING)  ← persist before anything
  │
  ▼
(2) Durable Message Queue (Kafka)  ← on disk, redelivers if worker dies
  │
  ▼
Worker ──► (dedup by event_id) ──► call 3rd-party
  │                                   │
  │                       success ────┘──► update DB (DELIVERED) + ack queue
  │                       failure ──► (3) retry → DLQ if exhausted
  ▼
(4) Background job re-scans DB for stuck/FAILED notifications
```

## Summary Table

| Risk of loss | Where | Prevention |
|---|---|---|
| Server crash after accepting | Notification server | **Persist to DB first** (PENDING) before queueing |
| Worker crash mid-processing | Worker | **Durable queue + ack-on-success** → redelivery |
| Third-party fails/times out | FCM/APNs/email | **Retry with backoff → Dead Letter Queue** |
| Something silently dropped | Anywhere | **Status tracking in DB + background re-scan** |
| Over-correction → duplicates | Retries | **Dedup via unique event ID (idempotency)** |

## One-Line Summary
**Prevent data loss by persisting every notification to a DB before processing, using a durable message queue that re-delivers on worker failure, retrying third-party sends (with a Dead Letter Queue for stubborn ones), and tracking delivery status so stuck notifications get reprocessed — accepting that this "at-least-once" approach may rarely produce duplicates, which we reduce with event-ID deduplication.**
