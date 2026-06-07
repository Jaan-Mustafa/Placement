# Normalization in DBMS

## 1. What is Normalization?

Normalization is the process of **organizing a database to reduce redundancy and improve data integrity** by decomposing tables into smaller, well-structured ones without losing information.

---

## 2. Why Normalization? — The Anomaly Problem

Without normalization, a poorly designed table causes **anomalies**:

### Example of a Bad Table

| StudentID | StudentName | CourseID | CourseName | InstructorID | InstructorName |
|---|---|---|---|---|---|
| 1 | Alice | C01 | DBMS | I01 | Dr. Smith |
| 1 | Alice | C02 | OS | I02 | Dr. Jones |
| 2 | Bob | C01 | DBMS | I01 | Dr. Smith |

### Anomalies

| Anomaly | Problem | Example |
|---|---|---|
| **Insert Anomaly** | Can't insert data without unrelated data | Can't add a new course unless a student is enrolled |
| **Update Anomaly** | Updating one fact requires changing multiple rows | Dr. Smith's name change requires updating all rows where he teaches |
| **Delete Anomaly** | Deleting one fact accidentally deletes another | Deleting Bob deletes the only record of C01 course info |

---

## 3. Functional Dependency (FD)

A **Functional Dependency** X → Y means: if two rows have the same value of X, they must have the same value of Y.

```
StudentID → StudentName     (StudentID determines StudentName)
CourseID  → CourseName      (CourseID determines CourseName)
```

### Types of FD

| Type | Meaning |
|---|---|
| **Trivial FD** | Y is a subset of X. e.g., {A,B} → A |
| **Non-trivial FD** | Y is not a subset of X. e.g., A → B |
| **Partial Dependency** | Non-key attribute depends on part of a composite key |
| **Transitive Dependency** | A → B → C (non-key depends on another non-key) |

### Keys (needed for NF rules)

| Term | Meaning |
|---|---|
| **Super Key** | Any set of attributes that uniquely identifies a row |
| **Candidate Key** | Minimal super key — no subset of it is also a super key |
| **Primary Key** | Chosen candidate key |
| **Prime Attribute** | Attribute that is part of ANY candidate key |
| **Non-prime Attribute** | Attribute that is NOT part of any candidate key |
| **Key Attribute** | Attribute that is part of the chosen Primary Key |
| **Non-Key Attribute** | Attribute that is NOT part of the chosen Primary Key |
| **Composite Key** | A primary key made of **two or more attributes** combined |

### Keys Explained with Example

Consider this table:

`Student(StudentID, Email, AadharNo, Name, Age)`

| StudentID | Email | AadharNo | Name | Age |
|---|---|---|---|---|
| 1 | alice@gmail.com | 1234-5678 | Alice | 20 |
| 2 | bob@gmail.com | 8765-4321 | Bob | 22 |

---

#### Super Key
Any set of attributes that **uniquely identifies a row** — can have extra unnecessary attributes.

```
{StudentID}                        ← unique ✓
{Email}                            ← unique ✓
{AadharNo}                         ← unique ✓
{StudentID, Email}                 ← unique ✓ (but has extra column)
{StudentID, Name}                  ← unique ✓ (but has extra column)
{StudentID, Email, AadharNo, Name} ← unique ✓ (but has many extra columns)
```

All of the above are **super keys** — they all uniquely identify a row.

---

#### Candidate Key
A **minimal** super key — removing any attribute from it makes it non-unique.

```
{StudentID}   ← remove StudentID → nothing left → not unique. So it's minimal ✓
{Email}       ← same reasoning → minimal ✓
{AadharNo}    ← same reasoning → minimal ✓

{StudentID, Email} ← NOT a candidate key — removing Email still gives {StudentID}
                     which is still unique. So it's NOT minimal ✗
```

**Candidate Keys = {StudentID}, {Email}, {AadharNo}**

---

#### Primary Key
The **one candidate key** the DBA chooses to use as the official identifier.

```
Primary Key = StudentID   (chosen from the three candidate keys above)
```

The others (Email, AadharNo) become **alternate keys**.

---

#### Prime Attribute
Any attribute that is **part of at least one candidate key**.

```
Candidate Keys = {StudentID}, {Email}, {AadharNo}

Prime Attributes = StudentID, Email, AadharNo
```

Even if `Email` is not the chosen primary key, it is still a **prime attribute** because it is a candidate key.

---

#### Non-Prime Attribute
Any attribute that is **NOT part of any candidate key**.

```
All attributes     = {StudentID, Email, AadharNo, Name, Age}
Prime attributes   = {StudentID, Email, AadharNo}

Non-Prime Attributes = Name, Age
```

