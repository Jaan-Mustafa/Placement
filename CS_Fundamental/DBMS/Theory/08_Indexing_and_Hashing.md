# Indexing in DBMS

## 1. What is Indexing?

An **index** is a separate data structure that stores a **sorted mapping of column values to their physical location (disk block address)** — allowing the DB to find rows fast without scanning the entire table.

### Without Index — Full Table Scan

```
SELECT * FROM Employee WHERE emp_id = 105;

Scan row 1  → emp_id = 1   ✗
Scan row 2  → emp_id = 2   ✗
...
Scan row 105 → emp_id = 105 ✓  ← found, but scanned 105 rows
```

Time complexity: **O(n)** — every row checked.

### With Index

```
Index on emp_id → B+ Tree lookup → pointer to exact disk block
→ fetch only that block

Time complexity: O(log n)
```

---

## 2. Why Indexing?

| Without Index | With Index |
|---|---|
| Full table scan — O(n) | Tree/hash lookup — O(log n) or O(1) |
| Slow on large tables | Fast even on millions of rows |
| Every query reads all blocks | Only relevant blocks read |

**Trade-off:** Indexes speed up reads but **slow down writes** (INSERT/UPDATE/DELETE must also update the index).

---

## 3. Index Structure — The Basics

An index has two parts:

```
┌─────────────────────────────────────────────┐
│               INDEX FILE                    │
│                                             │
│   Search Key    │   Pointer (Block Address) │
│   ─────────────────────────────────────────│
│   101           │   Block 4, Offset 0       │
│   102           │   Block 4, Offset 1       │
│   103           │   Block 7, Offset 2       │
│   ...           │   ...                     │
└─────────────────────────────────────────────┘

Search Key = the column(s) the index is built on
Pointer    = disk address of the actual row
```

---

## 4. Types of Indexes

### 4.1 Primary Index

- Built on the **ordering key field** of an **ordered (sorted) data file**
- One index entry per **disk block** (not per row) → **sparse**
- The data file is physically sorted on this field

```
Data file sorted by EmpID:

Block 1: [EmpID 1,  EmpID 2,  EmpID 3 ]
Block 2: [EmpID 4,  EmpID 5,  EmpID 6 ]
Block 3: [EmpID 7,  EmpID 8,  EmpID 9 ]

Primary Index:
  Key=1  → Block 1
  Key=4  → Block 2
  Key=7  → Block 3
```

To find EmpID=5:
- Index lookup: 4 ≤ 5 < 7 → Block 2
- Scan Block 2 → find EmpID=5

**Properties:**
- One index entry per block → index is small
- Data file must be sorted on the index key
- Only **one** primary index per table (data can only be sorted one way)

---

### 4.2 Secondary Index (Non-Ordering Index)

- Built on a **non-ordering field** — a field the data file is NOT sorted by
- One index entry per **record** → **dense**
- Can have multiple secondary indexes per table

```
Data file sorted by EmpID (not by DeptID):

Block 1: [E1/D10,  E2/D20,  E3/D10]
Block 2: [E4/D30,  E5/D20,  E6/D10]

Secondary Index on DeptID:
  D10 → [Block1/Off0,  Block1/Off2,  Block2/Off2]
  D20 → [Block1/Off1,  Block2/Off1]
  D30 → [Block2/Off0]
```

**Properties:**
- Dense — one entry per record
- Larger than primary index
- Multiple secondary indexes allowed
- Uses a **bucket of pointers** for duplicate key values

---

### 4.3 Clustered Index

- The **physical order of rows on disk matches** the order of the index
- Only **one** clustered index per table (disk can only be sorted one way)
- Usually the Primary Key is the clustered index by default (MySQL InnoDB)
- Range queries are very fast — related rows are physically adjacent on disk

```
Clustered Index on EmpID:

Disk layout:  [E1] [E2] [E3] [E4] [E5] [E6] [E7]
                ↑
          physically sorted by EmpID

SELECT * FROM Employee WHERE EmpID BETWEEN 3 AND 6;
→ find E3 in index → read contiguous blocks → get E3, E4, E5, E6
```

