# Load Balancer

A load balancer can be a server (software-based), or dedicated hardware built specifically for that job.

## Two Types

### Software Load Balancer (a server)
- Runs on a normal server with specialized software (e.g., **Nginx**, **HAProxy**)
- That server's only job is to receive requests and distribute them
- Example: An EC2 instance running Nginx as a load balancer

### Hardware Load Balancer
- A **physical dedicated device** — not a general-purpose server
- Extremely fast, handles millions of connections
- Example: **F5 BIG-IP** — used by banks, large enterprises
- Expensive, less flexible

## What It Actually Does

```
User Request
     ↓
Load Balancer  ←— only job: route traffic
   /    \
Server1  Server2
```

- Receives all incoming requests
- Decides which backend server to forward to (round robin, least connections, etc.)
- Does **not** process business logic — just routes

## Analogy

Like a **receptionist** — they don't do the actual work, they just direct you to the right person. Still a person (server), just optimized for one specific task (routing).

**TL;DR:** A load balancer is a server optimized for traffic routing, not business logic. It can be software on a normal server or dedicated hardware.
