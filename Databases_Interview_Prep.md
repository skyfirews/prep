# Databases (SQL, MySQL, PostgreSQL) – Interview Notes

> **Target**: Senior Engineers, Backend/Data | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: A database is an organized, persistent store of data. RDBMS (e.g. MySQL, PostgreSQL) store data in tables with rows and columns, enforced by schema, constraints, and ACID transactions.

**Why companies use it**: Structured data, relationships (joins), consistency, integrity (constraints, transactions), standard query language (SQL), mature tooling and ops.

**When NOT to use it**: Very high write throughput or schema-less evolution (consider NoSQL); heavy analytics over huge volume (consider warehouse); simple key-value (consider cache/NoSQL); real-time streaming (consider stream store).

---

## 2. Core Concepts (From PDF)

### Basics

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Database | Organized collection of data | Persistent store with structure | Why DB over files | ACID |
| Table | Rows and columns | Entity stored as rows | Schema design | Normalization |
| Primary Key | Unique row identifier | No duplicates; one per table | Index implication | Composite PK |
| Foreign Key | Reference to another table's PK | Enforces relationship | Referential integrity | ON DELETE behavior |
| Unique | No duplicate values in column(s) | e.g. email, username | Unique vs PK | Nullability |
| NOT NULL | Column must have a value | Data quality | Defaults | |
| Check | Value must satisfy expression | Business rules | When to use | |

### SQL & Queries

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| SELECT | Read data | Filter, project, join | Query optimization | EXPLAIN |
| JOIN (INNER) | Rows that match in both tables | Intersection | Most common join | Multiple JOINs |
| LEFT JOIN | All left + matching right | Keep left even if no match | Missing data, NULLs | RIGHT vs LEFT |
| RIGHT JOIN | All right + matching left | Keep right even if no match | Rarely used | Same as LEFT flipped |
| FULL JOIN | All from both | Union of left and right | PostgreSQL support | Use cases |
| WHERE | Filter rows | Before aggregation | Index usage | WHERE vs HAVING |
| GROUP BY | Aggregate by column(s) | One row per group | Mandatory with aggregates | GROUP BY columns |
| HAVING | Filter groups | After aggregation | HAVING vs WHERE | |
| ORDER BY | Sort result | ASC/DESC | Performance (index) | LIMIT |
| LIMIT / OFFSET | Pagination | Top N, skip M | Large offset cost | Cursor pagination |

### Indexes

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Index | Structure to speed up lookups | B-tree or hash; trade read vs write | When to index | Composite index |
| B-Tree | Default index type | Range and equality | Range queries | |
| Hash Index | Exact match only | No range (e.g. PostgreSQL) | When to use | |
| Composite Index | Multiple columns | Order matters (left prefix) | Query pattern match | Covering index |

### Transactions & Isolation

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Transaction | Unit of work (all or nothing) | BEGIN → COMMIT or ROLLBACK | ACID | Isolation levels |
| ACID | Atomicity, Consistency, Isolation, Durability | Core DB guarantee | What each letter means | |
| Isolation Level | How concurrent transactions see each other | Read Uncommitted → Serializable | Dirty read, phantom read | Default in PG/MySQL |
| Read Uncommitted | See uncommitted changes | Dirty reads | Rarely used | |
| Read Committed | See only committed (default in PG) | No dirty read | Row-level | |
| Repeatable Read | Same read result in transaction (MySQL default) | No non-repeatable read | Phantom read possible | |
| Serializable | Strictest | No phantom; serial order | Performance cost | |
| Deadlock | Circular wait on locks | Two txns wait for each other | Prevention, detection | |

### Normalization & Design

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| 1NF | Atomic values; no repeating groups | Each cell one value | Basics | |
| 2NF | 1NF + no partial dependency on PK | Non-key depends on full PK | Composite PK | |
| 3NF | 2NF + no transitive dependency | Non-key depends only on PK | Clean design | |
| Denormalization | Introduce redundancy | Fewer joins; more writes | When to use | Performance |