---

### 4.4 Non-Clustered Index (Unclustered Index)

- The index order is **different from the physical order** of rows on disk
- Rows are **NOT** physically sorted by this index
- Multiple non-clustered indexes allowed per table
- Each lookup may hit a **different disk block** → more I/O for range queries

```
Non-Clustered Index on DeptName:

Index (sorted):   Accounts → Block5   Finance → Block2   HR → Block8
Disk layout:  [HR row] [Accounts row] [Finance row] [HR row] [Accounts row]
                ↑               ↑               ↑
           Block 1          Block 2          Block 3
           (not sorted by DeptName)
```

---

### 4.5 Dense Index

**One index entry for every record in the data file.**

Table has 6 employees stored across 2 disk blocks:

```
DISK — Data File:

Block 1                      Block 2
┌─────────────────────┐      ┌─────────────────────┐
│ Slot 0 │ E1 │ Alice │      │ Slot 0 │ E4 │ Dave  │
│ Slot 1 │ E2 │ Bob   │      │ Slot 1 │ E5 │ Eve   │
│ Slot 2 │ E3 │ Carol │      │ Slot 2 │ E6 │ Frank │
└─────────────────────┘      └─────────────────────┘
```

```
DENSE INDEX on EmpID:

┌────────┬──────────────────┐
│ EmpID  │ Pointer          │
├────────┼──────────────────┤
│  E1    │ Block1, Slot 0   │
│  E2    │ Block1, Slot 1   │
│  E3    │ Block1, Slot 2   │
│  E4    │ Block2, Slot 0   │
│  E5    │ Block2, Slot 1   │
│  E6    │ Block2, Slot 2   │
└────────┴──────────────────┘
6 records → 6 index entries
```

**Lookup EmpID = E5:**
```
Index → E5 is at Block2, Slot1
→ directly fetch Block2, read Slot1
→ done in 1 disk I/O
```

**Properties:**
- Faster lookup — exact pointer, no scanning inside block
- Larger index size — one entry per record
- Does NOT require sorted data
- Used for **secondary indexes**

---

### 4.6 Sparse Index

**One index entry per disk block — only stores the first record of each block.**

```
DISK — Data File (must be sorted):

Block 1                      Block 2
┌─────────────────────┐      ┌─────────────────────┐
│ Slot 0 │ E1 │ Alice │      │ Slot 0 │ E4 │ Dave  │
│ Slot 1 │ E2 │ Bob   │      │ Slot 1 │ E5 │ Eve   │
│ Slot 2 │ E3 │ Carol │      │ Slot 2 │ E6 │ Frank │
└─────────────────────┘      └─────────────────────┘
```

```
SPARSE INDEX on EmpID:

┌────────┬──────────────────┐
│ EmpID  │ Pointer          │
├────────┼──────────────────┤
│  E1    │ Block1           │  ← first record of Block 1
│  E4    │ Block2           │  ← first record of Block 2
└────────┴──────────────────┘
6 records → only 2 index entries
```

**Lookup EmpID = E5:**
```
Index scan:
  E1 ≤ E5?  Yes
  E4 ≤ E5?  Yes → E5 must be in Block2 (between E4 and end)
→ load Block2, scan slots → Slot1 has E5
```

**Properties:**
- Smaller index — one entry per block → fits easily in memory
- Slightly slower — requires in-block scan after finding the block
- Requires sorted data (relies on physical ordering assumption)
- Used for **primary indexes**

### Dense vs Sparse — Side by Side

```
Dense Index             Sparse Index
─────────────────────   ─────────────────────
E1 → Block1/Slot0       E1 → Block1
E2 → Block1/Slot1
E3 → Block1/Slot2       E4 → Block2
E4 → Block2/Slot0
E5 → Block2/Slot1
E6 → Block2/Slot2
─────────────────────   ─────────────────────
6 entries               2 entries
```

| | Dense | Sparse |
|---|---|---|
| Entries | One per **record** | One per **block** |
| Index size | Large | Small |
| Lookup | Direct — exact pointer | Indirect — find block, then scan |
| Requires sorted data | No | **Yes** |
| Used for | Secondary index | Primary index |

