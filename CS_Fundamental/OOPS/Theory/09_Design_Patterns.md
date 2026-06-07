# Design Patterns

## What are Design Patterns?

**Reusable solutions to commonly occurring problems** in software design. Not code — but templates for how to solve a problem.

## Categories

```
Design Patterns
├── Creational  — how objects are created
├── Structural  — how objects are composed/structured
└── Behavioural — how objects communicate
```

---

## CREATIONAL PATTERNS

### 1. Singleton Pattern

**Ensures only ONE instance of a class exists throughout the application.**

```java
class DatabaseConnection {
    private static DatabaseConnection instance;   // single instance

    private DatabaseConnection() { }             // private constructor — no outside instantiation

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();  // create only once
        }
        return instance;
    }

    public void query(String sql) { }
}

// Usage:
DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
// db1 == db2 → true — same object
```

**Thread-safe Singleton (Double-Checked Locking):**

```java
class Singleton {
    private static volatile Singleton instance;

    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {           // double check inside lock
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Use cases:** DB connection pool, Logger, Config manager, Thread pool

---

### 2. Factory Pattern

**Lets a factory class decide which object to create** — the caller doesn't need to know the concrete class.

```java
interface Shape {
    void draw();
}

class Circle implements Shape {
    public void draw() { System.out.println("Drawing Circle"); }
}

class Rectangle implements Shape {
    public void draw() { System.out.println("Drawing Rectangle"); }
}

// Factory — creates the right object based on input
class ShapeFactory {
    public static Shape create(String type) {
        switch (type) {
            case "CIRCLE":    return new Circle();
            case "RECTANGLE": return new Rectangle();
            default: throw new IllegalArgumentException("Unknown shape");
        }
    }
}

// Usage — caller doesn't know Circle or Rectangle exists
Shape s = ShapeFactory.create("CIRCLE");
s.draw();
```

**Use cases:** Creating DB connections (MySQL/PostgreSQL), UI components, parsers

---

### 3. Builder Pattern

**Construct complex objects step by step** — useful when an object has many optional parameters.

```java
class Pizza {
    String size;
    String crust;
    boolean cheese;
    boolean pepperoni;

    private Pizza(Builder b) {
        this.size      = b.size;
        this.crust     = b.crust;
        this.cheese    = b.cheese;
        this.pepperoni = b.pepperoni;
    }

    static class Builder {
        String size;
        String crust  = "thin";
        boolean cheese    = false;
        boolean pepperoni = false;

        Builder(String size) { this.size = size; }

        Builder crust(String c)    { this.crust = c;      return this; }
        Builder cheese()           { this.cheese = true;  return this; }
        Builder pepperoni()        { this.pepperoni = true; return this; }

        Pizza build() { return new Pizza(this); }
    }
}

// Usage — clean, readable, no constructor with 10 params
Pizza p = new Pizza.Builder("Large")
                   .crust("thick")
                   .cheese()
                   .pepperoni()
                   .build();
```

**Use cases:** Building HTTP requests, SQL query builders, complex configuration objects

---

## STRUCTURAL PATTERNS

### 4. Decorator Pattern

**Dynamically add behaviour to an object without changing its class** — wraps the original object.

```java
interface Coffee {
    String getDescription();
    double getCost();
}

class SimpleCoffee implements Coffee {
    public String getDescription() { return "Coffee"; }
    public double getCost()        { return 50; }
}

// Decorator — wraps a Coffee and adds behaviour
class MilkDecorator implements Coffee {
    Coffee coffee;
    MilkDecorator(Coffee c) { this.coffee = c; }

    public String getDescription() { return coffee.getDescription() + ", Milk"; }
    public double getCost()        { return coffee.getCost() + 15; }
}

class SugarDecorator implements Coffee {
    Coffee coffee;
    SugarDecorator(Coffee c) { this.coffee = c; }

    public String getDescription() { return coffee.getDescription() + ", Sugar"; }
    public double getCost()        { return coffee.getCost() + 5; }
}

// Usage — stack decorators
Coffee c = new SimpleCoffee();
c = new MilkDecorator(c);
c = new SugarDecorator(c);

c.getDescription();  // "Coffee, Milk, Sugar"
c.getCost();         // 70
```

**Use cases:** Java I/O streams (`BufferedReader` wraps `FileReader`), UI component styling

---

### 5. Adapter Pattern

**Makes two incompatible interfaces work together** — converts one interface to another.

```java
// Existing interface
interface OldPrinter {
    void printOld(String text);
}

