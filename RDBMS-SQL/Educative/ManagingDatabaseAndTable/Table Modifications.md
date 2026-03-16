# Table Modifications
> *Learn to modify tables after creation in SQL.*

---

## Introduction

Imagine managing the `OnlineStore` database for months while business requirements keep evolving. We might need to record additional product details, remove outdated columns, or change a column's data type to store data more accurately. This **flexibility to adapt table structures** is essential for keeping our database relevant and maintainable.

### 🎯 Learning Goals

- ✅ Understand the purpose of `ALTER TABLE`
- ✅ Add and remove columns without losing other data
- ✅ Modify column data types to meet new requirements
- ✅ Change column constraints
- ✅ Drop entire tables and understand the consequences

---

## The `ALTER TABLE` Statement

`ALTER TABLE` is the primary tool for making structural changes to an existing table. It can:

| Operation | Keyword | What It Does |
|---|---|---|
| Add a new column | `ADD COLUMN` | Appends a new column to the table |
| Remove a column | `DROP COLUMN` | Permanently deletes the column and its data |
| Change data type or constraints | `MODIFY COLUMN` | Updates the column's definition |
| Rename a column | `RENAME COLUMN` | Changes the column's name |

---

## 1️⃣ Adding Columns

Use `ADD COLUMN` when a new attribute needs to be tracked that didn't exist at table creation.

### Syntax

```sql
ALTER TABLE TableName
ADD COLUMN ColumnName DataType [constraints];
```

### Example — Add `ProductDescription` to `Products`

```sql
ALTER TABLE Products
ADD COLUMN ProductDescription VARCHAR(255);

DESCRIBE Products;
```

#### Sample Output After Adding Column

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| ProductID | int | NO | PRI | NULL | auto_increment |
| ProductName | varchar(50) | NO | UNI | NULL | |
| CategoryID | int | YES | | NULL | |
| Price | decimal(10,2) | NO | | NULL | |
| Stock | int | NO | | NULL | |
| **ProductDescription** | **varchar(255)** | **YES** | | **NULL** | |

> 💡 New columns are added at the **end** of the table by default. The new column contains `NULL` for all existing rows until populated.

### Example — Add Column with a Default Value

```sql
-- Add a column with a default value so existing rows aren't NULL
ALTER TABLE Products
ADD COLUMN IsAvailable BOOLEAN DEFAULT TRUE;
```

---

## 2️⃣ Dropping Columns

Use `DROP COLUMN` to remove columns that are no longer needed to keep the schema clean.

### Syntax

```sql
ALTER TABLE TableName
DROP COLUMN ColumnName;
```

### Example — Remove `ProductDescription`

```sql
ALTER TABLE Products
DROP COLUMN ProductDescription;

DESCRIBE Products;
```

> ⚠️ **Dropping a column is permanent.** All data in that column is immediately and irreversibly lost. There is no undo.

#### Before vs. After Dropping a Column

| Before | After |
|---|---|
| ProductID, ProductName, CategoryID, Price, Stock, **ProductDescription** | ProductID, ProductName, CategoryID, Price, Stock |

---

## 3️⃣ Modifying Column Data Types

Use `MODIFY COLUMN` to change the data type of an existing column — for example, to increase the allowed length of text.

### Syntax

```sql
ALTER TABLE TableName
MODIFY COLUMN ColumnName NewDataType [constraints];
```

### Example — Increase `ProductName` from VARCHAR(50) to VARCHAR(100)

```sql
ALTER TABLE Products
MODIFY COLUMN ProductName VARCHAR(100);

DESCRIBE Products;
```

#### Sample Output

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| ProductID | int | NO | PRI | NULL | auto_increment |
| **ProductName** | **varchar(100)** | YES | UNI | NULL | |
| CategoryID | int | YES | | NULL | |
| Price | decimal(10,2) | NO | | NULL | |
| Stock | int | NO | | NULL | |

> ⚠️ **Important:** After using `MODIFY COLUMN`, any constraints like `NOT NULL` on that column must be **explicitly re-stated** in the new definition. If omitted, they are removed.

```sql
-- ❌ This removes NOT NULL from ProductName
ALTER TABLE Products
MODIFY COLUMN ProductName VARCHAR(100);

-- ✅ This keeps NOT NULL intact
ALTER TABLE Products
MODIFY COLUMN ProductName VARCHAR(100) NOT NULL;
```

