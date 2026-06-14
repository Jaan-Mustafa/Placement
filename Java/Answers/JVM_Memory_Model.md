# JVM Memory Model: Heap, Stack, and GC

## Big Picture — What is JVM Memory?

When your Java program runs, the JVM carves up the OS memory into distinct regions. Each region has a specific purpose, lifetime, and ownership rule.

```
JVM Process Memory
│
├── Heap              ← all objects live here (shared across threads)
├── Stack             ← each thread gets its own stack (method frames)
├── Metaspace         ← class metadata, static fields (Java 8+)
├── Code Cache        ← JIT-compiled native code
└── Native Memory     ← JVM internals, NIO buffers, thread stacks
```

---

## 1. The Heap

**What lives here:** Every object you create with `new` — `new String()`, `new ArrayList()`, your own POJOs — everything.

**Shared across all threads.** One heap per JVM process.

### Heap Layout (Generational GC model)

```
HEAP
│
├── Young Generation  (small, collected frequently — "Minor GC")
│     ├── Eden Space       ← new objects are born here
│     ├── Survivor S0      ← objects that survived one GC
│     └── Survivor S1      ← objects move between S0/S1 each GC
│
└── Old Generation    (large, collected rarely — "Major/Full GC")
      └── Tenured Space    ← objects that survived many GCs (age threshold ~15)
```

### Object Lifecycle on the Heap

```
new Object()
    │
    ▼
 Eden Space ──GC──► dead? → collected
                 ├► survived? → Survivor S0
                              ──GC──► survived again? → Survivor S1
                                                  ...repeat...
                                                  age >= threshold?
                                                       │
                                                       ▼
                                               Old Generation
                                                       │
                                               Major GC cleans it
```

### Key Heap Flags

```bash
-Xms512m          # initial heap size
-Xmx2g            # max heap size
-XX:NewRatio=2    # Old:Young ratio (2 means Old is 2x Young)
-XX:SurvivorRatio=8  # Eden:Survivor ratio
```

---

## 2. The Stack

**What lives here:** Method call frames. Each thread has its **own private stack** — no sharing, no thread-safety issues.

**Each frame contains:**
- Local variables (primitives stored directly: `int`, `boolean`, `double`)
- Reference variables (the reference/pointer to the heap object, not the object itself)
- Operand stack (intermediate calculation values)
- Return address (where to go after the method returns)

```java
public int add(int a, int b) {
    int result = a + b;   // a, b, result all live on the Stack frame
    return result;        // frame is popped when method returns
}
```

```
Thread 1 Stack          Thread 2 Stack
│                       │
│  add() frame          │  processOrder() frame
│  ├─ a = 5             │  ├─ orderId = 42
│  ├─ b = 3             │  └─ order ref → [Heap: Order@0x1a2]
│  └─ result = 8        │
│  main() frame         │  main() frame
└──────────────         └──────────────
```

### Stack vs Heap — where things actually live

```java
public void process() {
    int x = 10;                    // x (primitive) → Stack
    String name = "Alice";         // name (reference) → Stack
                                   // "Alice" object   → Heap
    Person p = new Person("Bob");  // p (reference)    → Stack
                                   // Person object    → Heap
}
// method returns → Stack frame is destroyed instantly
// Heap objects stay until GC collects them
```

### StackOverflowError

```java
void infinite() {
    infinite();   // each call pushes a new frame
}                 // stack runs out → StackOverflowError
```

### Key Stack Flag

```bash
-Xss512k    # stack size per thread (default ~512k-1m depending on OS)
```

---

## 3. Metaspace (Java 8+, replaced PermGen)

**What lives here:**
- Class definitions (bytecode structure)
- Method metadata
- Static variables
- String pool (interned strings — actually moved to Heap in Java 7+)
- Constant pool

**Not on the Heap.** Metaspace uses **native OS memory** — it grows automatically by default (unlike the old PermGen which had a fixed cap and caused `OutOfMemoryError: PermGen space`).

```bash
-XX:MetaspaceSize=128m       # initial size
-XX:MaxMetaspaceSize=256m    # cap it (optional)
```

---

## 4. Garbage Collection (GC)

GC automatically reclaims heap memory occupied by objects that are **no longer reachable** from any live thread or static reference.

### Reachability — the root concept

```
GC Roots (always alive):
├── Local variables on active thread stacks
├── Static fields in loaded classes
├── Active thread objects
└── JNI references

GC Root → Object A → Object B → Object C   ← all reachable, NOT collected
GC Root → Object D → Object E
                          ↑
                     nothing points here anymore → ELIGIBLE for GC
```

An object is collected when **no path from any GC root leads to it** — not when a variable goes out of scope (that just removes one reference).

---

### Minor GC (Young Generation)

