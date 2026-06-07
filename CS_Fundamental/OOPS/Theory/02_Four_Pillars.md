# Four Pillars of OOPS

## Overview

```
┌─────────────────────────────────────────────────────┐
│                  Four Pillars of OOP                │
├──────────────┬───────────────┬──────────┬───────────┤
│ Encapsulation│  Abstraction  │Inheritance│Polymorphism│
│              │               │           │            │
│ Wrapping data│ Hiding        │ Child     │ Same name, │
│ + methods    │ implementation│ inherits  │ different  │
│ together,    │ details,      │ from      │ behaviour  │
│ hiding       │ showing only  │ parent    │            │
│ internals    │ interface     │           │            │
└──────────────┴───────────────┴──────────┴────────────┘
```

---

## 1. Encapsulation

**Wrapping data (attributes) and methods (behaviour) together into a single unit, and restricting direct access to the data.**

### Real-World Analogy
A **capsule (medicine pill)** — the ingredients are wrapped inside, you can't directly touch them. You just take the pill.

### How it works in code
- Make attributes `private`
- Provide `public` getter/setter methods to access them

```java
class BankAccount {
    private double balance;       // hidden — can't access directly

    public double getBalance() {  // controlled read
        return balance;
    }

    public void deposit(double amount) {  // controlled write
        if (amount > 0)
            balance += amount;
    }
}

BankAccount acc = new BankAccount();
// acc.balance = -5000;          // ERROR — private, can't access
acc.deposit(1000);               // correct way
```

### Why it matters
- Prevents invalid data (`balance = -5000` blocked)
- Internal implementation can change without breaking external code
- Single place to put validation logic

---

## 2. Abstraction

**Hiding the internal implementation details and showing only the essential features to the user.**

### Real-World Analogy
A **car** — you use the steering wheel and pedals. You don't need to know how the engine, fuel injection, or braking system works internally.

### How it works in code
Achieved via **abstract classes** and **interfaces**.

```java
abstract class Shape {
    abstract double area();       // what to do — no implementation
}

class Circle extends Shape {
    double radius;
    double area() {               // how to do it — implementation
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    double w, h;
    double area() {
        return w * h;
    }
}

Shape s = new Circle();
s.area();   // user calls area() — doesn't care HOW it's computed
```

### Encapsulation vs Abstraction

| | Encapsulation | Abstraction |
|---|---|---|
| **Focus** | Hiding data | Hiding implementation |
| **Goal** | Data protection | Complexity reduction |
| **Achieved by** | private + getters/setters | abstract class / interface |
| **What is hidden** | Internal state | Internal logic |
| **What is shown** | Controlled access methods | Essential interface only |

---

## 3. Inheritance

**A child class acquires properties and behaviours of a parent class, enabling code reuse.**

### Real-World Analogy
A **child inherits traits from parents** — eye colour, height. But the child also has their own unique traits.

```java
class Animal {
    String name;
    void eat() { System.out.println(name + " is eating"); }
}

class Dog extends Animal {       // Dog inherits from Animal
    void bark() { System.out.println("Woof!"); }
}

Dog d = new Dog();
d.name = "Bruno";
d.eat();    // inherited from Animal ✓
d.bark();   // Dog's own method ✓
```

### Types (covered in detail in 03_Inheritance.md)
- Single, Multi-level, Hierarchical, Multiple (via interface), Hybrid

### IS-A Relationship
Inheritance models an **IS-A** relationship:
- Dog IS-A Animal ✓
- Car IS-A Vehicle ✓
- Car IS-A Animal ✗ — don't inherit just for code reuse

---

## 4. Polymorphism

**Same method name, different behaviour depending on the object or arguments.**

### Real-World Analogy
A person behaves differently in different roles — same person IS-A employee at work, IS-A parent at home, IS-A customer at a shop.

### Two Types

#### Compile-Time (Method Overloading)
Same method name, different parameters — resolved at **compile time**.

```java
class Calculator {
    int add(int a, int b)          { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c)   { return a + b + c; }
}

calc.add(1, 2);        // calls int version
calc.add(1.5, 2.5);    // calls double version
```

#### Runtime (Method Overriding)
Child class redefines a parent's method — resolved at **runtime** based on actual object type.

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}
class Dog extends Animal {
    void sound() { System.out.println("Woof"); }   // overrides
}
class Cat extends Animal {
    void sound() { System.out.println("Meow"); }   // overrides
}

Animal a = new Dog();   // reference is Animal, object is Dog
a.sound();              // prints "Woof" — runtime decision ✓
```

### Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Where | Same class | Parent–child classes |
| Parameters | Must differ | Must be same |
| Return type | Can differ | Must be same (or covariant) |
| Resolved at | Compile time | Runtime |
| Keyword | None needed | `@Override` annotation |
| Polymorphism type | Compile-time | Runtime |

---

## 5. Summary — One Line Each

| Pillar | One Line |
|---|---|
| **Encapsulation** | Bundle data + methods, hide data with private access |
| **Abstraction** | Show what, hide how — use abstract class/interface |
| **Inheritance** | Child reuses parent's code via IS-A relationship |
| **Polymorphism** | Same name, different behaviour — overloading or overriding |
