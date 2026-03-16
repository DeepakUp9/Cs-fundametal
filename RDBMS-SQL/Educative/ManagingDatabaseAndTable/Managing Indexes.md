# Managing Indexes
> *Learn to manage indexes on tables in SQL.*

---

## Introduction

Imagine running a store with thousands of products and customer orders. Without a structured way to locate information quickly, finding data in a massive table becomes increasingly time-consuming. This is where **indexes** come in — they act like the index of a book, allowing us to retrieve information swiftly without scanning the entire dataset.

### 🎯 Learning Goals

- ✅ Understand what an index is and why it is important
- ✅ Learn how to create a simple (single-column) index
- ✅ Learn how to create a composite (multi-column) index
- ✅ Learn how to view and drop indexes

---

## What Is an Index?

An **index** is a special data structure that the database maintains alongside a table to speed up data retrieval. Just as a book's index points you directly to the right page, a database index points the engine directly to the matching rows — no full table scan needed.

### Benefits vs. Trade-offs

| Aspect | Detail |
|---|---|
| ✅ **Faster reads** | Queries with `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY` on indexed columns execute faster |
| ✅ **Efficient lookups** | Database jumps directly to matching rows instead of scanning every row |
| ✅ **Sorted operations** | `ORDER BY` and `GROUP BY` on indexed columns are significantly faster |
| ⚠️ **Storage overhead** | Each index requires extra disk space |
| ⚠️ **Slower writes** | Every `INSERT`, `UPDATE`, and `DELETE` must also update all related indexes |

> 💡 **The core trade-off:** Indexes speed up reads but slow down writes. Index columns that are read frequently; avoid indexing columns that change constantly or are rarely queried.

---

## How Indexes Work — A Simple Analogy

```
Without index:
Table: [Row 1] [Row 2] [Row 3] ... [Row 10,000]
Query: "Find products with Price = 635.79"
Action: Scan EVERY row → slow on large tables

With index on Price:
Index: 5.00→Row6, 50.00→Row5, 75.62→Row4, 82.50→Row2, 635.79→Row1...
Query: "Find products with Price = 635.79"
Action: Jump directly to Row1 → fast
```

---

## 1️⃣ Creating a Simple Index

A **simple index** is built on a single column. It's ideal for queries that frequently filter, sort, or join on that one column.

### Syntax

```sql
CREATE INDEX IndexName ON TableName (ColumnName);
```

### Naming Convention

A widely used convention for index names is: `idx_tablename_columnname`

| Index Name | Meaning |
|---|---|
| `idx_products_price` | Index on `Products.Price` |
| `idx_customers_email` | Index on `Customers.Email` |
| `idx_orders_orderdate` | Index on `Orders.OrderDate` |

### Example — Index on `Price`

```sql
-- Enable profiling to measure query time
SET PROFILING = 1;

-- Query before the index exists
SELECT * FROM Products ORDER BY Price;

-- Create the index
CREATE INDEX idx_products_price ON Products(Price);

-- Same query after the index
SELECT * FROM Products ORDER BY Price;

-- View timing for both queries
SHOW PROFILES;
```

After creating `idx_products_price`, the second query runs significantly faster — the database uses the index instead of scanning every row.

### More Simple Index Examples

```sql
-- Index for fast customer lookup by email
CREATE INDEX idx_customers_email ON Customers(Email);

-- Index for fast order lookup by date
CREATE INDEX idx_orders_orderdate ON Orders(OrderDate);

-- Index for fast product lookup by name
CREATE INDEX idx_products_name ON Products(ProductName);
```

---

## 2️⃣ Viewing Indexes on a Table

Use `SHOW INDEX` to inspect all indexes currently on a table:

### Syntax

```sql
SHOW INDEX FROM TableName;
```

### Example

```sql
CREATE INDEX idx_products_price ON Products(Price);

SHOW INDEX FROM Products;
```

#### Sample Output (key columns)

| Table | Key_name | Column_name | Non_unique | Index_type |
|---|---|---|---|---|
| Products | PRIMARY | ProductID | 0 | BTREE |
| Products | ProductName | ProductName | 0 | BTREE |
| Products | idx_products_price | Price | 1 | BTREE |

> 💡 The `PRIMARY KEY` and `UNIQUE` columns automatically get indexes when a table is created — you don't need to create them manually.

---

## 3️⃣ Creating a Composite Index

A **composite index** is built on two or more columns. It's useful when queries frequently filter or sort on multiple columns together.

### Syntax

```sql
CREATE INDEX IndexName ON TableName (Column1, Column2);
```

### Example — Index on `CategoryID` and `Price`

```sql
CREATE INDEX idx_products_category_price ON Products(CategoryID, Price);

SHOW INDEX FROM Products;
```

This index is particularly useful for queries like:

```sql
-- Both conditions benefit from the composite index
SELECT ProductName, Price
FROM Products
WHERE CategoryID = 1
ORDER BY Price ASC;
```

### Column Order in Composite Indexes

The **order of columns matters** in a composite index:

```sql
-- Index: (CategoryID, Price)
-- Helps queries filtering by CategoryID alone, or CategoryID + Price together
WHERE CategoryID = 1                  -- ✅ Uses index
WHERE CategoryID = 1 AND Price < 100  -- ✅ Uses index fully
WHERE Price < 100                     -- ❌ Cannot use this index efficiently
```

> 💡 **Rule of thumb:** Put the most selective (most unique) column first, and the column used most often in `WHERE` conditions first. The index works left-to-right.

### More Composite Index Examples

