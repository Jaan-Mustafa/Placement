# D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.** And abstractions should not depend on details — details depend on abstractions.

## The Idea
Don't hard-wire a high-level class to a concrete low-level class. Instead, both talk through an **interface**. This lets you swap the low-level implementation (e.g. MySQL → MongoDB, Email → SMS) without touching the high-level logic.

- **High-level module** = business logic (what the app does).
- **Low-level module** = details/implementation (how it's done — DB, email, file).
- Glue them with an **abstraction** so they're decoupled.

> Note: DIP is often achieved via **Dependency Injection** (passing the dependency in), but they're not the same thing — DIP is the principle, DI is a technique.

---

## ❌ Violation (high-level depends directly on low-level concrete)

```java
// Low-level module
class MySQLDatabase {
    public void save(String data) { /* save to MySQL */ }
}

// High-level module depends DIRECTLY on the concrete class
class UserService {
    private MySQLDatabase db = new MySQLDatabase();  // hard-wired!

    public void register(String user) {
        db.save(user);
    }
}
```

**Why it's bad:** `UserService` (high-level) is glued to `MySQLDatabase` (low-level). To switch to MongoDB you must **edit `UserService`**. Also hard to unit-test (can't swap in a fake DB).

---

## ✅ Fix (both depend on an abstraction)

```java
// Abstraction
interface Database {
    void save(String data);
}

// Low-level modules implement the abstraction
class MySQLDatabase implements Database {
    public void save(String data) { /* MySQL */ }
}

class MongoDatabase implements Database {
    public void save(String data) { /* MongoDB */ }
}

// High-level module depends on the abstraction, injected in
class UserService {
    private final Database db;

    public UserService(Database db) {   // dependency injected
        this.db = db;
    }

    public void register(String user) {
        db.save(user);
    }
}
```

Usage — swap implementations freely without changing `UserService`:
```java
UserService s1 = new UserService(new MySQLDatabase());
UserService s2 = new UserService(new MongoDatabase());   // just pass a different impl
UserService test = new UserService(new FakeDatabase());  // easy to unit test
```

**Now the dependency is "inverted":** both `UserService` and `MySQLDatabase` depend on the `Database` interface — not on each other.

---

## The "Inversion" visual
```
BEFORE:  UserService ──────► MySQLDatabase        (high depends on low, concrete)

AFTER:   UserService ──► Database ◄── MySQLDatabase
                         (interface)   MongoDatabase
         (both depend on the abstraction — direction "inverted")
```

## How to Spot DIP Violations
- `new ConcreteClass()` for a dependency **inside** a high-level class.
- Business logic importing concrete infra classes (specific DB/HTTP/email libs).
- Impossible to unit-test without the real database/service.

## Benefits
- **Swappable** implementations (DB, payment gateway, notifier).
- **Testable** — inject mocks/fakes.
- **Decoupled** — change details without touching business logic.

## One-Line Summary
**DIP: high-level business logic and low-level details should both depend on an abstraction (interface), not on each other. Inject the concrete implementation instead of `new`-ing it inside — so you can swap MySQL↔Mongo or plug in a fake for tests without editing the high-level class.**
