# Module 04: Aggregations, Grouping & Subqueries

## 1. Aggregate Functions

PostgreSQL provides standard aggregates plus powerful array and string aggregation:

```sql
SELECT 
    COUNT(*) AS total_employees,
    COUNT(dept_id) AS employees_with_dept, -- Ignores NULLs
    SUM(salary) AS total_payroll,
    ROUND(AVG(salary), 2) AS average_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    STRING_AGG(first_name, ', ' ORDER BY first_name) AS employee_names,
    ARRAY_AGG(salary ORDER BY salary DESC) AS salaries_array
FROM employees;
```

---

## 2. Grouping: `GROUP BY` and `HAVING`

- `WHERE` filters rows **before** aggregation.
- `HAVING` filters aggregated groups **after** aggregation.

```sql
SELECT 
    d.dept_name,
    COUNT(e.emp_id) AS head_count,
    ROUND(AVG(e.salary), 2) AS avg_salary
FROM departments d
JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name
HAVING COUNT(e.emp_id) >= 2 AND AVG(e.salary) > 80000;
```

### Advanced Grouping: `ROLLUP` & `CUBE`

Calculate subtotals and grand totals automatically:

```sql
SELECT 
    d.dept_name,
    EXTRACT(YEAR FROM e.hired_at) AS hire_year,
    SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id
GROUP BY ROLLUP (d.dept_name, EXTRACT(YEAR FROM e.hired_at));
```

---

## 3. Subqueries & Predicates

### 1. `EXISTS` vs. `IN` (Semi-Joins)

```sql
-- Using EXISTS (Optimized by PostgreSQL query planner via Semi-Join)
SELECT d.dept_name 
FROM departments d
WHERE EXISTS (
    SELECT 1 
    FROM employees e 
    WHERE e.dept_id = d.dept_id AND e.salary > 100000
);

-- Using IN
SELECT dept_name 
FROM departments 
WHERE dept_id IN (
    SELECT dept_id FROM employees WHERE salary > 100000
);
```

### 2. Scalar Subqueries in `SELECT`

```sql
SELECT 
    e.first_name,
    e.salary,
    (SELECT ROUND(AVG(salary), 2) FROM employees) AS company_avg_salary,
    e.salary - (SELECT ROUND(AVG(salary), 2) FROM employees) AS diff_from_avg
FROM employees e;
```

---

## 4. Common Table Expressions (CTEs)

CTEs (`WITH` clauses) break complex queries into readable, modular steps.

### Standard CTE
```sql
WITH high_earners AS (
    SELECT emp_id, first_name, last_name, salary, dept_id
    FROM employees
    WHERE salary >= 100000
),
dept_stats AS (
    SELECT dept_id, COUNT(*) AS high_earner_count
    FROM high_earners
    GROUP BY dept_id
)
SELECT d.dept_name, COALESCE(s.high_earner_count, 0) AS high_earners
FROM departments d
LEFT JOIN dept_stats s ON d.dept_id = s.dept_id;
```

### Recursive CTEs (Graph & Hierarchy Traversal)

Recursive CTEs are essential for navigating trees, organization charts, and bill-of-materials:

```sql
-- Walk down the management hierarchy starting from Alice (emp_id = 1)
WITH RECURSIVE org_chart AS (
    -- Anchor member: base query
    SELECT 
        emp_id, 
        first_name || ' ' || last_name AS full_name, 
        manager_id, 
        1 AS level,
        first_name::TEXT AS path
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive member: joins back to org_chart
    SELECT 
        e.emp_id, 
        e.first_name || ' ' || e.last_name, 
        e.manager_id, 
        o.level + 1,
        o.path || ' -> ' || e.first_name
    FROM employees e
    INNER JOIN org_chart o ON e.manager_id = o.emp_id
)
SELECT level, full_name, path
FROM org_chart
ORDER BY level, full_name;
```
