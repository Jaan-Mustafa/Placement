# Polymorphism

## 1. What is Polymorphism?

**Poly = many, Morph = forms** — the ability of a single entity (method, object) to take multiple forms.

```
Same method name → different behaviour based on context
```

---

## 2. Types of Polymorphism

```
             Polymorphism
            /             \
    Compile-Time          Runtime
   (Static Binding)    (Dynamic Binding)
         |                    |
   Method Overloading   Method Overriding
```

---

## 3. Compile-Time Polymorphism — Method Overloading

**Same method name, different parameter list — resolved at compile time.**

```java
class Calculator {
    int add(int a, int b)             { return a + b; }
    double add(double a, double b)    { return a + b; }
    int add(int a, int b, int c)      { return a + b + c; }
    String add(String a, String b)    { return a + b; }
}

Calculator c = new Calculator();
c.add(1, 2);           // → int version
c.add(1.5, 2.5);       // → double version
c.add(1, 2, 3);        // → three-arg version
c.add("Hi", " there"); // → String version
```

The compiler decides **at compile time** which method to call based on the argument types and count.

### Rules for Overloading

| Rule | Detail |
|---|---|
| Method name | Must be the same |
| Parameters | Must differ — type, count, or order |
| Return type | Can be different (but alone is NOT enough to overload) |
| Access modifier | Can be different |

### What does NOT count as overloading

```java
int add(int a, int b)     { return a + b; }
double add(int a, int b)  { return a + b; }  // ✗ same params, only return type differs
                                               // COMPILE ERROR
```

### Operator Overloading
- **C++** supports operator overloading (`+`, `-`, `*`)
- **Java** does NOT support operator overloading (except `+` for String, which is built-in)

---

## 4. Runtime Polymorphism — Method Overriding

**Child class redefines a parent method — the actual method called is decided at runtime based on the object type.**

```java
class Animal {
    void sound() { System.out.println("Some sound"); }
}
class Dog extends Animal {
    @Override
    void sound() { System.out.println("Woof"); }
}
class Cat extends Animal {
    @Override
    void sound() { System.out.println("Meow"); }
}

Animal a;

a = new Dog();
a.sound();     // "Woof"  — Dog's version called at runtime

a = new Cat();
a.sound();     // "Meow"  — Cat's version called at runtime
```

The reference type is `Animal` — but the actual object is `Dog` or `Cat`. Java's JVM decides which `sound()` to call **at runtime** by checking the actual object type.

This is called **Dynamic Method Dispatch**.

---

## 5. Dynamic Method Dispatch

The mechanism by which a call to an overridden method is resolved at runtime.

```
Animal a = new Dog();

Compile time: compiler only checks Animal class — does Animal have sound()? Yes ✓
Runtime: JVM checks actual object type — it's a Dog → call Dog's sound()
```

```
Reference Variable      Actual Object
     Animal         →       Dog
       ↓                      ↓
   sees Animal's          JVM calls Dog's
   method signature        sound() at runtime
```

---

## 6. Overloading vs Overriding — Complete Comparison

| Feature | Overloading | Overriding |
|---|---|---|
| Definition | Same name, different parameters in same class | Child redefines parent's method |
| Where | Same class | Parent and child class |
| Parameters | Must differ | Must be identical |
| Return type | Can differ | Must be same (or covariant) |
| Access modifier | Can be anything | Cannot be more restrictive than parent |
| Resolved at | Compile time | Runtime |
| Polymorphism type | Compile-time (static) | Runtime (dynamic) |
| `@Override` | Not used | Should always use |
| Inheritance needed | No | Yes |
| Can override static? | N/A | No — static methods are hidden, not overridden |
| Can override private? | N/A | No — private methods not visible to child |

---

## 7. Covariant Return Type

An overriding method can return a **subtype** of the original return type.

```java
class Animal {
    Animal getInstance() { return new Animal(); }
}

class Dog extends Animal {
    @Override
    Dog getInstance() {    // Dog is a subtype of Animal — allowed ✓
        return new Dog();
    }
}
```

---

## 8. Polymorphism with Arrays

```java
Animal[] animals = new Animal[3];
animals[0] = new Dog();
animals[1] = new Cat();
animals[2] = new Animal();

for (Animal a : animals) {
    a.sound();    // Woof, Meow, Some sound — each calls its own version
}
```

This is the power of runtime polymorphism — process a mixed collection uniformly.

---

## 9. Interview Questions

1. **What is the difference between compile-time and runtime polymorphism?**
   → Compile-time: resolved by compiler based on method signature (overloading). Runtime: resolved by JVM at execution based on actual object type (overriding).

2. **Can you overload a method by changing only the return type?**
   → No. Return type alone is not sufficient to distinguish overloaded methods — the compiler uses parameter list to resolve calls.

3. **What is Dynamic Method Dispatch?**
   → The JVM mechanism that resolves an overridden method call at runtime based on the actual object type, not the reference type.

4. **Can static methods be overridden?**
   → No. Static methods are bound at compile time to the class, not the object. If a child defines the same static method, it's called method hiding — not overriding. No runtime polymorphism applies.

5. **What is method hiding?**
   → When a child class defines a static method with the same signature as a parent's static method — the parent method is hidden, not overridden. The method called depends on the reference type, not the object type.

6. **Can constructors be overloaded?**
   → Yes. A class can have multiple constructors with different parameter lists — constructor overloading.

7. **What is the difference between method overloading and method overriding in terms of binding?**
   → Overloading uses early/static binding — resolved at compile time. Overriding uses late/dynamic binding — resolved at runtime by the JVM.
