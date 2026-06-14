# Designing Scalable REST APIs

## What Makes a REST API "Scalable"?

A scalable API can handle **growing traffic, data volume, and team size** without degrading in performance, reliability, or maintainability. It involves decisions at three levels:

```
1. API Design       — URLs, methods, versioning, contracts
2. Performance      — caching, pagination, async, compression
3. Resilience       — rate limiting, circuit breakers, idempotency
```

---

## 1. URL Design — Resource-Oriented

REST is about **resources (nouns)**, not actions (verbs).

```
BAD  — action-based (RPC style)
POST /getUser
POST /createOrder
POST /deleteProduct

GOOD — resource-based (REST style)
GET    /users/{id}
POST   /orders
DELETE /products/{id}
```

### Hierarchy for relationships

```
GET    /users/{userId}/orders            — all orders of a user
GET    /users/{userId}/orders/{orderId}  — specific order of a user
POST   /users/{userId}/orders            — create order for a user
DELETE /users/{userId}/orders/{orderId}  — cancel specific order
```

### Keep URLs lowercase, hyphen-separated

```
/order-items      not  /orderItems  or  /order_items
/payment-methods  not  /paymentMethods
```

### Actions that don't fit CRUD — use sub-resources

```
POST /orders/{id}/cancel      — cancel an order
POST /orders/{id}/refund      — refund an order
POST /accounts/{id}/activate  — activate account
POST /emails/{id}/resend      — resend verification email
```

---

## 2. HTTP Methods — Use Correctly

| Method | Purpose | Idempotent | Safe |
|---|---|---|---|
| GET | Read resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace entire resource | Yes | No |
| PATCH | Partial update | No* | No |
| DELETE | Remove resource | Yes | No |

**Safe** = no side effects (read-only).
**Idempotent** = calling it N times = calling it once (same result).

```java
// PUT — send entire object (replace)
PUT /users/42
{ "name": "Alice", "email": "alice@x.com", "role": "ADMIN" }

// PATCH — send only changed fields
PATCH /users/42
{ "email": "newemail@x.com" }
```

---

## 3. HTTP Status Codes — Use the Right One

```
2xx — Success
  200 OK             — GET, PUT, PATCH success
  201 Created        — POST that creates a resource (+ Location header)
  204 No Content     — DELETE success, or PUT with no response body

4xx — Client error
  400 Bad Request    — invalid input, validation failed
  401 Unauthorized   — not authenticated (no/invalid token)
  403 Forbidden      — authenticated but not allowed
  404 Not Found      — resource doesn't exist
  409 Conflict       — duplicate resource, optimistic lock conflict
  422 Unprocessable  — syntactically valid but semantically wrong
  429 Too Many Req   — rate limit hit

5xx — Server error
  500 Internal Error — unexpected server failure
  502 Bad Gateway    — upstream service failed
  503 Unavailable    — service down, overloaded
  504 Gateway Timeout— upstream timed out
```

---

## 4. Request / Response Design

### Consistent error response body

```json
{
  "status": 400,
  "error": "VALIDATION_FAILED",
  "message": "Invalid request parameters",
  "errors": [
    { "field": "email", "message": "must be a valid email address" },
    { "field": "age",   "message": "must be greater than 0" }
  ],
  "traceId": "abc-123-xyz",
  "timestamp": "2025-06-10T10:30:00Z"
}
```

Always include `traceId` — correlates logs across services.

### POST response — return created resource + Location header

```
HTTP 201 Created
Location: /orders/789

{
  "id": 789,
  "status": "PENDING",
  "amount": 1500.00,
  "createdAt": "2025-06-10T10:30:00Z"
}
```

### Use envelopes only when needed

```json
// Unnecessary envelope for single resource
{ "data": { "id": 1, "name": "Alice" } }   // AVOID for simple responses

// Useful envelope for collections (metadata needed)
{
  "data": [ ... ],
  "pagination": { "page": 1, "size": 20, "total": 340 }
}
```

---

## 5. Pagination — Never Return Unbounded Collections

### Offset-based (simple, good for UI with page numbers)

```
GET /orders?page=2&size=20

Response:
{
  "data": [ ...20 orders... ],
  "pagination": {
    "page": 2,
    "size": 20,
    "totalElements": 340,
    "totalPages": 17,
    "hasNext": true
  }
}
```

