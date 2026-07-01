# L — Liskov Substitution Principle (LSP)

> **Subtypes must be substitutable for their base type** — anywhere you use a base class, you should be able to use any subclass without breaking behavior.

## The Idea
A subclass should **honor the contract** of its parent. If replacing the parent with a child causes wrong behavior, exceptions, or surprises, LSP is violated. Inheritance should model true "is-a" behavior, not just shared fields.

- Named after Barbara Liskov.
- Key test: *"Can I pass this subclass wherever the base is expected and everything still works correctly?"*

---

## ❌ Violation (the classic Rectangle/Square problem)

```java
class Rectangle {
    protected int width, height;
    public void setWidth(int w)  { this.width = w; }
    public void setHeight(int h) { this.height = h; }
    public int getArea() { return width * height; }
}

// A square "is-a" rectangle... mathematically. But behaviorally?
class Square extends Rectangle {
    public void setWidth(int w)  { this.width = w; this.height = w; }  // forces both
    public void setHeight(int h) { this.width = h; this.height = h; }  // forces both
}
```

Now client code written for `Rectangle`:
```java
void resize(Rectangle r) {
    r.setWidth(5);
    r.setHeight(4);
    assert r.getArea() == 20;   // expects 5 * 4 = 20
}
```

Pass a `Square` → `setHeight(4)` also sets width to 4 → area = 16, **not 20**. The assert fails. The subclass **broke the base class's contract** → LSP violated.

---

## ✅ Fix (don't force a false is-a; use a proper abstraction)

```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width, height;
    public Rectangle(int w, int h) { this.width = w; this.height = h; }
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private int side;
    public Square(int side) { this.side = side; }
    public int getArea() { return side * side; }
}
```

Now `Square` and `Rectangle` are **independent** shapes under a common `Shape` interface. Neither pretends to be the other, so no contract is broken. Any `Shape` can be substituted safely.

---

## Another everyday example of violation
```java
class Bird { void fly() { ... } }
class Ostrich extends Bird {
    void fly() { throw new UnsupportedOperationException(); }  // ostriches can't fly!
}
```
Code expecting `Bird.fly()` breaks on `Ostrich`. Fix: separate `Bird` and `FlyingBird` (interface segregation helps here too).

---

## How to Spot LSP Violations
- A subclass **overriding a method to throw an exception** or do nothing.
- A subclass that **weakens** guarantees or changes expected behavior.
- Needing `if (obj instanceof SubType)` checks in client code to handle a subclass specially.

## Benefits
- Polymorphism actually works — you can rely on the base contract.
- No nasty surprises when swapping implementations.

## One-Line Summary
**LSP: a subclass must be usable wherever its parent is, without breaking behavior or the parent's contract. If a subclass throws on inherited methods or changes expected results (Square breaking Rectangle), the inheritance is wrong — model them as independent types under a shared interface instead.**