// New interface the client expects
interface NewPrinter {
    void print(String text);
}

class OldPrinterImpl implements OldPrinter {
    public void printOld(String text) { System.out.println("Old: " + text); }
}

// Adapter — wraps OldPrinter, exposes NewPrinter interface
class PrinterAdapter implements NewPrinter {
    OldPrinter oldPrinter;
    PrinterAdapter(OldPrinter p) { this.oldPrinter = p; }

    public void print(String text) {
        oldPrinter.printOld(text);    // delegates to old implementation
    }
}

// Client only knows NewPrinter
NewPrinter printer = new PrinterAdapter(new OldPrinterImpl());
printer.print("Hello");
```

**Use cases:** Third-party library integration, legacy system compatibility

---

## BEHAVIOURAL PATTERNS

### 6. Observer Pattern

**When one object changes state, all its dependents are notified automatically.**

```java
import java.util.*;

interface Observer {
    void update(String event);
}

class EventSystem {
    List<Observer> observers = new ArrayList<>();

    void subscribe(Observer o)   { observers.add(o); }
    void unsubscribe(Observer o) { observers.remove(o); }

    void notify(String event) {
        for (Observer o : observers)
            o.update(event);
    }
}

class EmailNotifier implements Observer {
    public void update(String event) {
        System.out.println("Email sent for: " + event);
    }
}

class SMSNotifier implements Observer {
    public void update(String event) {
        System.out.println("SMS sent for: " + event);
    }
}

// Usage
EventSystem es = new EventSystem();
es.subscribe(new EmailNotifier());
es.subscribe(new SMSNotifier());

es.notify("ORDER_PLACED");
// Email sent for: ORDER_PLACED
// SMS sent for: ORDER_PLACED
```

**Use cases:** Event systems, MVC (View observes Model), notification services

---

### 7. Strategy Pattern

**Define a family of algorithms, encapsulate each one, and make them interchangeable** — lets the algorithm vary independently from clients.

```java
interface SortStrategy {
    void sort(int[] arr);
}

class BubbleSort implements SortStrategy {
    public void sort(int[] arr) { System.out.println("Bubble sort"); }
}

class QuickSort implements SortStrategy {
    public void sort(int[] arr) { System.out.println("Quick sort"); }
}

class Sorter {
    SortStrategy strategy;

    Sorter(SortStrategy s) { this.strategy = s; }

    void setStrategy(SortStrategy s) { this.strategy = s; }

    void sort(int[] arr) { strategy.sort(arr); }
}

// Usage — swap strategies at runtime
Sorter sorter = new Sorter(new BubbleSort());
sorter.sort(arr);                         // Bubble sort

sorter.setStrategy(new QuickSort());
sorter.sort(arr);                         // Quick sort
```

**Use cases:** Payment methods (UPI/Card/NetBanking), compression algorithms, authentication strategies

---

## Summary — Most Asked in Interviews

| Pattern | Category | Key idea | Real-world use |
|---|---|---|---|
| **Singleton** | Creational | One instance only | DB connection, Logger |
| **Factory** | Creational | Factory decides which class | Shape/DB/UI creation |
| **Builder** | Creational | Build step-by-step | HTTP request, SQL query |
| **Decorator** | Structural | Wrap to add behaviour | Java I/O, coffee toppings |
| **Adapter** | Structural | Bridge incompatible interfaces | Legacy integration |
| **Observer** | Behavioural | Notify all dependents | Events, MVC, notifications |
| **Strategy** | Behavioural | Swap algorithms at runtime | Sorting, payment, auth |

---

## Interview Questions

1. **What is the Singleton pattern? How do you make it thread-safe?**
   → Ensures one instance. Thread-safe via double-checked locking with `volatile` and `synchronized`.

2. **What is the difference between Factory and Builder pattern?**
   → Factory creates objects in one step — chooses which class to instantiate. Builder constructs one complex object step by step with many optional parameters.

3. **What is the difference between Decorator and Inheritance?**
   → Inheritance adds behaviour at compile time (static). Decorator adds behaviour at runtime (dynamic) by wrapping objects — more flexible than inheritance.

4. **What is the Observer pattern?**
   → A subject maintains a list of observers. When the subject's state changes, all observers are automatically notified. Used in event-driven systems.

5. **What is the difference between Strategy and Factory pattern?**
   → Factory is about object creation — which class to instantiate. Strategy is about behaviour — which algorithm to use at runtime.

6. **Where is Singleton used in real projects?**
   → Logger (one log file), DB connection pool, application config, thread pool manager.
