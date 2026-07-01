# S — Single Responsibility Principle (SRP)

> **A class should have only ONE reason to change** — i.e. it should do only one job / have one responsibility.

## The Idea
If a class handles multiple concerns, a change in one concern can break the others, and the class becomes hard to test and maintain. Split responsibilities into separate classes.

- "One reason to change" = one actor/concern owns the class.
- Symptoms of violation: a class that does business logic **and** DB access **and** printing/formatting.

---

## ❌ Violation (one class doing too much)

```java
class Employee {
    private String name;
    private double salary;

    // Responsibility 1: business logic
    public double calculateTax() {
        return salary * 0.2;
    }

    // Responsibility 2: database access
    public void saveToDatabase() {
        // open connection, write SQL...
    }

    // Responsibility 3: reporting / formatting
    public void generateReport() {
        System.out.println("Report for " + name);
    }
}
```

**Why it's bad:** this class has **three reasons to change** —
1. tax rules change → edit here
2. database changes (SQL → NoSQL) → edit here
3. report format changes → edit here

Three unrelated concerns tangled in one class. A DB change risks breaking tax logic.

---

## ✅ Fix (split by responsibility)

```java
// Just the data + core business rule
class Employee {
    private String name;
    private double salary;

    public double getSalary() { return salary; }
    public String getName()   { return name; }
}

// Responsibility 1: tax calculation
class TaxCalculator {
    public double calculateTax(Employee e) {
        return e.getSalary() * 0.2;
    }
}

// Responsibility 2: persistence
class EmployeeRepository {
    public void save(Employee e) {
        // DB logic only
    }
}

// Responsibility 3: reporting
class EmployeeReport {
    public void generate(Employee e) {
        System.out.println("Report for " + e.getName());
    }
}
```

**Now each class has exactly one reason to change:**
- Tax rule changes → only `TaxCalculator`.
- DB switch → only `EmployeeRepository`.
- Report format → only `EmployeeReport`.

---

## How to Spot SRP Violations
- The word **"and"** in a class description ("handles orders **and** sends emails").
- A class importing unrelated libraries (SQL + PDF + HTTP).
- Giant "God classes" / "Manager" / "Utils" doing everything.

## Benefits
- Easier to **test** (test one concern in isolation).
- Easier to **maintain** (changes are localized).
- Better **reusability** (TaxCalculator can be reused elsewhere).

## One-Line Summary
**SRP: one class = one job = one reason to change. Split unrelated responsibilities (data, persistence, formatting, business logic) into separate classes so a change in one never risks breaking another.**
