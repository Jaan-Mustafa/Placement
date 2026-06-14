# Why Might a @Transactional Method Fail to Roll Back Changes?

## How @Transactional Works Internally (Quick Recap)

Spring wraps your bean in a **proxy**. When you call a `@Transactional` method, the call goes through the proxy — not directly to your object.

```
Your Code
    │
    ▼
Spring Proxy (AOP)
    │  begin transaction
    ▼
Your actual method runs
    │
    ├── success → proxy commits
    └── exception → proxy rolls back
```

**This proxy mechanism is the root cause of most rollback failures.** Anything that bypasses the proxy breaks the transaction.

---

## Reason 1 — Self-Invocation (Most Common)

Calling a `@Transactional` method **from within the same class** bypasses the proxy entirely. Spring never sees the call.

```java
@Service
public class OrderService {

    public void placeOrder(Order order) {
        saveOrder(order);       // direct call — bypasses proxy!
        // if saveOrder throws, NO rollback happens
    }

    @Transactional
    public void saveOrder(Order order) {
        orderRepository.save(order);
        paymentRepository.save(order.getPayment());
    }
}
```

```
What actually happens:
  External caller → Proxy → placeOrder() [no transaction started]
                              │
                              └── this.saveOrder()  ← calls real object directly
                                                       proxy is SKIPPED
                                                       @Transactional is IGNORED
```

### Fix

```java
// Option A — Move the @Transactional to the public entry point
@Service
public class OrderService {

    @Transactional                 // transaction starts here at the proxy level
    public void placeOrder(Order order) {
        orderRepository.save(order);
        paymentRepository.save(order.getPayment());
    }
}

// Option B — Inject the bean into itself (Spring 4.3+)
@Service
public class OrderService {

    @Autowired
    private OrderService self;     // injected proxy, not 'this'

    public void placeOrder(Order order) {
        self.saveOrder(order);     // goes through proxy → transaction starts
    }

    @Transactional
    public void saveOrder(Order order) { ... }
}

// Option C — Extract to a separate bean
@Service
public class OrderService {
    @Autowired private OrderPersistenceService persistenceService;

    public void placeOrder(Order order) {
        persistenceService.saveOrder(order);  // different bean → proxy works
    }
}
```

---

## Reason 2 — Wrong Exception Type (Checked Exception)

By default, Spring **only rolls back on unchecked exceptions** (`RuntimeException` and `Error`). Checked exceptions (`Exception`, `IOException`, `SQLException`) do NOT trigger rollback.

```java
@Transactional
public void processPayment(Payment payment) throws PaymentException {
    orderRepository.save(payment.getOrder());
    paymentRepository.save(payment);

    if (payment.isFailed()) {
        throw new PaymentException("Payment declined");  // CHECKED exception
        // transaction COMMITS despite the exception!
    }
}
```

```
Default rollback rules:
  RuntimeException (unchecked) → ROLLBACK  ✓
  Error                        → ROLLBACK  ✓
  Exception (checked)          → COMMIT    ✗  ← surprising!
  IOException (checked)        → COMMIT    ✗
  PaymentException extends Exception → COMMIT ✗
```

### Fix

```java
// Option A — rollbackFor explicitly
@Transactional(rollbackFor = Exception.class)       // roll back on ANY exception
public void processPayment(Payment payment) throws PaymentException { ... }

@Transactional(rollbackFor = PaymentException.class) // specific checked exception
public void processPayment(Payment payment) throws PaymentException { ... }

// Option B — make your exception unchecked
public class PaymentException extends RuntimeException {  // extends RuntimeException
    public PaymentException(String msg) { super(msg); }
}

// Option C — noRollbackFor for cases you want to commit despite exception
@Transactional(noRollbackFor = StaleDataException.class)
public void updateOrder(Order order) { ... }
```

---

## Reason 3 — Exception is Caught and Swallowed

Exception never reaches the Spring proxy — proxy thinks everything succeeded.

```java
@Transactional
public void processOrder(Order order) {
    try {
        orderRepository.save(order);
        paymentService.charge(order);    // throws RuntimeException
    } catch (Exception e) {
        log.error("Payment failed", e);  // exception swallowed here
        // Spring proxy sees: method returned normally → COMMIT
        // DB now has order saved but no payment — CORRUPTED STATE
    }
}
```

### Fix

```java
// Option A — rethrow after logging
@Transactional
public void processOrder(Order order) {
    try {
        orderRepository.save(order);
        paymentService.charge(order);
    } catch (Exception e) {
        log.error("Payment failed", e);
        throw e;                          // rethrow → proxy sees it → ROLLBACK
    }
}

// Option B — mark transaction for rollback manually
@Transactional
public void processOrder(Order order) {
    try {
        orderRepository.save(order);
        paymentService.charge(order);
    } catch (Exception e) {
        log.error("Payment failed", e);
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
        // transaction will roll back when method exits, even without rethrow
    }
}
```

---

## Reason 4 — Method is Not Public

Spring AOP proxy only intercepts `public` methods. `@Transactional` on `private`, `protected`, or package-private methods is silently ignored.

```java
@Service
public class OrderService {

    @Transactional          // IGNORED — Spring proxy can't intercept private methods
    private void saveOrder(Order order) {
        orderRepository.save(order);
        paymentRepository.save(order.getPayment());
    }
}
```

No error thrown, no warning — it just runs without a transaction.

### Fix

```java
@Transactional          // works — public method intercepted by proxy
public void saveOrder(Order order) {
    orderRepository.save(order);
    paymentRepository.save(order.getPayment());
}
```

---

## Reason 5 — Wrong Propagation Behavior

`@Transactional` propagation controls what happens when a transactional method calls another transactional method.