### Stored Objects & Performance

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| View | Virtual table (saved query) | Security, abstraction | Materialized vs view | |
| Materialized View | Stored result (refreshed) | PostgreSQL; analytics | Refresh strategy | |
| Stored Procedure | Precompiled SQL (MySQL) | Logic in DB | When to use | |
| Trigger | Run on event (insert/update/delete) | Audit, derived data | Side effects | |
| EXPLAIN | Show execution plan | Index use, full scan | Query tuning | EXPLAIN ANALYZE |

### Security & Operations

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| SQL Injection | Attacker injects SQL via input | Parameterized queries | Prevention | Prepared statement |
| GRANT / REVOKE | Permissions to users/roles | Least privilege | Production safety | |
| Backup / Restore | Copy data; recover from copy | DR, point-in-time | Strategy | |

### MySQL vs PostgreSQL (from PDF)

| Concept | MySQL | PostgreSQL |
|---------|--------|------------|
| Default engine | InnoDB (ACID, FK) | — |
| PK auto | AUTO_INCREMENT | SERIAL / IDENTITY |
| JSON | JSON type | JSON, JSONB (indexed) |
| CTE | Supported | WITH (recursive) |
| Window functions | Supported | Rich support |

---

## 3. How It Works (Internals)

**Query execution**  
Parse SQL → Validate → Optimizer (plan) → Executor. Executor uses indexes (index scan) or full table (seq scan). EXPLAIN shows the plan.

**Transaction**  
BEGIN starts a transaction; changes are visible to others only after COMMIT. ROLLBACK discards. Locks (row/table) protect concurrent access; isolation level defines what one txn can see of others.

**B-tree index**  
Sorted structure; lookup/range O(log n). Insert/delete update tree. Composite index (A, B): useful for WHERE A = ? and B = ?, or WHERE A = ? ORDER BY B.

**Word diagram**  
```
Client → [Parser] → [Optimizer] → [Executor] → [Storage]
                           ↓
                    [Index / Seq Scan]
```

---

## 4. Real-World Use Case

**E-commerce orders and inventory**

- **Where DB fits**: Orders table (order_id PK, user_id FK, status, created_at); order_items (order_id FK, product_id FK, qty); products (product_id PK, name, stock); users (user_id PK). Transactions for "place order" (deduct stock, insert order and items).
- **Why RDBMS**: Strong consistency for money and stock; joins for order history and reports; constraints (FK, check) and ACID.
- **Trade-offs**: Scale reads with replicas and caching; scale writes with partitioning or async; complex analytics offload to warehouse.

---

## 5. Code Example (Minimal but Powerful)

