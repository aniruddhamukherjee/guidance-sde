# Module 03: Querying, Filtering & Joins

## Sample Schema Setup

```sql
CREATE TABLE departments (
    dept_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    dept_name TEXT NOT NULL UNIQUE
);

CREATE TABLE employees (
    emp_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name TEXT NOT NULL,
    last_name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    salary NUMERIC(10, 2) NOT NULL,
    dept_id INT REFERENCES departments(dept_id),
    manager_id INT REFERENCES employees(emp_id),
    hired_at DATE NOT NULL
);

INSERT INTO departments (dept_name) VALUES ('Engineering'), ('Sales'), ('HR'), ('Marketing');

INSERT INTO employees (first_name, last_name, email, salary, dept_id, manager_id, hired_at)
VALUES
    ('Alice', 'Smith', 'alice@example.com', 120000, 1, NULL, '2021-03-15'),
    ('Bob', 'Jones', 'bob@example.com', 95000, 1, 1, '2022-06-01'),
    ('Charlie', 'Brown', 'charlie@example.com', 85000, 2, NULL, '2020-01-10'),
    ('Diana', 'Prince', 'diana@example.com', 105000, 2, 3, '2023-08-20'),
    ('Evan', 'Wright', 'evan@example.com', 70000, NULL, NULL, '2024-02-01');
```

---

## 1. Advanced Filtering & Pattern Matching

### Text Matching: `LIKE`, `ILIKE`, and Regex

```sql
-- ILIKE: Case-insensitive pattern match (% = any chars, _ = single char)
SELECT emp_id, first_name, email 
FROM employees 
WHERE email ILIKE '%@example.com' AND first_name LIKE 'A%';

-- POSIX Regular Expressions in Postgres:
-- ~ (case-sensitive regex match), ~* (case-insensitive regex match)
SELECT first_name, last_name 
FROM employees 
WHERE last_name ~* '^(smith|brown)$';
```

### Handling `NULL` Values Correctly

In SQL, `NULL` represents unknown value; comparisons with `= NULL` always evaluate to `UNKNOWN` (falsy):

```sql
-- Select employees with no department assigned
SELECT first_name, last_name 
FROM employees 
WHERE dept_id IS NULL;

-- COALESCE: Returns first non-null argument
SELECT first_name, COALESCE(dept_id::TEXT, 'Unassigned') AS department
FROM employees;
```

---

## 2. Pagination: Offset vs. Keyset Pagination

### Offset Pagination (Traditional)
```sql
-- ⚠️ Inefficient on large tables because it scans and discards 10,000 rows
SELECT * FROM employees 
ORDER BY emp_id 
LIMIT 10 OFFSET 10000;
```

### Keyset (Cursor-based) Pagination (High Performance)
```sql
-- ⚡ Fast: Uses index on (emp_id) directly without scanning prior rows
SELECT * FROM employees 
WHERE emp_id > 10000 
ORDER BY emp_id ASC 
LIMIT 10;
```

---

## 3. Relational Joins in PostgreSQL

```
                Types of SQL Joins
  INNER JOIN         LEFT JOIN         RIGHT JOIN        FULL JOIN
   (Matching)     (All Left + Match) (All Right + Match)   (Union All)
    ┌───┬───┐         ┌───┬───┐         ┌───┬───┐         ┌───┬───┐
    │   █   │         █████   │         │   █████         █████████
    └───┴───┘         └───┴───┘         └───┴───┘         └───┴───┘
```

### 1. `INNER JOIN`
Returns only rows where matching keys exist in both tables.

```sql
SELECT e.first_name, e.last_name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id;
```

### 2. `LEFT OUTER JOIN`
Returns all rows from the left table (`employees`), with matching rows from the right (`departments`). Non-matches produce `NULL`s.

```sql
SELECT e.first_name, e.last_name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id;
```

### 3. `FULL OUTER JOIN`
Returns all rows when there is a match in either table (includes employees without departments AND departments with zero employees).

```sql
SELECT e.first_name, d.dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.dept_id = d.dept_id;
```

### 4. `SELF JOIN`
Joining a table to itself (e.g. hierarchical employee $\rightarrow$ manager relation):

```sql
SELECT 
    e.first_name || ' ' || e.last_name AS employee,
    COALESCE(m.first_name || ' ' || m.last_name, 'None (Top Executive)') AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;
```

### 5. `CROSS JOIN LATERAL` (Correlated Subquery Join)
PostgreSQL supports `LATERAL` joins which allow subqueries in the `FROM` clause to reference columns from previous `FROM` items:

```sql
-- Fetch the top 2 highest paid employees per department
SELECT d.dept_name, top_emp.first_name, top_emp.salary
FROM departments d
CROSS JOIN LATERAL (
    SELECT e.first_name, e.salary
    FROM employees e
    WHERE e.dept_id = d.dept_id
    ORDER BY e.salary DESC
    LIMIT 2
) top_emp;
```

---

## 4. Set Operations

Combine rows from multiple query results:

```sql
-- UNION: Distinct union (deduplicates)
SELECT email FROM employees
UNION
SELECT email FROM customers;

-- UNION ALL: Appends all rows without deduplication (faster)
SELECT email FROM employees
UNION ALL
SELECT email FROM customers;

-- INTERSECT: Only rows present in both queries
SELECT email FROM employees
INTERSECT
SELECT email FROM customers;

-- EXCEPT: Rows in first query that are not in second query
SELECT email FROM employees
EXCEPT
SELECT email FROM customers;
```
