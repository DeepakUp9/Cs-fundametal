# Query Database Information
> *Learn to query database information using SQL.*

---

## Introduction

Suppose we need to ensure everything is organized and optimized in our growing database. To achieve this, we need quick ways to retrieve structural information about the database itself. This is where **querying database information** becomes incredibly valuable — it lets us uncover details about tables, columns, constraints, and other structural elements without manually inspecting the database design.

### 🎯 Learning Goals

- ✅ Understand why querying database information is essential for development and maintenance
- ✅ Use SQL commands to view the list of databases and tables
- ✅ Retrieve details about table structures and columns
- ✅ Access constraint, index, and foreign key metadata

---

## What Is Database Metadata?

**Metadata** is data about data — it describes the structure of the database itself: table names, column definitions, constraints, indexes, and more. Accessing metadata is crucial for:

- Debugging query errors (wrong column name, wrong type)
- Documenting the database structure for teammates
- Verifying constraints and relationships
- Performance tuning and index management

---

## Two Ways to Query Metadata

| Approach | Method | When to Use |
|---|---|---|
| **`INFORMATION_SCHEMA`** | Standard SQL views | Detailed, filterable metadata queries |
| **`SHOW` statements** | MySQL shorthand | Quick, convenient lookups |

Both approaches give access to the same underlying information — `INFORMATION_SCHEMA` is more flexible; `SHOW` statements are faster to type.

---

## The `INFORMATION_SCHEMA`

`INFORMATION_SCHEMA` is a special built-in database that contains read-only views describing the database structure. Think of it as a **dictionary of your database**.

### Common `INFORMATION_SCHEMA` Views

| View | Contains |
|---|---|
| `SCHEMATA` | All databases on the server |
| `TABLES` | All tables (and views) in a database |
| `COLUMNS` | Column definitions for all tables |
| `TABLE_CONSTRAINTS` | Constraint names and types |
| `KEY_COLUMN_USAGE` | Key columns and foreign key references |
| `STATISTICS` | Index information |

> 💡 `INFORMATION_SCHEMA` is itself a database. You can query it like any other: `USE INFORMATION_SCHEMA;` or `SELECT ... FROM INFORMATION_SCHEMA.view_name`.

---

## 1️⃣ Listing All Databases

### Using `INFORMATION_SCHEMA`

```sql
SELECT SCHEMA_NAME AS DatabaseName
FROM INFORMATION_SCHEMA.SCHEMATA;
```

#### Sample Output

| DatabaseName |
|---|
| information_schema |
| mysql |
| OnlineStore |

### Using `SHOW`

```sql
SHOW DATABASES;
```

---

## 2️⃣ Querying the Currently Selected Database

```sql
-- Shows which database is currently active
SELECT DATABASE();

-- Alternative using DUAL (a special single-row table)
SELECT DATABASE() FROM DUAL;
```

> 💡 `DUAL` is a special one-row, one-column table for queries that don't need actual table data. `SELECT 1+1;` and `SELECT 1+1 FROM DUAL;` produce identical results.

---

## 3️⃣ Listing Tables in a Database

### Using `INFORMATION_SCHEMA`

```sql
SELECT TABLE_NAME
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'OnlineStore';
```

#### Sample Output

| TABLE_NAME |
|---|
| Categories |
| Customers |
| Order_Details |
| Orders |
| Product_Suppliers |
| Products |
| Suppliers |

### Using `SHOW`

```sql
USE OnlineStore;
SHOW TABLES;
```

---

## 4️⃣ Retrieving Column Information

### Using `INFORMATION_SCHEMA`

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE, COLUMN_KEY
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Products'
  AND TABLE_SCHEMA = 'OnlineStore';
```

#### Column Meanings

| Column | Meaning |
|---|---|
| `COLUMN_NAME` | Name of the column |
| `DATA_TYPE` | The data type (int, varchar, decimal, etc.) |
| `IS_NULLABLE` | `YES` if the column allows NULL, `NO` if NOT NULL |
| `COLUMN_KEY` | `PRI` = primary key, `UNI` = unique, `MUL` = foreign key or index |

#### Sample Output

| COLUMN_NAME | DATA_TYPE | IS_NULLABLE | COLUMN_KEY |
|---|---|---|---|
| ProductID | int | NO | PRI |
| ProductName | varchar | NO | UNI |
| CategoryID | int | YES | MUL |
| Price | decimal | NO | |
| Stock | int | NO | |

### Using `DESCRIBE` / `SHOW COLUMNS`

```sql
-- Option 1: DESCRIBE (most common shorthand)
DESCRIBE Products;

