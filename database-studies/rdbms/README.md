# Relational Database Management Systems (RDBMS) & PostgreSQL Guide

Welcome to the comprehensive RDBMS guide using **PostgreSQL**. This curriculum covers core theoretical database concepts paired with practical, production-ready PostgreSQL examples, progressing from foundational to intermediate/advanced topics.

---

## 🗺️ Learning Roadmap & Table of Contents

```
database-studies/rdbms/
├── db-setup.md
├── 01-introduction-and-core-concepts.md
├── 02-ddl-dml-and-postgresql-data-types.md
├── 03-querying-filtering-and-joins.md
├── 04-aggregations-grouping-and-subqueries.md
├── 05-window-functions-and-advanced-analytics.md
├── 06-indexing-query-performance-and-execution-plans.md
└── 07-transactions-concurrency-and-locks.md
```

### Module Breakdown

| Module | Topics Covered |
|---|---|
| [**01. Introduction & Core Concepts**](file:///workspaces/guidance-sde/database-studies/rdbms/01-introduction-and-core-concepts.md) | Relational Model, Codd's Rules, Tables/Tuples/Attributes, ACID Properties, Keys & Constraints, Normalization (1NF, 2NF, 3NF, BCNF) vs Denormalization. |
| [**02. DDL, DML & PostgreSQL Data Types**](file:///workspaces/guidance-sde/database-studies/rdbms/02-ddl-dml-and-postgresql-data-types.md) | Core Data Types (Numeric, String, Timestamp with Time Zone, JSONB, Arrays, UUID), DDL (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`), Identity columns, DML (`INSERT`, `UPDATE`, `DELETE`, `UPSERT` via `ON CONFLICT`, `RETURNING`). |
| [**03. Querying, Filtering & Joins**](file:///workspaces/guidance-sde/database-studies/rdbms/03-querying-filtering-and-joins.md) | `SELECT`, `WHERE` filtering, Pattern matching (`LIKE`, `ILIKE`, Regex), Keyset vs Offset Pagination, `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, `CROSS`, `SELF`, and `LATERAL` Joins, Set Operations (`UNION`, `INTERSECT`, `EXCEPT`). |
| [**04. Aggregations, Grouping & Subqueries**](file:///workspaces/guidance-sde/database-studies/rdbms/04-aggregations-grouping-and-subqueries.md) | Aggregate functions (`COUNT`, `SUM`, `AVG`, `STRING_AGG`, `ARRAY_AGG`), `GROUP BY`, `HAVING`, `ROLLUP`, `CUBE`, Subqueries (`EXISTS`, `IN`, `ANY`, `ALL`), Common Table Expressions (CTEs), Recursive CTEs. |
| [**05. Window Functions & Analytics**](file:///workspaces/guidance-sde/database-studies/rdbms/05-window-functions-and-advanced-analytics.md) | `OVER (PARTITION BY ... ORDER BY ... [FRAME])`, Ranking (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`), Value navigation (`LAG`, `LEAD`, `FIRST_VALUE`), Running totals & Moving averages. |
| [**06. Indexing & Query Performance**](file:///workspaces/guidance-sde/database-studies/rdbms/06-indexing-query-performance-and-execution-plans.md) | Index Types (B-Tree, GIN, GiST, BRIN, Hash), Single vs Composite Indexes (Column order rule), Partial & Expression Indexes, Reading `EXPLAIN (ANALYZE, BUFFERS)` plans, Index Scan vs Bitmap Index Scan vs Seq Scan. |
| [**07. Transactions, Concurrency & Locks**](file:///workspaces/guidance-sde/database-studies/rdbms/07-transactions-concurrency-and-locks.md) | `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, Isolation Levels (Read Committed, Repeatable Read, Serializable), Concurrency anomalies, Row-level locking (`FOR UPDATE`, `FOR SHARE`, `SKIP LOCKED`), Deadlock handling. |

---

## 🚀 Running Examples in the PostgreSQL DevContainer

If you have the dev container running, connect to PostgreSQL in your terminal:

```bash
# Connect using psql inside the app container
psql -h localhost -U postgres -d postgres
# Password is: postgres
```

Or connect via any GUI client (e.g. DBeaver, pgAdmin, TablePlus, VS Code PostgreSQL extension) at:
`postgresql://postgres:postgres@localhost:5432/postgres`