**Problem at scale:** `OFFSET 10000 LIMIT 20` forces the DB to scan 10,020 rows.

### Cursor-based (scalable, good for feeds / infinite scroll)

```
GET /orders?cursor=eyJpZCI6MTAwfQ==&size=20

Response:
{
  "data": [ ...20 orders... ],
  "nextCursor": "eyJpZCI6MTIwfQ==",
  "hasNext": true
}
```

Cursor encodes the last seen position (e.g., `{"id": 100, "createdAt": "..."}`).
DB query: `WHERE id > 100 LIMIT 20` — always fast regardless of depth.

**Use cursor pagination for high-volume, append-heavy data (feeds, audit logs, events).**

---

## 6. Filtering, Sorting, Field Selection

```
GET /orders?status=PENDING&minAmount=100&maxAmount=500
GET /orders?sort=createdAt,desc&sort=amount,asc
GET /users?fields=id,name,email          // sparse fieldsets — reduce payload
GET /orders?include=items,payment        // side-loading related resources
```

```java
// Spring example — filter via request params
@GetMapping("/orders")
public Page<Order> getOrders(
    @RequestParam(required = false) String status,
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "20") int size,
    @RequestParam(defaultValue = "createdAt,desc") String[] sort
) { ... }
```

---

## 7. API Versioning — Plan From Day One

Once an API is public, **breaking changes break clients.**

### URL versioning (most common, most visible)

```
/v1/orders
/v2/orders
```

Easy to route, easy to deprecate, easy to test in browser. Preferred for public APIs.

### Header versioning

```
GET /orders
Accept: application/vnd.myapp.v2+json
```

Clean URLs but harder to test and debug.

### Query param versioning (avoid)

```
GET /orders?version=2   // messy, easy to forget
```

### Strategy in practice

```
v1 → v2 migration:
1. Release v2 alongside v1
2. Set deprecation header on v1: Deprecation: true, Sunset: 2026-01-01
3. Give clients 6–12 months to migrate
4. Shut down v1 after Sunset date
```

---

## 8. Idempotency — Safe Retries

Network failures cause clients to retry. Without idempotency, retries cause duplicates.

```
Client sends POST /orders → network drops → client retries → two orders created
```

**Fix: Idempotency Key**

```
POST /orders
Idempotency-Key: client-generated-uuid-abc123

{
  "productId": 42,
  "quantity": 2
}
```

Server logic:
```
1. Check if Idempotency-Key "abc123" was seen before
2. If yes → return the original response (no duplicate created)
3. If no  → process, store key + response, return result
```

```java
@PostMapping("/orders")
public ResponseEntity<Order> createOrder(
    @RequestHeader("Idempotency-Key") String idempotencyKey,
    @RequestBody OrderRequest request
) {
    // check Redis/DB for existing result
    Optional<Order> existing = idempotencyStore.get(idempotencyKey);
    if (existing.isPresent()) return ResponseEntity.ok(existing.get());

    Order order = orderService.create(request);
    idempotencyStore.save(idempotencyKey, order, Duration.ofHours(24));
    return ResponseEntity.status(201).body(order);
}
```

---

## 9. Caching — Avoid Redundant Computation

### HTTP Cache Headers

```
GET /products/42

Response headers:
Cache-Control: max-age=300, public    // cache for 5 minutes
ETag: "abc123"                         // version fingerprint
Last-Modified: Tue, 10 Jun 2025 10:00:00 GMT
```

Client revalidation:
```
GET /products/42
If-None-Match: "abc123"

→ 304 Not Modified  (no body sent — saves bandwidth)
→ 200 OK + new body (if resource changed)
```

### Cache-Control directives

```
public          — can be cached by browser + CDN
private         — only browser cache (user-specific data)
no-cache        — must revalidate with server before using cached copy
no-store        — never cache (sensitive data)
max-age=300     — fresh for 300 seconds
s-maxage=600    — CDN cache duration (overrides max-age for shared caches)
```

### Server-side caching (Spring)

```java
@Cacheable(value = "products", key = "#id")
public Product getProduct(Long id) {
    return productRepository.findById(id).orElseThrow();
}

@CacheEvict(value = "products", key = "#id")
public Product updateProduct(Long id, ProductRequest req) { ... }
```