```sql
-- Index for orders filtered by customer and date
CREATE INDEX idx_orders_customer_date ON Orders(CustomerID, OrderDate);

-- Index for order details filtered by order and product
CREATE INDEX idx_orderdetails_order_product ON Order_Details(OrderID, ProductID);
```

---

## 4️⃣ Dropping an Index

Unused indexes waste storage and slow down write operations. Drop indexes that are no longer helpful.

### Syntax

```sql
DROP INDEX IndexName ON TableName;
```

### Example — Drop and Verify

```sql
-- Create the index
CREATE INDEX idx_products_price ON Products(Price);
SHOW INDEX FROM Products;  -- Index appears

-- Drop the index
DROP INDEX idx_products_price ON Products;
SHOW INDEX FROM Products;  -- Index is gone
```

> ⚠️ **Be careful** when dropping indexes related to `PRIMARY KEY` or `FOREIGN KEY` constraints — removing them can break data integrity and referential relationships.

### When to Drop an Index

| Scenario | Action |
|---|---|
| Index is rarely used in queries | Drop it |
| Table has too many indexes slowing writes | Drop the least-used ones |
| Before a large bulk data import | Drop indexes, import, then recreate |
| Column is being dropped from the table | Drop its index first |

> 💡 **Bulk import tip:** Dropping indexes before a large `INSERT` operation, then recreating them after, can make the import **dramatically faster** — the database doesn't have to update indexes for each inserted row.

---

## Index Types Comparison

| Index Type | Columns | Best For |
|---|---|---|
| **Simple index** | Single column | Filtering or sorting on one column |
| **Composite index** | Multiple columns | Filtering on two or more columns together |
| **Primary key index** | Single column (auto-created) | Unique row identification |
| **Unique index** | Single or multiple columns | Enforcing uniqueness + fast lookup |

---

## When to Index — Decision Guide

```
Should I create an index on this column?

Is the column frequently used in WHERE, JOIN, ORDER BY, or GROUP BY?
    YES → Consider creating an index
    NO  → Probably not needed

Does the table have a large number of rows (thousands+)?
    YES → Index will provide more benefit
    NO  → Full scan on a small table is fast enough

Is the column's data highly varied (many unique values)?
    YES → Index is very effective (high selectivity)
    NO  → Index may not help much (e.g., a BOOLEAN column)

Is the column written to (INSERT/UPDATE/DELETE) very frequently?
    YES → Index may slow writes; weigh carefully
    NO  → Safe to index
```

---

## Profiling Query Performance

Use `SET PROFILING` to measure how long queries take — before and after adding an index:

```sql
-- Enable profiling
SET PROFILING = 1;

-- Run query without index
SELECT * FROM Products WHERE Price > 100;

-- Create index
CREATE INDEX idx_products_price ON Products(Price);

-- Run same query with index
SELECT * FROM Products WHERE Price > 100;

-- Compare execution times
SHOW PROFILES;
```

The `SHOW PROFILES` output shows the duration of each query. The difference in time demonstrates the performance improvement from the index.

---

## Index Management Quick Reference

| Operation | Syntax |
|---|---|
| Create simple index | `CREATE INDEX idx_name ON TableName (col);` |
| Create composite index | `CREATE INDEX idx_name ON TableName (col1, col2);` |
| View indexes on a table | `SHOW INDEX FROM TableName;` |
| Drop an index | `DROP INDEX idx_name ON TableName;` |
| Enable query profiling | `SET PROFILING = 1;` |
| View query profiles | `SHOW PROFILES;` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Index columns used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY`** | These operations benefit most from indexes |
| **Use descriptive index names** | `idx_products_price` is clearer than `idx1` |
| **Avoid over-indexing** | Too many indexes slow down writes and waste storage |
| **Drop indexes before bulk imports** | Dramatically speeds up large INSERT operations |
| **Recreate indexes after bulk imports** | Ensures query performance returns to normal |
| **Monitor index usage** | Drop indexes that are rarely or never used |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Indexing rarely-queried columns** — unused indexes waste storage and add overhead to every write operation.

> ⚠️ **Creating too many indexes on one table** — every index must be updated on every write. A table with 10 indexes is 10x slower to insert into than one with no indexes.

> ⚠️ **Getting composite index column order wrong** — `(CategoryID, Price)` and `(Price, CategoryID)` are different indexes. The left-most column determines which queries benefit most.

> ⚠️ **Not dropping indexes before large bulk imports** — inserting millions of rows with indexes active is far slower than importing first, then indexing.

> ⚠️ **Dropping `PRIMARY KEY` or `FOREIGN KEY` indexes carelessly** — these indexes enforce data integrity. Removing them can break table relationships.

> ⚠️ **Indexing low-selectivity columns** — a column with only 2 distinct values (like `TRUE`/`FALSE`) provides little benefit as an index; the database may still scan most of the table.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Index | A data structure that speeds up data retrieval |
| Simple index | Index on one column — ideal for single-column filtering/sorting |
| Composite index | Index on multiple columns — ideal for multi-column queries |
| Trade-off | Faster reads vs. slower writes and more storage |
| `CREATE INDEX` | Builds a new index on specified columns |
| `SHOW INDEX` | Lists all indexes on a table |
| `DROP INDEX` | Removes an unused or unwanted index |
| Profiling | Use `SET PROFILING` to measure query time improvements |

---

## What's Next?

Now that you can optimize query performance with indexes, explore more advanced database management!

- 🔒 **Managing user privileges and roles**
- 🔄 **Transactions, `COMMIT`, and `ROLLBACK`**
- 📋 **Query optimization and execution plans (`EXPLAIN`)**
- ⚡ **Advanced performance tuning**

> *Indexes are your database's speed boost — index smartly, and your queries will fly even as your data grows!* 🚀

---

