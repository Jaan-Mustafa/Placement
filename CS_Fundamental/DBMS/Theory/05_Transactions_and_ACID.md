# Transactions & ACID Properties

## 1. What is a Transaction?

A **transaction** is a sequence of one or more SQL operations executed as a **single logical unit of work** — either all operations succeed together, or none of them take effect.

### Example

Transfer ₹500 from Alice's account to Bob's account:

```sql
BEGIN TRANSACTION;
  UPDATE Account SET balance = balance - 500 WHERE name = 'Alice';
  UPDATE Account SET balance = balance + 500 WHERE name = 'Bob';
COMMIT;
```

Both updates must succeed together. If the second one fails after the first, Alice loses ₹500 and Bob gets nothing — the database is corrupt.

---

## 2. Transaction States

```
                        ┌─────────┐
                        │ ACTIVE  │  ← transaction is executing
                        └────┬────┘
                             │ all operations done
                        ┌────▼──────────┐
                        │ PARTIALLY     │  ← last operation executed,
                        │ COMMITTED     │    not yet written to disk
                        └────┬──────────┘
               success  ────▶│◀──── failure
              ┌──────────────┘└──────────────┐
         ┌────▼─────┐                  ┌─────▼─────┐
         │ COMMITTED│                  │  FAILED   │
         └──────────┘                  └─────┬─────┘
                                             │ rollback
                                       ┌─────▼─────┐
                                       │ ABORTED   │  ← rolled back, DB restored
                                       └───────────┘
```

| State | Meaning |
|---|---|
| **Active** | Transaction is currently executing |
| **Partially Committed** | Last statement executed, awaiting final write |
| **Committed** | All changes permanently saved to DB |
| **Failed** | Error occurred, cannot proceed |
| **Aborted** | Rolled back — DB restored to state before transaction |

---

## 3. ACID Properties

ACID is a set of 4 properties that guarantee a transaction is processed reliably.

| Property | One-line meaning |
|---|---|
| **Atomicity** | All or nothing |
| **Consistency** | DB moves from one valid state to another |
| **Isolation** | Concurrent transactions don't interfere |
| **Durability** | Committed changes survive crashes |

---

### A — Atomicity

**All operations in a transaction succeed, or none of them are applied.**

#### Example

```
Alice: ₹1000,  Bob: ₹500

BEGIN;
  UPDATE Account SET balance = balance - 500 WHERE name = 'Alice';  ← executes ✓
  UPDATE Account SET balance = balance + 500 WHERE name = 'Bob';    ← CRASH ✗
```

Without atomicity: Alice loses ₹500, Bob gets nothing.  
With atomicity: entire transaction is **rolled back** → Alice: ₹1000, Bob: ₹500 (original state).

**How it's implemented:** UNDO logs — on failure, reverse all completed steps.

---

### C — Consistency

**A transaction brings the database from one valid state to another valid state — no integrity rules are broken.**

#### Example

Rule: `Account balance must never go below ₹0`

```
Alice: ₹200

BEGIN;
  UPDATE Account SET balance = balance - 500 WHERE name = 'Alice';
  -- balance would become -300 → violates constraint
ROLLBACK;  ← DB rejects, rolls back
```

The DB enforces constraints (PK, FK, CHECK, NOT NULL) — a transaction that would break them is aborted.

**How it's implemented:** Constraint checking by the DB engine before commit.

---

### I — Isolation

**Concurrent transactions execute as if they were running one at a time — each transaction is unaware of others running simultaneously.**

#### The Problem Without Isolation

Two users read and update the same data at the same time:

```
Alice's account: ₹1000

Transaction T1 (Alice pays ₹300 to Bob)    Transaction T2 (Alice pays ₹400 to Carol)
─────────────────────────────────────────   ─────────────────────────────────────────
1. Read Alice balance → 1000
                                            2. Read Alice balance → 1000
3. Write Alice balance = 1000 - 300 = 700
                                            4. Write Alice balance = 1000 - 400 = 600
```

