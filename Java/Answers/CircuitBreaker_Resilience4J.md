# Circuit Breaker Pattern with Resilience4J

## The Problem — Cascading Failures

In microservices, services call each other. If one service goes down or slows down, callers pile up waiting — threads get exhausted — the caller goes down too — its callers go down — the whole system collapses.

```
Normal flow:
OrderService → PaymentService → BankAPI   ← all healthy

BankAPI goes down:
OrderService → PaymentService → BankAPI (timeout 30s)
                                         (timeout 30s)
                                         (timeout 30s)
                                         ...threads pile up...
PaymentService runs out of threads → PaymentService dies
OrderService runs out of threads  → OrderService dies
Frontend shows 500 errors to all users
```

**Circuit Breaker prevents this** — it detects the failure and stops calling the broken service immediately, returning a fallback response instead of waiting.

---

## The Electrical Analogy

```
Electrical circuit breaker:
  Too much current → breaker TRIPS → circuit opens → no more current flows
  Reset after a while → test if safe → close again

Software circuit breaker:
  Too many failures → breaker OPENS → no more calls to broken service
  Wait a while → test with one call → if OK, close again
```

---

## Three States of a Circuit Breaker

```
         failure threshold exceeded
CLOSED ──────────────────────────────► OPEN
  │                                      │
  │ (normal operation,                   │ (all calls fail fast,
  │  all calls go through)               │  fallback returned immediately)
  │                                      │
  │                          waitDuration passes
  │                                      │
  │        success                       ▼
  │◄────────────────────────────── HALF-OPEN
                                    │
                                    │ (limited test calls allowed)
                                    │
                                    ▼ failure
                                   OPEN (again)
```

### CLOSED (normal)
- All calls go through to the real service
- Failures are counted in a sliding window
- If failure rate exceeds threshold → trips to OPEN

### OPEN (tripped)
- All calls **immediately** return fallback — real service is NOT called
- No threads wasted waiting for a dead service
- After `waitDuration` (e.g. 60s) → transitions to HALF-OPEN

### HALF-OPEN (testing)
- Allows a limited number of test calls through
- If test calls succeed → back to CLOSED (service recovered)
- If test calls fail → back to OPEN (still broken, wait more)

---

## Resilience4J — The Library

Resilience4J is the modern replacement for Netflix Hystrix (now in maintenance mode). It provides:

```
resilience4j-circuitbreaker   — circuit breaker
resilience4j-retry            — automatic retry with backoff
resilience4j-ratelimiter      — rate limiting
resilience4j-bulkhead         — concurrency / thread isolation
resilience4j-timelimiter      — timeout
```

All are **decorators** — they wrap your existing functions without changing them.

---

## Setup

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId> <!-- required for annotations -->
</dependency>
```

---

## Configuration

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:                        # circuit breaker name
        sliding-window-type: COUNT_BASED     # COUNT_BASED or TIME_BASED
        sliding-window-size: 10              # last 10 calls measured
        failure-rate-threshold: 50           # open if 50%+ of calls fail
        slow-call-rate-threshold: 80         # also open if 80%+ calls are slow
        slow-call-duration-threshold: 2000   # "slow" = > 2 seconds
        wait-duration-in-open-state: 60s     # stay OPEN for 60s before trying HALF-OPEN
        permitted-calls-in-half-open-state: 3 # allow 3 test calls in HALF-OPEN
        minimum-number-of-calls: 5           # need at least 5 calls before evaluating
        automatic-transition-from-open-to-half-open-enabled: true
```

---

## Two Sliding Window Types

### COUNT_BASED
```
sliding-window-size: 10
→ evaluates last 10 calls

calls: [OK, OK, FAIL, FAIL, FAIL, FAIL, FAIL, OK, OK, FAIL]
         ↑ oldest                                    ↑ newest
failure count = 6/10 = 60% > threshold(50%) → OPEN
```

### TIME_BASED
```
sliding-window-size: 10
→ evaluates all calls in the last 10 seconds

Useful when traffic is low — COUNT_BASED might never trigger
if only 2 calls happen per minute even if both fail
```