-- Option 2: SHOW COLUMNS (equivalent to DESCRIBE)
SHOW COLUMNS FROM Products;
```

> 💡 `DESCRIBE TableName` is a quick alias for `SHOW COLUMNS FROM TableName`. Both display the same output.

---

## 5️⃣ Understanding Constraints

### Using `INFORMATION_SCHEMA`

```sql
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'Products'
  AND TABLE_SCHEMA = 'OnlineStore';
```

#### Sample Output

| CONSTRAINT_NAME | CONSTRAINT_TYPE |
|---|---|
| PRIMARY | PRIMARY KEY |
| ProductName | UNIQUE |
| Products_ibfk_1 | FOREIGN KEY |

### `CONSTRAINT_TYPE` Values

| Value | Meaning |
|---|---|
| `PRIMARY KEY` | Primary key constraint |
| `UNIQUE` | Unique constraint |
| `FOREIGN KEY` | Foreign key referencing another table |
| `CHECK` | Check constraint (value must satisfy a condition) |

---

## 6️⃣ Retrieving Index Information

### Using `INFORMATION_SCHEMA`

```sql
SELECT TABLE_NAME, INDEX_NAME, COLUMN_NAME, NON_UNIQUE
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'OnlineStore';
```

#### Sample Output (partial)

| TABLE_NAME | INDEX_NAME | COLUMN_NAME | NON_UNIQUE |
|---|---|---|---|
| Products | PRIMARY | ProductID | 0 |
| Products | ProductName | ProductName | 0 |
| Products | idx_products_price | Price | 1 |
| Orders | PRIMARY | OrderID | 0 |

> 💡 `NON_UNIQUE = 0` means the index enforces uniqueness (like `PRIMARY KEY` or `UNIQUE`). `NON_UNIQUE = 1` means duplicate values are allowed in that column.

### Using `SHOW`

```sql
SHOW INDEX FROM Products;
```

---

## 7️⃣ Retrieving Foreign Key Constraints

```sql
SELECT CONSTRAINT_NAME,
       TABLE_NAME,
       COLUMN_NAME,
       REFERENCED_TABLE_NAME,
       REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'OnlineStore'
  AND REFERENCED_TABLE_NAME IS NOT NULL;
```

#### Sample Output

| CONSTRAINT_NAME | TABLE_NAME | COLUMN_NAME | REFERENCED_TABLE_NAME | REFERENCED_COLUMN_NAME |
|---|---|---|---|---|
| Products_ibfk_1 | Products | CategoryID | Categories | CategoryID |
| Orders_ibfk_1 | Orders | CustomerID | Customers | CustomerID |
| Order_Details_ibfk_1 | Order_Details | OrderID | Orders | OrderID |
| Order_Details_ibfk_2 | Order_Details | ProductID | Products | ProductID |

> 💡 Filtering `WHERE REFERENCED_TABLE_NAME IS NOT NULL` excludes primary key entries and shows only actual foreign key relationships.

---

## 8️⃣ Viewing Table Creation Scripts

```sql
-- Shows the exact CREATE TABLE statement used to define a table
SHOW CREATE TABLE Products;
```

This is useful for:
- Documenting the database structure
- Copying table definitions to another database
- Auditing constraints and indexes on a table

---

## 9️⃣ Accessing Views Metadata

Views are treated similarly to tables in `INFORMATION_SCHEMA`.

### List All Views in a Database

```sql
SELECT TABLE_NAME AS ViewName
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'OnlineStore'
  AND TABLE_TYPE = 'VIEW';
```

### View the Definition of a View

```sql
-- Shows the original CREATE VIEW query
SHOW CREATE VIEW ViewName;
```

---

## Full `SHOW` Statements Reference

```sql
-- List all databases on the server
SHOW DATABASES;

-- List all tables in the currently selected database
USE OnlineStore;
SHOW TABLES;

-- Show column details of a table (two equivalent forms)
DESCRIBE Customers;
SHOW COLUMNS FROM Customers;

-- Show all indexes on a table
SHOW INDEX FROM Customers;

-- Show the SQL script that created a table
SHOW CREATE TABLE Customers;

