# Tricky SOLID Interview Questions (depth vs shallow test)

These are the questions that trip up people who "know" SOLID at surface level. Each has the **shallow answer (the trap)** and the **deep answer** that shows real understanding.

---

## Q1. What does "one reason to change" in SRP actually mean? Isn't every class changed for many reasons?

**Shallow trap:** "A class should do only one thing / have one method."

**Deep answer:** SRP is **not** "do one thing" — it's **one reason to change = one actor/stakeholder**. Robert C. Martin's precise definition: *"A class should have only one reason to change, meaning it should be responsible to one, and only one, actor."*

Example: a `Payroll` class with `calculatePay()` (owned by Finance) and `reportHours()` (owned by HR). Both are "payroll," but **two different departments** can request changes → two reasons to change → SRP violation. Split them.

> Key insight: SRP is about **who asks for the change (people/actors)**, not about counting methods or "things." A class can have many methods and still satisfy SRP if they all serve one actor.

---

## Q2. OCP says "closed for modification" — but when I add a new class, I still have to modify the factory/main that creates it. Isn't that a contradiction?

**Shallow trap:** "You never modify any code" (impossible, and wrong).

**Deep answer:** OCP applies to the **core/business logic modules**, not to the composition root. The *policy* code (the part with the algorithm/rules) stays closed. Some **wiring code** (factory, DI config, main) will always change — that's expected and acceptable because it's trivial glue, not logic.

The goal: adding a feature shouldn't force you to **reopen and re-test the classes that contain behavior**. Pushing the `new SomeType()` decisions to the edges (factory/DI) is exactly how you keep the core closed. So it's not a contradiction — you *localize* the unavoidable change to a dumb wiring point.

---

## Q3. Can you achieve OCP WITHOUT inheritance?

**Shallow trap:** "No, OCP needs inheritance / abstract classes."

**Deep answer:** Yes. There are two flavors of OCP:
- **Meyer's original (1988):** via **inheritance** — extend a class to add behavior.
- **Polymorphic OCP (Martin):** via **interfaces + composition/strategy** — the modern, preferred way.

You can also get OCP via **composition, higher-order functions, plugins, configuration**. Inheritance is just one mechanism (and often the worst one). Saying "OCP = inheritance" reveals shallow knowledge.

---

## Q4. A `Square extends Rectangle` compiles fine and a square IS mathematically a rectangle. Why does it violate LSP?

**Shallow trap:** "It doesn't — a square is a rectangle."

**Deep answer:** LSP is about **behavioral substitutability, not real-world taxonomy**. Setting `Rectangle.width=5, height=4` should give area 20. A `Square` forces width==height, so `setHeight(4)` also changes width → area 16 → **the base class's contract is broken**. Client code relying on Rectangle behavior fails.

> The lesson: "is-a" in the **English/math sense ≠ "is-a" in the behavioral sense.** Inheritance must preserve behavior, not just categorization. This is the single most-cited LSP example precisely because the naive "is-a" reasoning fails.

---

## Q5. State the formal rules a subclass must obey for LSP (beyond "don't throw exceptions").

**Shallow trap:** "Just don't override methods to throw exceptions."

