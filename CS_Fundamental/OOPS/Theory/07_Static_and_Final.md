# Static and Final Keywords

## 1. Static Keyword

`static` means the member **belongs to the class**, not to any specific object.

### Static Variable

Shared across **all objects** of the class — one copy in memory regardless of how many objects exist.

```java
class Employee {
    String name;
    static String company = "Google";   // shared by all employees
    static int count = 0;               // track total employees

    Employee(String name) {
        this.name = name;
        count++;
    }
}

Employee e1 = new Employee("Alice");
Employee e2 = new Employee("Bob");

System.out.println(Employee.company);  // "Google" — accessed via class name
System.out.println(Employee.count);    // 2
```

```
Memory:
              ┌──────────────────┐
              │   Class (Method  │
              │   Area / MetaSpace│
              │   company=Google │  ← one copy, shared
              │   count=2        │
              └──────────────────┘
Heap:
 e1 → { name: "Alice" }
 e2 → { name: "Bob"   }    (no company/count here — it's in class)
```

---

### Static Method

Belongs to the class — **can be called without creating an object**.

```java
class MathUtils {
    static int square(int n) { return n * n; }
}

MathUtils.square(5);   // called without object — ✓
```

**Restrictions of static methods:**
- Cannot access `this` or `super`
- Cannot access non-static (instance) variables or methods directly
- Can only directly access static members

```java
class Example {
    int x = 10;
    static int y = 20;

    static void show() {
        // System.out.println(x);  // ✗ can't access instance variable
        System.out.println(y);     // ✓ can access static variable
    }
}
```

---

### Static Block

Runs **once** when the class is first loaded — used for static initialisation.

```java
class Config {
    static String db;

    static {                       // runs once at class loading
        db = "postgresql://localhost:5432/mydb";
        System.out.println("Config loaded");
    }
}
```

---

### Static Class (Nested)

Only inner classes can be static. A static nested class does NOT need an instance of the outer class.

```java
class Outer {
    static class StaticNested {
        void display() { System.out.println("Static Nested"); }
    }
}

Outer.StaticNested obj = new Outer.StaticNested();   // no Outer instance needed
```

---

### Static — Summary

| | static variable | static method | instance variable | instance method |
|---|---|---|---|---|
| Belongs to | Class | Class | Object | Object |
| Memory | Once (class area) | Once | Per object (heap) | Once (shared code) |
| Access via | ClassName.var | ClassName.method() | object.var | object.method() |
| `this` available | No | No | Yes | Yes |

---

## 2. Final Keyword

`final` means **cannot be changed** after initial assignment.

### Final Variable
Value cannot be changed once assigned — becomes a **constant**.

```java
final double PI = 3.14159;
// PI = 3.0;                  // ✗ compile error

final int MAX_SIZE = 100;     // convention: UPPER_SNAKE_CASE for constants
```

**Blank final:** Declared but not initialised — must be assigned in constructor.

```java
class Circle {
    final double radius;

    Circle(double r) {
        radius = r;            // assigned in constructor — valid ✓
    }
}
```

---

### Final Method
Cannot be **overridden** in a subclass.

```java
class Parent {
    final void show() {
        System.out.println("Parent show");
    }
}

class Child extends Parent {
    // @Override
    // void show() { }        // ✗ compile error — cannot override final method
}
```

**Use for:** Methods where the implementation must never change (security-critical logic, template method that must not be altered).

---

### Final Class
Cannot be **extended** (no subclass can inherit from it).

```java
final class Immutable {
    private int value;
    Immutable(int v) { value = v; }
    int getValue() { return value; }
}

// class Child extends Immutable { }   // ✗ compile error

// Examples of final classes in Java:
// String, Integer, Double, Boolean — all are final
```

**Why String is final in Java:**
- Security — if String could be subclassed, someone could override methods and tamper with string operations used in sensitive places (class loading, network URLs)
- Immutability guarantee — String's contract (immutable) cannot be broken by a subclass

---

## 3. Static vs Final

| | `static` | `final` |
|---|---|---|
| Applies to | Variables, methods, classes, blocks | Variables, methods, classes |
| Meaning | Belongs to class, not object | Cannot be changed/overridden/extended |
| Memory | Single copy in class area | N/A (just a constraint) |
| Can combine? | Yes — `static final` = class-level constant | Yes |

### static final — Class-level Constant

```java
class Constants {
    static final double PI = 3.14159;
    static final int MAX_CONNECTIONS = 100;
}

// accessed as:
Constants.PI
Constants.MAX_CONNECTIONS
```

---

## 4. Interview Questions

1. **What is the difference between static and instance methods?**
   → Static methods belong to the class and can be called without an object — they cannot access instance variables. Instance methods belong to objects and can access both instance and static members.

2. **Can a static method access a non-static variable?**
   → No. Static methods don't have a `this` reference — they have no access to instance variables. They can only access static members directly.

3. **Why is the main method static in Java?**
   → Because it must be called by the JVM before any object is created. Since no object exists yet, it must be static so the JVM can call it directly via the class name.

4. **What is a static block and when does it run?**
   → A block of code inside `static { }` — runs exactly once when the class is first loaded into memory, before any objects are created or methods are called.

5. **What is the difference between final variable and static final variable?**
   → Final variable: constant per object (each object has its own constant). Static final variable: one constant for the entire class — a true global constant.

6. **Why is String immutable (final) in Java?**
   → Security (prevents subclass tampering), thread safety (immutable objects are inherently thread-safe), and performance (String pool caching relies on immutability).

7. **Can you declare a constructor as final?**
   → No. Constructors are not inherited, so there's nothing to prevent overriding — making them final has no meaning and is a compile error.

8. **Can a final class have abstract methods?**
   → No. A final class cannot be subclassed, and abstract methods require subclasses to implement them — the combination is a contradiction and causes a compile error.