Triggered when **Eden is full.**

```
Before Minor GC:
Eden: [Obj1][Obj2][Obj3][Obj4]   ← full
S0:   [OldObj]
S1:   empty

GC runs:
- Obj2, Obj4 still reachable → copy to S1, increment age
- Obj1, Obj3 unreachable → wiped

After Minor GC:
Eden: empty (all cleared)
S0:   empty
S1:   [Obj2 age=1][Obj4 age=1][OldObj age=2]
```

- Very fast (milliseconds) — only scans Young Gen
- "Stop-The-World" (STW) — all threads pause briefly

---

### Major GC / Full GC (Old Generation)

Triggered when **Old Gen is full** or explicitly via `System.gc()`.

- Scans the entire heap (Young + Old)
- Much slower (hundreds of ms to seconds)
- Long STW pause — this is what causes latency spikes in production

---

### GC Algorithms

| Collector | Use Case | STW Pause | How |
|---|---|---|---|
| **Serial GC** | Single-threaded, small apps | Long | One thread does everything |
| **Parallel GC** | Throughput-focused (batch jobs) | Long but parallel | Multiple threads, still STW |
| **G1 GC** | Balanced (default Java 9+) | Short, predictable | Divides heap into regions, concurrent marking |
| **ZGC** | Ultra-low latency (<1ms pauses) | Near-zero | Concurrent everything, colored pointers |
| **Shenandoah** | Low latency (Red Hat) | Near-zero | Concurrent compaction |

```bash
-XX:+UseG1GC          # use G1 (default Java 9+)
-XX:+UseZGC           # use ZGC (Java 15+)
-XX:MaxGCPauseMillis=200  # G1: target max pause time
```

---

### G1 GC — How it works (most common in production)

```
Heap divided into equal-sized REGIONS (1MB–32MB each)

[E][E][E][S][O][O][E][O][H][E][S][O]
 E=Eden  S=Survivor  O=Old  H=Humongous (large objects)

G1 prioritizes collecting regions with most garbage first
→ "Garbage First" = highest ROI regions collected first
→ Concurrent marking happens while app runs
→ STW only for short evacuation pauses
```

---

## 5. Common Memory Problems

### OutOfMemoryError: Java heap space
```
Cause: Heap is full, GC can't free enough
Fix: -Xmx increase, fix memory leak (use heap dump + VisualVM/MAT)
```

### OutOfMemoryError: Metaspace
```
Cause: Too many classes loaded (classloader leak, reflection abuse)
Fix: -XX:MaxMetaspaceSize, fix classloader leak
```

### StackOverflowError
```
Cause: Deep/infinite recursion
Fix: Reduce recursion depth, convert to iterative, increase -Xss
```

### GC Overhead Limit Exceeded
```
Cause: GC spending >98% of time freeing <2% of heap
Fix: Fix memory leak, increase heap, review object retention
```

---

## 6. Memory Across Threads — The Visibility Problem

The JVM allows each thread to cache variables in CPU registers/L1 cache — not always reading from main memory (heap).

```java
// Thread 1 writes:
running = false;   // may stay in CPU cache, not flushed to heap

// Thread 2 reads:
while (running) { }  // may read stale cached value → infinite loop
```

**Fix:** `volatile` forces reads/writes to go directly to main memory.

```java
volatile boolean running = true;  // Thread 2 always sees Thread 1's write
```

This is the **Java Memory Model (JMM)** — the rules about when one thread's writes are guaranteed visible to another thread. Key rules:
- `volatile` write **happens-before** any subsequent volatile read of same variable
- `synchronized` block exit **happens-before** entry of same monitor by another thread
- `Thread.start()` **happens-before** any action in the started thread

---

## Full Picture

```
JVM
│
├── HEAP (shared)
│     ├── Young Gen
│     │     ├── Eden       ← new Object() lands here
│     │     ├── Survivor0
│     │     └── Survivor1
│     └── Old Gen          ← long-lived objects promoted here
│
├── METASPACE (native memory)
│     └── Class definitions, static fields, method metadata
│
├── THREAD STACKS (one per thread, private)
│     └── Method frames → local vars, references, return addresses
│
├── CODE CACHE
│     └── JIT-compiled hot methods (native machine code)
│
└── GC
      ├── Minor GC  → cleans Young Gen  (fast, frequent)
      ├── Major GC  → cleans Old Gen    (slow, rare)
      └── Full GC   → cleans everything (avoid in production)
```

---

## One-Line Summary for Interviews

> "The JVM heap stores all objects and is split into Young and Old generations for generational GC; each thread gets its own private stack holding method frames with local variables and references; GC traces reachability from GC roots and reclaims unreachable heap objects — Minor GC is fast and frequent on Young Gen, Major GC is slow and rare on Old Gen."