**Deep answer:** The formal design-by-contract rules:
1. **Preconditions cannot be strengthened** in a subtype. (Can't demand *more* than the parent — e.g. parent accepts any int, child rejecting negatives breaks callers.)
2. **Postconditions cannot be weakened.** (Must guarantee *at least* what the parent promised.)
3. **Invariants must be preserved.**
4. **History constraint** — a subtype must not allow state changes the base forbids (e.g. making a mutable subclass of an immutable base).

Also covariance of return types / contravariance of parameters. Naming these separates deep from shallow candidates.

---

## Q6. What's the difference between SRP and ISP? They both sound like "keep things small."

**Shallow trap:** "They're basically the same."

**Deep answer:**
- **SRP** is about **classes** — a class should have one reason to change (one actor).
- **ISP** is about **interfaces** — clients shouldn't be forced to depend on methods they don't use.

Different level, different victim: SRP protects the *class* from tangled responsibilities; ISP protects the *client/implementer* from a fat interface. You can violate ISP while each class has a single responsibility, and vice versa. ISP is essentially "SRP applied to interface consumers."

---

## Q7. (The big one) Is Dependency Inversion the same as Dependency Injection?

**Shallow trap:** "Yes, DIP = DI = passing dependencies in the constructor."

**Deep answer:** **No — different things.**
- **DIP (Principle):** high-level and low-level modules both depend on **abstractions**; details depend on abstractions.
- **DI (Technique/Pattern):** a way to *supply* a dependency (constructor/setter/interface injection).
- **IoC (Inversion of Control):** the broader concept where a framework controls flow/wiring.

You can do DI **without** DIP (inject a **concrete** class → you injected it but still depend on a concretion → DIP violated). You can achieve DIP without a DI framework. DI is *a tool* that *helps* achieve DIP; they are not equal. Confusing them is the #1 giveaway of shallow knowledge.

---

## Q8. I made my class depend on an interface. Does that automatically satisfy DIP?

**Shallow trap:** "Yes — using any interface = DIP."

**Deep answer:** Not necessarily. DIP has a subtle **ownership** requirement: the **abstraction should be owned/defined by the high-level (client) module, not by the low-level implementation.**

If your `UserService` depends on an interface that lives in and is dictated by the database package, the arrow of ownership still points from high-level → low-level. True inversion means the high-level module **declares the interface it needs** (e.g. `UserService` defines `UserRepository`), and the low-level module implements it. The dependency direction is *inverted* toward the policy, not the detail.

> Deep tell: "Who owns/defines the abstraction?" If it's the low-level module, you haven't really inverted anything.

---

## Q9. Does the Singleton pattern violate any SOLID principle?

**Shallow trap:** "No, Singleton is a standard GoF pattern, it's fine."

**Deep answer:** Singleton commonly violates **DIP** (and often **SRP**):
- Classes call `Singleton.getInstance()` internally → they depend on a **concrete** global, not an abstraction → DIP violation, and untestable (can't inject a mock).
- The singleton often mixes "manage my single instance" with its actual job → SRP smell.

Better: register a single instance in a DI container and inject it via an interface — you get "one instance" **without** the global-concrete dependency.

---

## Q10. Can code follow all 5 SOLID principles and STILL be bad design?

**Shallow trap:** "No — SOLID guarantees good design."

**Deep answer:** Yes. SOLID is a set of **guidelines**, not a guarantee. You can:
- **Over-apply** them → interface explosion, needless abstraction layers, "AbstractSingletonProxyFactoryBean" astronaut architecture.
- Satisfy every rule mechanically yet have poor **cohesion, naming, or wrong domain modeling**.
- Add abstractions for changes that **never happen** (violating YAGNI).

Deep candidates note the tension: SOLID must be balanced with **KISS, YAGNI, DRY** and actual, likely change. Blindly maximizing SOLID produces its own kind of bad code.

---

## Q11. Doesn't the Open/Closed Principle conflict with YAGNI ("You Aren't Gonna Need It")?

**Shallow trap:** unaware of the tension.

**Deep answer:** There *is* real tension. OCP tempts you to add abstraction points "in case" something changes; YAGNI says don't build for imaginary futures. The resolution: **apply OCP reactively at the axis of change you actually observe.** First time is fine to hardcode; when a *second* variant appears, refactor to an abstraction (the "Rule of Three"). Pre-emptively abstracting everything for OCP is over-engineering.

---

## Q12. Give an example where following LSP requires you to REJECT a natural-looking inheritance.

**Deep answer:** `Ostrich extends Bird` where `Bird.fly()` exists — ostriches can't fly, so the override throws → LSP broken. Or `ReadOnlyList extends List` where `add()` throws. The fix is to **not** inherit: split into `Bird` / `FlyingBird`, or `ReadableList` / `MutableList`. Recognizing that **the right answer is sometimes "don't use inheritance here"** is a strong depth signal.

---

## Quick "who's deep" cheat sheet

| Question | Shallow says | Deep says |
|---|---|---|
| SRP meaning | "do one thing" | "one reason to change = one actor" |
| OCP mechanism | "needs inheritance" | "interfaces/composition; inheritance is one option" |
| Square/Rectangle | "square is a rectangle" | "behavioral, not taxonomic, substitutability" |
| SRP vs ISP | "same thing" | "classes vs interface-clients" |
| DIP vs DI | "same thing" | "principle vs technique; DI≠DIP" |
| Interface = DIP? | "yes" | "only if high-level owns the abstraction" |
| Singleton | "fine" | "violates DIP/testability" |
| All SOLID = good? | "yes" | "guideline, can be over-applied; balance w/ YAGNI/KISS" |

## One-Line Summary
**Depth in SOLID shows up in the nuances: SRP is about actors not verbs, OCP isn't limited to inheritance, LSP is behavioral (not taxonomic) substitutability with formal contract rules, ISP≠SRP (interfaces vs classes), and DIP≠DI (the abstraction must be owned by the high-level module). Knowing SOLID can be over-applied — and balanced with YAGNI/KISS — is the final tell.**