`Name` and `Age` don't uniquely identify any row on their own — they are non-prime.

---

#### Key Attribute
Attribute that is part of the **chosen Primary Key only**.

```
Primary Key = StudentID

Key Attribute     = StudentID
Non-Key Attribute = Email, AadharNo, Name, Age
```

---

#### Non-Key Attribute
Any attribute **not in the chosen Primary Key** — includes alternate keys and regular attributes.

```
Non-Key Attributes = Email, AadharNo, Name, Age
```

Note: `Email` and `AadharNo` are non-key but they ARE prime (they belong to a candidate key).  
`Name` and `Age` are non-key AND non-prime (they belong to no candidate key).

---

#### Key vs Prime — Where the Confusion Happens

`Email` and `AadharNo` are candidate keys but NOT the chosen primary key:

| Attribute | Key Attribute? | Non-Key Attribute? | Prime Attribute? | Non-Prime Attribute? |
|---|---|---|---|---|
| StudentID | Yes (is PK) | No | Yes | No |
| Email | No | Yes | Yes (is CK) | No |
| AadharNo | No | Yes | Yes (is CK) | No |
| Name | No | Yes | No | Yes |
| Age | No | Yes | No | Yes |

> `Email` is a **non-key but prime attribute** — it is not the chosen PK, but it does belong to a candidate key.

---

#### Why This Distinction Matters

- **2NF and 3NF** use **prime / non-prime** in their formal definitions — not key/non-key
- Using "non-key" loosely is technically imprecise: `Email` is non-key but still prime, so 3NF rules treat it differently than `Name`
- BCNF is the strictest — it doesn't care about prime/non-prime at all; every determinant must be a super key

---

#### Why This Matters for Normalization

| NF Rule | Uses These Concepts |
|---|---|
| **2NF** | Non-prime attributes must depend on the **whole** primary key, not part of it |
| **3NF** | Non-prime attributes must not depend on other non-prime attributes |
| **BCNF** | Every determinant must be a super key (no exceptions even for prime attributes) |

---

#### Composite Key

A **Primary Key made of two or more attributes combined** — no single attribute alone is enough to uniquely identify a row.

##### Example

`Enrollment(StudentID, CourseID, Grade)`

| StudentID | CourseID | Grade |
|---|---|---|
| 1 | C01 | A |
| 1 | C02 | B |
| 2 | C01 | A |

- `StudentID` alone → not unique (student 1 appears twice)
- `CourseID` alone → not unique (C01 appears twice)
- `(StudentID, CourseID)` together → **unique** ✓

So Primary Key = `(StudentID, CourseID)` — a **composite key**.

##### Key Points

- A composite key is always made of **2 or more columns**
- Each column in the composite key is a **key attribute** (prime attribute)
- **Partial dependency** — a concept that only exists because of composite keys (a non-prime attribute depending on only *part* of the composite key) — this is exactly what **2NF** fixes

##### Composite Key vs Composite Candidate Key

| Term | Meaning |
|---|---|
| **Composite Key** | A primary key with 2+ attributes |
| **Composite Candidate Key** | A candidate key with 2+ attributes (may or may not be chosen as PK) |

If `(StudentID, CourseID)` is a candidate key and you choose it as PK → it's both a composite candidate key AND a composite primary key.

##### Why Composite Keys Matter

```
Without composite key:
  StudentID=1 enrolled in C01 → Grade A
  StudentID=1 enrolled in C01 → Grade B  ← duplicate! no way to distinguish

With composite key (StudentID, CourseID):
  (1, C01) → A   ← unique row ✓
  (1, C02) → B   ← unique row ✓
```

---

## 4. First Normal Form (1NF)

### Rule
- Every column must have **atomic (indivisible) values**
- No **repeating groups** or arrays in a column
- Each row must be **uniquely identifiable**

### Violation Example

| StudentID | Name | Courses |
|---|---|---|
| 1 | Alice | DBMS, OS, CN |
| 2 | Bob | DBMS |

`Courses` column has multiple values — violates 1NF.

### After 1NF

| StudentID | Name | Course |
|---|---|---|
| 1 | Alice | DBMS |
| 1 | Alice | OS |
| 1 | Alice | CN |
| 2 | Bob | DBMS |

Each cell now has exactly one value. Primary Key = (StudentID, Course).

### Key Point
1NF only ensures atomicity — it does **not** remove redundancy.

---

## 5. Second Normal Form (2NF)

### Rule
- Must be in **1NF**
- **No partial dependency** — every non-prime attribute must depend on the **whole** primary key, not just part of it