---

## 4️⃣ Changing Column Constraints

`MODIFY COLUMN` also lets us add or remove constraints like `NOT NULL`, `UNIQUE`, and `DEFAULT`.

### Syntax

```sql
ALTER TABLE TableName
MODIFY ColumnName DataType Constraint;
```

### Example — Make `Phone` Mandatory (NOT NULL)

```sql
ALTER TABLE Customers
MODIFY Phone VARCHAR(15) NOT NULL;
```

> ⚠️ If the column contains any `NULL` values, this command will **fail**. First update all `NULL` values to a valid placeholder, then apply the constraint.

```sql
-- Step 1: Fill in NULL values first
UPDATE Customers SET Phone = 'N/A' WHERE Phone IS NULL;

-- Step 2: Then apply NOT NULL
ALTER TABLE Customers
MODIFY Phone VARCHAR(15) NOT NULL;
```

### Example — Allow NULL Values Again

```sql
ALTER TABLE Customers
MODIFY Phone VARCHAR(15) NULL;

DESCRIBE Customers;
```

### Constraints That Can Be Added or Removed with `ALTER TABLE`

| Constraint | Add with | Remove with |
|---|---|---|
| `NOT NULL` | `MODIFY col DataType NOT NULL` | `MODIFY col DataType NULL` |
| `DEFAULT` | `MODIFY col DataType DEFAULT value` | `MODIFY col DataType` (omit DEFAULT) |
| `UNIQUE` | `MODIFY col DataType UNIQUE` | `DROP INDEX col_name` |
| `PRIMARY KEY` | `ADD PRIMARY KEY (col)` | `DROP PRIMARY KEY` |
| `FOREIGN KEY` | `ADD FOREIGN KEY (col) REFERENCES ...` | `DROP FOREIGN KEY fk_name` |

---

## 5️⃣ Dropping an Entire Table

`DROP TABLE` permanently removes a table and all of its data from the database.

### Syntax

```sql
DROP TABLE TableName;
```

### Example

```sql
DROP TABLE TempTable;
```

This permanently deletes the table structure **and** all data it contains.

> ⚠️ **There is no undo.** Once a table is dropped, it is gone unless you have a backup.

### Implications of Dropping a Table

| Impact | Description |
|---|---|
| ❌ Data lost | All rows are permanently deleted |
| ❌ Structure lost | The column definitions and indexes are removed |
| ❌ Foreign keys broken | Any table referencing this one via `FOREIGN KEY` will error |
| ❌ Views broken | Any view built on this table will stop working |
| ❌ Queries fail | Any stored procedures or queries referencing the table will fail |

---

## `ALTER TABLE` vs. `DROP TABLE` — Key Differences

| Feature | `ALTER TABLE` | `DROP TABLE` |
|---|---|---|
| **Scope** | Modifies structure (adds/removes/changes columns) | Removes the entire table |
| **Data impact** | Preserves existing data (unless dropping a column) | All data permanently deleted |
| **Reversible?** | Column drops are not reversible; type changes may be | Not reversible without a backup |
| **Use when** | Evolving requirements, adding/removing fields | Table is completely obsolete |

---

## 🧩 Code Exercises

---

### Task 1 — Add a Column with a Default Value

**Add a column named `IsActive` of type `BOOLEAN` to the `Categories` table with a default value of `TRUE`.**

```sql
ALTER TABLE Categories
ADD COLUMN IsActive BOOLEAN DEFAULT TRUE;

DESCRIBE Categories;
```

#### Expected Output

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| CategoryID | int | NO | PRI | NULL | auto_increment |
| CategoryName | varchar(50) | NO | UNI | NULL | |
| **IsActive** | **tinyint(1)** | **YES** | | **1** | |

> 💡 `BOOLEAN` is stored as `tinyint(1)` in MySQL. The default `TRUE` is stored as `1`.

---

### Task 2 — Drop a Column

**Remove the `OldCategoryCode` column from the `Categories` table.**

```sql
ALTER TABLE Categories
DROP COLUMN OldCategoryCode;

DESCRIBE Categories;
```

#### Expected Output

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| CategoryID | int | NO | PRI | NULL | auto_increment |
| CategoryName | varchar(50) | NO | UNI | NULL | |

