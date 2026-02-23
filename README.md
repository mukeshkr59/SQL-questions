# SQL-questions

## 🟢 EASY LEVEL (Q1–Q7)

**Q1. What is the difference between DELETE, TRUNCATE, and DROP?**

"DELETE is a DML command that removes rows one by one based on a WHERE condition, and it can be rolled back because it logs each deletion. TRUNCATE is a DDL command that removes all rows at once — it's faster than DELETE, but you typically can't roll it back in most databases. DROP removes the entire table structure along with its data — the table simply ceases to exist after DROP.
For example:

```sql
DELETE FROM employees WHERE id = 5; — removes only one row, rollback possible.
TRUNCATE TABLE employees; — wipes all rows instantly.
DROP TABLE employees; — the table itself is gone."
```

**Q2. What is a Primary Key and how is it different from a Unique Key?**

"A Primary Key uniquely identifies every row in a table. It cannot be NULL and a table can have only one Primary Key. A Unique Key also enforces uniqueness, but it can accept one NULL value (in most databases like MySQL and SQL Server), and a table can have multiple Unique Keys.
Example:

```````sql
CREATE TABLE users (
  user_id INT PRIMARY KEY,        -- only one, no NULL
  email VARCHAR(100) UNIQUE,      -- can be NULL once
  phone VARCHAR(15) UNIQUE        -- multiple unique keys allowed
);
````````


**Q3. What is a Foreign Key?**

"A Foreign Key is a column in one table that references the Primary Key of another table. It enforces referential integrity — meaning you can't insert a value in the child table that doesn't exist in the parent table.

Example:
```````sql
CREATE TABLE orders (
  order_id INT PRIMARY KEY,
  customer_id INT,
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```````
If `customer_id = 99` doesn't exist in the `customers` table, the insert will be rejected."

---

**Q4. What is the difference between WHERE and HAVING?**

"WHERE filters rows before grouping happens, while HAVING filters after the GROUP BY aggregation.

Example:
```````sql
-- WHERE filters individual rows
SELECT department, COUNT(*) 
FROM employees
WHERE salary > 50000
GROUP BY department;

-- HAVING filters grouped results
SELECT department, COUNT(*) 
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
```````
A common mistake I've seen is trying to use an aggregate function like COUNT() inside WHERE — that will throw an error. HAVING is the right place for that."

---

**Q5. What are the different types of JOINs?**

"There are four main types:

- **INNER JOIN** — returns only rows where there's a match in both tables.
- **LEFT JOIN** — returns all rows from the left table and matched rows from the right; unmatched right rows are NULL.
- **RIGHT JOIN** — opposite of LEFT JOIN.
- **FULL OUTER JOIN** — returns all rows from both tables; NULLs fill in where there's no match.

Example:
```````sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```````
This returns all employees even if they haven't been assigned a department yet."

---

**Q6. What is normalization? What are 1NF, 2NF, and 3NF?**

"Normalization is the process of organizing a database to reduce redundancy and improve data integrity.

- **1NF (First Normal Form):** Each column must hold atomic (indivisible) values, and each row must be unique. No repeating groups or arrays in a column.
- **2NF:** Must be in 1NF, and every non-key attribute must be fully dependent on the entire primary key — not just part of it. This applies when you have a composite key.
- **3NF:** Must be in 2NF, and no transitive dependency — meaning a non-key column should not depend on another non-key column.

Example of a 3NF violation: If a table has `employee_id`, `department_id`, and `department_name`, then `department_name` depends on `department_id`, not on `employee_id`. That's a transitive dependency — we'd split it into a separate `departments` table."

---

**Q7. What is the difference between CHAR and VARCHAR?**

"CHAR is a fixed-length data type. If I declare `CHAR(10)` and store 'hello', it pads it with spaces to fill all 10 characters. VARCHAR is variable-length — `VARCHAR(10)` storing 'hello' will only use 5 bytes of storage.

CHAR is faster for fixed-size data like country codes or gender flags ('M'/'F'). VARCHAR is more storage-efficient for variable-length strings like names or addresses."

---

## 🟡 INTERMEDIATE LEVEL (Q8–Q14)

---

**Q8. What are indexes and how do they improve performance?**

"An index is a data structure — typically a B-Tree — that the database builds on one or more columns to speed up data retrieval. Instead of doing a full table scan, the query engine can jump directly to the relevant rows.

Example:
```````sql
CREATE INDEX idx_employee_name ON employees(last_name);
```````
Now a query like `SELECT * FROM employees WHERE last_name = 'Smith'` is much faster.

The trade-off is that indexes slow down INSERT, UPDATE, and DELETE operations because the index also needs to be updated. So I always recommend indexing columns that are frequently used in WHERE clauses, JOINs, or ORDER BY — not every column blindly."

---

**Q9. What is a subquery vs a JOIN? When would you prefer one over the other?**

"A subquery is a query nested inside another query. A JOIN combines rows from two tables based on a related column.

Example using subquery:
```````sql
SELECT name FROM employees
WHERE dept_id = (SELECT dept_id FROM departments WHERE dept_name = 'Engineering');
```````

Same result using JOIN:
```````sql
SELECT e.name FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
WHERE d.dept_name = 'Engineering';
```````

I prefer JOINs for performance in most cases because the query optimizer handles them better. Subqueries are more readable sometimes, and correlated subqueries are useful when you need row-by-row comparison. For large datasets, I'd almost always benchmark both."

---

**Q10. What are window functions? Give an example.**

"Window functions perform calculations across a set of rows related to the current row, without collapsing them into groups like GROUP BY does. They're one of my favorite features in modern SQL.

Common ones: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`, `LEAD()`, `SUM() OVER()`.

Example — rank employees by salary within each department:
```````sql
SELECT name, department, salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```````
This gives each employee a rank within their department without losing any rows from the result set. I use this heavily in reporting and analytics queries."

---

**Q11. What is the difference between RANK(), DENSE_RANK(), and ROW_NUMBER()?**

"All three are window functions that assign numbers to rows, but they handle ties differently.

- `ROW_NUMBER()` — assigns a unique sequential number regardless of ties. No two rows get the same number.
- `RANK()` — same rank for ties, but then skips numbers. So if two people tie for rank 2, the next rank is 4.
- `DENSE_RANK()` — same rank for ties, but does NOT skip. If two people tie for rank 2, the next rank is 3.

Example:
```````sql
SELECT name, salary,
  ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn,
  RANK()       OVER (ORDER BY salary DESC) AS rnk,
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rnk
FROM employees;
```````
If salaries are 90k, 90k, 80k:
- ROW_NUMBER: 1, 2, 3
- RANK: 1, 1, 3
- DENSE_RANK: 1, 1, 2"

---

**Q12. What are CTEs (Common Table Expressions) and when do you use them?**

"A CTE is a temporary named result set defined with the `WITH` keyword. It improves readability and can be referenced multiple times within the same query.

Example:
```````sql
WITH high_earners AS (
  SELECT * FROM employees WHERE salary > 100000
)
SELECT department, COUNT(*) 
FROM high_earners
GROUP BY department;
```````

I use CTEs when a subquery would be repeated or nested too deeply. They make complex queries much more readable. Recursive CTEs are especially powerful for hierarchical data like org charts or category trees:
```````sql
WITH RECURSIVE org_chart AS (
  SELECT id, name, manager_id FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employees e
  JOIN org_chart o ON e.manager_id = o.id
)
SELECT * FROM org_chart;
``````"

---

**Q13. What is a transaction and what are ACID properties?**

"A transaction is a unit of work that must either complete fully or not at all. ACID stands for:

- **Atomicity:** All operations in a transaction succeed or all are rolled back. Like a bank transfer — debit AND credit must both happen.
- **Consistency:** The database moves from one valid state to another. Constraints and rules are never violated.
- **Isolation:** Concurrent transactions don't interfere with each other. The intermediate state of one transaction is invisible to others.
- **Durability:** Once committed, the changes persist even if the system crashes — thanks to write-ahead logging.

Example:
`````sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
-- If anything fails between, ROLLBACK keeps both accounts intact.
````"

---

**Q14. What are different types of indexes — clustered vs non-clustered?**

"A **clustered index** determines the physical order of data in the table. There can be only one per table, and in SQL Server, the Primary Key is clustered by default. The actual data rows are stored in the order of the clustered index.

A **non-clustered index** is a separate structure that holds a pointer back to the actual data rows. You can have many non-clustered indexes on a table.

Analogy I like to use: A clustered index is like a phone book sorted by last name — the data itself is ordered. A non-clustered index is like the index at the back of a textbook — it points you to where the real content is.

For queries that retrieve a large range of rows, a clustered index performs better. For highly selective point lookups on non-primary-key columns, non-clustered indexes are ideal."

---

## 🔴 ADVANCED LEVEL (Q15–Q20)

---

**Q15. Explain query execution order in SQL.**

"This is something I always clarify because people write SQL in one order but it executes in a completely different order. The logical execution sequence is:

1. **FROM** — identify the tables and apply JOINs
2. **WHERE** — filter rows
3. **GROUP BY** — group the filtered rows
4. **HAVING** — filter groups
5. **SELECT** — evaluate expressions and select columns
6. **DISTINCT** — remove duplicates
7. **ORDER BY** — sort the result
8. **LIMIT/OFFSET** — return the final rows

This is why you can't reference a SELECT alias in a WHERE clause — WHERE runs before SELECT. And why aggregate functions can't be in WHERE — they're evaluated at the GROUP BY/HAVING stage."

---

**Q16. What are isolation levels in database transactions?**

"Isolation levels control how much a transaction is exposed to the effects of other concurrent transactions. There are four standard levels defined by SQL standard:

- **Read Uncommitted:** A transaction can read data that hasn't been committed yet. This causes **dirty reads** — you might read data that gets rolled back.
- **Read Committed:** Only committed data can be read. Prevents dirty reads, but **non-repeatable reads** can occur — reading the same row twice in a transaction may give different results if another transaction commits between the reads.
- **Repeatable Read:** Guarantees that if you read a row once, it won't change within the same transaction. But **phantom reads** can occur — new rows inserted by another transaction may appear.
- **Serializable:** The strictest level. Transactions execute as if they were serial (one after another). Prevents all anomalies but has the highest performance cost.

In my experience, most production systems run on **Read Committed** (SQL Server default) or **Repeatable Read** (MySQL InnoDB default) as a balance between safety and performance."

---

**Q17. What is query optimization? How do you approach a slow query?**

"When I encounter a slow query, I follow a systematic approach:

**Step 1 — Use EXPLAIN / EXPLAIN ANALYZE:** This shows the query execution plan. I look for full table scans (`Seq Scan` in PostgreSQL or `ALL` in MySQL EXPLAIN) on large tables.

**Step 2 — Check indexes:** Is the column in the WHERE or JOIN clause indexed? Is the index being used, or is it being ignored due to a function wrapping the column?

Example of index being bypassed:
```sql
-- Bad: function on column kills the index
WHERE YEAR(created_at) = 2024

-- Good: range condition uses the index
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'
```

**Step 3 — Avoid SELECT *:** Fetching unnecessary columns increases I/O.

**Step 4 — Optimize JOINs:** Ensure join columns are indexed. Watch for Cartesian products.

**Step 5 — Consider query rewrite:** Sometimes a correlated subquery can be rewritten as a JOIN for massive performance gain.

**Step 6 — Partitioning / Archiving:** For very large tables, I look into table partitioning so queries only scan relevant partitions."

---

**Q18. What is database sharding and how does it differ from replication?**

"Both are horizontal scaling strategies but they serve different purposes.

**Replication** copies the same data to multiple nodes. You have a primary (write) node and one or more replicas (read) nodes. It improves read performance and provides high availability — if the primary fails, a replica can be promoted. But every node holds the full dataset.

**Sharding** splits the dataset across multiple nodes called shards. Each shard holds a subset of the data. For example, users with IDs 1–1M go to Shard A, users 1M–2M go to Shard B. This improves both read and write scalability since data and load are distributed.

The challenge with sharding is **cross-shard queries** — if you need data spanning multiple shards, it becomes complex and expensive. Choosing the right shard key is critical. A poor shard key leads to hotspots where one shard gets overloaded.

In practice, I'd recommend replication first for read-heavy workloads, and consider sharding only when a single node's write throughput becomes the bottleneck."

---

**Q19. What are deadlocks in a database and how do you prevent them?**

"A deadlock occurs when two or more transactions are waiting for each other to release locks, creating a circular dependency that neither can escape.

Classic Example:
````
Transaction A locks Row 1, wants Row 2.
Transaction B locks Row 2, wants Row 1.
Both wait forever → deadlock.
`````

How I approach preventing deadlocks:

**1. Consistent lock ordering:** Always acquire locks in the same order across all transactions. If every transaction locks Row 1 before Row 2, the cycle can never form.

**2. Keep transactions short:** The longer a transaction holds a lock, the higher the chance of conflict. Do as little work as possible inside a transaction.

**3. Use lower isolation levels where appropriate:** Read Committed reduces the scope of locks compared to Serializable.

**4. Timeout and retry logic:** Set a lock timeout so a transaction doesn't wait indefinitely. Implement retry logic in the application layer when a deadlock is detected.

**5. Deadlock detection:** Most modern databases (MySQL, PostgreSQL, SQL Server) have built-in deadlock detection and will automatically kill the transaction with the lower cost to resolve it, returning an error code that the application can catch and retry."

---

**Q20. What is the difference between SQL and NoSQL databases? When would you choose one over the other?**

"This is a fundamental architectural decision I've had to make in real projects.

**SQL (Relational) databases** like PostgreSQL, MySQL, and SQL Server store data in structured tables with a fixed schema. They excel at complex queries with JOINs, enforce ACID transactions strongly, and are ideal when data relationships are well-defined.

**NoSQL databases** like MongoDB (document), Cassandra (wide-column), Redis (key-value), and Neo4j (graph) offer flexible schemas, horizontal scalability, and high throughput for specific access patterns.

**I'd choose SQL when:**
- Data has clear relationships (e.g., e-commerce: orders, products, customers)
- Strong consistency is required (e.g., financial transactions)
- Complex ad-hoc queries and reporting are needed

**I'd choose NoSQL when:**
- Schema is dynamic or evolving rapidly (e.g., a product catalog with varied attributes)
- I need massive write throughput at scale (e.g., logging, IoT telemetry — Cassandra is great here)
- Data is naturally document-oriented (e.g., user profiles in MongoDB)
- I need sub-millisecond lookups (e.g., session caching in Redis)

In modern systems, I often use **both** — a PostgreSQL database for transactional core data and Redis for caching, or MongoDB for flexible product data alongside a relational DB for orders and billing. This is called polyglot persistence."

---

## Quick Reference Table

| # | Question | Level |
|---|----------|-------|
| 1 | DELETE vs TRUNCATE vs DROP | 🟢 Easy |
| 2 | Primary Key vs Unique Key | 🟢 Easy |
| 3 | Foreign Key | 🟢 Easy |
| 4 | WHERE vs HAVING | 🟢 Easy |
| 5 | Types of JOINs | 🟢 Easy |
| 6 | Normalization (1NF, 2NF, 3NF) | 🟢 Easy |
| 7 | CHAR vs VARCHAR | 🟢 Easy |
| 8 | Indexes & Performance | 🟡 Intermediate |
| 9 | Subquery vs JOIN | 🟡 Intermediate |
| 10 | Window Functions | 🟡 Intermediate |
| 11 | RANK vs DENSE_RANK vs ROW_NUMBER | 🟡 Intermediate |
| 12 | CTEs | 🟡 Intermediate |
| 13 | Transactions & ACID | 🟡 Intermediate |
| 14 | Clustered vs Non-Clustered Index | 🟡 Intermediate |
| 15 | SQL Execution Order | 🔴 Advanced |
| 16 | Isolation Levels | 🔴 Advanced |
| 17 | Query Optimization | 🔴 Advanced |
| 18 | Sharding vs Replication | 🔴 Advanced |
| 19 | Deadlocks & Prevention | 🔴 Advanced |
| 20 | SQL vs NoSQL | 🔴 Advanced |

---

Practice delivering these out loud — interviewers are as much evaluating *how clearly you communicate* as whether you know the answer. Good luck!
