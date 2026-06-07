# Inheritance

## 1. What is Inheritance?

Inheritance allows a **child class (subclass)** to acquire properties and methods of a **parent class (superclass)**, promoting code reuse.

```java
class Parent {
    void greet() { System.out.println("Hello from Parent"); }
}

class Child extends Parent {    // Child inherits from Parent
    void play() { System.out.println("Child is playing"); }
}

Child c = new Child();
c.greet();   // inherited ✓
c.play();    // own method ✓
```

---

## 2. Types of Inheritance

### 2.1 Single Inheritance
One child, one parent.

```
    Animal
      ↑
     Dog
```

```java
class Animal { void eat() {} }
class Dog extends Animal { void bark() {} }
```

---

### 2.2 Multi-Level Inheritance
Chain of inheritance — grandparent → parent → child.

```
   Animal
     ↑
  Mammal
     ↑
    Dog
```

```java
class Animal  { void breathe() {} }
class Mammal  extends Animal  { void feed() {} }
class Dog     extends Mammal  { void bark()  {} }

Dog d = new Dog();
d.breathe();  // from Animal ✓
d.feed();     // from Mammal ✓
d.bark();     // from Dog ✓
```

---

### 2.3 Hierarchical Inheritance
One parent, multiple children.

```
      Animal
     /      \
   Dog       Cat
```

```java
class Animal { void eat() {} }
class Dog extends Animal { void bark() {} }
class Cat extends Animal { void meow() {} }
```

---

### 2.4 Multiple Inheritance
One child inherits from multiple parents.

```
   A     B
    \   /
      C
```

**Java does NOT support multiple inheritance with classes** — causes the Diamond Problem.  
Java supports it only through **interfaces**.

```java
// NOT allowed in Java:
class C extends A, B { }   // ✗ compile error

// Allowed via interfaces:
interface A { void doA(); }
interface B { void doB(); }
class C implements A, B { } // ✓
```

---

### 2.5 Hybrid Inheritance
Combination of two or more types. Possible in Java only through interfaces.

```
     A
    / \
   B   C
    \ /
     D
```

---

## 3. Diamond Problem

Occurs in multiple inheritance when two parent classes have the same method — the child doesn't know which one to use.

```
        Animal
       void eat()
       /        \
    Dog          Cat
  void eat()   void eat()
       \        /
        ???
      which eat()?
```

**Java's solution:** No multiple class inheritance. Use interfaces — and if two interfaces have the same default method, the implementing class must override it explicitly.

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    public void hello() {       // must override — resolve ambiguity
        A.super.hello();        // explicitly choose A's version
    }
}
```

---

## 4. `super` Keyword

Used to refer to the **parent class** from inside a child class.

### Use 1 — Call parent constructor

```java
class Animal {
    String name;
    Animal(String name) { this.name = name; }
}

class Dog extends Animal {
    String breed;
    Dog(String name, String breed) {
        super(name);           // calls Animal's constructor
        this.breed = breed;
    }
}
```

### Use 2 — Call parent method (overridden)

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}

class Dog extends Animal {
    void sound() {
        super.sound();         // calls Animal's sound()
        System.out.println("Woof");
    }
}
```

### Use 3 — Access parent field

```java
class Animal { String type = "Animal"; }

class Dog extends Animal {
    String type = "Dog";
    void printType() {
        System.out.println(super.type);  // "Animal"
        System.out.println(this.type);   // "Dog"
    }
}
```

---

## 5. Method Overriding

Child class provides its own implementation of a method already defined in the parent.

### Rules
- Same method name, same parameters, same return type
- Access modifier can be **same or wider** (not narrower)
- Cannot override `static`, `final`, or `private` methods
- Use `@Override` annotation (catches mistakes at compile time)

```java
class Animal {
    void sound() { System.out.println("..."); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("Woof"); }   // overrides
}

Animal a = new Dog();
a.sound();    // "Woof" — runtime polymorphism
```

---

## 6. What is Inherited and What is NOT

| | Inherited? |
|---|---|
| public methods | Yes |
| protected methods | Yes |
| public fields | Yes |
| protected fields | Yes |
| private methods | No |
| private fields | No |
| Constructors | No (but can be called via `super()`) |
| static methods | Yes (but NOT overridden — hidden instead) |

---

## 7. `final` and Inheritance

- `final class` — cannot be extended
- `final method` — cannot be overridden
- `final variable` — cannot be reassigned

```java
final class Immutable { }       // no class can extend this
class Child extends Immutable { } // ✗ compile error

class Parent {
    final void show() { }        // cannot be overridden
}
```

---

## 8. Interview Questions

1. **What is the difference between single and multiple inheritance?**
   → Single: one child, one parent. Multiple: one child, two+ parents. Java doesn't support multiple inheritance with classes — only with interfaces.

2. **Why doesn't Java support multiple inheritance with classes?**
   → Diamond problem — if two parent classes have the same method, the child is ambiguous about which to inherit. Java avoids this by disallowing it for classes.

3. **What is the diamond problem?**
   → When a class inherits from two classes that both inherit from the same grandparent — the method resolution becomes ambiguous.

4. **What is the difference between `this` and `super`?**
   → `this` refers to the current object. `super` refers to the parent class — used to call parent constructors or overridden methods.

5. **Can you override a private method?**
   → No. Private methods are not visible to child classes, so they cannot be overridden. The child class can define a method with the same name, but it's a new method, not an override.

6. **Can you override a static method?**
   → No. Static methods belong to the class, not instances. A child can define a static method with the same name, but this is called **method hiding**, not overriding — it's resolved at compile time, not runtime.

7. **What is the use of `@Override` annotation?**
   → It tells the compiler to verify that the method actually overrides a parent method. If it doesn't (due to typo or wrong parameters), the compiler throws an error — catches mistakes early.

8. **What is IS-A vs HAS-A relationship?**
   → IS-A is inheritance (Dog IS-A Animal). HAS-A is composition (Car HAS-A Engine). Prefer HAS-A over IS-A when the relationship isn't truly hierarchical — composition is more flexible than inheritance.
