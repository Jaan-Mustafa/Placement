# OOPS Cheatsheet — Quick Revision

## Four Pillars

| Pillar | One Line | How |
|---|---|---|
| **Encapsulation** | Bundle data + methods, hide internals | `private` fields + `public` getters/setters |
| **Abstraction** | Show what, hide how | Abstract class / Interface |
| **Inheritance** | Child reuses parent code | `extends` keyword |
| **Polymorphism** | Same name, different behaviour | Overloading (compile) / Overriding (runtime) |

---

## Overloading vs Overriding

| | Overloading | Overriding |
|---|---|---|
| Where | Same class | Parent–child |
| Parameters | Must differ | Must be same |
| Resolved | Compile time | Runtime |
| Type | Static polymorphism | Dynamic polymorphism |

---

## Abstract Class vs Interface

| | Abstract Class | Interface |
|---|---|---|
| Methods | Abstract + concrete | Abstract (default/static in Java 8+) |
| Variables | Any | `public static final` only |
| Constructor | Yes | No |
| Multiple inheritance | No | Yes |
| Use when | IS-A + shared code | CAN-DO / capability |

---

## Access Modifiers

| Modifier | Class | Package | Subclass | World |
|---|---|---|---|---|
| `private` | ✓ | ✗ | ✗ | ✗ |
| `default` | ✓ | ✓ | ✗ | ✗ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `public` | ✓ | ✓ | ✓ | ✓ |

---

## SOLID in One Line

| | Principle | One Line |
|---|---|---|
| S | Single Responsibility | One class = one reason to change |
| O | Open/Closed | Extend with new code, don't modify existing |
| L | Liskov Substitution | Subclass must safely replace parent |
| I | Interface Segregation | Small specific interfaces > one fat interface |
| D | Dependency Inversion | Depend on abstractions, not concrete classes |

---

## Design Patterns Quick Reference

| Pattern | Type | One Line |
|---|---|---|
| Singleton | Creational | One instance only — private constructor |
| Factory | Creational | Factory decides which class to create |
| Builder | Creational | Build complex objects step by step |
| Decorator | Structural | Wrap object to add behaviour dynamically |
| Adapter | Structural | Bridge two incompatible interfaces |
| Observer | Behavioural | Notify all subscribers on state change |
| Strategy | Behavioural | Swap algorithms at runtime |

---

## Inheritance Types

```
Single:     A → B
Multilevel: A → B → C
Hierarchical: A → B, A → C
Multiple:   A + B → C  (only via interfaces in Java)
```

## Diamond Problem
Two parents have same method → child is ambiguous.
Java's fix: No multiple inheritance for classes. Interfaces with same default method → child must override explicitly.

## Key Keywords

| Keyword | Use |
|---|---|
| `this` | Current object — resolve name conflict, chain constructors |
| `super` | Parent class — call parent constructor/method |
| `final` | Variable: constant. Method: no override. Class: no extend |
| `static` | Belongs to class, not object — no `this` available |
| `abstract` | Method: no body. Class: cannot instantiate |
| `@Override` | Verify method actually overrides parent — compile-time check |
