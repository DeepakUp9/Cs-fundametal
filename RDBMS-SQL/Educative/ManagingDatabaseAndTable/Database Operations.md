# Database Operations
> *Learn about basic operations to manage a database.*

---

## Introduction

We often need to restructure or clean up our database to keep data accurate and well-organized. For instance, we might need to remove all data from a test table quickly, rename a table to reflect new business logic, or delete an entire database that has become obsolete. All of this is possible with `TRUNCATE TABLE`, `RENAME TABLE`, and `DROP DATABASE`.

### 🎯 Learning Goals

- ✅ Understand the purpose and usage of `TRUNCATE TABLE`
- ✅ Learn how to rename tables using `RENAME TABLE`
- ✅ Explore how to remove an entire database with `DROP DATABASE`

---

## Operations at a Glance

| Statement | What It Does | Reversible? | Keeps Structure? |
|---|---|---|---|
| `TRUNCATE TABLE` | Removes all rows from a table | ❌ No | ✅ Yes |
| `DELETE FROM` | Removes rows (can be filtered) | ✅ Yes (with transaction) | ✅ Yes |
| `RENAME TABLE` | Renames a table | ✅ Yes (rename back) | ✅ Yes |
| `DROP TABLE` | Deletes a table entirely | ❌ No | ❌ No |
| `DROP DATABASE` | Deletes an entire database | ❌ No | ❌ No |

---

## 1️⃣ `TRUNCATE TABLE` — Clear All Rows

`TRUNCATE TABLE` is a fast way to remove **all rows** from a table at once — without deleting the table itself. The table structure, columns, constraints, and indexes all remain intact.

### Syntax

```sql
TRUNCATE TABLE TableName;
```

### Example — Clear `Order_Details`

```sql
-- Check row count before
SELECT COUNT(*) AS RowsBefore FROM Order_Details;

-- Truncate the table
TRUNCATE TABLE Order_Details;

-- Verify the table is empty
SELECT COUNT(*) AS RowsAfter FROM Order_Details;
```

#### Output

| RowsBefore | → | RowsAfter |
|---|---|---|
| 25 | → | 0 |

The table remains in the database — it's just empty. The schema and all column definitions are preserved.

### `TRUNCATE TABLE` vs. `DELETE FROM`

| Feature | `TRUNCATE TABLE` | `DELETE FROM` |
|---|---|---|
| **Removes rows** | ✅ All rows | ✅ All or filtered rows |
| **Speed** | ⚡ Very fast (deallocates pages) | 🐢 Slower (row by row) |
| **Resets `AUTO_INCREMENT`** | ✅ Yes — resets to 1 | ❌ No — counter continues |
| **Can filter with `WHERE`** | ❌ No | ✅ Yes |
| **Works with `FOREIGN KEY`** | ❌ No — fails if referenced | ✅ Yes (if rows are removed correctly) |
| **Reversible (with transaction)** | ❌ No | ✅ Yes |
| **Triggers fire** | ❌ No | ✅ Yes |
| **Best used for** | Resetting a table completely | Selective or reversible deletes |

> ⚠️ `TRUNCATE TABLE` **cannot be used** on a table that is referenced by a `FOREIGN KEY` constraint in another table. Drop the foreign key first, or use `DELETE FROM` instead.

### `AUTO_INCREMENT` Reset Behaviour

```sql
-- Before truncate: last AUTO_INCREMENT was 20
INSERT INTO Products (ProductName, ...) VALUES ('Widget', ...);
-- ProductID = 21 (continues from 20)

TRUNCATE TABLE Products;

-- After truncate: AUTO_INCREMENT resets to 1
INSERT INTO Products (ProductName, ...) VALUES ('Widget', ...);
-- ProductID = 1 (starts fresh)
```

---

## 2️⃣ `RENAME TABLE` — Rename a Table

`RENAME TABLE` changes a table's name without altering its data, columns, constraints, or indexes. Any queries that reference the old name must be updated after renaming.

### Syntax

```sql
RENAME TABLE OldName TO NewName;
```

### Example — Rename `Customers` to `Clients`

```sql
-- See current tables
SHOW TABLES;

-- Rename the table
RENAME TABLE Customers TO Clients;

-- Verify the name change
SHOW TABLES;
```

#### Before

| Tables_in_OnlineStore |
|---|
| Categories |
| **Customers** |
| Orders |
| Products |
| … |

#### After

| Tables_in_OnlineStore |
|---|
| Categories |
| **Clients** |
| Orders |
| Products |
| … |

The data, columns, and constraints are completely unchanged — only the name is different.

### Renaming Multiple Tables at Once

`RENAME TABLE` can rename several tables in a single statement:

```sql
-- Rename multiple tables in one statement
RENAME TABLE
    Customers TO Clients,
    Products TO Inventory,
    Orders TO Transactions;
```

> ✅ Renaming multiple tables atomically reduces the window where some tables have new names and others don't — useful when updating related tables together.

### What Changes After a Rename?

| What | After Rename |
|---|---|
| ✅ **Data** | Unchanged |
| ✅ **Columns and types** | Unchanged |
| ✅ **Constraints and indexes** | Unchanged |
| ❌ **Queries using the old name** | Will fail — must be updated |
| ❌ **Views referencing the old name** | Will fail — must be recreated |
| ❌ **Stored procedures using old name** | Will fail — must be updated |

