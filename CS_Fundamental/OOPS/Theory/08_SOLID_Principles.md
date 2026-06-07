# SOLID Principles

## What is SOLID?

SOLID is a set of **5 design principles** for writing clean, maintainable, and scalable object-oriented code.

| Letter | Principle |
|---|---|
| **S** | Single Responsibility Principle |
| **O** | Open/Closed Principle |
| **L** | Liskov Substitution Principle |
| **I** | Interface Segregation Principle |
| **D** | Dependency Inversion Principle |

---

## S — Single Responsibility Principle (SRP)

> **A class should have only one reason to change.**

Every class should do exactly one thing — one job, one responsibility.

### Violation

```java
class Employee {
    String name;
    double salary;

    void calculateSalary() { }     // responsibility 1: business logic
    void saveToDatabase() { }      // responsibility 2: database
    void generatePayslip() { }     // responsibility 3: reporting
}
```

This class has 3 reasons to change — if DB changes, if salary logic changes, if report format changes.

### Fixed

```java
class Employee {
    String name;
    double salary;
    void calculateSalary() { }     // only business logic
}

class EmployeeRepository {
    void save(Employee e) { }      // only DB operations
}

class PayslipGenerator {
    void generate(Employee e) { }  // only reporting
}
```

Each class has one job — one reason to change.

---

## O — Open/Closed Principle (OCP)

> **A class should be open for extension but closed for modification.**

Add new features by adding new code — not by changing existing, tested code.

### Violation

```java
class DiscountCalculator {
    double calculate(String type, double price) {
        if (type.equals("STUDENT"))  return price * 0.8;
        if (type.equals("SENIOR"))   return price * 0.7;
        // adding new type requires modifying this class ✗
    }
}
```

Every new discount type requires changing this class — risks breaking existing logic.

### Fixed — using polymorphism

```java
interface Discount {
    double apply(double price);
}

class StudentDiscount implements Discount {
    public double apply(double price) { return price * 0.8; }
}

class SeniorDiscount implements Discount {
    public double apply(double price) { return price * 0.7; }
}

// New discount type: just add a new class — existing code untouched
class SeasonalDiscount implements Discount {
    public double apply(double price) { return price * 0.9; }
}

class DiscountCalculator {
    double calculate(Discount discount, double price) {
        return discount.apply(price);   // closed for modification ✓
    }
}
```

---

## L — Liskov Substitution Principle (LSP)

> **Subclasses should be substitutable for their parent class without breaking the program.**

If class B extends class A, then anywhere A is used, B should work correctly without any change.

### Violation

```java
class Bird {
    void fly() { System.out.println("Flying"); }
}

class Penguin extends Bird {
    @Override
    void fly() {
        throw new RuntimeException("Penguins can't fly!");  // ✗ breaks LSP
    }
}

// This breaks:
Bird b = new Penguin();
b.fly();   // throws exception — substitution broke the program
```

### Fixed

```java
class Bird {
    void eat() { }
    void sleep() { }
}

class FlyingBird extends Bird {
    void fly() { System.out.println("Flying"); }
}

class Penguin extends Bird {         // Penguin IS-A Bird, but NOT a FlyingBird ✓
    void swim() { System.out.println("Swimming"); }
}

class Eagle extends FlyingBird { }   // Eagle CAN fly ✓
```

Restructure the hierarchy so substitution always works.

---

## I — Interface Segregation Principle (ISP)

> **A class should not be forced to implement methods it doesn't need.**

Prefer many small, specific interfaces over one large general-purpose interface.

### Violation

```java
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    public void work()  { }
    public void eat()   { throw new UnsupportedOperationException(); }  // ✗ robots don't eat
    public void sleep() { throw new UnsupportedOperationException(); }  // ✗ robots don't sleep
}
```

### Fixed — split into smaller interfaces

```java
interface Workable  { void work(); }
interface Eatable   { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable {
    public void work()  { }
    public void eat()   { }
    public void sleep() { }
}

class Robot implements Workable {       // only implements what it needs ✓
    public void work() { }
}
```

---

## D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

Code to abstractions (interfaces), not to concrete implementations.

### Violation

```java
class MySQLDatabase {
    void save(String data) { System.out.println("Saving to MySQL: " + data); }
}

class UserService {
    MySQLDatabase db = new MySQLDatabase();    // directly depends on MySQL ✗

    void createUser(String name) {
        db.save(name);
    }
}
// Want to switch to PostgreSQL? Must change UserService code.
```

### Fixed — depend on abstraction

```java
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    public void save(String data) { System.out.println("MySQL: " + data); }
}

class PostgreSQLDatabase implements Database {
    public void save(String data) { System.out.println("PostgreSQL: " + data); }
}

class UserService {
    Database db;                              // depends on interface ✓

    UserService(Database db) {                // injected from outside (DI)
        this.db = db;
    }

    void createUser(String name) {
        db.save(name);
    }
}

// Switch DB without changing UserService:
UserService s1 = new UserService(new MySQLDatabase());
UserService s2 = new UserService(new PostgreSQLDatabase());
```

This is also called **Dependency Injection** — the concrete implementation is injected from outside.

---

## Summary Table

| Principle | Problem it solves | Key idea |
|---|---|---|
| **SRP** | Class doing too many things | One class = one responsibility |
| **OCP** | Modifying existing code for new features | Extend via new classes, not edits |
| **LSP** | Subclass breaking parent's contract | Subclass must be safely substitutable |
| **ISP** | Fat interfaces forcing unwanted implementations | Many small interfaces > one big one |
| **DIP** | Tight coupling to concrete classes | Depend on interfaces, not implementations |

---

## Interview Questions

1. **What is the Single Responsibility Principle?**
   → A class should have only one reason to change — one job, one responsibility.

2. **What is the Open/Closed Principle?**
   → Classes should be open for extension (add new behaviour) but closed for modification (don't change existing tested code). Achieved via polymorphism and interfaces.

3. **What is the Liskov Substitution Principle?**
   → A subclass object should be substitutable for a parent class object without breaking the program — the subclass must honour the parent's contract.

4. **What is the difference between ISP and SRP?**
   → SRP is about classes — one class, one responsibility. ISP is about interfaces — don't force a class to implement methods it doesn't need; split fat interfaces into smaller ones.

5. **What is Dependency Inversion? How is it different from Dependency Injection?**
   → DIP is the principle — depend on abstractions, not concrete classes. Dependency Injection is the technique to implement DIP — inject the concrete implementation from outside rather than creating it inside the class.

6. **Which SOLID principle does polymorphism most directly enable?**
   → Open/Closed Principle — polymorphism lets you add new behaviour (new subclasses) without modifying existing code.
