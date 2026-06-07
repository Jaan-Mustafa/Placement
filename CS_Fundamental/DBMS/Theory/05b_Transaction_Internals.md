# Transaction Internals — How Atomicity & Durability Work

## 1. The Core Problem

When a transaction runs, changes happen in **RAM (buffer pool)** first — not directly on disk. Disk writes are slow, so the DB batches them. This creates two problems:

```
Problem 1 — Atomicity:
  Transaction half-completes → system crashes → disk has partial changes
  How do we undo the incomplete work?

Problem 2 — Durability:
  Transaction commits → system crashes before RAM is flushed to disk
  How do we ensure the committed data is not lost?
```

Both are solved by the same mechanism: **Write-Ahead Logging (WAL)**.

---

## 2. Key Components

```
┌─────────────────────────────────────────────────────────────┐
│                        RAM                                  │
│  ┌─────────────────────┐    ┌──────────────────────────┐   │
│  │    Buffer Pool      │    │      Log Buffer          │   │
│  │  (dirty pages of    │    │  (log records waiting    │   │
│  │   actual DB data)   │    │   to be flushed)         │   │
│  └──────────┬──────────┘    └────────────┬─────────────┘   │
└─────────────┼───────────────────────────┼─────────────────-┘
              │ flush (later)             │ flush (first — WAL rule)
              ▼                           ▼
┌─────────────────────┐    ┌──────────────────────────────┐
│   Data Files        │    │        WAL Log File          │
│  (heap, indexes)    │    │  (append-only, sequential)   │
└─────────────────────┘    └──────────────────────────────┘
              DISK
```

| Component | What it holds |
|---|---|
| **Buffer Pool** | Pages of actual data (tables, indexes) cached in RAM |
| **Dirty Page** | A buffer pool page that has been modified but not yet written to disk |
| **Log Buffer** | Log records in RAM waiting to be flushed to the WAL file |
| **WAL Log File** | Append-only file on disk recording every change before it happens |
| **Data File** | The actual database files on disk (heap files, index files) |

---

## 3. Write-Ahead Logging (WAL)

### The Golden Rule of WAL

> **The log record for a change must reach disk BEFORE the actual data page reaches disk.**

This single rule is what makes both atomicity and durability possible.

### What a Log Record Contains

Each log record has:

```
┌──────────┬──────────┬──────────┬──────────────┬──────────────┐
│  LSN     │  TxnID   │  Type    │  Before      │  After       │
│ (Log     │          │ (UPDATE/ │  Image       │  Image       │
│ Sequence │          │  INSERT/ │  (old value) │  (new value) │
│ Number)  │          │  DELETE) │              │              │
└──────────┴──────────┴──────────┴──────────────┴──────────────┘
```

| Field | Purpose |
|---|---|
| **LSN** | Unique sequential number — orders all log records globally |
| **TxnID** | Which transaction made this change |
| **Type** | Operation type — UPDATE, INSERT, DELETE, COMMIT, ABORT |
| **Before Image** | Value before the change — used for UNDO (atomicity) |
| **After Image** | Value after the change — used for REDO (durability) |

### Example Log Sequence

Transfer ₹300 from Alice to Bob:

```
LSN  TxnID  Type     Page   Before   After
───────────────────────────────────────────────
001  T1     BEGIN
002  T1     UPDATE   P1     1000     700      ← Alice: 1000 → 700
003  T1     UPDATE   P2     500      800      ← Bob:    500 → 800
004  T1     COMMIT
```

---

## 4. How Atomicity is Maintained — UNDO Log

### Scenario: Crash mid-transaction

```
LSN 001  T1  BEGIN
LSN 002  T1  UPDATE  Alice: 1000 → 700   ← flushed to data file on disk
LSN 003  T1  UPDATE  Bob:    500 → 800
              ↑
           CRASH HERE — T1 never committed
```

On restart, the DB scans the log and finds T1 has a BEGIN but no COMMIT → T1 must be **undone**.

### UNDO Process (Rolling Back)

The DB reads the log **backwards** from the crash point:

```
Step 1: Find LSN 003 → T1 UPDATE Bob   → undo: set Bob back to 500
Step 2: Find LSN 002 → T1 UPDATE Alice → undo: set Alice back to 1000
Step 3: Find LSN 001 → T1 BEGIN        → write ABORT record, stop
```

