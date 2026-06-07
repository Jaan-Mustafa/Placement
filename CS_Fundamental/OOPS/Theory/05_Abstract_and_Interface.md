# Abstract Class & Interface

## 1. Abstract Class

A class declared with `abstract` keyword — **cannot be instantiated** directly. May have abstract methods (no body) and concrete methods (with body).

```java
abstract class Shape {
    String color;

    abstract double area();          // abstract — no body, child MUST implement

    void display() {                 // concrete — has body
        System.out.println("Color: " + color);
    }
}

class Circle extends Shape {
    double radius;

    @Override
    double area() {                  // child must implement
        return Math.PI * radius * radius;
    }
}

// Shape s = new Shape();           // ✗ cannot instantiate abstract class
Shape s = new Circle();             // ✓ reference of parent, object of child
```

### Rules
- Declared with `abstract` keyword
- Can have abstract and non-abstract methods
- Can have constructors (called via `super()` from child)
- Can have instance variables
- A child MUST implement all abstract methods, or also be declared abstract
- Cannot be instantiated directly

---

## 2. Interface

A **contract** — defines what a class must do, not how. All methods are implicitly `public abstract` (before Java 8). No instance variables — only constants.

```java
interface Drawable {
    void draw();                     // implicitly public abstract
    double area();
}

interface Colorable {
    void setColor(String color);
}

class Circle implements Drawable, Colorable {
    @Override
    public void draw() { System.out.println("Drawing circle"); }

    @Override
    public double area() { return Math.PI * 5 * 5; }

    @Override
    public void setColor(String color) { System.out.println("Color: " + color); }
}
```

### Rules
- Declared with `interface` keyword
- All methods are `public abstract` by default (Java 7 and below)
- All variables are `public static final` (constants)
- A class `implements` an interface (not `extends`)
- A class can implement **multiple interfaces** ✓
- Cannot be instantiated directly

### Java 8+ additions to Interface
- **Default methods** — methods with a body (so old implementations don't break)
- **Static methods** — utility methods inside interface

```java
interface Drawable {
    void draw();

    default void print() {           // default method — Java 8+
        System.out.println("Printing shape");
    }

    static void info() {             // static method — Java 8+
        System.out.println("Drawable interface");
    }
}
```

---

## 3. Abstract Class vs Interface — Full Comparison

| Feature | Abstract Class | Interface |
|---|---|---|
| Keyword | `abstract class` | `interface` |
| Methods | Abstract + concrete | Abstract by default (default/static in Java 8+) |
| Variables | Any type | `public static final` only (constants) |
| Constructor | Yes | No |
| Multiple inheritance | No (single `extends`) | Yes (multiple `implements`) |
| Access modifiers | Any | `public` only |
| Speed | Slightly faster | Slightly slower (extra indirection) |
| When to use | IS-A with shared code | CAN-DO / capability contract |
| `extends`/`implements` | `extends` | `implements` |

---

## 4. When to Use Which?

### Use Abstract Class when:
- Classes share **common code** — put it in the abstract class
- You want to provide a **default implementation** for some methods
- The relationship is truly **IS-A** — `Dog IS-A Animal`
- You need **constructors** or **instance variables**

```java
// Good use of abstract class:
abstract class Vehicle {
    int speed;
    String fuel;

    void refuel() { System.out.println("Refuelling..."); }  // shared code

    abstract void drive();   // each vehicle drives differently
}

class Car extends Vehicle {
    void drive() { System.out.println("Driving car"); }
}
```

### Use Interface when:
- Defining a **capability** that unrelated classes can share — `CAN-DO` relationship
- You need **multiple inheritance**
- You want to define a **contract** without any shared implementation
- Classes from different hierarchies need to share behaviour

```java
// Good use of interface:
interface Flyable  { void fly(); }
interface Swimmable { void swim(); }

class Duck implements Flyable, Swimmable {
    public void fly()  { System.out.println("Duck flying"); }
    public void swim() { System.out.println("Duck swimming"); }
}

class Airplane implements Flyable {
    public void fly() { System.out.println("Airplane flying"); }
}
// Duck and Airplane are unrelated — but both CAN fly
```

---

## 5. Interface Extending Interface

An interface can extend another interface (or multiple interfaces):

```java
interface A { void doA(); }
interface B { void doB(); }
interface C extends A, B { void doC(); }   // C inherits doA and doB

class D implements C {
    public void doA() { }
    public void doB() { }
    public void doC() { }   // must implement all three
}
```

---

## 6. Real-World Example — Both Together

```java
// Abstract class — shared structure
abstract class Animal {
    String name;
    Animal(String name) { this.name = name; }
    abstract void sound();
    void eat() { System.out.println(name + " is eating"); }
}

// Interface — capability
interface Trainable {
    void train(String command);
}

// Dog IS-A Animal AND CAN-DO Trainable
class Dog extends Animal implements Trainable {
    Dog(String name) { super(name); }

    @Override
    public void sound() { System.out.println("Woof"); }

    @Override
    public void train(String cmd) {
        System.out.println(name + " learned: " + cmd);
    }
}
```

---

## 7. Interview Questions

1. **What is the difference between abstract class and interface?**
   → Abstract class can have concrete methods, constructors, and any access modifiers. Interface methods are public abstract by default, no constructors, only constants. A class can implement multiple interfaces but extend only one abstract class.

2. **Can an abstract class have a constructor?**
   → Yes. Even though it can't be instantiated directly, its constructor is called via `super()` when a child class is instantiated.

3. **Can an interface have a constructor?**
   → No. Interfaces cannot be instantiated and have no constructors.

4. **When would you use an abstract class over an interface?**
   → When there is shared implementation code among related classes (IS-A relationship). Interface is better for unrelated classes sharing a capability (CAN-DO).

5. **What is a marker interface?**
   → An empty interface with no methods — used to mark a class for special treatment by the JVM or frameworks. Examples: `Serializable`, `Cloneable` in Java.

6. **Can you instantiate an abstract class?**
   → No. But you can create an anonymous subclass inline: `Shape s = new Shape() { double area() { return 0; } };`

7. **What is the difference between IS-A and CAN-DO relationships?**
   → IS-A is inheritance (Dog IS-A Animal → use abstract class or class). CAN-DO is a capability (Duck CAN-DO Flyable → use interface).

8. **Can an interface extend a class?**
   → No. An interface can only extend another interface. A class implements an interface.