**Key point:** Sparse only works on sorted data because it assumes all records between two consecutive index entries are physically stored between those two blocks. If data is unsorted, that assumption breaks and wrong blocks would be fetched.

---

## 5. Indexing Methods (Data Structures)

### 5.1 B-Tree Index

A **self-balancing tree** where every node stores both keys and data pointers.

```
              [30 | 70]
             /    |    \
        [10|20] [40|60] [80|90]
```

- Both internal nodes and leaf nodes store data pointers
- Search, insert, delete: **O(log n)**
- Good for equality and range queries
- **Problem:** Data pointers scattered across all levels → inconsistent depth

---

### 5.2 B+ Tree Index (Most Common)

An improvement over B-Tree — **only leaf nodes store data pointers**. Internal nodes store only keys for routing.

```
              [30 | 70]                    ← Internal nodes (keys only)
             /    |    \
        [10|20] [40|60] [80|90]           ← Leaf nodes (keys + data pointers)
            ↓       ↓       ↓
          data    data    data
           ↕       ↕       ↕              ← Leaf nodes linked as a linked list
```

**Key properties:**
- All leaf nodes are at the same depth → consistent O(log n)
- Leaf nodes are **linked** → range queries are fast (just traverse the linked list)
- Internal nodes are denser → more keys fit → shorter tree height → fewer disk I/Os
- Used by: **MySQL InnoDB, PostgreSQL, Oracle**

#### B+ Tree Range Query

```sql
SELECT * FROM Employee WHERE EmpID BETWEEN 20 AND 60;
```

1. Traverse tree to find EmpID=20 → reach leaf node
2. Walk linked list of leaf nodes → collect 20, 30, 40, 50, 60
3. Fetch actual rows using pointers

No need to go back up the tree — linked list makes range scan linear.

---

### 5.3 Hash Index

Uses a **hash function** to map a key directly to a bucket containing the row pointer.

```
hash(EmpID=105) → bucket 7 → [pointer to row with EmpID=105]
```

**Properties:**
- Equality lookup: **O(1)** — fastest for exact match
- **Cannot** do range queries (hash destroys ordering)
- **Cannot** do partial key lookup (LIKE 'Ali%')
- Used by: memory-based tables (MySQL MEMORY engine), hash join internally

| | B+ Tree | Hash Index |
|---|---|---|
| Equality (=) | O(log n) | **O(1)** |
| Range (BETWEEN, >, <) | **Fast** | Not supported |
| ORDER BY | Supported | Not supported |
| LIKE prefix | Supported | Not supported |
| Used for | General purpose | Exact match only |

---

### 5.4 Bitmap Index

Uses a **bit array** for each distinct value of the indexed column.

```
Column: Gender (M/F)    100,000 rows

M: [1, 0, 1, 0, 1, 0, 1, 1, 0 ...]   ← 1 if row has M, else 0
F: [0, 1, 0, 1, 0, 1, 0, 0, 1 ...]
```

**AND/OR operations become bitwise operations — extremely fast.**

```sql
SELECT * FROM Employee WHERE Gender = 'M' AND DeptID = 'D1';
→ Bitmap(M) AND Bitmap(D1) → bitwise AND → result bitmap → fetch rows
```

**Properties:**
- Best for **low cardinality** columns (few distinct values — Gender, Status, DeptID)
- Very compact storage
- Fast for read-heavy analytical queries (OLAP)
- Slow for writes (updating bitmaps on INSERT/UPDATE is expensive)
- Used by: Oracle, data warehouses

---

## 6. Multilevel Index

When even the index itself becomes too large to fit in memory, build an **index on the index**.

```
Level 2 (sparse index on Level 1 index):
  [1→Block1, 100→Block3, 200→Block5]

Level 1 (sparse index on data):
  Block1: [1, 5, 10, 15...]
  Block3: [100, 105, 110...]
  Block5: [200, 205, 210...]

Data File:
  [actual rows...]
```

