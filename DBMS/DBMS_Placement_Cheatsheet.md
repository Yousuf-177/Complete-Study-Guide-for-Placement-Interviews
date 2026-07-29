```
╔══════════════════════════════════════════════════════╗
║   DBMS — Placement Interview Cheat Sheet             ║
║   Subject: DBMS  |  Level: Campus Placements/SDE     ║
╚══════════════════════════════════════════════════════╝
```

---

## ⚡ 1. CORE CONCEPTS (30-second recall)

- **Database** → organized collection of related data stored & accessed electronically.
- **DBMS** → software to create, manage, and manipulate databases (MySQL, Oracle, PostgreSQL).
- **RDBMS** → DBMS based on relational model; data stored in tables (relations) with rows & columns.
- **Schema** → logical structure/blueprint of the database (tables, fields, relationships).
- **Instance** → actual data stored in DB at a given point in time.
- **Data Independence** → ability to change schema at one level without affecting another (Logical & Physical).
- **Primary Key (PK)** → uniquely identifies each row; can't be NULL or duplicate.
- **Foreign Key (FK)** → field referencing PK of another table; maintains referential integrity.
- **Candidate Key** → minimal set of attributes that can uniquely identify a row; one becomes PK, rest are Alternate Keys.
- **Super Key** → any set of attributes (may include extra) that uniquely identifies a row.
- **Composite Key** → PK made of 2+ columns combined.
- **Surrogate Key** → artificial key (like auto-increment ID) with no business meaning.
- **Normalization** → process of organizing data to reduce redundancy & avoid anomalies.
- **Denormalization** → intentionally adding redundancy to improve read performance.
- **ACID** → Atomicity, Consistency, Isolation, Durability — properties guaranteeing reliable transactions.
- **Index** → data structure (usually B-Tree) that speeds up data retrieval at cost of write speed & storage.
- **View** → virtual table based on result of a SQL query; doesn't store data physically.
- **Trigger** → block of code auto-executed before/after INSERT/UPDATE/DELETE.
- **Stored Procedure** → precompiled set of SQL statements stored in DB, callable by name.
- **Deadlock** → two+ transactions waiting on each other's locks forever, none can proceed.
- **Cursor** → pointer used to traverse rows returned by a query one at a time.

---

## 📐 2. KEYS & CONSTRAINTS TABLE

| Key/Constraint | Meaning | Allows NULL? | Allows Duplicate? |
|---|---|---|---|
| Primary Key | Uniquely identifies row | ❌ No | ❌ No |
| Unique Key | Ensures uniqueness | ✅ Yes (1 NULL) | ❌ No |
| Foreign Key | Links to PK of another table | ✅ Yes | ✅ Yes |
| Candidate Key | Eligible to be PK | ❌ No | ❌ No |
| Not Null | Column must have value | ❌ No | ✅ Yes |
| Check | Restricts value range | ✅ Yes | ✅ Yes |
| Default | Auto-fills value if none given | ✅ Yes | ✅ Yes |

---

## 🧩 3. NORMALIZATION (VERY frequently asked)

| Normal Form | Rule | Removes |
|---|---|---|
| **1NF** | Atomic values only, no repeating groups | Multi-valued attributes |
| **2NF** | 1NF + no partial dependency (non-key attr depends on WHOLE composite PK) | Partial dependency |
| **3NF** | 2NF + no transitive dependency (non-key attr depends on another non-key attr) | Transitive dependency |
| **BCNF** | 3NF + every determinant is a candidate key | Anomalies from overlapping candidate keys |
| **4NF** | BCNF + no multi-valued dependency | Multi-valued dependency |
| **5NF** | 4NF + no join dependency (lossless decomposition only) | Join dependency |

💡 **Mnemonic** → "**1**-Atomic, **2**-Partial gone, **3**-Transitive gone, **B**-CNF Boss(key), **4**-Multivalued gone, **5**-Join dependency gone"

**Functional Dependency (FD):** X → Y means value of X determines value of Y.
- **Partial Dependency**: non-prime attribute depends on part of a composite candidate key.
- **Transitive Dependency**: A → B → C, so A → C indirectly (non-key to non-key).

