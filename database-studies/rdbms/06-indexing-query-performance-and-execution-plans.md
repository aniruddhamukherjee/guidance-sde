# Module 06: Indexing, Query Performance & Execution Plans

## 1. PostgreSQL Index Types

Indexes are specialized data structures that speed up data retrieval at the cost of additional storage and slower `INSERT`/`UPDATE` operations.

| Index Method | Best Used For | PostgreSQL Example Syntax |
|---|---|---|
| **B-Tree** (Default) | Equality (`=`) and Range (`<`, `<=`, `>`, `>=`, `BETWEEN`) | `CREATE INDEX idx_emp_salary ON employees(salary);` |
| **GIN** (Generalized Inverted Index) | Semi-structured arrays, JSONB keys, Full-Text Search | `CREATE INDEX idx_prod_meta ON products USING GIN(metadata);` |
| **GiST** (Generalized Search Tree) | Geometric, spatial (PostGIS), ranges (`tsrange`) | `CREATE INDEX idx_locations ON points USING GIST(geom);` |
| **BRIN** (Block Range Index) | Huge naturally ordered append-only tables (time-series) | `CREATE INDEX idx_logs_ts ON logs USING BRIN(logged_at);` |
| **Hash** | Exact equality lookups only (`=`) | `CREATE INDEX idx_users_token ON tokens USING HASH(token);` |

---

## 2. Advanced Index Strategies

### 1. Composite Indexes & The Leftmost Prefix Rule
```sql
CREATE INDEX idx_emp_dept_salary ON employees(dept_id, salary);
```
- ✅ Fast for: `WHERE dept_id = 1 AND salary > 50000`
- ✅ Fast for: `WHERE dept_id = 1`
- ❌ **Cannot** use this index for: `WHERE salary > 50000` (does not use leftmost column).

### 2. Partial (Filtered) Indexes
Reduces index size by indexing only the subset of rows you query often:
```sql
-- Indexes only active orders instead of millions of archived ones
CREATE INDEX idx_orders_active_unprocessed 
ON orders(customer_id) 
WHERE status = 'unprocessed';
```

### 3. Expression (Functional) Indexes
Indexes the computed output of a function:
```sql
-- Speeds up case-insensitive lookups
CREATE INDEX idx_users_lower_email 
ON employees (LOWER(email));

-- Matches this query:
SELECT * FROM employees WHERE LOWER(email) = 'alice@example.com';
```

### 4. Covering Indexes (`INCLUDE` Clause)
Performs **Index-Only Scans** without visiting heap pages:
```sql
CREATE INDEX idx_emp_dept_inc_salary 
ON employees(dept_id) 
INCLUDE (salary, first_name);
```

---

## 3. Reading PostgreSQL `EXPLAIN` and `EXPLAIN ANALYZE`

To understand how PostgreSQL executes queries:

```sql
EXPLAIN (ANALYZE, BUFFERS, COSTS, VERBOSE)
SELECT e.first_name, e.salary, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > 90000;
```

### Key Execution Node Types:
1. **Sequential Scan (`Seq Scan`)**: Reads every block in the table sequentially. Normal for small tables; bad for large tables filtered on selective conditions.
2. **Index Scan**: Traverses the B-tree to find matching tuple pointers, then fetches each row from table storage.
3. **Index-Only Scan**: All requested columns exist directly in the index; table heap pages are never read.
4. **Bitmap Index Scan + Bitmap Heap Scan**: Gathers row locations into an in-memory bitmap before fetching rows to ensure sequential disk access.
5. **Join Nodes**:
   - `Nested Loop`: Efficient for small datasets or when one side is indexed.
   - `Hash Join`: Builds an in-memory hash table of the smaller table, scans the larger table.
   - `Merge Join`: Both inputs are pre-sorted; fast for large joined sets.

---

## 4. Query Anti-Patterns to Avoid

| Anti-Pattern | Why It Hurts | Recommended Fix |
|---|---|---|
| `SELECT *` | Fetches unnecessary columns, prevents Index-Only Scans | Explicitly list columns: `SELECT id, name` |
| Function wrapped on indexed column: `WHERE YEAR(created_at) = 2026` | Invalidates normal index on `created_at` | Range filter: `WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'` |
| Leading wildcard: `WHERE name LIKE '%tech'` | Forces a full Seq Scan | Use Full-Text Search or `pg_trgm` GIN index |
| Unindexed Foreign Keys | Slows down joins and cascading deletes/updates | Always create indexes on FK columns |
