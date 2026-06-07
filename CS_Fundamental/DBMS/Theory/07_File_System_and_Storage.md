# File System & Storage in DBMS

## 1. Atomicity Problem in File Systems

### What is Atomicity?

**Atomicity** means an operation either completes fully or not at all — no partial state is ever visible or persisted.

### The Core Problem

File systems and databases often need to perform **multi-step operations** that must succeed or fail as a unit. If a crash or failure happens mid-way, you get **partial writes** — a corrupted, inconsistent state.

### Classic Example: Database Write

Suppose a DB writes a record that spans two disk blocks:

```
Step 1: Write block A  ✓
Step 2: Write block B  ← CRASH HERE
```

Now block A has new data, block B has old data. The record is **torn** — neither old nor new. The database is corrupted.

### Specific Scenarios

| Problem | Description |
|---|---|
| **Torn write** | A single logical write spans multiple physical sectors; crash mid-write leaves mixed old/new data |
| **Lost update** | Two concurrent writers; one overwrites the other's partial write |
| **Metadata inconsistency** | File size updated but data blocks not written yet (or vice versa) |
| **Directory corruption** | File created in directory but inode not initialized |

### Why File Systems Don't Solve This

The OS `write()` syscall is **not atomic** for large writes — the kernel may flush partial data to disk at any point. `fsync()` ensures durability but not atomicity across multiple calls.

---

## 2. How Databases Solve the Atomicity Problem

### 2.1 Write-Ahead Logging (WAL)

- Log the **intended change before applying it** to actual data
- On crash: replay committed transactions, rollback uncommitted ones
- Used by: PostgreSQL, SQLite

```
[LOG: BEGIN TX-101]
[LOG: UPDATE row X from A to B]
[LOG: COMMIT TX-101]
         ↓
[Apply change to actual data file]
```

If crash happens before COMMIT log entry → rollback (undo the partial write).  
If crash happens after COMMIT → redo (re-apply from log).

### 2.2 Shadow Paging

- Write new version to a **new page**
- Atomically swap the **root pointer** to point to new page
- Old page remains until GC; crash before swap → old version intact
- Used by: older SQLite versions

### 2.3 Copy-on-Write (CoW)

- **Never overwrite in place** — always write to a new location
- Update the reference atomically after write succeeds
- Used by: **ZFS**, **Btrfs**, APFS

### 2.4 Double-Write Buffer (InnoDB)

- Write data to a **safe sequential buffer** first
- Only then write to the actual random data file
- On crash: recover from double-write buffer
- Protects against torn writes at 16KB page boundaries

### 2.5 Hardware-Level Atomicity

- Disk hardware guarantees **sector-level atomicity** (~512 bytes or 4KB with 4Kn drives)
- Use `O_DIRECT` + aligned writes to stay within atomic hardware boundaries
- Only practical for very small writes

---

## 3. Storage Architecture Overview

```
Application (SQL)
      ↓
Query Processor
      ↓
Storage Engine  ← manages atomicity, buffering, indexing
      ↓
Buffer Pool (RAM cache of disk pages)
      ↓
File System (OS)
      ↓
Disk (HDD / SSD / NVMe)
```

### Buffer Pool

- Pages are read from disk into RAM
- **Dirty pages** = modified but not yet flushed to disk
- **Page replacement** uses LRU or clock algorithm
- WAL ensures dirty pages can be flushed safely

---

## 4. Key Terms

| Term | Meaning |
|---|---|
| **Torn write** | Partial write leaving inconsistent page on disk |
| **WAL** | Write-Ahead Log — log before applying changes |
| **Checkpoint** | Point where all dirty pages are guaranteed flushed |
| **REDO log** | Replay committed changes after crash |
| **UNDO log** | Rollback uncommitted changes after crash |
| **fsync()** | OS call to force flush from kernel buffer to disk |
| **O_DIRECT** | Bypass OS page cache, write directly to disk |
| **Double-write** | Write to safe area first before actual location |

---

## 5. Interview Questions on This Topic

1. **What is the atomicity problem in file systems?**  
   → Multi-step writes can be interrupted mid-way by a crash, leaving data in a partial/inconsistent state.

2. **Why can't we rely on the OS `write()` syscall for atomicity?**  
   → `write()` is not atomic for large writes — kernel can flush partial data at any time.

3. **How does WAL solve the atomicity problem?**  
   → Changes are logged before being applied. On crash, the log is used to either redo committed changes or undo partial ones.

4. **What is a torn write?**  
   → A crash during a write that spans multiple disk sectors, leaving some sectors with old data and some with new data.

5. **What does InnoDB's double-write buffer protect against?**  
   → Torn writes at the 16KB page level — it writes to a sequential safe area first, then to the actual location.