Result: Database restored to exact state before T1 started.

### UNDO Log Rule

> Before a dirty page is flushed to disk, its **UNDO log records** must already be on disk.

This ensures that if the dirty page reaches disk but the transaction later aborts, we can always reverse the change.

```
Correct order:
  1. Write UNDO log record to disk
  2. Flush dirty data page to disk
  3. (Transaction can now safely abort and be undone)
```

---

## 5. How Durability is Maintained — REDO Log

### Scenario: Crash after COMMIT but before data flush

```
LSN 001  T1  BEGIN
LSN 002  T1  UPDATE  Alice: 1000 → 700
LSN 003  T1  UPDATE  Bob:    500 → 800
LSN 004  T1  COMMIT  ← log flushed to disk ✓
              ↑
           CRASH HERE — dirty pages (Alice=700, Bob=800) still in RAM, not on disk
```

On restart, the data files still show Alice=1000, Bob=500 — the committed changes are lost from disk.

### REDO Process (Replaying)

The DB scans the log **forward** from the last checkpoint:

```
Step 1: Find LSN 002 → T1 UPDATE Alice 1000 → 700 → re-apply to data file
Step 2: Find LSN 003 → T1 UPDATE Bob    500 → 800 → re-apply to data file
Step 3: Find LSN 004 → T1 COMMIT → mark T1 as done
```

Result: Committed changes are reapplied — durability preserved.

### REDO Log Rule (The WAL Rule)

> The **COMMIT log record** must be flushed to disk **before** telling the client that the transaction committed.

```
Correct order:
  1. Write all log records to log buffer
  2. Flush log buffer to WAL file on disk  ← COMMIT log must hit disk
  3. Tell client "COMMIT successful"
  4. (Data pages can be flushed lazily later)
```

If the system crashes between step 2 and step 4 — on recovery, the log shows COMMIT, so REDO re-applies the changes. The client never got a success response, but data is consistent.

---

## 6. Checkpoint — Limiting Recovery Work

Without checkpoints, crash recovery would need to replay the **entire log history** — which could be gigabytes.

### What a Checkpoint Does

At regular intervals, the DB:
1. Forces all dirty pages in the buffer pool to disk
2. Writes a `CHECKPOINT` record to the log

```
... LSN 100  CHECKPOINT  ← all dirty pages flushed to disk up to this point
... LSN 101  T2  BEGIN
... LSN 102  T2  UPDATE
... LSN 103  T2  COMMIT
... LSN 104  T3  BEGIN
... LSN 105  T3  UPDATE
             ↑ CRASH
```

On recovery, the DB only needs to replay from **LSN 100** — everything before the checkpoint is already safely on disk.

### Fuzzy Checkpoint (used by most real DBs)

A strict checkpoint stalls all transactions while dirty pages flush — too slow. A **fuzzy checkpoint** marks the start and end of flushing without stopping transactions:

```
LSN 100  BEGIN_CHECKPOINT  (start flushing dirty pages)
  ... transactions continue running ...
LSN 150  END_CHECKPOINT    (all dirty pages at LSN 100 are now on disk)
```

Recovery starts from LSN 100, not the beginning of the entire log.

---

## 7. ARIES Recovery Algorithm

Most modern databases (PostgreSQL, MySQL InnoDB) use **ARIES** (Algorithm for Recovery and Isolation Exploiting Semantics).

### Three Phases of ARIES Recovery

```
Crash happens
     ↓
┌────────────────────────────────────────────────────────┐
│  Phase 1: ANALYSIS                                     │
│  Scan log forward from last checkpoint                 │
│  → Find which transactions were active at crash        │
│  → Find which pages were dirty (not flushed)           │
│  Output: Winner list (committed), Loser list (active)  │
└────────────────────────┬───────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────────┐
│  Phase 2: REDO                                         │
│  Scan log forward, re-apply ALL changes                │
│  (even from uncommitted transactions)                  │
│  → Restores DB to exact state at crash moment          │
└────────────────────────┬───────────────────────────────┘
                         ↓
┌────────────────────────────────────────────────────────┐
│  Phase 3: UNDO                                         │
│  Scan log backward, undo changes from LOSER txns       │
│  → Rolls back all transactions that never committed    │
└────────────────────────────────────────────────────────┘
```

