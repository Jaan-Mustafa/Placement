# Access Modifiers

## 1. What are Access Modifiers?

Access modifiers control the **visibility and accessibility** of classes, methods, and variables from different parts of the program.

---

## 2. Types (Java)

| Modifier | Same Class | Same Package | Subclass (diff package) | Everywhere |
|---|---|---|---|---|
| `private` | ✓ | ✗ | ✗ | ✗ |
| `default` (no keyword) | ✓ | ✓ | ✗ | ✗ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `public` | ✓ | ✓ | ✓ | ✓ |

---

## 3. Each Modifier Explained

### private
Only accessible **within the same class**.

```java
class BankAccount {
    private double balance = 1000;   // only BankAccount can access this

    public double getBalance() {     // public method to expose controlled access
        return balance;
    }
}

BankAccount acc = new BankAccount();
// acc.balance = 500;               // ✗ compile error
acc.getBalance();                   // ✓
```

**Use for:** Internal implementation details, sensitive data.

---

### default (package-private)
Accessible within the **same package** only. No keyword needed.

```java
class Helper {
    void assist() { }    // default — accessible only within same package
}
```

**Use for:** Internal utility classes/methods meant only for the package.

---

### protected
Accessible within the **same package** AND by **subclasses** (even in different packages).

```java
class Animal {
    protected String name;

    protected void breathe() { }    // subclasses can access this
}

class Dog extends Animal {          // even in a different package
    void bark() {
        System.out.println(name);   // ✓ — protected field accessible
        breathe();                  // ✓ — protected method accessible
    }
}
```

**Use for:** Methods/fields designed to be used by subclasses but not public API.

---

### public
Accessible **everywhere** — any class, any package.

```java
public class Car {
    public String brand;
    public void drive() { }
}
```

**Use for:** Public API — things meant to be used by anyone.

---

## 4. Access Modifiers with Inheritance

When overriding, you can **widen** access but not **narrow** it:

```java
class Parent {
    protected void show() { }
}

class Child extends Parent {
    @Override
    public void show() { }      // ✓ widened from protected → public

    // @Override
    // private void show() { }  // ✗ narrowed — compile error
}
```

---

## 5. Access Modifiers for Classes

- Top-level class: only `public` or `default`
- Inner class: any modifier

```java
public class Outer {
    private class Inner { }    // private inner class — only Outer can use it
}
```

---

## 6. Interview Questions

1. **What is the difference between private and protected?**
   → Private is accessible only within the same class. Protected is accessible within the same package and in subclasses (even across packages).

2. **What is the default access modifier in Java?**
   → Package-private — accessible only within the same package. Used when no modifier is explicitly specified.

3. **Can we override a private method?**
   → No. Private methods are not visible to subclasses, so they can't be overridden. A child can define a method with the same name, but it's a new method, not an override.

4. **Why is encapsulation achieved through private fields and public getters/setters?**
   → Private fields prevent direct external access (protecting data integrity). Public getters/setters provide controlled access with validation logic if needed.