```python
# Safe query with parameterized SQL (Python – psycopg2)
import psycopg2

conn = psycopg2.connect("dbname=mydb user=app")
cur = conn.cursor()

# CORRECT: Parameterized – no SQL injection
user_id = request.get("user_id")  # From user input
cur.execute("SELECT id, name FROM users WHERE id = %s", (user_id,))
rows = cur.fetchall()

# WRONG: String concatenation – SQL injection
# cur.execute("SELECT * FROM users WHERE id = " + user_id)  # NEVER do this

# Transaction for consistency
try:
    cur.execute("UPDATE products SET stock = stock - %s WHERE id = %s", (qty, pid))
    cur.execute("INSERT INTO orders (user_id, product_id, qty) VALUES (%s, %s, %s)", (uid, pid, qty))
    conn.commit()
except Exception:
    conn.rollback()
finally:
    cur.close()
    conn.close()
# Time: O(1) for indexed lookup; space: O(result set). Always use parameterized queries.
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use parameterized queries (prepared statements) | Don't concatenate user input into SQL |
| Index columns in WHERE, JOIN, ORDER BY | Don't over-index (writes get slower) |
| Use transactions for multi-step writes | Don't hold transactions long (lock contention) |
| Prefer small, focused queries | Avoid SELECT * in production code |
| Use EXPLAIN for slow queries | Don't guess; measure |
| Backup and test restore | Don't assume backups work |

**Security**: Least privilege; no raw SQL from frontend; encrypt sensitive columns if needed.  
**Scalability**: Read replicas, connection pooling, query tuning, consider sharding for very high write.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "WHERE and HAVING are the same" | WHERE filters rows before aggregation; HAVING filters groups after | Use WHERE for row filter, HAVING for group filter |
| "More indexes = faster" | Indexes slow writes and use space | Index only columns used in predicates/joins/order |
| "No need for transactions for one query" | Multi-statement updates (e.g. order + stock) must be atomic | Use transaction for any multi-step consistency |
| "String concatenation for SQL is fine if we validate" | Validation is error-prone; one bug = injection | Always use parameterized queries |
| "LEFT JOIN and INNER JOIN are interchangeable" | LEFT keeps all left rows; INNER only matches | Choose by whether you need unmatched rows |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between WHERE and HAVING?**  
**Short answer**: WHERE filters rows before grouping; HAVING filters groups after GROUP BY. You use HAVING when the condition involves an aggregate (e.g. COUNT, SUM).  
**Follow-up**: "Can you use WHERE with an aggregate?"  
**Ideal answer**: No. WHERE is evaluated per row before aggregation. For conditions on aggregates (e.g. COUNT(*) > 1) use HAVING.

**Q: What is a primary key?**  
**Short answer**: A column (or set of columns) that uniquely identifies each row; NOT NULL and UNIQUE. One per table.  
**Follow-up**: "Can a table have two primary keys?"  
**Ideal answer**: One primary key, but it can be composite (multiple columns). You can have additional UNIQUE constraints.

### B. Intermediate (Most common)

**Q: Explain INNER JOIN vs LEFT JOIN.**  
**Short answer**: INNER JOIN returns only rows where both tables match. LEFT JOIN returns all rows from the left table and matching rows from the right; non-matching right side is NULL.  
**Follow-up**: "When would you use LEFT JOIN?"  
**Ideal answer**: When you need all rows from the "left" entity even if there's no related row (e.g. all customers and their orders; customers with no orders still appear).

**Q: How do you prevent SQL injection?**  
**Short answer**: Use parameterized queries (prepared statements). Never build SQL by concatenating user input.  
**Follow-up**: "Is input validation enough?"  
**Ideal answer**: No. Validation helps for business rules but not for injection. One bypass or mistake and you're vulnerable. Parameterized queries are the fix.

### C. Advanced / Tricky (Senior-level)

**Q: What are isolation levels and what problems does each solve?**  
**Short answer**: Read Uncommitted: dirty reads. Read Committed: no dirty reads. Repeatable Read: same row read twice in txn stays same. Serializable: no phantoms. Higher isolation = more consistency, more locking, lower concurrency.  
**Follow-up**: "What is a phantom read?"  
**Ideal answer**: In one transaction you run the same query twice and get different rows because another transaction inserted/deleted rows. Repeatable Read may still allow phantoms in some DBs; Serializable prevents it.

**Q: How would you optimize a slow query that does a full table scan?**  
**Ideal answer**: Run EXPLAIN to confirm full scan. Add an index on columns used in WHERE, JOIN, ORDER BY. Ensure index is used (no functions on column, type match). Consider composite index for multiple columns. If table is huge, consider partitioning or archiving.

---

## 9. Quick Revision

- **Primary Key** = unique, NOT NULL; **Foreign Key** = reference to another table.
- **WHERE** = row filter; **HAVING** = group filter (after GROUP BY).
- **INNER JOIN** = matching rows; **LEFT JOIN** = all left + match (NULL if no match).
- **Index** = faster read, slower write; **B-tree** for range; **composite** order matters.
- **ACID** = Atomicity, Consistency, Isolation, Durability.
- **Always use parameterized queries** – no SQL injection.
- **EXPLAIN** to see plan; index to avoid full scan.

---

## 10. References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MySQL Reference Manual](https://dev.mysql.com/doc/)
- [Use The Index, Luke](https://use-the-index-luke.com/) – SQL indexing
