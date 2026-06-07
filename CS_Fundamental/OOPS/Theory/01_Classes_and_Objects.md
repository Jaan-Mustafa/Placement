# Classes and Objects

## 1. Class

A **class** is a blueprint/template that defines the properties (attributes) and behaviours (methods) that objects of that type will have.

```java
class Car {
    // attributes
    String brand;
    int speed;

    // behaviour
    void accelerate() {
        speed += 10;
    }
}
```

- A class occupies **no memory** by itself
- Memory is allocated only when an **object** is created

---

## 2. Object

An **object** is an instance of a class — a real entity created from the blueprint.

```java
Car myCar = new Car();   // object created
myCar.brand = "Toyota";
myCar.accelerate();
```

```
Class (Blueprint)          Objects (Instances)
┌──────────────────┐       ┌──────────┐  ┌──────────┐
│ Car              │  →    │ myCar    │  │ yourCar  │
│ - brand          │       │ Toyota   │  │ Honda    │
│ - speed          │       │ 60 kmph  │  │80 kmph   │
│ + accelerate()   │       └──────────┘  └──────────┘
└──────────────────┘
```

---

## 3. Constructor

A **constructor** is a special method called automatically when an object is created. It initialises the object's attributes.

### Rules
- Same name as the class
- No return type (not even void)
- Called automatically on `new`

### Types of Constructors

#### Default Constructor
No parameters — compiler provides one if you don't define any constructor.

```java
class Car {
    String brand;
    Car() {                    // default constructor
        brand = "Unknown";
    }
}
Car c = new Car();             // brand = "Unknown"
```

#### Parameterised Constructor
Takes arguments to initialise with specific values.

```java
class Car {
    String brand;
    int speed;
    Car(String b, int s) {     // parameterised constructor
        brand = b;
        speed = s;
    }
}
Car c = new Car("Toyota", 100);
```

#### Copy Constructor
Creates a new object as a copy of an existing object.

```java
class Car {
    String brand;
    Car(Car other) {           // copy constructor
        brand = other.brand;
    }
}
Car c1 = new Car("Toyota", 100);
Car c2 = new Car(c1);          // c2 is a copy of c1
```

### Constructor Overloading
Multiple constructors with different parameter lists.

```java
class Car {
    Car() { }                          // no-arg
    Car(String brand) { }              // one-arg
    Car(String brand, int speed) { }   // two-arg
}
```

---

## 4. Destructor

Called automatically when an object is **destroyed** (goes out of scope or garbage collected).

- **C++**: `~Car() { }` — explicitly defined, called deterministically
- **Java/Python**: Garbage Collector handles memory — no explicit destructor needed
  - Java: `finalize()` (deprecated), use `try-with-resources` instead
  - Python: `__del__()` method

---

## 5. `this` Keyword

Refers to the **current object** inside a method or constructor.

### Use 1 — Resolve name conflict between parameter and field

```java
class Car {
    String brand;
    Car(String brand) {
        this.brand = brand;    // this.brand = field, brand = parameter
    }
}
```

### Use 2 — Call another constructor from a constructor (constructor chaining)

```java
class Car {
    String brand;
    int speed;

    Car() {
        this("Unknown", 0);    // calls the 2-arg constructor
    }
    Car(String brand, int speed) {
        this.brand = brand;
        this.speed = speed;
    }
}
```

### Use 3 — Pass current object as argument

```java
void register(Car c) { ... }

Car myCar = new Car("Toyota", 100);
myCar.register(this);          // passes myCar itself
```

---

## 6. Object Memory Layout

```
Stack                    Heap
┌──────────────┐         ┌──────────────────────┐
│ myCar  ──────┼────────▶│ Car object           │
│ (reference)  │         │  brand = "Toyota"    │
└──────────────┘         │  speed = 100         │
                         └──────────────────────┘
```

- The **reference** (variable name) lives on the Stack
- The actual **object data** lives on the Heap
- Multiple references can point to the same object

---

## 7. Interview Questions

1. **What is the difference between a class and an object?**
   → Class is a blueprint (no memory allocated). Object is an instance of the class (memory allocated on heap).

2. **What happens if you don't define a constructor?**
   → The compiler provides a default no-argument constructor automatically. But if you define any constructor, the compiler no longer provides the default one.

3. **Can a constructor return a value?**
   → No. Constructors have no return type — not even void. They implicitly return the new object.

4. **What is constructor overloading?**
   → Defining multiple constructors in the same class with different parameter lists. The correct one is called based on the arguments passed.

5. **What is the use of the `this` keyword?**
   → Refers to the current object. Used to resolve naming conflicts, call other constructors, or pass the current object as an argument.

6. **What is the difference between stack and heap memory in OOP?**
   → Object references are stored on the stack. Actual object data is stored on the heap. Stack memory is automatically managed; heap memory is managed by the garbage collector.