```java
@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    notificationService.sendEmail(order);  // has @Transactional(propagation = REQUIRES_NEW)
    // if sendEmail throws, does placeOrder roll back?
}
```

```
Propagation types and rollback behavior:

REQUIRED (default)
  → joins existing transaction
  → exception in inner method rolls back ENTIRE transaction

REQUIRES_NEW
  → suspends outer transaction, starts NEW one
  → inner transaction can commit/rollback INDEPENDENTLY
  → exception in inner does NOT automatically roll back outer

NESTED
  → savepoint within the outer transaction
  → inner rollback goes back to savepoint, outer continues

SUPPORTS
  → runs in transaction if one exists, without one if not
  → risky: no guaranteed rollback if called without a transaction
```

```java
// REQUIRES_NEW example — inner commits independently
@Service
public class AuditService {

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAudit(String event) {
        auditRepository.save(new AuditLog(event));
        // this commits even if outer transaction rolls back
        // useful for audit logs — you want the audit even if main tx fails
    }
}

@Transactional
public void placeOrder(Order order) {
    orderRepository.save(order);
    auditService.logAudit("ORDER_PLACED");  // commits in its own transaction
    throw new RuntimeException("Order failed");
    // outer rolls back — order not saved
    // BUT audit log IS saved — separate committed transaction
}
```

---

## Reason 6 — Bean Not Managed by Spring

`@Transactional` only works on Spring-managed beans. If you create an object with `new`, Spring has no proxy around it.

```java
// NOT a Spring bean — @Transactional does nothing
OrderService service = new OrderService();
service.placeOrder(order);   // no proxy, no transaction

// Fix: always inject, never new
@Autowired
private OrderService orderService;
orderService.placeOrder(order);   // goes through Spring proxy
```

---

## Reason 7 — Database / Table Doesn't Support Transactions

Some storage engines don't support transactions at all.

```sql
-- MySQL MyISAM engine — no transaction support
CREATE TABLE orders (id INT, status VARCHAR) ENGINE=MyISAM;
-- @Transactional on Java side does nothing — DB ignores BEGIN/ROLLBACK

-- Fix: use InnoDB
CREATE TABLE orders (id INT, status VARCHAR) ENGINE=InnoDB;
```

DDL statements (CREATE TABLE, ALTER TABLE, DROP TABLE) in most databases **auto-commit** and cannot be rolled back regardless of `@Transactional`.

---

## Reason 8 — Transaction Already Marked for Rollback (Unexpected)

If an inner method marks the transaction as rollback-only, the outer method can't commit even if it catches the exception.

```java
@Transactional
public void outerMethod() {
    try {
        innerMethod();        // throws, marks transaction rollback-only
    } catch (Exception e) {
        // you caught it — but transaction is already poisoned
        log.warn("Ignored error", e);
    }
    // outer tries to commit → UnexpectedRollbackException
    // "Transaction rolled back because it has been marked as rollback-only"
}

@Transactional
public void innerMethod() {
    throw new RuntimeException("fail");   // marks current transaction rollback-only
}
```

### Fix

```java
// Option A — use REQUIRES_NEW for inner so it has its own transaction
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void innerMethod() {
    throw new RuntimeException("fail");   // rolls back only inner tx
}

// Option B — don't catch if you can't fully recover
// let the exception propagate and roll back the whole thing cleanly
```

---

## Summary Table

| Cause | What Happens | Fix |
|---|---|---|
| Self-invocation | Proxy bypassed, no transaction | Move annotation to entry point or use separate bean |
| Checked exception | Spring commits by default | `rollbackFor = Exception.class` or make exception unchecked |
| Exception swallowed | Proxy sees success | Rethrow or use `setRollbackOnly()` |
| Non-public method | `@Transactional` silently ignored | Make method `public` |
| Wrong propagation | Inner tx independent of outer | Understand REQUIRED vs REQUIRES_NEW vs NESTED |
| Not a Spring bean | No proxy exists | Always inject, never `new` |
| DB doesn't support tx | DB ignores BEGIN/ROLLBACK | Use InnoDB, avoid DDL in transactions |
| Rollback-only poisoning | `UnexpectedRollbackException` | Use `REQUIRES_NEW` or don't catch inner exceptions |

---

## Quick Mental Checklist

```
@Transactional not rolling back?

1. Is the method public?                         → if not, ignored
2. Is it called from the same class?             → self-invocation, proxy bypassed
3. Is it a checked exception?                    → add rollbackFor
4. Is the exception being caught and swallowed?  → rethrow or setRollbackOnly
5. Is the object a Spring bean (@Autowired)?     → if new, no proxy
6. What propagation is set?                      → REQUIRES_NEW = independent tx
7. Does the DB/table support transactions?       → MyISAM, DDL statements
```

---

## Key Interview Points

1. **Self-invocation is the #1 trap** — `this.method()` skips the proxy. Always call transactional methods from a different bean.

2. **Only `RuntimeException` rolls back by default** — checked exceptions commit. Use `rollbackFor = Exception.class` to be safe.

3. **Catching and swallowing an exception hides it from the proxy** — the proxy sees a normal return and commits. Rethrow or call `setRollbackOnly()`.

4. **`@Transactional` on private methods is silently ignored** — no error, no warning, just no transaction.

5. **`REQUIRES_NEW` creates an independent transaction** — it commits or rolls back regardless of what the outer transaction does.

6. **`setRollbackOnly()`** is the escape hatch — lets you log and handle the error without rethrowing, but still forces rollback.

---

## One-Line Summary for Interviews

> "`@Transactional` fails to roll back most often due to self-invocation bypassing the Spring proxy, a checked exception not matching the default rollback rules, or the exception being caught before it reaches the proxy — all three silently let the transaction commit when you expect a rollback."
