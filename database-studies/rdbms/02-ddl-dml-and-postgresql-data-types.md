# Module 02: DDL, DML & PostgreSQL Data Types

## 1. Key PostgreSQL Data Types

Choosing the correct data type optimizes storage footprint, indexes, and execution performance.

| Category | Type | Description | Example |
|---|---|---|---|
| **Integer** | `SMALLINT` | 2 bytes (-32,768 to +32,767) | Small counts, status flags |
| | `INTEGER` (`INT`) | 4 bytes (-2.1B to +2.1B) | Default integer |
| | `BIGINT` | 8 bytes (-9.22e18 to +9.22e18) | Large IDs, financial counters |
| **Decimal / Float** | `NUMERIC(p, s)` / `DECIMAL` | Exact precision numeric, no rounding errors | Money: `NUMERIC(12, 2)` |
| | `REAL` / `DOUBLE PRECISION` | Inexact floating-point (scientific calculations) | Sensor metrics: `3.14159` |
| **String / Text** | `VARCHAR(n)` | Variable-length with limit `n` | `VARCHAR(100)` |
| | `TEXT` | Unlimited variable-length (Postgres optimizes this natively) | Bio, article bodies |
| **Date & Time** | `TIMESTAMPTZ` | Timestamp with timezone (always recommended) | `2026-09-26 15:30:00+00` |
| | `DATE` | Calendar date (no time) | `2026-09-26` |
| | `INTERVAL` | Time spans / durations | `INTERVAL '3 days 2 hours'` |
| **Binary & Identifiers** | `UUID` | 128-bit universally unique identifier | `gen_random_uuid()` |
| | `BOOLEAN` | `TRUE`, `FALSE`, or `NULL` | `is_verified` |
| **Complex / Semi-structured** | `JSONB` | Binary JSON (indexed, decomposed for fast queries) | `{"tags": ["dev", "db"]}` |
| | `TEXT[]` / `INT[]` | Native array data types | `ARRAY['admin', 'editor']` |

---

## 2. Data Definition Language (DDL)

DDL commands define and modify database structures and schemas.

### Creating Tables with Identity & Modern Types

```sql
CREATE EXTENSION IF NOT EXISTS "pgcrypto"; -- For UUID generation if on older postgres

CREATE TABLE products (
    product_id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    sku VARCHAR(50) NOT NULL UNIQUE,
    name TEXT NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    metadata JSONB DEFAULT '{}'::jsonb,
    tags TEXT[] DEFAULT ARRAY[]::TEXT[],
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);
```

### Altering and Dropping Tables

```sql
-- Add a column
ALTER TABLE products ADD COLUMN category_id INT;

-- Modify column type or constraints
ALTER TABLE products ALTER COLUMN description SET NOT NULL;
ALTER TABLE products ADD CONSTRAINT chk_sku_uppercase CHECK (sku = UPPER(sku));

-- Rename column
ALTER TABLE products RENAME COLUMN name TO title;

-- Drop column
ALTER TABLE products DROP COLUMN category_id;

-- Truncate all rows quickly (resets identity counters)
TRUNCATE TABLE products RESTART IDENTITY;

-- Drop table
DROP TABLE IF EXISTS products CASCADE;
```

---

## 3. Data Manipulation Language (DML)

DML commands manage and retrieve the data within the tables.

### 1. `INSERT` Operations

```sql
-- Single-row insert
INSERT INTO products (sku, title, price, stock_quantity, metadata, tags)
VALUES (
    'PROD-001', 
    'Mechanical Keyboard', 
    129.99, 
    50, 
    '{"switch": "cherry-mx-red", "layout": "ANSI"}'::jsonb, 
    ARRAY['electronics', 'peripherals']
);

-- Multi-row insert with RETURNING clause
INSERT INTO products (sku, title, price, stock_quantity)
VALUES 
    ('PROD-002', 'Wireless Mouse', 49.99, 120),
    ('PROD-003', 'USB-C Cable', 14.50, 300)
RETURNING product_id, sku, title, created_at;
```

### 2. `UPSERT` (Insert on Conflict Do Update / Nothing)

PostgreSQL provides native upsert semantics via `ON CONFLICT`:

```sql
-- Insert or update existing SKU
INSERT INTO products (sku, title, price, stock_quantity)
VALUES ('PROD-001', 'RGB Mechanical Keyboard', 139.99, 45)
ON CONFLICT (sku) 
DO UPDATE SET
    title = EXCLUDED.title,
    price = EXCLUDED.price,
    stock_quantity = EXCLUDED.stock_quantity,
    updated_at = clock_timestamp()
RETURNING product_id, sku, title, price;

-- Insert or ignore if conflict exists
INSERT INTO products (sku, title, price)
VALUES ('PROD-001', 'Mechanical Keyboard', 129.99)
ON CONFLICT (sku) DO NOTHING;
```

### 3. `UPDATE` and `DELETE` with `RETURNING`

```sql
-- Update with conditional logic and returning affected rows
UPDATE products
SET 
    price = price * 0.90, -- Apply 10% discount
    updated_at = clock_timestamp()
WHERE stock_quantity > 100
RETURNING product_id, title, price AS discounted_price;

-- Delete with returning deleted rows
DELETE FROM products
WHERE is_active = FALSE AND stock_quantity = 0
RETURNING product_id, sku, title;
```

---

## 4. Querying Semi-Structured JSONB Data

PostgreSQL's `JSONB` allows querying structured and semi-structured documents:

```sql
-- Extract field as text using ->>
SELECT title, metadata->>'switch' AS switch_type
FROM products
WHERE metadata->>'layout' = 'ANSI';

-- Check if JSON contains a key/value pair using @>
SELECT title, price, metadata
FROM products
WHERE metadata @> '{"switch": "cherry-mx-red"}';

-- Update a key inside JSONB using jsonb_set
UPDATE products
SET metadata = jsonb_set(metadata, '{wireless}', 'true'::jsonb)
WHERE sku = 'PROD-002';
```
