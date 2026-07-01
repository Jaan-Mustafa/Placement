# O — Open/Closed Principle (OCP)

> **Software entities should be OPEN for extension but CLOSED for modification.**

## The Idea
You should be able to **add new behavior without changing existing, tested code**. Achieve this with abstractions (interfaces/abstract classes) — new features come as new classes that implement the interface, not as edits to old classes.

- **Open for extension** → you can add new functionality.
- **Closed for modification** → you don't touch code that already works.

Every time you edit a working class to add a case, you risk breaking it. OCP avoids that.

---

## ❌ Violation (must edit the class for every new type)

```java
class PaymentProcessor {
    public void pay(String type, double amount) {
        if (type.equals("CreditCard")) {
            // credit card logic
        } else if (type.equals("PayPal")) {
            // paypal logic
        } else if (type.equals("UPI")) {          // <- had to EDIT this class
            // upi logic
        }
        // every new method = another if-else here = modifying tested code
    }
}
```

**Why it's bad:** adding a new payment method (e.g. Crypto) forces you to **modify** `PaymentProcessor` and re-test the whole thing. The class is not closed for modification.

---

## ✅ Fix (extend via an interface)

```java
// Abstraction
interface PaymentMethod {
    void pay(double amount);
}

// Each method = its own class (extension)
class CreditCardPayment implements PaymentMethod {
    public void pay(double amount) { /* credit card logic */ }
}

class PayPalPayment implements PaymentMethod {
    public void pay(double amount) { /* paypal logic */ }
}

class UpiPayment implements PaymentMethod {
    public void pay(double amount) { /* upi logic */ }
}

// Processor never changes — it works on the abstraction
class PaymentProcessor {
    public void pay(PaymentMethod method, double amount) {
        method.pay(amount);
    }
}
```

**Now adding Crypto** = just create a new class:
```java
class CryptoPayment implements PaymentMethod {
    public void pay(double amount) { /* crypto logic */ }
}
```
No existing class is touched. `PaymentProcessor` stays **closed for modification** but the system is **open for extension**. (This is also the Strategy pattern.)

---

## How to Spot OCP Violations
- Long **if-else / switch** chains on a "type" that grow every time a feature is added.
- Having to reopen and edit the same class for each new variant.

## Benefits
- Existing, tested code stays untouched → fewer regressions.
- New features = new classes → safer, parallel-friendly development.

## One-Line Summary
**OCP: add new behavior by writing new classes (extension), not by editing existing ones (modification). Put the varying part behind an interface so new variants plug in without touching tested code — killing the growing if-else chain.**