---

## Usage — Annotation Style (simplest)

```java
@Service
public class PaymentService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        // calls external BankAPI — might fail or be slow
        return bankApiClient.charge(request);
    }

    // fallback — called when circuit is OPEN or real call fails
    // must have same return type + same params + Throwable as last param
    private PaymentResponse paymentFallback(PaymentRequest request, Throwable ex) {
        log.warn("Payment circuit open, using fallback. Cause: {}", ex.getMessage());
        return PaymentResponse.builder()
            .status("PENDING")
            .message("Payment queued for retry")
            .build();
    }
}
```

---

## Usage — Programmatic Style (more control)

```java
@Service
public class PaymentService {

    private final CircuitBreaker circuitBreaker;
    private final BankApiClient bankApiClient;

    public PaymentService(CircuitBreakerRegistry registry, BankApiClient bankApiClient) {
        this.circuitBreaker = registry.circuitBreaker("paymentService");
        this.bankApiClient = bankApiClient;
    }

    public PaymentResponse processPayment(PaymentRequest request) {
        // wrap the call
        Supplier<PaymentResponse> decorated = CircuitBreaker
            .decorateSupplier(circuitBreaker, () -> bankApiClient.charge(request));

        // execute with fallback
        return Try.ofSupplier(decorated)
            .recover(CallNotPermittedException.class, ex -> paymentFallback(request, ex))
            .recover(Exception.class, ex -> paymentFallback(request, ex))
            .get();
    }

    private PaymentResponse paymentFallback(PaymentRequest request, Throwable ex) {
        return PaymentResponse.pending("Payment queued");
    }
}
```

---

## Combining Circuit Breaker + Retry + Timeout

Real production setup — all three together:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 30s
        permitted-calls-in-half-open-state: 3

  retry:
    instances:
      paymentService:
        max-attempts: 3                        # retry up to 3 times
        wait-duration: 500ms                   # wait 500ms between retries
        exponential-backoff-multiplier: 2      # 500ms → 1s → 2s
        retry-exceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
        ignore-exceptions:
          - com.myapp.exception.InvalidRequestException  # don't retry business errors

  timelimiter:
    instances:
      paymentService:
        timeout-duration: 3s    # fail fast if call takes > 3s
```

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
@Retry(name = "paymentService")
@TimeLimiter(name = "paymentService")
public CompletableFuture<PaymentResponse> processPayment(PaymentRequest request) {
    return CompletableFuture.supplyAsync(() -> bankApiClient.charge(request));
}
```

**Execution order (outer to inner):**
```
TimeLimiter → Retry → CircuitBreaker → actual call

1. TimeLimiter wraps everything — if total time > 3s, fail fast
2. Retry sees failure → retries up to 3 times (with backoff)
3. CircuitBreaker counts each retry attempt as a separate call
4. If too many fail → circuit opens → Retry stops (CallNotPermittedException)
```

---

## Bulkhead — Companion Pattern

Circuit Breaker stops calls when a service is failing.
**Bulkhead** limits how many concurrent calls can run at once — prevents one service from consuming all threads.

```yaml
resilience4j:
  bulkhead:
    instances:
      paymentService:
        max-concurrent-calls: 10     # max 10 parallel calls to this service
        max-wait-duration: 100ms     # wait up to 100ms for a slot, else reject
```

```java
@Bulkhead(name = "paymentService", fallbackMethod = "paymentFallback")
@CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
public PaymentResponse processPayment(PaymentRequest request) {
    return bankApiClient.charge(request);
}
```

```
Without Bulkhead:
1000 threads all calling PaymentService → thread pool exhausted → app dead

With Bulkhead(max=10):
10 calls in progress → 11th call waits 100ms → rejects with fallback
App stays alive, only PaymentService calls are limited
```

---

## Monitoring Circuit Breaker State

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, circuitbreakers, metrics
  health:
    circuitbreakers:
      enabled: true