📝 Example: Table(StudentID, CourseID, StudentName, CourseName)
- PK = (StudentID, CourseID) → composite
- StudentName depends only on StudentID → **partial dependency** → violates 2NF

---

## 🔄 4. ACID PROPERTIES

```
A — Atomicity   → Transaction is all-or-nothing (rollback on failure)
C — Consistency → DB moves from one valid state to another valid state
I — Isolation   → Concurrent transactions don't interfere with each other
D — Durability  → Once committed, changes persist even after crash
```

💡 **Mnemonic** → "**A**ll **C**hanges **I**solated **D**one" — commit is final, failure is total.

---

## 🔐 5. TRANSACTION ISOLATION LEVELS (Low → High isolation)

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | ✅ Possible | ✅ Possible | ✅ Possible |
| Read Committed | ❌ No | ✅ Possible | ✅ Possible |
| Repeatable Read | ❌ No | ❌ No | ✅ Possible |
| Serializable | ❌ No | ❌ No | ❌ No |

- **Dirty Read** → reading uncommitted data of another transaction.
- **Non-Repeatable Read** → same row gives different values on re-read (updated by another txn).
- **Phantom Read** → same query returns different SET of rows (new rows inserted by another txn).

---

## 🗝️ 6. KEYS FOR CONCURRENCY CONTROL

