# OOPS Interview Questions

## Four Pillars

1. **What are the four pillars of OOP?**
   → Encapsulation (bundle data + methods, hide internals), Abstraction (hide implementation, show interface), Inheritance (child reuses parent code), Polymorphism (same name, different behaviour).

2. **Explain encapsulation with a real-world example.**
   → A bank account — balance is private, exposed only through deposit/withdraw methods. Direct modification (`account.balance = -5000`) is blocked; validation happens inside the methods.

3. **What is the difference between encapsulation and abstraction?**
   → Encapsulation hides **data** (using private + getters/setters). Abstraction hides **implementation** (using abstract class/interface — shows what to do, not how).

4. **What is polymorphism? Give a real-world example.**
   → Same action, different behaviour. A person IS-A employee at work, IS-A parent at home, IS-A customer at a shop — same person, different roles/behaviour.

---

## Classes, Objects, Constructors

5. **What is the difference between a class and an object?**
   → Class is a blueprint — no memory allocated. Object is an instance — memory allocated on heap.

6. **Can a constructor be private? What is the use?**
   → Yes. Used in Singleton pattern — prevents external instantiation, forces use of `getInstance()`.

7. **What is constructor chaining?**
   → Calling one constructor from another using `this()` (same class) or `super()` (parent class). Must be the first statement.

8. **What is the difference between shallow copy and deep copy?**
   → Shallow copy: copies references — both objects point to the same nested objects. Deep copy: copies everything recursively — completely independent objects.

---

## Inheritance

9. **Why doesn't Java support multiple inheritance with classes?**
   → Diamond problem — if two parents have the same method, the child is ambiguous. Java allows multiple inheritance only through interfaces.

10. **What is the difference between method overriding and method hiding?**
    → Overriding: instance method in child redefines parent's instance method — resolved at runtime (dynamic). Hiding: child defines a static method same as parent's static method — resolved at compile time (static).

11. **Can you call a parent class method that has been overridden?**
    → Yes. Using `super.methodName()` inside the child class.

12. **What is the IS-A vs HAS-A relationship?**
    → IS-A is inheritance (`Dog IS-A Animal`). HAS-A is composition (`Car HAS-A Engine`). Prefer HAS-A when there's no true hierarchical relationship — more flexible, less coupled.

---

## Polymorphism

13. **What is Dynamic Method Dispatch?**
    → JVM mechanism that resolves an overridden method at runtime based on actual object type, not reference type.

14. **Can you overload a method by changing only the return type?**
    → No. The compiler uses the parameter list to resolve overloaded calls — return type alone is not sufficient.

15. **What is the difference between early binding and late binding?**
    → Early (static) binding: method resolved at compile time (overloading, static methods). Late (dynamic) binding: method resolved at runtime (overriding via polymorphism).

---

## Abstract Class & Interface

16. **Can an abstract class have a constructor? What is its use?**
    → Yes. Called via `super()` from the child constructor — used to initialise common fields defined in the abstract class.

17. **What is a marker interface?**
    → An empty interface with no methods — marks a class for special JVM/framework treatment. Examples: `Serializable`, `Cloneable`.

18. **Can an interface have instance variables?**
    → No. All interface variables are implicitly `public static final` — constants only.

19. **What is the difference between abstract class and interface in Java 8+?**
    → Java 8 added `default` and `static` methods to interfaces. The remaining key differences: abstract class can have constructors and instance variables; a class can implement multiple interfaces but extend only one abstract class.

---

## Static & Final

20. **Why is the `main` method static in Java?**
    → JVM calls it before any object is created. Being static allows it to be called directly via the class name without instantiation.

21. **What is the difference between `final`, `finally`, and `finalize`?**
    → `final`: keyword — prevents change/override/extension. `finally`: block in try-catch — always executes. `finalize()`: method called by GC before object is destroyed (deprecated in Java 9+).

22. **Can a static method be overridden?**
    → No. Static methods are bound to the class at compile time. A child can define the same static method, but it's method hiding — no runtime polymorphism.

---

## SOLID Principles

23. **What is the Single Responsibility Principle? Give an example of a violation.**
    → A class should have one reason to change. Violation: an `Employee` class that handles salary calculation, DB persistence, and report generation — three separate responsibilities in one class.

24. **What is the Open/Closed Principle?**
    → Open for extension, closed for modification. Add new behaviour through new classes, not by editing existing ones. Achieved with polymorphism and interfaces.

25. **What is the Liskov Substitution Principle? Give an example of a violation.**
    → Subclass must be substitutable for parent without breaking behaviour. Violation: `Penguin extends Bird` where `Bird.fly()` throws an exception — breaks any code that treats a Bird and calls `fly()`.

---

## Design Patterns

26. **What is the Singleton pattern? How is it implemented thread-safely?**
    → One instance throughout the app. Thread-safe: double-checked locking with `volatile` field and `synchronized` block.

27. **What is the Factory pattern? What problem does it solve?**
    → Delegates object creation to a factory class — decouples the caller from knowing which concrete class to instantiate.

28. **What is the difference between Factory and Abstract Factory?**
    → Factory creates one type of object. Abstract Factory creates families of related objects — a factory of factories.

29. **What is the difference between Decorator and Inheritance for adding behaviour?**
    → Inheritance is static — decided at compile time. Decorator is dynamic — behaviour added at runtime by wrapping. Decorator avoids class explosion from combining multiple features.

30. **What is Dependency Injection? How does it relate to DIP?**
    → DIP (Dependency Inversion Principle) says depend on abstractions. DI is the technique: inject the concrete dependency from outside rather than creating it inside — makes code testable and swappable.

---

## Miscellaneous

31. **What is the difference between composition and inheritance? Which is preferred?**
    → Inheritance: IS-A (child is a type of parent). Composition: HAS-A (class contains another class). Composition is generally preferred — more flexible, less coupled, avoids deep inheritance chains.

32. **What is coupling and cohesion?**
    → Coupling: degree of dependency between classes — lower is better. Cohesion: degree to which elements of a class belong together — higher is better. Goal: low coupling, high cohesion.

33. **What is an anonymous class?**
    → A class defined and instantiated in one expression — no class name. Used for one-time implementations of interfaces or abstract classes.
    ```java
    Runnable r = new Runnable() {
        public void run() { System.out.println("Running"); }
    };
    ```

34. **What is the difference between aggregation and composition?**
    → Aggregation: weak HAS-A — child can exist independently (Department HAS-A Employee; Employee can exist without Department). Composition: strong HAS-A — child cannot exist without parent (House HAS-A Room; Room cannot exist without House).
