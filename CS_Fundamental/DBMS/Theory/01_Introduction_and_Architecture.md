# Introduction & Architecture of DBMS

## 1. Three-Level Architecture (ANSI/SPARC Model)

Also called the **3-Schema Architecture**, it separates how data is stored from how users see it.

```
┌─────────────────────────────────────┐
│         External Level              │  ← User views
│   View A    View B    View C        │
└──────────────┬──────────────────────┘
               │  Mapping
┌──────────────▼──────────────────────┐
│         Conceptual Level            │  ← Logical structure
│   (Tables, Relations, Constraints)  │
└──────────────┬──────────────────────┘
               │  Mapping
┌──────────────▼──────────────────────┐
│         Internal Level              │  ← Physical storage
│   (Files, Indexes, Blocks)          │
└─────────────────────────────────────┘
```

---

### Level 1 — External Level (View Level)

- **What users see** — each user/application gets a customized view
- Different users can see different subsets or formats of data
- Hides everything below it from the user
- Example: HR sees employee salary, but a manager only sees name + department

### Level 2 — Conceptual Level (Logical Level)

- **What data exists** — the complete logical structure of the entire database
- Defines tables, columns, data types, relationships, constraints
- No concern for how data is physically stored
- Managed by the **DBA**
- Example: `Employee(id, name, dept, salary)` with a FK to `Department`

### Level 3 — Internal Level (Physical Level)

- **How data is stored** — actual physical storage details
- Defines file organization, indexing (B+ trees, hash), compression, encryption
- Closest to the actual hardware
- Example: `Employee` stored as a heap file with a B+ tree index on `id`

---

## 2. Why This Architecture?

The key benefit is **Data Independence** — change one level without affecting others:

| Type | Meaning |
|---|---|
| **Physical Data Independence** | Change storage format/indexes without changing the logical schema |
| **Logical Data Independence** | Change the logical schema (add a column) without changing user views |

---

## 3. Mappings

- **External → Conceptual mapping**: Translates a user's view into the full logical schema
- **Conceptual → Internal mapping**: Translates logical queries into physical storage operations

This is why you can write `SELECT * FROM Employee` without knowing whether the data is stored in a B+ tree, a heap, or compressed blocks on disk.

---

## 4. Interview Questions

1. **What is the 3-level architecture of DBMS?**  
   → ANSI/SPARC model with External (user views), Conceptual (logical schema), and Internal (physical storage) levels.

2. **What is the difference between physical and logical data independence?**  
   → Physical: change storage without affecting logical schema. Logical: change schema without affecting user views.

3. **Who manages the conceptual level?**  
   → The DBA (Database Administrator).

4. **Why do we need data independence?**  
   → So changes in one level (e.g., adding an index) don't force changes in application code or user views.

5. **What is a view in the context of the external level?**  
   → A virtual table derived from the actual tables, showing only the data relevant to a particular user or role.