- **Locking** → Shared Lock (S) for read, Exclusive Lock (X) for write.
- **Two-Phase Locking (2PL)** → Growing phase (acquire locks) then Shrinking phase (release locks); guarantees serializability.
- **Deadlock Handling** → Prevention (wait-die/wound-wait), Detection (wait-for graph), Avoidance (banker's algorithm-style).
- **Timestamp Ordering** → transactions ordered by timestamps to avoid conflicts.

---

## 🗺️ 7. ER MODEL CONCEPT MAP

```
Entity → Attribute → Relationship → Cardinality
  |          |              |             |
Table     Column      Foreign Key    1:1, 1:N, M:N
```

- **Entity** → real-world object (Student, Course).
- **Attribute Types** → Simple, Composite, Derived, Multi-valued.
- **Relationship Cardinality**:
  - **1:1** → One Employee has One Locker.
  - **1:N** → One Department has Many Employees.
  - **M:N** → Many Students enroll in Many Courses (needs junction table).
- **Weak Entity** → depends on another entity for existence (has partial key, no own PK).
- **Generalization/Specialization** → merging/splitting entities into super/sub types.

---

## 📊 8. SQL vs NoSQL

| Basis | SQL (RDBMS) | NoSQL |
|---|---|---|
| Schema | Fixed, predefined | Dynamic/flexible |
| Structure | Tables (rows/columns) | Document, Key-Value, Graph, Column-family |
| Scalability | Vertical (scale-up) | Horizontal (scale-out) |
| ACID | Strongly follows | Often eventual consistency (BASE) |
| Examples | MySQL, PostgreSQL, Oracle | MongoDB, Redis, Cassandra, Neo4j |
| Best for | Structured, relational data | Big data, unstructured, fast writes |

---

## 📊 9. DELETE vs TRUNCATE vs DROP

| Basis | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Rollback | ✅ Possible | ❌ (mostly not) | ❌ No |
| WHERE clause | ✅ Yes | ❌ No | ❌ No |
| Removes structure | ❌ No, only rows | ❌ No, only rows | ✅ Yes, entire table |
| Speed | Slow (row by row + logs) | Fast (deallocates pages) | Fast |

---

## 📊 10. CLUSTERED vs NON-CLUSTERED INDEX

| Basis | Clustered Index | Non-Clustered Index |
|---|---|---|
| Data order | Physically reorders table data | Separate structure, points to data |
| Count per table | Only 1 | Multiple allowed |
| Speed | Faster for range queries | Slightly slower (extra lookup) |
| Default | Usually on Primary Key | Created explicitly |

---

## 🔗 11. JOINS — MUST KNOW COLD

```sql
-- INNER JOIN: only matching rows in both tables
SELECT * FROM A INNER JOIN B ON A.id = B.id;

-- LEFT JOIN: all rows from A + matched from B (NULL if no match)
SELECT * FROM A LEFT JOIN B ON A.id = B.id;

-- RIGHT JOIN: all rows from B + matched from A
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;

-- FULL OUTER JOIN: all rows from both, NULL where no match
SELECT * FROM A FULL OUTER JOIN B ON A.id = B.id;

-- SELF JOIN: table joined with itself
SELECT A.name, B.name FROM Emp A, Emp B WHERE A.mgr_id = B.emp_id;

-- CROSS JOIN: Cartesian product (every row x every row)
SELECT * FROM A CROSS JOIN B;
```

💡 **Mnemonic** → "**I**nner=Intersection, **L**eft=Left+match, **R**ight=Right+match, **F**ull=Union of both"

---

## 📝 12. SQL QUICK SYNTAX REFERENCE

```sql
-- Basic query structure order of WRITING vs EXECUTION
-- WRITE order:  SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY
-- EXEC order:   FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY

SELECT col1, COUNT(*) FROM table
WHERE condition
GROUP BY col1
HAVING COUNT(*) > 1
ORDER BY col1 DESC
LIMIT 10;

-- Subquery
SELECT name FROM Emp WHERE salary > (SELECT AVG(salary) FROM Emp);

-- Find Nth highest salary
SELECT DISTINCT salary FROM Emp
ORDER BY salary DESC
LIMIT 1 OFFSET (N-1);

-- Using window function (common interview ask)
SELECT name, salary,
DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
FROM Emp;

-- Find duplicate rows
SELECT name, COUNT(*) FROM Emp
GROUP BY name HAVING COUNT(*) > 1;

-- Delete duplicates keeping one
DELETE FROM Emp WHERE id NOT IN (
  SELECT MIN(id) FROM Emp GROUP BY name
);
```

---

## ⚠️ 13. COMMON EXAM/INTERVIEW TRAPS

- ❌ Wrong: Confusing DELETE (DML, rollback-able) with DROP/TRUNCATE (DDL) → ✅ Right: DELETE can use WHERE + rollback, DROP/TRUNCATE cannot.
- ❌ Wrong: Thinking WHERE can filter aggregated results → ✅ Right: Use **HAVING** for filtering after GROUP BY, WHERE filters before.
- ❌ Wrong: Believing 3NF automatically implies BCNF → ✅ Right: BCNF is stricter; a table can be in 3NF but not BCNF.
- ❌ Wrong: Foreign Key must always be unique → ✅ Right: FK can have duplicate values; only PK must be unique.
- ❌ Wrong: NULL = NULL evaluates to TRUE → ✅ Right: NULL comparisons return UNKNOWN; use IS NULL.
- ❌ Wrong: Index always improves performance → ✅ Right: Indexes speed up SELECT but slow down INSERT/UPDATE/DELETE.
- ❌ Wrong: Thinking 2PL prevents deadlocks → ✅ Right: 2PL guarantees serializability, NOT deadlock-freedom.
- ❌ Wrong: View stores data physically → ✅ Right: View is virtual; only the query is stored (unless materialized view).

---

## 🃏 14. MNEMONICS & MEMORY TRICKS

- 💡 **ACID** → "Atomic Changes Isolate Durably"
- 💡 **1NF→5NF** → "Atomic, Partial, Transitive, BCNF-key, Multivalued, Join"
- 💡 **JOIN types** → "I Left the Right, Full circle" (Inner, Left, Right, Full)
- 💡 **DDL/DML/DCL/TCL** →
  - DDL = **D**efine (CREATE, ALTER, DROP, TRUNCATE)
  - DML = **M**anipulate (SELECT, INSERT, UPDATE, DELETE)
  - DCL = **C**ontrol access (GRANT, REVOKE)
  - TCL = **T**ransaction control (COMMIT, ROLLBACK, SAVEPOINT)
- 💡 **Isolation levels ladder** → "Read Uncommitted < Read Committed < Repeatable Read < Serializable" (increasing safety, decreasing concurrency)

---

## 📝 15. TOP INTERVIEW Q&A (rapid fire)

**Q: Difference between Primary Key and Unique Key?**
📝 PK: no NULLs, only 1 per table, used for identification. Unique Key: allows 1 NULL, multiple per table.

**Q: What is a Deadlock and how to prevent it?**
📝 Circular wait for locks. Prevent via: lock ordering, timeout, wait-die/wound-wait schemes, avoiding nested transactions.

**Q: What is Referential Integrity?**
📝 FK value must either be NULL or match an existing PK value in the referenced table.

**Q: Difference between Clustered and Non-Clustered Index?**
📝 Clustered physically sorts table data (1 per table); Non-clustered is a separate lookup structure (many per table).

**Q: What is a Join and when would you use each type?**
📝 INNER for common data, LEFT/RIGHT when you need all rows from one side even without a match, FULL for all data from both.

**Q: Explain Normalization with an example.**
📝 Split a table having (StudentID, StudentName, CourseID, CourseName) into Student(StudentID, StudentName) and Course(CourseID, CourseName) + Enrollment(StudentID, CourseID) to remove redundancy.

**Q: What's the difference between HAVING and WHERE?**
📝 WHERE filters rows before grouping; HAVING filters groups after GROUP BY/aggregation.

**Q: What is a Materialized View?**
📝 A view that physically stores query results, refreshed periodically — trades storage for speed (unlike a normal view).

**Q: Explain CAP theorem (often cross-asked with DBMS).**
📝 Distributed system can guarantee only 2 of 3: Consistency, Availability, Partition Tolerance.

---

## 🎯 16. LAST-MINUTE INTERVIEW TIPS

1. Always be ready to **write a query live** — practice JOIN, GROUP BY, subquery, and window functions on paper/whiteboard.
2. If asked "explain a concept," give **definition → example → real DB use case** structure — interviewers love examples.
3. For **normalization questions**, always be ready to normalize a sample denormalized table step-by-step (1NF→2NF→3NF).
4. Know **indexing tradeoffs** — interviewers often ask "would you add an index here?" — mention read vs write tradeoff.
5. Be ready to explain **ACID with a real transaction example** (e.g., bank transfer: debit + credit as one atomic unit).
6. If you don't know exact SQL syntax, **explain the logic first** — interviewers value approach over perfect syntax.
7. Revise **DBMS vs RDBMS** and **SQL vs NoSQL** — very common opening questions.
8. Mention **real-world DB** you've used (MySQL/PostgreSQL/MongoDB) to sound practical, not just theoretical.

---

## 🔑 17. ONE-GLANCE SUMMARY BOX

```
┌───────────────────────────────────────────────────────────┐
│  🔑 MUST-KNOW ESSENTIALS — DBMS PLACEMENT                  │
│  1. ACID = Atomicity, Consistency, Isolation, Durability   │
│  2. Normal Forms: 1NF(atomic)→2NF(no partial)→             │
│     3NF(no transitive)→BCNF(every determinant is CK)       │
│  3. PK: unique+not null | FK: refers to PK, can duplicate  │
│  4. JOINS: Inner(match)/Left/Right/Full/Self/Cross         │
│  5. Isolation levels: Read Uncommitted→Committed→          │
│     Repeatable Read→Serializable                           │
│  6. DELETE(DML,rollback) vs TRUNCATE vs DROP(DDL, no roll) │
│  7. Clustered index = 1/table, sorts data physically       │
│  8. Deadlock = circular wait; solved by 2PL + wait schemes │
│  9. WHERE filters rows, HAVING filters groups               │
│  10. View=virtual query, Materialized View=stored result   │
└───────────────────────────────────────────────────────────┘
```

---

*Want me to go deeper on any section (e.g., detailed normalization examples, transaction scheduling problems, or a mock DBMS interview Q&A round), or create a practice quiz to test yourself?*