> Only relevant when the primary key is **composite** (more than one column).

### Violation Example

Table: `Enrollment(StudentID, CourseID, StudentName, CourseName, Grade)`  
Primary Key = (StudentID, CourseID)

**Actual data in the table:**

| StudentID | CourseID | StudentName | CourseName | Grade |
|---|---|---|---|---|
| 1 | C01 | Alice | DBMS | A |
| 1 | C02 | Alice | OS | B |
| 2 | C01 | Bob | DBMS | A |
| 2 | C03 | Bob | CN | C |

**Functional Dependencies:**

| Dependency | Type |
|---|---|
| (StudentID, CourseID) → Grade | Full dependency ✓ |
| StudentID → StudentName | **Partial dependency** ✗ |
| CourseID → CourseName | **Partial dependency** ✗ |

`StudentName` depends only on `StudentID` — not the full key. Same for `CourseName`.

**Problems this causes:**
- `Alice` is repeated in every row she is enrolled in → **redundancy**
- If Alice changes her name, must update multiple rows → **update anomaly**
- Deleting Bob's last course deletes Bob's name too → **delete anomaly**

### After 2NF — Decompose

**Student table:**
| StudentID | StudentName |
|---|---|
| 1 | Alice |
| 2 | Bob |

**Course table:**
| CourseID | CourseName |
|---|---|
| C01 | DBMS |
| C02 | OS |
| C03 | CN |

**Enrollment table:**
| StudentID | CourseID | Grade |
|---|---|---|
| 1 | C01 | A |
| 1 | C02 | B |
| 2 | C01 | A |
| 2 | C03 | C |

Now `StudentName` is stored once, `CourseName` is stored once — no partial dependencies.

### Visual: Partial Dependency Map

```
Composite PK = (StudentID, CourseID)

StudentID ──────────────→ StudentName   ← partial (only left part of PK)
           CourseID ────→ CourseName    ← partial (only right part of PK)
(StudentID, CourseID) ──→ Grade         ← full dependency ✓
```

### Key Point
2NF violation is **only possible with a composite primary key**. If a table has a single-column PK, it is automatically in 2NF (there is no "part of the key" to partially depend on).

---

## 6. Third Normal Form (3NF)

### Rule
- Must be in **2NF**
- **No transitive dependency** — non-prime attribute must not depend on another non-prime attribute

Formal rule: For every FD X → Y in the table, at least one must be true:
1. X is a super key, **OR**
2. Y is a prime attribute (part of some candidate key)

### Violation Example

Table: `Employee(EmpID, DeptID, DeptName)`  
Primary Key = EmpID

| Dependency | Type |
|---|---|
| EmpID → DeptID | Direct ✓ |
| DeptID → DeptName | **Transitive** ✗ (non-prime → non-prime) |
| EmpID → DeptName | Transitive through DeptID ✗ |

`DeptName` depends on `DeptID`, not on `EmpID` directly.

### After 3NF — Decompose

**Employee table:**
| EmpID | DeptID |
|---|---|
| E1 | D1 |

**Department table:**
| DeptID | DeptName |
|---|---|
| D1 | Engineering |

Transitive dependency removed.

---

## 7. Boyce-Codd Normal Form (BCNF)

### Rule
- Must be in **3NF**
- For every FD **X → Y**, X must be a **super key**

> BCNF is a stricter version of 3NF. 3NF allows Y to be a prime attribute even if X is not a super key. BCNF does not allow this exception.

### When 3NF is satisfied but BCNF is not

Table: `CourseInstructor(StudentID, CourseID, InstructorID)`  
- Each instructor teaches only one course
- Each student can enroll with only one instructor per course

Candidate Keys: (StudentID, CourseID) and (StudentID, InstructorID)  
FDs:
- (StudentID, CourseID) → InstructorID
- InstructorID → CourseID  ← **InstructorID is not a super key, but CourseID is a prime attribute**

This satisfies 3NF (Y=CourseID is prime) but **violates BCNF** (X=InstructorID is not a super key).

### After BCNF — Decompose

**InstructorCourse:**
| InstructorID | CourseID |
|---|---|
| I01 | DBMS |

**StudentInstructor:**
| StudentID | InstructorID |
|---|---|
| 1 | I01 |

### 3NF vs BCNF

| | 3NF | BCNF |
|---|---|---|
| Allows X → Y where X is not super key | Yes, if Y is prime | No |
| Always lossless decomposition possible | Yes | Yes |
| Always dependency preserving | Yes | **Not always** |
| Preferred when | Dependency preservation needed | Redundancy elimination priority |

