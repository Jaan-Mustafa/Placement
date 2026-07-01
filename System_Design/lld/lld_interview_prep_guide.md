# LLD Interview Preparation Guide (SDE-1 / SDE-2 / any level)

Low-Level Design (LLD) = designing the **classes, objects, methods, and their relationships** for a specific feature or module. It's about writing clean, extensible, object-oriented code — not distributed systems (that's HLD).

## LLD vs HLD (know the difference)

| | HLD (High-Level Design) | LLD (Low-Level Design) |
|---|---|---|
| **Scope** | Whole system architecture | One module/feature in depth |
| **Focus** | Scalability, DBs, caching, queues, load balancers | Classes, objects, methods, relationships |
| **Output** | Boxes-and-arrows architecture diagram | Class diagram + working code |
| **Example** | "Design Instagram" (feed, storage, CDN) | "Design the notification module / a parking lot" |
| **Skills** | Distributed systems, trade-offs | OOP, SOLID, design patterns |
| **Level emphasis** | More at senior/staff | Heavy at SDE-1 & SDE-2 |

> At **SDE-2**, LLD is often the **make-or-break round**. They want to see you write extensible, maintainable OOP code and reason about future changes.

---

## What Interviewers Actually Evaluate

1. **Requirement clarification** — do you ask questions before coding?
2. **Object modeling** — can you identify the right classes/entities and their relationships?
3. **SOLID principles** — is your design extensible and loosely coupled?
4. **Design patterns** — do you apply the right pattern (not over-engineer)?
5. **Clean code** — naming, encapsulation, no giant God classes.
6. **Extensibility** — "now add feature X" → does your design absorb it gracefully?
7. **Working code** — many companies want compilable, runnable code (not just diagrams).

---

## Foundation 1: OOP Fundamentals (must be rock-solid)

| Concept | Meaning | LLD use |
|---|---|---|
| **Encapsulation** | Hide internal state, expose via methods | Private fields + getters/behavior methods |
| **Abstraction** | Expose *what*, hide *how* | Interfaces / abstract classes |
| **Inheritance** | "is-a" relationship | Base class → subclasses |
| **Polymorphism** | Same interface, different behavior | Override methods; strategy-like dispatch |

Also know: **composition over inheritance** ("has-a" often beats "is-a"), and the relationships:
- **Association** — uses-a (A uses B)
- **Aggregation** — has-a, independent lifetime (Team has Players)
- **Composition** — has-a, dependent lifetime (House has Rooms; rooms die with house)

---

## Foundation 2: SOLID Principles (the backbone of LLD)

| Letter | Principle | One-liner |
|---|---|---|
| **S** | Single Responsibility | A class should have one reason to change |
| **O** | Open/Closed | Open for extension, closed for modification |
| **L** | Liskov Substitution | Subclass must be usable wherever base is |
| **I** | Interface Segregation | Many small interfaces > one fat interface |
| **D** | Dependency Inversion | Depend on abstractions, not concrete classes |

> Interviewers love asking "which SOLID principle does this violate?" — be ready to name and fix violations.

---

## Foundation 3: Design Patterns (know when, not just what)

Group them and learn 2-3 deeply per group. **Creational / Structural / Behavioral:**

### Creational (how objects are made)
- **Singleton** — one instance (config, logger). *Caveat: overused; know thread-safety.*
- **Factory / Factory Method** — create objects without exposing instantiation logic.
- **Abstract Factory** — factory of factories (families of related objects).
- **Builder** — build complex objects step by step (e.g. `Pizza.Builder()`).
- **Prototype** — clone existing objects.

### Structural (how objects are composed)
- **Adapter** — make incompatible interfaces work together.
- **Decorator** — add behavior dynamically (e.g. coffee + milk + sugar).
- **Facade** — simple interface over a complex subsystem.
- **Composite** — tree structures (files/folders).
- **Proxy** — placeholder/control access (lazy load, access control).

### Behavioral (how objects interact)
- **Strategy** — swap algorithms at runtime (payment methods, sorting). ⭐ most-asked.
- **Observer** — pub/sub, notify subscribers on change (notifications). ⭐ most-asked.
- **State** — behavior changes with state (vending machine, order status).
- **Command** — encapsulate a request as an object (undo/redo).
- **Chain of Responsibility** — pass request along a chain (middleware, approval flow).
- **Template Method** — skeleton in base, steps in subclasses.

> ⭐ **Most frequently used in interviews:** Strategy, Observer, Factory, Singleton, State, Decorator, Builder. Master these first.

---

## Foundation 4: UML Class Diagrams (communicate the design)

Know how to draw:
- **Class box**: name / attributes / methods.
- **Relationships**: inheritance (▷), composition (◆ filled), aggregation (◇ hollow), association (→).
- **Multiplicity**: 1, *, 1..*.