Lookup: search Level 2 → narrow to Level 1 block → narrow to data block → find row.  
This is essentially what a **B+ Tree** implements automatically.

---

## 7. Composite Index (Multi-Column Index)

An index built on **two or more columns together**.

```sql
CREATE INDEX idx_dept_salary ON Employee(DeptID, Salary);
```

```
Index structure:
  (D1, 30000) → Block 3
  (D1, 45000) → Block 5
  (D2, 25000) → Block 1
  (D2, 60000) → Block 8
```

**Left-prefix rule:** The index is only usable if the query filters on columns from the **left side** of the index definition.

```sql
WHERE DeptID = 'D1'                    → uses index ✓  (leftmost column)
WHERE DeptID = 'D1' AND Salary > 40000 → uses index ✓  (both columns)
WHERE Salary > 40000                   → does NOT use index ✗  (skips DeptID)
```

---

## 8. Covering Index

An index that contains **all columns needed by a query** — the DB never needs to touch the actual table.

```sql
CREATE INDEX idx_covering ON Employee(DeptID, EmpName, Salary);

SELECT EmpName, Salary FROM Employee WHERE DeptID = 'D1';
```

All 3 columns (DeptID, EmpName, Salary) are in the index → query answered entirely from the index → **zero table I/O** → very fast.

---

## 9. Full Summary Table

| Index Type | Based On | Dense/Sparse | Sorted Data Required | Count per Table |
|---|---|---|---|---|
| Primary | Ordering key | Sparse | Yes | 1 |
| Secondary | Non-ordering field | Dense | No | Many |
| Clustered | Physical row order | — | Yes (physically) | 1 |
| Non-Clustered | Logical order | — | No | Many |
| Dense | Every record | Dense | No | — |
| Sparse | Every block | Sparse | Yes | — |

| Index Method | Best For | Not Good For | Complexity |
|---|---|---|---|
| B+ Tree | Range, equality, ORDER BY | Nothing major | O(log n) |
| Hash | Exact equality | Range queries | O(1) |
| Bitmap | Low cardinality, OLAP | High cardinality, writes | O(1) bitwise |

---

## 10. Interview Questions

1. **What is an index in DBMS and why is it used?**
   → A data structure mapping column values to disk locations. Used to avoid full table scans — speeds up queries from O(n) to O(log n).

2. **What is the difference between clustered and non-clustered index?**
   → Clustered: physical row order matches index order — one per table. Non-clustered: logical order only, rows not physically sorted — multiple allowed per table.

3. **Why is B+ Tree preferred over B-Tree for indexing?**
   → B+ Tree stores data only at leaf nodes, and leaf nodes are linked — making range queries fast by simply traversing the linked list. B-Tree stores data at all levels, making range scans slower and inconsistent.

4. **What is the difference between dense and sparse index?**
   → Dense: one entry per record — faster lookup, larger size. Sparse: one entry per block — smaller size, requires sorted data, needs in-block scan after lookup.

5. **When would you use a Hash index over a B+ Tree?**
   → When queries are only equality comparisons (=) and range queries are never needed. Hash gives O(1) lookup vs O(log n) for B+ Tree.

6. **What is the left-prefix rule in composite indexes?**
   → A composite index on (A, B, C) is only usable if the query filters start from the leftmost column A. Filtering on B or C alone skips A and cannot use the index.

7. **What is a covering index?**
   → An index that contains all columns a query needs — the query is answered entirely from the index without touching the actual table rows.

8. **Why do indexes slow down writes?**
   → Every INSERT/UPDATE/DELETE must also update the index structure (e.g., insert a new entry into the B+ Tree and rebalance) — additional I/O on every write.

9. **What is a bitmap index and when is it used?**
   → Uses bit arrays per distinct value. Best for low-cardinality columns (few distinct values) in read-heavy OLAP systems. Bitwise AND/OR makes multi-condition queries extremely fast.

10. **Can a table have multiple clustered indexes?**
    → No. A clustered index defines the physical order of rows on disk — data can only be physically sorted one way. Only one clustered index per table is possible.
