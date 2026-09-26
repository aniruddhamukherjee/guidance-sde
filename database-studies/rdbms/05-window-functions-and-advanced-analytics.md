# Module 05: Window Functions & Advanced Analytics

## 1. What are Window Functions?

Unlike `GROUP BY` (which collapses multiple rows into a single summary row), **Window Functions** perform calculations across a set of table rows related to the current row **without collapsing the result set**.

```
Syntax:
FUNCTION_NAME() OVER (
    [PARTITION BY partition_column]
    [ORDER BY sort_column [ASC|DESC]]
    [ROWS|RANGE frame_specification]
)
```

---

## 2. Ranking Functions

| Function | Description | Example Output for Salaries [100k, 90k, 90k, 80k] |
|---|---|---|
| `ROW_NUMBER()` | Unique sequential integer for each row | 1, 2, 3, 4 |
| `RANK()` | Same rank for ties, skips subsequent ranks | 1, 2, 2, 4 |
| `DENSE_RANK()` | Same rank for ties, does NOT skip ranks | 1, 2, 2, 3 |
| `NTILE(n)` | Divides partition into `n` roughly equal buckets | Bucket 1, 1, 2, 2 |

### Example: Top Earners Per Department (Dense Rank)

```sql
WITH ranked_employees AS (
    SELECT 
        e.emp_id,
        e.first_name,
        e.last_name,
        d.dept_name,
        e.salary,
        DENSE_RANK() OVER (
            PARTITION BY e.dept_id 
            ORDER BY e.salary DESC
        ) as dept_salary_rank
    FROM employees e
    JOIN departments d ON e.dept_id = d.dept_id
)
SELECT * 
FROM ranked_employees 
WHERE dept_salary_rank <= 2;
```

---

## 3. Value & Offset Functions (`LAG`, `LEAD`, `FIRST_VALUE`)

Inspect prior or upcoming rows in the partition without self-joining:

```sql
SELECT 
    emp_id,
    first_name,
    salary,
    hired_at,
    -- Previous hire's salary
    LAG(salary, 1) OVER (ORDER BY hired_at) AS prev_hired_salary,
    -- Next hire's salary
    LEAD(salary, 1) OVER (ORDER BY hired_at) AS next_hired_salary,
    -- Difference from previous hire
    salary - LAG(salary, 1) OVER (ORDER BY hired_at) AS salary_diff_from_prev
FROM employees
ORDER BY hired_at;
```

---

## 4. Running Totals & Moving Averages (Window Frames)

Window frames (`ROWS BETWEEN ...`) define the exact sliding boundaries of calculation:

```sql
SELECT 
    hired_at,
    first_name,
    salary,
    -- Cumulative running total of salaries over time
    SUM(salary) OVER (
        ORDER BY hired_at 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_payroll,

    -- 3-hire moving average (current hire + 2 previous hires)
    ROUND(AVG(salary) OVER (
        ORDER BY hired_at 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 2) AS moving_avg_3_hires
FROM employees;
```
