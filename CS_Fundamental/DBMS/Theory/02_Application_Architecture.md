# DBMS Application Architecture

## Overview

DBMS applications are classified by how many **separate layers** exist between the user and the database.

| Tier | Layers | Example |
|---|---|---|
| T1 | 1 — everything on one machine | MS Access desktop app |
| T2 | 2 — client + DB server | College LAN desktop app |
| T3 | 3 — client + app server + DB server | Any modern web app |

---

## T1 — 1-Tier Architecture

### Structure

```
┌─────────────────────────┐
│   User Interface        │
│   Business Logic        │  ← Everything on one machine
│   Database              │
└─────────────────────────┘
```

- User interacts **directly** with the database
- No network, no separation — UI, logic, and data all on the same machine
- Example: developer querying a local SQLite/MySQL, MS Access file-based app

### Why T1 is NOT Good

| Problem | Explanation |
|---|---|
| No separation of concerns | UI, logic, and data are tangled — hard to maintain |
| Not scalable | Everything on one machine, can't handle multiple users |
| Security risk | Direct DB access, no access control layer |
| No centralised data | Each user has their own local DB — data is inconsistent |
| Tight coupling | Changing DB schema forces changes in UI code directly |

### When T1 is acceptable
- Single-user local tools
- Developer testing / quick prototypes

---

## T2 — 2-Tier Architecture

### Structure

```
TIER 1 (Client)                    TIER 2 (Database Server)
┌──────────────────────┐           ┌──────────────────────┐
│  UI                  │           │                      │
│  Business Logic      │  ──────→  │   Database Engine    │
│  (validation, rules) │   JDBC/   │   (just data + SQL)  │
└──────────────────────┘   ODBC    └──────────────────────┘
     User's machine                    Separate machine
                                    (MySQL/Oracle running
                                     as a service on port
                                     3306/5432)
```

### Key Point — What is the "Server" here?

- **Server in T2 = only the DB server** (a separate machine running MySQL/PostgreSQL)
- There is **no backend logic server** — business logic lives on the **client itself**
- Client = UI + business logic, both together on the user's machine
- Client connects using a connection string: `jdbc:mysql://192.168.1.10:3306/mydb`

### How It Works — Step by Step

```
1. User fills a form on the client app
         ↓
2. Client builds a SQL query
         ↓
3. Query sent to DB server over network via JDBC/ODBC
         ↓
4. DB server executes query, returns result set
         ↓
5. Client renders the result to the user
```

### Real Example

50 staff computers (clients) in a college, all connecting over LAN to 1 central MySQL server in the IT room. All 50 see the same data.

### Advantages over T1

- Centralised data — all clients share one DB
- Multi-user support — DB server handles concurrent connections
- DB is on a separate machine — slightly better security than T1

### Problems with T2

| Problem | Explanation |
|---|---|
| Fat client | Client carries both UI and business logic — heavy |
| Logic duplication | Every client must implement the same business rules |
| Hard to scale | DB server becomes bottleneck; no middle layer to distribute load |
| Tight coupling | Client directly coupled to DB schema — change a table, update every client |
| Security gap | Client holds DB credentials — if client is compromised, DB is exposed |
| No reusability | Can't share logic between web app and mobile app on the same DB |

---

## T3 — 3-Tier Architecture

### Structure

```
TIER 1 (Client)       TIER 2 (App Server)        TIER 3 (DB Server)
┌───────────────┐     ┌─────────────────────┐     ┌──────────────────┐
│               │     │                     │     │                  │
│   UI only     │ ──→ │   Business Logic    │ ──→ │  Database Engine │
│ (browser/app) │HTTP │   (Node, Django,    │JDBC │  (MySQL, Postgres)│
│               │     │    Spring Boot)     │     │                  │
└───────────────┘     └─────────────────────┘     └──────────────────┘
  User's machine         Separate server              Separate server
```

### Key Point — What changed from T2?

- Business logic is **pulled out of the client** into its own dedicated server (Tier 2)
- Client becomes **thin** — only handles UI
- Client never touches the DB directly — it calls the app server via HTTP/REST/GraphQL
- App server holds DB credentials, not the client

### How It Works — Step by Step

```
1. User clicks "Search" on browser
         ↓
2. Browser sends HTTP request to app server
         ↓
3. App server validates request, applies business rules
         ↓
4. App server queries DB using JDBC/ORM
         ↓
5. DB returns result to app server
         ↓
6. App server formats response (JSON) and sends to client
         ↓
7. Browser renders the result
```

### Advantages over T2

| Advantage | Explanation |
|---|---|
| Thin client | Client only does UI — lightweight, works on any device |
| Centralised logic | Business rules live in one place — easy to update |
| Better security | Client never has DB credentials; DB is not exposed to internet |
| Scalable | App server layer can be scaled horizontally (multiple instances) |
| Reusable | Same app server serves web, mobile, and API consumers |
| Loose coupling | Client doesn't care about DB schema — only knows the API contract |

### Problems with T3

| Problem | Explanation |
|---|---|
| More complex | Three separate components to deploy and maintain |
| Network latency | Two network hops instead of one (client→app server→DB) |
| More infrastructure | Need to manage app server + DB server separately |

### Real Example

Any modern web app:
- Browser (React/HTML) — Tier 1
- Node.js / Django / Spring Boot API — Tier 2
- MySQL / PostgreSQL / MongoDB — Tier 3

---

## Side-by-Side Comparison

| Feature | T1 | T2 | T3 |
|---|---|---|---|
| Tiers | 1 | 2 | 3 |
| UI location | Local | Client | Browser / App |
| Business logic | Local | Client | App Server |
| DB location | Local | Separate server | Separate server |
| Client type | Fat | Fat | Thin |
| Scalability | None | Low | High |
| Security | Poor | Medium | Good |
| Maintenance | Hard | Medium | Easy |
| Example | MS Access | LAN desktop app | Web / Mobile app |

---

## Interview Questions

1. **What is the difference between T2 and T3 architecture?**
   → In T2, business logic is on the client. In T3, it's on a separate application server — making the client thin and the system more secure and scalable.

2. **Where does business logic live in T2?**
   → On the client itself — there is no separate backend/app server in T2.

3. **Why is T3 more secure than T2?**
   → In T3, the client never holds DB credentials or talks directly to the DB. All DB access goes through the app server.

4. **What is a "fat client" and "thin client"?**
   → Fat client handles both UI and logic (T2). Thin client handles only UI and delegates logic to a server (T3).

5. **Why is T2 hard to scale?**
   → The DB server becomes a bottleneck with many users and there's no intermediate layer to distribute or cache load.