```

```bash
# Check circuit breaker state
GET /actuator/health

{
  "components": {
    "circuitBreakers": {
      "status": "UP",
      "details": {
        "paymentService": {
          "status": "CIRCUIT_CLOSED",
          "details": {
            "failureRate": "20.0%",
            "slowCallRate": "0.0%",
            "bufferedCalls": 10,
            "failedCalls": 2,
            "state": "CLOSED"
          }
        }
      }
    }
  }
}
```

```bash
# Metrics via Micrometer
GET /actuator/metrics/resilience4j.circuitbreaker.state
GET /actuator/metrics/resilience4j.circuitbreaker.failure.rate
GET /actuator/metrics/resilience4j.circuitbreaker.calls
```

---

## Listen to State Change Events

```java
@Component
public class CircuitBreakerEventListener {

    @EventListener
    public void onStateChange(CircuitBreakerOnStateTransitionEvent event) {
        log.warn("Circuit Breaker [{}] state changed: {} → {}",
            event.getCircuitBreakerName(),
            event.getStateTransition().getFromState(),
            event.getStateTransition().getToState()
        );
        // alert PagerDuty / Slack when circuit opens
    }
}
```

---

## Full Flow — What Happens in Production

```
Normal (CLOSED):
  OrderService → [CB: CLOSED] → PaymentService → BankAPI  ✓

BankAPI starts failing (50% failure rate):
  calls: [OK, FAIL, OK, FAIL, FAIL, FAIL] → 5/6 fail → threshold hit
  CB → OPEN

While OPEN (next 60 seconds):
  OrderService → [CB: OPEN] → CallNotPermittedException
                             → fallbackMethod() called immediately
                             → "Payment queued, will retry shortly"
  BankAPI gets ZERO calls — no thread waste, no cascading load

After 60s wait → HALF-OPEN:
  Test call 1: [CB: HALF-OPEN] → PaymentService → BankAPI  ✓
  Test call 2: [CB: HALF-OPEN] → PaymentService → BankAPI  ✓
  Test call 3: [CB: HALF-OPEN] → PaymentService → BankAPI  ✓
  All 3 succeed → CB → CLOSED (service recovered)

Normal again (CLOSED):
  OrderService → [CB: CLOSED] → PaymentService → BankAPI  ✓
```

---

## Key Differences — Hystrix vs Resilience4J

| Feature | Hystrix | Resilience4J |
|---|---|---|
| Status | Maintenance mode | Actively maintained |
| Thread isolation | Thread pool per command | Bulkhead (semaphore or thread) |
| Functional style | No | Yes (decorators, lambdas) |
| Spring Boot 3 | No | Yes |
| Metrics | Hystrix dashboard | Micrometer (Prometheus/Grafana) |
| Sliding window | Time-based | Count or Time based |

---

## Key Interview Points

1. **Three states: CLOSED → OPEN → HALF-OPEN** — CLOSED is normal, OPEN fails fast, HALF-OPEN tests recovery.

2. **Fail fast, not fail slow** — OPEN state returns fallback in microseconds instead of waiting 30s for timeout.

3. **Prevents cascading failures** — one slow downstream can't exhaust all threads and take the caller down.

4. **Fallback must be meaningful** — not just an error; return cached data, a queued-for-later response, or a degraded but usable result.

5. **Retry + CircuitBreaker order matters** — Retry is inside CircuitBreaker. Retries consume CB call budget. When CB opens, Retry stops immediately.

6. **Bulkhead is the companion** — CB protects against failure rate; Bulkhead protects against concurrency overload. Use both.

7. **Monitor state transitions** — alert when circuit opens (means a downstream is down). Never let it silently stay open.

---

## One-Line Summary for Interviews

> "A Circuit Breaker wraps downstream calls and trips OPEN after a failure rate threshold is crossed — failing fast with a fallback instead of waiting — then tests recovery in HALF-OPEN state before closing again, preventing a slow or failed downstream service from exhausting threads and cascading failures through the entire system."