---

## 8. Fourth Normal Form (4NF)

### Rule
- Must be in **BCNF**
- No **multi-valued dependency (MVD)** unless the determinant is a super key

### Multi-Valued Dependency (MVD)

X →→ Y means: for a given X, Y can have multiple independent values (unrelated to other attributes).

### Violation Example

Table: `Employee(EmpID, Skill, Language)`

| EmpID | Skill | Language |
|---|---|---|
| E1 | Java | English |
| E1 | Java | French |
| E1 | Python | English |
| E1 | Python | French |

Skills and Languages are **independent** of each other — but stored together, causing redundancy. If E1 learns a new language, you must add a row for every skill.

FDs: EmpID →→ Skill and EmpID →→ Language (two independent MVDs)

### After 4NF — Decompose

**EmpSkill:**
| EmpID | Skill |
|---|---|
| E1 | Java |
| E1 | Python |

**EmpLanguage:**
| EmpID | Language |
|---|---|
| E1 | English |
| E1 | French |

---

## 9. Fifth Normal Form (5NF) / Project-Join Normal Form (PJNF)

### Rule
- Must be in **4NF**
- No **join dependency** that is not implied by the candidate keys
- A table is in 5NF if every join dependency in it is implied by the candidate keys

### When it Applies

5NF deals with cases where a table can be **split into 3 or more tables** and joined back without losing or gaining rows.

### Example

Table: `Supplier_Part_Project(SupplierID, PartID, ProjectID)`

Meaning: Supplier S supplies Part P to Project J — but only when:
- S supplies P
- S works on J
- P is used in J

This **3-way relationship** cannot be decomposed into two tables without losing information — but if the data has a special cyclic pattern, it can be decomposed into three tables losslessly.

5NF is rare in practice and mostly theoretical.

---

## 10. Summary Table

| NF | Requirement | Removes |
|---|---|---|
| **1NF** | Atomic values, no repeating groups | Multi-valued cells |
| **2NF** | 1NF + no partial dependency | Partial dependency on composite key |
| **3NF** | 2NF + no transitive dependency | Non-prime depending on non-prime |
| **BCNF** | 3NF + every determinant is a super key | Anomalies 3NF misses with prime attributes |
| **4NF** | BCNF + no multi-valued dependency | Independent multi-valued facts in one table |
| **5NF** | 4NF + no join dependency | Lossless decomposition into 3+ tables |

---

## 11. Denormalization

The **intentional reversal** of normalization — merging tables back together for **performance**.

### When to Denormalize

- Too many JOINs slow down read-heavy queries
- Reporting/analytics systems (OLAP) where reads >> writes
- Caching frequently accessed combined data

### Trade-off

| | Normalized | Denormalized |
|---|---|---|
| Redundancy | Low | High |
| Write performance | Better | Worse (update many rows) |
| Read performance | Slower (JOINs) | Faster (fewer JOINs) |
| Data integrity | High | Lower |
| Use case | OLTP | OLAP / Reporting |

---

## 12. Interview Questions

1. **What is normalization and why is it needed?**
   → Process of organizing tables to reduce redundancy and prevent anomalies (insert, update, delete).

2. **What are the three anomalies in an unnormalized table?**
   → Insert anomaly, Update anomaly, Delete anomaly.

3. **What is the difference between 2NF and 3NF?**
   → 2NF removes partial dependencies (non-prime depending on part of composite key). 3NF removes transitive dependencies (non-prime depending on non-prime).

4. **What is the difference between 3NF and BCNF?**
   → 3NF allows X → Y where X is not a super key if Y is a prime attribute. BCNF strictly requires X to always be a super key. BCNF is stricter.

5. **Can BCNF decomposition lose functional dependencies?**
   → Yes. BCNF may not always preserve all FDs. 3NF always preserves FDs. This is the main trade-off.

6. **What is a partial dependency?**
   → A non-prime attribute depends on only a part of a composite primary key, not the whole key.

7. **What is a transitive dependency?**
   → A → B → C, where A is the primary key, B and C are non-prime. C depends on A only through B.

8. **What is a multi-valued dependency?**
   → X →→ Y means X independently determines multiple values of Y, unrelated to other attributes in the table.

9. **What is denormalization? When would you use it?**
   → Intentionally introducing redundancy by merging tables for query performance. Used in read-heavy OLAP/reporting systems.

10. **Is BCNF always better than 3NF?**
    → Not always. If dependency preservation is critical, 3NF is preferred. BCNF is better for eliminating redundancy at the cost of possibly losing some FDs.
