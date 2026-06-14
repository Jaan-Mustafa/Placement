# Monolith vs Microservices — When Monolith is Better

## What is a Monolith?

All application code — web layer, business logic, data access — deployed as a **single unit**.

```
Monolith
┌─────────────────────────────────────────┐
│  OrderController                        │
│  PaymentService                         │
│  UserService          Single JAR /      │
│  InventoryService     Single Process /  │
│  NotificationService  Single Deploy     │
│  OrderRepository                        │
│  PaymentRepository                      │
└─────────────────────────────────────────┘
             │
             ▼
      Single Database
```

## What is a Microservice?

Each business capability is a **separate, independently deployable service** with its own DB.

```
Microservices
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Order   │  │ Payment  │  │  User    │  │Inventory │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
│          │  │          │  │          │  │          │
│  Own DB  │  │  Own DB  │  │  Own DB  │  │  Own DB  │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
     │               │            │              │
     └───────────────┴────────────┴──────────────┘
                   API Gateway / Message Bus
```

---

## The Real Cost of Microservices Nobody Talks About

Before saying "microservices are better", understand what you're signing up for:

```
Every microservice needs:
  ├── Its own CI/CD pipeline
  ├── Its own Docker image + K8s deployment
  ├── Service discovery (Consul, Eureka, K8s DNS)
  ├── Distributed tracing (Jaeger, Zipkin)
  ├── Centralized logging (ELK Stack)
  ├── API Gateway (Kong, AWS API Gateway)
  ├── Inter-service auth (mTLS, JWT)
  ├── Circuit breakers (Resilience4J)
  ├── Health checks + readiness probes
  ├── Schema registry (if using Kafka)
  └── Distributed transaction handling (Saga pattern)
```

**This infrastructure cost is FIXED — you pay it even for 2 microservices.**
A monolith pays none of it. That's a massive advantage at small scale.

---

## When Monolith is the Right Choice

### 1. Small Team (< 8 engineers)

Conway's Law: *"Organizations design systems that mirror their own communication structure."*

```
Microservices require team-per-service ownership:
  Payment team  → owns PaymentService
  Order team    → owns OrderService
  Infra team    → manages K8s, service mesh

3 engineers trying to own 8 services:
  → context switching between services constantly
  → no one knows the full system deeply
  → infra overhead consumes 40% of eng time
  → features ship slower than monolith

Monolith with 3 engineers:
  → everyone knows everything
  → one deploy pipeline
  → zero network hops between "services"
  → features ship 2–3x faster
```

**Real example — Basecamp (37signals):**
Basecamp runs on a Rails monolith serving millions of users. DHH (creator of Rails) has written extensively about how their small team (~50 engineers total) would be destroyed by microservices overhead. They run a modular monolith with clear internal boundaries — no inter-service HTTP calls, no distributed transactions, no Kubernetes. One of the most profitable SaaS companies per engineer in the world.

---

### 2. Early-Stage / Unclear Domain Boundaries

Microservices require **stable domain boundaries**. If you split wrong, you get distributed monolith — the worst of both worlds.

```
Wrong split (unclear domain):
  "Order Management Service"
  "Order Processing Service"
  "Order Fulfillment Service"
  → All call each other synchronously for every request
  → Every feature change touches 3 services
  → Deploy all 3 together anyway
  → Distributed monolith — complexity of microservices, none of the benefits
```

**Real example — Twitter (early days):**
Twitter started as a Ruby on Rails monolith called "Monorail". They understood their domain deeply before splitting. The premature split of some services caused the famous "Fail Whale" era — not because of monolith, but because of poorly split services with unclear boundaries creating cascade failures. They eventually rebuilt with well-understood service boundaries.

**Real example — Segment (2020):**
Segment publicly wrote about migrating from microservices BACK to a monolith. They had 140+ microservices. The cost:
- A simple bug fix required coordinating deploys across 4 services
- New engineers took weeks to understand data flow
- 140 CI/CD pipelines to maintain
- Integration tests were nearly impossible

They merged everything into a single "Centrifuge" monolith. Result: deployment time dropped from hours to minutes, on-call incidents reduced dramatically.

---

### 3. Strong Data Consistency Requirements

Microservices with separate DBs make ACID transactions across services **impossible**.

```
Monolith — simple:
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);         // same DB
    inventoryRepository.reduce(order);   // same DB
    paymentRepository.charge(order);     // same DB
    // all commit or all rollback — ACID guaranteed
}

Microservices — complex:
POST /orders          → OrderService DB   (committed)
POST /inventory/reduce → InventoryService DB (committed)
POST /payments/charge  → PaymentService DB
                              ↑ FAILS HERE

Now what?
- Order is saved ✓
- Inventory reduced ✓
- Payment failed ✗
- System is in inconsistent state
- Need Saga pattern to compensate (complex, eventual consistency only)
```