**Final balance: ₹600** — but it should be ₹300 (1000 - 300 - 400).  
T1's update was **overwritten** by T2 — this is called a **Lost Update** problem.

#### With Isolation

T2 must wait until T1 finishes before it can read Alice's balance:

```
T1: Read(1000) → Write(700) → COMMIT
T2:                                    Read(700) → Write(300) → COMMIT
```

**Final balance: ₹300** ✓ — correct result.

#### Isolation is NOT binary — it has levels

Full isolation (Serializable) is expensive. So databases offer trade-off levels:

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| **Read Uncommitted** | ✓ possible | ✓ possible | ✓ possible |
| **Read Committed** | ✗ prevented | ✓ possible | ✓ possible |
| **Repeatable Read** | ✗ prevented | ✗ prevented | ✓ possible |
| **Serializable** | ✗ prevented | ✗ prevented | ✗ prevented |

> These problems (Dirty Read, Non-Repeatable Read, Phantom Read) are explained in detail in [Concurrency Control](06_Concurrency_Control.md).

**How it's implemented:** Locks, MVCC (Multi-Version Concurrency Control), timestamps.

---

### D — Durability

**Once a transaction is committed, its changes are permanent — even if the system crashes immediately after.**

#### Example

```
BEGIN;
  UPDATE Account SET balance = 700 WHERE name = 'Alice';
COMMIT;  ← system crashes 1ms after this
```

On restart, Alice's balance is still ₹700 — the committed change survived the crash.

**How it's implemented:** Write-Ahead Logging (WAL) — changes are written to a log on disk before being applied. On recovery, the log is replayed.

---

## 4. ACID — Full Example Together

```
Alice: ₹1000, Bob: ₹500
Rule: balance >= 0

BEGIN TRANSACTION;
  UPDATE Account SET balance = balance - 300 WHERE name = 'Alice';
  UPDATE Account SET balance = balance + 300 WHERE name = 'Bob';
COMMIT;
```

| Property | What it guarantees here |
|---|---|
| **Atomicity** | Both updates happen, or neither does |
| **Consistency** | Balance never goes below 0; total money (₹1500) stays the same |
| **Isolation** | Another transaction reading Alice's balance won't see the intermediate state (deducted but not yet added to Bob) |
| **Durability** | After COMMIT, even a crash won't lose the transfer |

---

## 5. Interview Questions

1. **What is a transaction in DBMS?**
   → A sequence of operations executed as a single logical unit — all succeed or all are rolled back.

2. **What are ACID properties?**
   → Atomicity (all or nothing), Consistency (valid state to valid state), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes).

3. **What is the difference between Atomicity and Durability?**
   → Atomicity ensures a transaction fully completes or fully rolls back. Durability ensures that once committed, changes persist even after a crash.

4. **Why is Isolation important?**
   → Without isolation, concurrent transactions can interfere — causing lost updates, dirty reads, or inconsistent data.

5. **What is a Lost Update problem?**
   → Two transactions read the same value, both modify it, and one overwrites the other's change — net result is wrong.

6. **How is Atomicity implemented?**
   → Using UNDO logs — if a transaction fails mid-way, all completed steps are reversed.

7. **How is Durability implemented?**
   → Using Write-Ahead Logging (WAL) — changes are logged to disk before being applied, so they can be replayed after a crash.

8. **What is the difference between Consistency in ACID and Consistency in CAP theorem?**
   → ACID Consistency means DB constraints are never violated. CAP Consistency means all nodes in a distributed system see the same data at the same time — they are different concepts.

9. **Can a transaction be Atomic but not Durable?**
   → Yes — in an in-memory DB with no persistence. The transaction completes fully (atomic), but a crash erases everything (not durable).

10. **What happens after an Aborted transaction?**
    → The DB is rolled back to its exact state before the transaction started — as if the transaction never happened.