> 💡 Always search your codebase and views for the old table name after renaming. Any reference to the old name will immediately produce an error.

---

## 3️⃣ `DROP DATABASE` — Remove an Entire Database

`DROP DATABASE` permanently removes a database along with **all** of its tables, data, views, stored procedures, and schema information. This is irreversible — use with extreme caution.

### Syntax

```sql
DROP DATABASE DatabaseName;
```

### Example — Drop `TestDB`

```sql
-- See all databases before
SHOW DATABASES;

-- Drop the database
DROP DATABASE TestDB;

-- Confirm it's gone
SHOW DATABASES;
```

#### Before

| Database |
|---|
| information_schema |
| mysql |
| OnlineStore |
| **TestDB** |

#### After

| Database |
|---|
| information_schema |
| mysql |
| OnlineStore |

`TestDB` is completely gone. Every table, row, view, and constraint inside it has been permanently deleted.

### What `DROP DATABASE` Destroys

| Object | Destroyed? |
|---|---|
| All tables | ✅ Yes |
| All rows in every table | ✅ Yes |
| All views | ✅ Yes |
| All indexes | ✅ Yes |
| All constraints | ✅ Yes |
| All stored procedures | ✅ Yes |
| The database itself | ✅ Yes |

> ⚠️ There is **no undo**. Without a backup, all data is permanently lost. Always confirm you are dropping the correct database.

---

## Danger Level Comparison

```
LOW RISK ──────────────────────────────────── HIGH RISK

RENAME TABLE    TRUNCATE TABLE    DROP TABLE    DROP DATABASE
     │                │               │               │
  Reversible       Fast but        Permanent       Nuclear option
  (rename back)   irreversible     data + schema   Entire database
  Data intact     data loss        lost            gone forever
```

---

## 🧩 Code Exercise

### Task 1 — Clear a Test Table

**We created a temporary table `Temp_Products` for testing. Remove all rows quickly without destroying the table structure.**

#### Current Table (before)

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 2 | Dumbbell Set | 3 | 82.50 | 15 |
| 3 | Laptop | 1 | 1099.99 | 30 |
| … | … | … | … | … |
| 20 | Smart Fitness Band | 3 | 350.99 | 60 |

```sql
-- Remove all rows
TRUNCATE TABLE Temp_Products;

-- Verify the table is empty
SELECT * FROM Temp_Products;
```

#### Expected Output

*(Empty table — no rows returned)*

> ✅ The `Temp_Products` table still exists with all its columns and structure — only the rows are gone. The `AUTO_INCREMENT` counter has also reset to 1.

---

## Quick Reference

| Task | Statement |
|---|---|
| Clear all rows, keep structure | `TRUNCATE TABLE TableName;` |
| Clear all rows (reversible) | `DELETE FROM TableName;` |
| Rename a table | `RENAME TABLE OldName TO NewName;` |
| Rename multiple tables | `RENAME TABLE A TO B, C TO D;` |
| Drop an entire database | `DROP DATABASE DatabaseName;` |
| Verify a table is empty | `SELECT COUNT(*) FROM TableName;` |
| Confirm table still exists | `SHOW TABLES;` |
| Confirm database is gone | `SHOW DATABASES;` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always back up before truncating or dropping** | These operations cannot be reversed without a backup |
| **Test in a development environment first** | Prevents accidental data loss in production |
| **Document table renames** | Keeps teammates informed and prevents broken queries |
| **Update all queries and views after renaming** | Old names cause immediate errors |
| **Verify you're in the correct database before dropping** | `SELECT DATABASE();` confirms which database is active |
| **Use `DELETE FROM` when in doubt** | Unlike `TRUNCATE`, `DELETE` can be wrapped in a transaction and rolled back |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Truncating the wrong table** — there is no undo. Double-check the table name and always run `SELECT COUNT(*) FROM TableName;` first to confirm it's the right table.

> ⚠️ **Using `TRUNCATE TABLE` on a foreign-key-referenced table** — this will fail with a constraint error. Use `DELETE FROM` instead, or temporarily disable the foreign key check.

> ⚠️ **Dropping a database that is still in use** — always confirm the database is truly obsolete and that no applications or users are connected to it.

> ⚠️ **Forgetting to update queries after renaming** — any view, stored procedure, or application query using the old table name will immediately return an error.

> ⚠️ **Assuming `TRUNCATE` is the same as `DELETE`** — they differ in speed, rollback capability, `AUTO_INCREMENT` reset, `FOREIGN KEY` compatibility, and trigger firing.

---

## Summary

| Statement | Purpose | Keeps Table? | Keeps Data? | Reversible? |
|---|---|---|---|---|
| `TRUNCATE TABLE` | Remove all rows instantly | ✅ Yes | ❌ No | ❌ No |
| `RENAME TABLE` | Change a table's name | ✅ Yes | ✅ Yes | ✅ Yes (rename back) |
| `DROP DATABASE` | Delete an entire database | ❌ No | ❌ No | ❌ No |

---

## What's Next?

With these database management tools mastered, you're ready for the final advanced topics!

- 🔄 **Transactions, `COMMIT`, and `ROLLBACK`**
- 🔒 **User privileges and access control**
- ⚡ **Query optimization and `EXPLAIN`**
- 📋 **Database backup and recovery strategies**

> *`TRUNCATE`, `RENAME`, and `DROP` are powerful — use them deliberately, and always have a backup ready!* 🚀

---

