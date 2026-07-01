# Server-Initiated Connection

A **server-initiated connection** is when the **server sends data to the client without the client asking first** — the server "pushes" updates on its own, instead of waiting for a request. Core to real-time features like chat.

## The Contrast: Who Starts the Conversation?

Normally the web is **client-initiated** (request-response):
```
CLIENT-INITIATED (normal HTTP):
   Client: "Hey server, give me X"   →
                                      ←   Server: "Here's X"
   Server NEVER speaks unless asked first.
```

For real-time features (chat messages, notifications, live scores), the **server** has new data and wants to deliver it immediately — it can't wait to be asked:
```
SERVER-INITIATED:
   (no request from client)
                                      ←   Server: "New message arrived!"
   Server pushes data on its own, whenever something happens.
```

## The Problem It Solves

How does a client get *new* data appearing on the server (e.g. a friend sends a DM)?

| Approach | How | Problem |
|---|---|---|
| **Polling** | Client asks "anything new?" every few seconds | Wasteful — most replies "nothing"; delayed |
| **Long polling** | Client asks, server holds request open until data | Better, but still client-initiated, connection churn |
| **Server-initiated push** | Server keeps a connection, sends data the instant it has it | Real-time, efficient ✅ |

## How It's Done (technologies)

True push needs a **persistent connection** kept open, because normal HTTP closes after each response.

### 1. WebSocket (the main one)
**Full-duplex, persistent** — once open, **both** client and server can send anytime, both directions.
```
Client ⟷ Server   (one long-lived connection, two-way)
Server can push: "new message", "typing...", etc. instantly.
```
Used by: chat apps (WhatsApp Web), live notifications, multiplayer games, collaborative editors.

### 2. Server-Sent Events (SSE)
**One-way** persistent connection: server → client only. Simpler when the client doesn't push back.
```
Client opens connection → Server streams updates → → → (one-way)
```
Used by: live feeds, stock tickers, notifications.

### 3. Long polling
A "fake" push — client sends a request, server **holds it open** until it has data, then responds (client immediately reconnects). Mimics server-initiated over plain HTTP.

## Why It Matters for a Chat System

A chat system is the classic use case — messages must arrive in real time without refreshing:
```
User A sends message → Chat server receives it
        │
        ▼
Server PUSHES it to User B over a WebSocket connection
        │
        ▼
User B sees it instantly — no polling, no refresh
```
- **WebSocket** is the standard for chat (two-way: send + receive, typing indicators, presence/online status).
- For **mobile/offline** delivery, the server pushes via **FCM/APNs** — the device keeps a persistent connection to Google/Apple, and the chat server pushes through them.

## One-Line Summary
**A server-initiated connection is one where the server sends data to the client on its own (push), rather than only responding to requests — enabled by persistent connections like WebSocket (two-way) or SSE (one-way), and used for real-time features like chat messages, presence, and notifications.**
