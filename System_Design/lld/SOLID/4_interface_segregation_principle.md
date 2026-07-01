# I — Interface Segregation Principle (ISP)

> **Clients should not be forced to depend on methods they do not use.** Prefer many small, specific interfaces over one large, general-purpose ("fat") interface.

## The Idea
A big interface forces implementers to provide methods that don't apply to them — often by throwing exceptions or leaving them empty. Split the fat interface into focused ones, so each class implements only what it actually needs.

---

## ❌ Violation (one fat interface)

```java
interface Worker {
    void work();
    void eat();
}

// Human worker: fine
class HumanWorker implements Worker {
    public void work() { /* works */ }
    public void eat()  { /* lunch break */ }
}

// Robot worker: robots don't eat!
class RobotWorker implements Worker {
    public void work() { /* works */ }
    public void eat()  {
        throw new UnsupportedOperationException(); // forced to implement junk
    }
}
```

**Why it's bad:** `RobotWorker` is **forced to depend on `eat()`** which makes no sense for it. It ends up throwing an exception (which also violates LSP). The interface is too fat.

---

## ✅ Fix (split into focused interfaces)

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

// Human needs both
class HumanWorker implements Workable, Eatable {
    public void work() { /* works */ }
    public void eat()  { /* lunch break */ }
}

// Robot needs only Workable
class RobotWorker implements Workable {
    public void work() { /* works */ }
    // no eat() to implement — clean!
}
```

**Now each class implements only what applies to it.** Robots aren't dragged into `eat()`. Adding a new capability = a new small interface, composed as needed.

---

## Real-world flavor
A classic case: a fat `Machine` interface with `print()`, `scan()`, `fax()`. A simple printer that can only print shouldn't be forced to implement `scan()` and `fax()`. Split into `Printer`, `Scanner`, `Fax` interfaces, and a multi-function device implements all three.

---

## How to Spot ISP Violations
- Implementations with **empty method bodies** or `throw new UnsupportedOperationException()`.
- An interface where different implementers use **different subsets** of the methods.
- Interfaces named vaguely and broadly ("Manager", "Service") with 10+ unrelated methods.

## ISP vs SRP (don't confuse)
- **SRP** is about **classes** having one responsibility.
- **ISP** is about **interfaces** being small/focused so *clients* aren't forced into unused methods.

## Benefits
- Implementers only depend on what they use → less coupling.
- Changing one capability's interface doesn't ripple to unrelated classes.

## One-Line Summary
**ISP: don't force a class to implement methods it doesn't need. Break fat interfaces into small, role-specific ones (Workable, Eatable) so each class depends only on the behavior it actually uses — no empty methods, no "not supported" exceptions.**