**Real example — Banking core systems:**
Most core banking systems (account ledgers, transaction processing) are monoliths or tightly coupled systems — not microservices. ACID guarantees across account debit/credit operations are non-negotiable. Deutsche Bank, JP Morgan's core ledger systems are large Java monoliths. The risk of an inconsistent distributed transaction in core banking is catastrophic — no Saga pattern is acceptable when money is involved.

---

### 4. Latency-Sensitive Applications

Every microservice call is a network hop. Latency adds up.

```
Monolith — method call:
  orderService.placeOrder()
    → paymentService.processPayment()    // in-process method call: ~0.001ms
    → inventoryService.reserve()         // in-process method call: ~0.001ms
    → notificationService.send()         // in-process method call: ~0.001ms
  Total overhead: ~0.003ms

Microservices — network call:
  OrderService → PaymentService          // HTTP/gRPC: ~2–10ms
              → InventoryService         // HTTP/gRPC: ~2–10ms
              → NotificationService      // HTTP/gRPC: ~2–10ms
  Total overhead: ~6–30ms (before your actual business logic)
  
  With serialization + deserialization + auth + tracing headers:
  Each hop: 5–15ms
  6 hops in a request chain: 30–90ms of pure overhead
```

**Real example — High-Frequency Trading systems:**
HFT systems (Jane Street, Citadel, Virtu) run monoliths or in-process components in C++/Java. A microsecond matters. Any network boundary is unacceptable. Their "services" are in-process modules — same JVM, direct method calls.

**Real example — Stack Overflow:**
Stack Overflow serves ~1.5 billion page views/month with 9 web servers running a monolith (.NET). They intentionally kept it as a monolith for performance reasons. Their CTO Marco Cecconi wrote: "The monolith means we have a single context. No network overhead between parts of our application." They handle massive traffic with a tiny team — impossible with microservices overhead on their team size.

---

### 5. Simple CRUD / Internal Tools

Not every application has complex scaling needs.

```
Example: Internal HR system
  - 500 employees use it
  - CRUD operations on employee data
  - Monthly payroll reports
  - Leave management

Microservices for this:
  EmployeeService + LeaveService + PayrollService + NotificationService
  + API Gateway + Service Discovery + Distributed Tracing
  = 6 months to build infrastructure before writing any business logic

Monolith for this:
  Spring Boot app + PostgreSQL
  = 2 weeks to build the whole thing
  = works perfectly for the next 10 years
```

---

## When Microservices ARE the Right Choice

To be balanced — microservices genuinely win in these scenarios:

```
1. Independent scaling needs
   → Checkout service gets 100x Black Friday traffic
   → User profile service stays flat
   → Scale ONLY checkout, not everything

2. Team autonomy at scale
   → Amazon: "Two-pizza teams" — small teams own services end-to-end
   → 200 teams deploying independently without coordination
   → Each team chooses their own tech stack

3. Different SLAs per service
   → Real-time payment processing: 99.999% uptime
   → Recommendation engine: 99.5% uptime OK
   → Different reliability requirements = different service tiers

4. Polyglot persistence / tech
   → Search: Elasticsearch
   → Sessions: Redis
   → Orders: PostgreSQL
   → Analytics: Cassandra
   → Impossible in one monolith codebase cleanly

5. Fault isolation
   → Recommendation engine crashes
   → Rest of Amazon still works
   → Monolith: one bug in recommendations brings down checkout
```

---

## The Companies That Went Microservices and Regretted It

### Amazon — Lessons Learned
Amazon is famous for microservices (Service Oriented Architecture from 2002). But Werner Vogels himself said:

> *"We didn't start with microservices because it was fashionable. We started because we had 10,000 engineers and needed independent deployment. If you have 10 engineers, you don't have that problem."*

The lesson: **Amazon needed microservices because of team scale, not traffic scale.** Most companies have the traffic but not the 10,000 engineers.

### Netflix — Microservices Done Right (but at huge cost)
Netflix runs 700+ microservices. But:
- 2,000+ engineers
- Dedicated teams for each service
- Built their own tooling (Hystrix, Eureka, Zuul, Ribbon — all open-sourced)
- Spent years building the infrastructure before reaping benefits
- Still has incidents caused by microservice complexity regularly (dependency hell, version mismatches)

### Uber — Migration Pain
Uber migrated from monolith to microservices to handle growth. By 2018 they had 2,200+ microservices. The result:
- Engineers needed to understand 100+ services to debug one issue
- "Dependency hell" — service A depends on B which depends on C which depends on A
- Built their own internal service mesh (Uber Ringpop, YARPC) to manage it
- Admitted in engineering blogs: "We went too far. We should have had fewer, larger services."