---

## 10. Rate Limiting — Protect from Abuse

```
HTTP 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1718012400
```

### Common strategies

```
Fixed Window   — 100 req/minute, counter resets at :00 each minute
                 Problem: burst at :59 + :00 = 200 req in 2 seconds

Sliding Window — 100 req in any rolling 60-second window
                 Smoother, no burst at window boundary

Token Bucket   — bucket fills at N tokens/sec, each request consumes 1
                 Allows short bursts up to bucket capacity

Leaky Bucket   — requests queue and drain at fixed rate
                 No bursts, strictly smooth output
```

In production: implement at **API Gateway** level (Kong, AWS API Gateway, Nginx) — not in application code.

---

## 11. Async APIs — For Long-Running Operations

Never make clients wait >5 seconds on a synchronous call.

```
Client: POST /reports/generate   { "type": "MONTHLY" }

Server: HTTP 202 Accepted
        Location: /reports/jobs/job-456
        {
          "jobId": "job-456",
          "status": "PROCESSING",
          "estimatedTime": "30s"
        }

Client polls:
GET /reports/jobs/job-456
→ { "status": "PROCESSING", "progress": 40 }

GET /reports/jobs/job-456
→ { "status": "COMPLETED", "resultUrl": "/reports/789" }

Client fetches result:
GET /reports/789
→ full report
```

Or use **webhooks** — server calls client when done:
```
POST /reports/generate
{
  "type": "MONTHLY",
  "webhookUrl": "https://client.com/hooks/report-ready"
}
```

---

## 12. Security Basics

```
ALWAYS use HTTPS — never HTTP for APIs

Authentication:
  JWT Bearer Token    → Authorization: Bearer eyJhbGci...
  API Key             → X-API-Key: secret123
  OAuth2              → for third-party access delegation

Authorization:
  @PreAuthorize("hasRole('ADMIN')")   — Spring Security method-level
  Check resource ownership:  order.getUserId().equals(currentUserId)

Input Validation:
  Validate ALL inputs — never trust client data
  Use @Valid + Bean Validation constraints

Sensitive Data:
  Never return passwords, full card numbers, SSNs
  Mask in logs:  card ending in ****4242
```

---

## 13. Scalability Patterns

### Stateless services

```
Each request carries all the context needed (JWT token, not server session)
→ any instance can handle any request
→ horizontal scaling is trivial
```

### Database read replicas

```
Writes → Primary DB
Reads  → Read Replica(s)

GET /orders → read replica  (most traffic)
POST /orders → primary
```

### CQRS (Command Query Responsibility Segregation)

```
Write path: POST/PUT/DELETE → Command Service → Write DB (normalized)
Read path:  GET             → Query Service  → Read DB (denormalized, optimized views)
```

### Async with message queues

```
POST /orders → save to DB → publish OrderCreated event → return 201
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
             EmailService    InventoryService   AnalyticsService
```

Decouples services — order API doesn't wait for email to send.

---

## Full Design Checklist

```
API Design
  ✓ Resource-based URLs (nouns not verbs)
  ✓ Correct HTTP methods
  ✓ Correct status codes
  ✓ Versioned from day one
  ✓ Consistent error response format with traceId

Performance
  ✓ Pagination on all list endpoints (cursor for scale)
  ✓ Filtering and sorting via query params
  ✓ HTTP cache headers on read endpoints
  ✓ Async pattern for long-running operations
  ✓ Sparse fieldsets / include to reduce payload

Resilience
  ✓ Idempotency keys on POST/PUT
  ✓ Rate limiting (at gateway level)
  ✓ Circuit breaker for downstream calls
  ✓ Timeout on all outbound calls

Security
  ✓ HTTPS only
  ✓ Authentication on all endpoints
  ✓ Authorization checks (ownership + role)
  ✓ Input validation with @Valid
  ✓ No sensitive data in responses or logs
```

---

## One-Line Summary for Interviews

> "A scalable REST API uses resource-oriented URLs, correct HTTP verbs and status codes, cursor-based pagination, HTTP caching headers, idempotency keys for safe retries, async patterns for long operations, rate limiting at the gateway, and stateless JWT auth — so individual services stay simple and horizontal scaling is frictionless."