> ✅ The `OldCategoryCode` column is gone. The rest of the table — and all its data — remains intact.

---

### Task 3 — Modify Column Type and Constraints

**Change the `Phone` column in the `Suppliers` table from `VARCHAR(15)` to `VARCHAR(20)`, and make it `NOT NULL` and `UNIQUE`.**

```sql
ALTER TABLE Suppliers
MODIFY COLUMN Phone VARCHAR(20) NOT NULL UNIQUE;

DESCRIBE Suppliers;
```

#### Expected Output

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| SupplierID | int | NO | PRI | NULL | auto_increment |
| SupplierName | varchar(50) | NO | | NULL | |
| Email | varchar(50) | YES | | NULL | |
| **Phone** | **varchar(20)** | **NO** | **UNI** | **NULL** | |
| Address | varchar(100) | YES | | NULL | |

> 💡 One `MODIFY COLUMN` statement handles the type change (`VARCHAR(20)`), the mandatory constraint (`NOT NULL`), and the uniqueness constraint (`UNIQUE`) simultaneously.

---

## `ALTER TABLE` Quick Reference

| Task | Syntax |
|---|---|
| Add a column | `ALTER TABLE t ADD COLUMN col DataType;` |
| Add with default | `ALTER TABLE t ADD COLUMN col BOOLEAN DEFAULT TRUE;` |
| Drop a column | `ALTER TABLE t DROP COLUMN col;` |
| Change data type | `ALTER TABLE t MODIFY COLUMN col VARCHAR(100);` |
| Add NOT NULL | `ALTER TABLE t MODIFY COLUMN col DataType NOT NULL;` |
| Remove NOT NULL | `ALTER TABLE t MODIFY COLUMN col DataType NULL;` |
| Add UNIQUE | `ALTER TABLE t MODIFY COLUMN col DataType UNIQUE;` |
| Drop entire table | `DROP TABLE TableName;` |
| Inspect structure | `DESCRIBE TableName;` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Back up data before altering tables** | Structural changes can cause irreversible data loss |
| **Test changes in a dev environment first** | Prevents breaking live data with untested alterations |
| **Re-state all constraints after `MODIFY COLUMN`** | Omitting them silently removes existing constraints |
| **Fill NULL values before adding `NOT NULL`** | Avoids command failure on columns with existing NULLs |
| **Check for foreign key dependencies before dropping** | Dropping a referenced table breaks other tables |
| **Verify with `DESCRIBE` after every change** | Confirms the alteration was applied exactly as intended |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Dropping the wrong column or table due to a typo** — always double-check table and column names before running `DROP`. There is no undo.

> ⚠️ **Forgetting to re-apply `NOT NULL` after `MODIFY COLUMN`** — if omitted, the constraint is silently removed.

> ⚠️ **Trying to add `NOT NULL` to a column with existing `NULL` values** — the command fails. Update those `NULL` values first.

> ⚠️ **Dropping a table that another table's `FOREIGN KEY` references** — this breaks the referential integrity and causes errors in the child table.

> ⚠️ **Incompatible type changes** — changing `VARCHAR(100)` to `INT` on a column containing text will cause data truncation or errors. Always verify existing data is compatible with the new type.

> ⚠️ **Excessive alterations on production tables** — frequent large-scale structural changes impact performance. Consider whether a schema redesign is more appropriate.

---

## Summary

| Statement | Purpose | Reversible? |
|---|---|---|
| `ALTER TABLE ... ADD COLUMN` | Add a new column | ✅ (drop it to undo) |
| `ALTER TABLE ... DROP COLUMN` | Remove a column and its data | ❌ Not without a backup |
| `ALTER TABLE ... MODIFY COLUMN` | Change data type or constraints | ⚠️ Depends on data compatibility |
| `DROP TABLE` | Remove the entire table and all data | ❌ Not without a backup |

---

## What's Next?

Now that you can shape your database structure, explore more advanced SQL techniques!

- 🔒 **Managing user privileges and permissions**
- ⚡ **Indexes for query performance optimization**
- 🔄 **Transactions and rollback safety**
- 📋 **Schema migration best practices**

> *Tables evolve as your business evolves — `ALTER TABLE` is your tool for keeping pace without starting from scratch!* 🚀

---