You don't need perfect UML — a clear class diagram on the whiteboard/editor is enough.

---

## The LLD Interview Framework (step-by-step approach)

Follow this every time — structure impresses interviewers:

```
1. CLARIFY REQUIREMENTS (2-3 min)
   - Ask about scope, features, scale, edge cases.
   - Nail down functional requirements. Confirm what's IN vs OUT of scope.

2. IDENTIFY CORE ENTITIES / OBJECTS (nouns)
   - Extract nouns from requirements → candidate classes.
   - e.g. Parking Lot → ParkingLot, Floor, Slot, Vehicle, Ticket, Payment

3. DEFINE RELATIONSHIPS & ATTRIBUTES
   - How do entities relate (has-a, is-a)? What fields does each hold?

4. DEFINE BEHAVIORS (methods / verbs)
   - What actions? park(), unpark(), calculateFee(), assignSlot()

5. APPLY SOLID + PATTERNS
   - Where will change happen? Put an abstraction/pattern there.
   - e.g. multiple fee strategies → Strategy pattern.

6. WRITE THE CODE
   - Interfaces first, then classes. Clean naming, encapsulation.

7. HANDLE EDGE CASES + EXTENSIBILITY
   - "What if we add electric-vehicle slots / dynamic pricing?"
   - Show your design absorbs it with minimal change (Open/Closed).
```

> Golden rule: **think about what will change in the future, and isolate it behind an interface.** That's the essence of good LLD.

---

## Must-Practice LLD Problems (by frequency)

### Tier 1 — very commonly asked (do these first)
- **Parking Lot** (the classic; teaches entities, strategy, state)
- **Elevator / Lift system** (state machine, scheduling)
- **Vending Machine** (State pattern)
- **Design a Logger** (Singleton, Chain of Responsibility)
- **Tic-Tac-Toe / Chess** (board games — OOP modeling)
- **Splitwise** (expense splitting — relationships, strategies)

### Tier 2 — frequently asked
- **BookMyShow / Movie ticket booking** (concurrency, seat locking)
- **Rate Limiter** (Strategy — you already have HLD notes on this)
- **Notification system** (Observer, Strategy for channels)
- **Cache (LRU/LFU)** (data structure design — HashMap + DLL)
- **Snake and Ladder** (board game)
- **ATM machine** (State pattern)

### Tier 3 — asked at SDE-2+
- **Food delivery (Swiggy/Zomato)**
- **Ride hailing (Uber)** matching logic
- **Online food ordering / cart & order system**
- **Library management system**
- **Meeting scheduler / Calendar**
- **Stock exchange / order matching**

> For each: model entities → apply patterns → write code → then answer "add feature X."

---

## Preparation Roadmap

```
Week 1-2: Foundations
   - Solidify OOP (encapsulation, composition vs inheritance, relationships)
   - Master SOLID (be able to spot + fix violations)

Week 2-4: Design Patterns
   - Learn the ⭐ core 7 deeply (Strategy, Observer, Factory, Singleton,
     State, Decorator, Builder) with a small code example each.

Week 4-6: Problem practice
   - Do Tier 1 problems end-to-end (entities → diagram → code → extend).
   - Write actual runnable code in your language (Java/C++/Python).

Week 6+: Mock + refine
   - Time-box (35-45 min) full problems.
   - Practice explaining out loud + drawing class diagrams.
   - Do Tier 2/3 problems.
```

---

## Practical Tips

- **Always clarify first.** Jumping into code without questions is the #1 red flag.
- **Start simple, then extend.** Get a working core model, then add patterns where needed. Don't over-engineer upfront.
- **Verbalize trade-offs.** "I'll use Strategy here so we can add payment methods without touching existing code."
- **Encapsulate what varies.** The line that scores points.
- **Write real code** if allowed — interfaces, classes, a small `main()` demo showing it works.
- **Don't force patterns.** Using a pattern where it's not needed is as bad as missing one.
- **Manage time.** ~5 min clarify, ~10 min model, ~20 min code, ~5 min edge cases/extend.
- **Pick one language and stick to it** (Java is most common for LLD; C++/Python fine too).

---

## Quick Checklist Before the Round

- [ ] Can I explain all 5 SOLID with an example + a fix?
- [ ] Can I code the core 7 patterns from memory?
- [ ] Can I do Parking Lot / Elevator / Splitwise end-to-end in 40 min?
- [ ] Do I draw a class diagram before coding?
- [ ] Do I always ask clarifying questions first?
- [ ] Can I answer "now add feature X" without rewriting everything?

## One-Line Summary
**LLD interviews test clean, extensible OOP: clarify requirements → model entities & relationships → apply SOLID and the right design pattern (isolating what will change) → write working code → extend gracefully. Master OOP + SOLID + the core 7 patterns, then practice Tier-1 problems (Parking Lot, Elevator, Splitwise) end-to-end.**
