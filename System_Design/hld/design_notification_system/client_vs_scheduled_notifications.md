# Client-Triggered vs Server-Side Scheduled Notifications

The difference comes down to **what causes a notification to fire** — a user/client action vs. a pre-planned time or condition on the server.

## 1. Client-Triggered Notification

Fires **as a direct result of something the user (or client app) does** — an action triggers a request to the server, which sends the notification.

```
User does something  →  client calls server  →  server sends notification
```

**Examples:**
- You **like** a photo → owner gets "X liked your post."
- You **send a message** → recipient gets a push.
- You **place an order** → "Order confirmed."
- You **comment / tag** someone → they get notified.

**Key traits:**
- **Event-driven / real-time** — immediate response to an action.
- Trigger is **unpredictable** — depends on when users act.
- Flow: client → API → notification service → push (FCM/APNs).

## 2. Server-Side Scheduled Notification

The **server decides to send at a predetermined time or condition**, with **no user action** at that moment.

```
Server has a schedule/rule  →  time/condition is met  →  server sends notification
```

**Examples:**
- **Daily reminder** at 9 AM: "Log your workout."
- **Subscription expiring** in 3 days → reminder.
- **Abandoned cart** → "You left items in your cart" 2 hours later.
- **Birthday wishes**, weekly digest emails, "bill due tomorrow."
- **Flash sale** announcement scheduled for 6 PM.

**Key traits:**
- **Time-based / batch-based** — driven by a clock or recurring job, not a live action.
- **Predictable & planned** — known in advance.
- Built with a **scheduler / cron job / delayed queue** (e.g. cron runs every minute, checks "who needs a reminder now").

## Side-by-Side

| | **Client-triggered** | **Server-side scheduled** |
|---|---|---|
| **What starts it** | A user/client action | A time, schedule, or server-evaluated condition |
| **Timing** | Immediate / real-time | Pre-planned / delayed / recurring |
| **Predictable?** | No — depends on user behavior | Yes — fired on a schedule |
| **Trigger source** | Client → server request | Server's scheduler / cron / job queue |
| **Examples** | likes, DMs, comments, order placed | daily reminders, expiry alerts, digests, abandoned cart |
| **Infra needed** | API + notification service | Scheduler/cron + job store + worker |

## How They're Built (system design view)

```
Client-triggered:
   App → API Gateway → Notification Service → message queue → push (FCM/APNs/SMS/email)

Server-scheduled:
   Cron / Scheduler → checks DB for "due" notifications → Notification Service → same push pipeline
   (often a "delayed queue" or a job running every minute scanning for things due now)
```

> Both **converge on the same delivery pipeline** (notification service → FCM/APNs/email). The only real difference is **the trigger**: a live client request vs. a scheduler firing on time/condition.

## One-Line Summary
**Client-triggered = "someone did something, notify now" (real-time, event-driven). Server-side scheduled = "it's time / a condition is met, notify" (planned, clock-/cron-driven). Same delivery pipeline, different trigger.**