| Phase | Direction | Purpose |
|---|---|---|
| **Analysis** | Forward from checkpoint | Identify winners and losers |
| **REDO** | Forward from checkpoint | Restore crash-time state |
| **UNDO** | Backward from crash | Roll back uncommitted transactions |

### Why REDO even uncommitted changes first?

Because a dirty page from an uncommitted transaction may have been flushed to disk before the crash. REDO brings the DB to the exact crash state, then UNDO cleanly reverses the losers — this is simpler and more correct than trying to skip uncommitted changes during REDO.

---

## 8. Full Picture — What Happens During a Transaction

```
BEGIN TRANSACTION T1
       ↓
  Operation executes (e.g., UPDATE Alice balance)
       ↓
  ┌─────────────────────────────────────┐
  │ 1. Write log record to Log Buffer   │  (RAM)
  │    {LSN, T1, UPDATE, before, after} │
  └─────────────┬───────────────────────┘
                ↓
  ┌─────────────────────────────────────┐
  │ 2. Apply change to Buffer Pool page │  (RAM)
  │    (page marked dirty)              │
  └─────────────┬───────────────────────┘
                ↓
             (more operations...)
                ↓
  COMMIT called
       ↓
  ┌─────────────────────────────────────┐
  │ 3. Flush Log Buffer → WAL file      │  (DISK) ← must happen before ACK
  │    (includes COMMIT log record)     │
  └─────────────┬───────────────────────┘
                ↓
  ┌─────────────────────────────────────┐
  │ 4. ACK client: "Transaction done"   │
  └─────────────┬───────────────────────┘
                ↓
  ┌─────────────────────────────────────┐
  │ 5. Dirty pages flushed lazily       │  (DISK) ← happens later, async
  │    by background writer             │
  └─────────────────────────────────────┘
```

---

## 9. Summary

| Goal | Mechanism | How |
|---|---|---|
| **Atomicity** | UNDO Log | Before image stored in log; on crash, scan log backward and reverse incomplete transactions |
| **Durability** | REDO Log + WAL rule | After image stored in log; COMMIT log flushed to disk before ACK; on crash, replay forward |
| **Both** | WAL (Write-Ahead Logging) | Log always written to disk before the data page — enables both undo and redo |
| **Fast Recovery** | Checkpoints | Periodically flush dirty pages; recovery only replays log from last checkpoint |
| **Complete Recovery** | ARIES | Three-phase: Analysis → REDO → UNDO |

---

## 10. Interview Questions

1. **What is WAL and why is it needed?**
   → Write-Ahead Logging — every change is logged to disk before the data page is modified on disk. Needed to support both atomicity (undo) and durability (redo) after crashes.

2. **What is the difference between UNDO and REDO logs?**
   → UNDO log stores the before-image — used to reverse incomplete transactions. REDO log stores the after-image — used to reapply committed changes lost from RAM on crash.

3. **What is a dirty page?**
   → A buffer pool page that has been modified in RAM but not yet written to disk.

4. **Why must the COMMIT log record reach disk before the client gets an ACK?**
   → Because if the system crashes between commit and disk flush, the log lets us REDO the changes. Without the log on disk, committed data would be permanently lost.

5. **What is a checkpoint and why is it used?**
   → A point where all dirty pages are flushed to disk and a marker is written in the log. On recovery, replay only starts from the checkpoint — avoids replaying the entire log history.

6. **What are the three phases of ARIES?**
   → Analysis (find winners/losers), REDO (replay all changes to reach crash state), UNDO (roll back uncommitted transactions).

7. **Why does ARIES REDO even uncommitted transactions before undoing them?**
   → Uncommitted dirty pages may have already reached disk. REDO brings the DB to the exact crash state cleanly, then UNDO reverses the losers — simpler and more correct than trying to skip them.

8. **What happens if a crash occurs during the UNDO phase of recovery?**
   → ARIES handles this — it uses Compensation Log Records (CLRs) during UNDO, so if a crash happens mid-UNDO, recovery can safely restart and continue undoing from where it left off.