They introduced **"macro-services"** — pulling several related microservices back together. A reversal toward modular monolith thinking.

---

## The Modular Monolith — Best of Both Worlds

The real answer many mature companies land on: **Modular Monolith** — strict module boundaries inside one deployable unit.

```
Modular Monolith
┌──────────────────────────────────────────────────────┐
│                                                      │
│  ┌─────────────┐   ┌─────────────┐   ┌───────────┐  │
│  │   Order     │   │   Payment   │   │  User     │  │
│  │   Module    │   │   Module    │   │  Module   │  │
│  │             │   │             │   │           │  │
│  │ - Own pkg   │   │ - Own pkg   │   │ - Own pkg │  │
│  │ - Own DB    │   │ - Own DB    │   │ - Own DB  │  │
│  │   schema    │   │   schema    │   │   schema  │  │
│  │ - Public    │   │ - Public    │   │ - Public  │  │
│  │   API only  │   │   API only  │   │   API     │  │
│  └──────┬──────┘   └──────┬──────┘   └─────┬─────┘  │
│         │                 │                │         │
│         └─────────────────┴────────────────┘         │
│                    Internal calls                    │
│                  (direct method, not HTTP)           │
│                                                      │
└──────────────────────────────────────────────────────┘
              │
        Single Deploy
        Single DB (separate schemas per module)
```

```java
// Module boundary enforced — Payment module exposes only interface
public interface PaymentModule {
    PaymentResult charge(PaymentRequest request);
    void refund(String transactionId);
}

// Order module depends on the interface, not the implementation
@Service
public class OrderService {
    private final PaymentModule paymentModule;  // injected — clean boundary
}

// No direct class-to-class calls across module boundaries
// Cross-module = through published interface only
// When you're ready to split → interface already exists → trivial to extract
```

**Real example — Shopify:**
Shopify runs one of the largest Rails monoliths in the world, serving millions of merchants. They built a modular monolith called "Component Based Rails Applications" (CBRA). Clear module boundaries, separate DB schemas per component, but single deployment. 3,000+ engineers working on one codebase. They explicitly chose this over microservices because:
- Deploy coordination across services was too costly
- Rollback is trivial (one deploy unit)
- Database transactions work across components

---

## Technical Decision Framework

```
Should you use Microservices?

Team size < 8?                        → Monolith
Domain boundaries unclear?            → Monolith (understand domain first)
ACID transactions required?           → Monolith (or very careful Saga)
Early stage / MVP?                    → Monolith
Latency < 10ms required?             → Monolith (eliminate network hops)
Simple CRUD / internal tool?          → Monolith

Team size > 50 per service area?      → Microservices
Independent scaling requirements?     → Microservices
Different release cadences per team?  → Microservices
Fault isolation critical?             → Microservices
Polyglot tech requirements?           → Microservices

In between?                           → Modular Monolith
                                        (modular now, extract later when needed)
```

---

## The Rule Amazon Lives By

Jeff Bezos issued the "API Mandate" in 2002:

> *"All teams will henceforth expose their data and functionality through service interfaces. Teams must communicate with each other through these interfaces. There will be no other form of interprocess communication."*

But this was for a company with **thousands of engineers** where teams literally could not coordinate. For a 10-person startup, that mandate would kill you.

**The key insight:** Amazon didn't split services for performance — they split for **organizational independence**. If you don't have that organizational scaling problem, you don't need the solution.

---

## Key Interview Points

1. **Microservices solve an organizational problem, not just a technical one** — designed for large teams that need independent deployment, not for small teams chasing "best practices."

2. **Segment went from 140 microservices back to a monolith** — a real example of microservice complexity exceeding its value.

3. **Distributed transactions are fundamentally harder** — ACID is free in a monolith, costs significant complexity (Saga pattern, eventual consistency, compensating transactions) in microservices.

4. **Each network hop adds 5–15ms** — a 6-service call chain adds 30–90ms of pure overhead before any business logic runs.

5. **Modular monolith is the pragmatic middle ground** — enforced module boundaries, separate schemas, single deployment. Shopify does this at massive scale.

6. **"Start with a monolith, extract services when pain is proven"** — Martin Fowler's actual recommendation.

7. **Conway's Law determines the right architecture** — your architecture will mirror your org structure whether you plan it or not.

---

## One-Line Summary for Interviews

> "Microservices solve organizational scaling problems — independent deployment for large teams — not performance problems; a monolith is strictly better when teams are small, domain boundaries are unclear, ACID transactions are required, or latency is critical, which is why companies like Shopify, Stack Overflow, and Basecamp run monoliths at massive scale and why Segment migrated 140 microservices back into one."
