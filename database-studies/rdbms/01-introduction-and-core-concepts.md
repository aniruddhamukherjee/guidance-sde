# Module 01: Introduction & Core RDBMS Concepts

## 1. What is an RDBMS?

A **Relational Database Management System (RDBMS)** is a database engine based on the **Relational Model** introduced by Edgar F. Codd in 1970. In an RDBMS:
- Data is represented mathematically as **relations** (visualized as **tables**).
- Each table contains rows (**tuples** / records) representing entities.
- Each table contains columns (**attributes** / fields) representing characteristics with specific data types.
- Relationships between entities are established through foreign keys matching primary keys across tables.

---

## 2. The ACID Properties

ACID is the foundation of reliability in transactional relational databases:

### 1. **Atomicity ("All or Nothing")**
A transaction is treated as a single indivisible unit of work. If any statement within the transaction fails, the entire transaction is rolled back, leaving the database unchanged.

```sql
-- Transfer $100 from Alice (acc 1) to Bob (acc 2)
BEGIN;

UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1 AND balance >= 100;

UPDATE accounts 
SET balance = balance + 100 
WHERE id = 2;

-- If both updates succeed, persist changes:
COMMIT;
-- If an error happens midway, restore original state:
-- ROLLBACK;
```

### 2. **Consistency**
A transaction brings the database from one valid state to another, maintaining all predefined rules, schemas, constraints (NOT NULL, UNIQUE, CHECK, FOREIGN KEY), and triggers.

```sql
CREATE TABLE accounts (
    id INT PRIMARY KEY,
    name TEXT NOT NULL,
    balance NUMERIC(12, 2) NOT NULL CHECK (balance >= 0) -- Constraint guarantees consistency
);
```

### 3. **Isolation**
Transactions execute concurrently without interfering with each other. Intermediate, uncommitted states of one transaction are not visible to other transactions (governed by isolation levels).

### 4. **Durability**
Once a transaction is committed, its changes survive system crashes or power outages. PostgreSQL achieves this using the **Write-Ahead Log (WAL)**.

---

## 3. Keys and Integrity Constraints

Integrity constraints ensure database accuracy and reliability.

| Constraint | Purpose | PostgreSQL Syntax Example |
|---|---|---|
| **Primary Key (PK)** | Uniquely identifies each row; implies `NOT NULL` and `UNIQUE`. | `id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY` |
| **Foreign Key (FK)** | Enforces referential integrity between child and parent tables. | `customer_id BIGINT REFERENCES customers(id) ON DELETE CASCADE` |
| **Unique Key** | Enforces that all values in a column or column group are unique (allows `NULL`s unless `NOT NULL`). | `email VARCHAR(255) UNIQUE` |
| **Not Null** | Prohibits `NULL` (empty/unknown) values. | `first_name VARCHAR(50) NOT NULL` |
| **Check** | Enforces custom boolean expressions for validity. | `CHECK (age >= 18 AND status IN ('active', 'pending'))` |
| **Default** | Provides a fallback value if none is supplied upon insert. | `created_at TIMESTAMPTZ DEFAULT clock_timestamp()` |

### Concrete Schema Example:

```sql
-- Parent Table
CREATE TABLE customers (
    customer_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(100) NOT NULL,
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'deleted')),
    created_at TIMESTAMPTZ DEFAULT clock_timestamp()
);

-- Child Table
CREATE TABLE orders (
    order_id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    order_date TIMESTAMPTZ DEFAULT clock_timestamp(),
    total_amount NUMERIC(10, 2) NOT NULL CHECK (total_amount >= 0),
    
    -- Referential Integrity Constraint
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT -- Prevents deleting a customer if orders exist
        ON UPDATE CASCADE
);
```

---

## 4. Database Normalization & Normal Forms

Normalization organizes table columns to reduce **data redundancy** and avoid **anomalies** (Insertion, Update, Deletion anomalies).

### 1NF (First Normal Form)
- Each cell must contain a single **atomic** value (no multi-valued lists or arrays stored as text).
- Each row must be unique (must have a primary key).

❌ **Non-1NF:**
| order_id | customer_name | item_names |
|---|---|---|
| 101 | John Doe | Laptop, Mouse, Keyboard |

✅ **1NF:**
| order_id | customer_name | item_name |
|---|---|---|
| 101 | John Doe | Laptop |
| 101 | John Doe | Mouse |
| 101 | John Doe | Keyboard |

---

### 2NF (Second Normal Form)
- Must be in **1NF**.
- No **partial dependencies**: Every non-key attribute must depend on the **entire** primary key (relevant when PK is composite).

❌ **Non-2NF (Composite PK: `order_id` + `item_id`):**
| order_id (PK) | item_id (PK) | item_name | item_price | order_date |
|---|---|---|---|---|
| 101 | 501 | Mouse | \$25 | 2026-01-01 |

*Problem:* `item_name` depends only on `item_id`, not `order_id`. `order_date` depends only on `order_id`.

✅ **2NF (Split into separate entities):**
- `orders`: (`order_id`, `order_date`)
- `items`: (`item_id`, `item_name`, `item_price`)
- `order_items`: (`order_id`, `item_id`, `quantity`)

---

### 3NF (Third Normal Form)
- Must be in **2NF**.
- No **transitive dependencies**: Non-key attributes must depend directly on the primary key, not on another non-key attribute.

❌ **Non-3NF:**
| customer_id (PK) | name | zip_code | city | state |
|---|---|---|---|---|
| 1 | Alice | 94016 | Daly City | CA |

*Problem:* `city` and `state` depend on `zip_code`, which depends on `customer_id`. If zip code rules change, multiple customer records must be updated.

✅ **3NF:**
- `customers`: (`customer_id`, `name`, `zip_code`)
- `postal_codes`: (`zip_code`, `city`, `state`)

---

### 5. Normalization vs. Denormalization Trade-offs

| Factor | Normalized (3NF / BCNF) | Denormalized |
|---|---|---|
| **Write Performance** | ⚡ Fast (updates occur in one place) | ⚠️ Slower (must update duplicate values) |
| **Read Performance** | ⚠️ Slower for deep analytics (requires many `JOIN`s) | ⚡ Fast for read queries (fewer `JOIN`s) |
| **Data Integrity** | 🔒 High (no redundancy anomalies) | ⚠️ Risk of inconsistency across duplicated fields |
| **Storage** | 📉 Compact | 📈 Larger due to duplicate fields |
| **Primary Use-case** | OLTP (Transactional apps, e-commerce, banking) | OLAP / Data Warehouses / Reporting dashboards |