-- Show the SQL script that created a view
SHOW CREATE VIEW HighPricedElectronics;
```

---

## `INFORMATION_SCHEMA` vs. `SHOW` Statements

| Task | `INFORMATION_SCHEMA` | `SHOW` Statement |
|---|---|---|
| List databases | `SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA` | `SHOW DATABASES` |
| List tables | `SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'db'` | `SHOW TABLES` |
| Column details | `SELECT ... FROM INFORMATION_SCHEMA.COLUMNS WHERE ...` | `DESCRIBE TableName` |
| Index info | `SELECT ... FROM INFORMATION_SCHEMA.STATISTICS WHERE ...` | `SHOW INDEX FROM TableName` |
| Constraints | `SELECT ... FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS WHERE ...` | — |
| Foreign keys | `SELECT ... FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE ...` | — |
| Table script | — | `SHOW CREATE TABLE TableName` |
| **Filterable?** | ✅ Yes — full `WHERE` clause support | ⚠️ Limited filtering |
| **Joinable?** | ✅ Yes — join with other views | ❌ No |
| **Best for** | Complex, targeted metadata queries | Quick, everyday lookups |

---

## 🧩 Code Exercise

### Task 1 — List All Columns with Data Types and Nullability

**Write a query to list all columns of the `Customers` table, including their data types and nullability.**

```sql
SELECT COLUMN_NAME, DATA_TYPE, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Customers'
  AND TABLE_SCHEMA = 'OnlineStore';
```

#### Expected Output

| COLUMN_NAME | DATA_TYPE | IS_NULLABLE |
|---|---|---|
| Address | varchar | YES |
| CustomerID | int | NO |
| CustomerName | varchar | NO |
| Email | varchar | YES |
| Phone | varchar | YES |

> 💡 Results are returned in alphabetical order by `COLUMN_NAME` by default. Add `ORDER BY ORDINAL_POSITION` to get them in table-definition order.

---

## Metadata Quick Reference

| Goal | Query |
|---|---|
| List all databases | `SHOW DATABASES;` or `SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;` |
| Current database | `SELECT DATABASE();` |
| List tables | `SHOW TABLES;` or `SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='db';` |
| Column details | `DESCRIBE TableName;` or `SELECT ... FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='t';` |
| Constraints | `SELECT ... FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS WHERE TABLE_NAME='t';` |
| Indexes | `SHOW INDEX FROM TableName;` or `SELECT ... FROM INFORMATION_SCHEMA.STATISTICS WHERE ...;` |
| Foreign keys | `SELECT ... FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE REFERENCED_TABLE_NAME IS NOT NULL;` |
| Table script | `SHOW CREATE TABLE TableName;` |
| List views | `SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE='VIEW';` |
| View script | `SHOW CREATE VIEW ViewName;` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always filter by `TABLE_SCHEMA`** | Prevents mixing results from multiple databases |
| **Run `USE db` before `SHOW TABLES`** | `SHOW TABLES` only works when a database is selected |
| **Check structure before inserting** | `DESCRIBE` before `INSERT` prevents column mismatch errors |
| **Use `INFORMATION_SCHEMA` for joinable queries** | Enables combining metadata from multiple views in one query |
| **Use metadata for documentation** | Keeps team members informed about the current database structure |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting `TABLE_SCHEMA` in `INFORMATION_SCHEMA` queries** — without it, results include all databases on the server, producing a confusing mix.

> ⚠️ **Running `SHOW TABLES` without first selecting a database** — always run `USE DatabaseName;` first, or use `SHOW TABLES IN DatabaseName;`.

> ⚠️ **Misreading `COLUMN_KEY`** — `PRI` = primary key, `UNI` = unique, `MUL` = index or foreign key (can have duplicates). They are not interchangeable.

> ⚠️ **Running `DESCRIBE` on a non-existent table** — double-check table names before running. A typo produces an error immediately.

> ⚠️ **Confusing `SHOW TABLES` with `SHOW DATABASES`** — `SHOW TABLES` lists tables within the selected database; `SHOW DATABASES` lists all databases on the server.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Metadata | Structural information about the database — tables, columns, constraints, indexes |
| `INFORMATION_SCHEMA` | Built-in read-only views describing all database objects |
| `SHOW` statements | Quick MySQL shorthand for common metadata lookups |
| `TABLE_SCHEMA` filter | Always specify the database name to avoid mixing results |
| `DESCRIBE` | Fast way to inspect a table's column structure |
| Foreign key metadata | Use `KEY_COLUMN_USAGE` with `REFERENCED_TABLE_NAME IS NOT NULL` |

---

## What's Next?

Now that you can inspect and understand your database's structure, explore the final advanced topics!

- 🔒 **Managing user privileges and access control**
- 🔄 **Transactions, `COMMIT`, and `ROLLBACK`**
- ⚡ **Query optimization with `EXPLAIN`**
- 📋 **Database backup and recovery strategies**

> *Metadata is your map of the database — query it regularly and you'll always know exactly what your data looks like!* 🚀

---

