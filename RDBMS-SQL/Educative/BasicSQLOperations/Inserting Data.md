# Inserting Data
> *Learn to insert data in databases.*

---

## Introduction

Imagine we are managing an online store and have just set up our database structure with tables for products, customers, and orders. The next crucial step is **populating these tables with data** so our application can function effectively. Think of this step as stocking the shelves of a store — without products in the inventory, customers wouldn't have anything to buy. In the same way, databases need data to function and be useful.

### 🎯 Learning Goals

- ✅ Use the `INSERT INTO` statement to add single and multiple rows of data
- ✅ Use `AUTO_INCREMENT` and understand its concept

---

## The `INSERT INTO` Statement

In SQL, we insert data into database tables using the `INSERT INTO` statement. There are several ways to use it depending on how many records we need to add and how the table is set up.

---

## 1️⃣ Inserting a Single Record

### Syntax

```sql
INSERT INTO TableName (Column1, Column2, ...)
VALUES (Value1, Value2, ...);
```

| Part | Description |
|---|---|
| `TableName` | The table we are inserting data into |
| `Column1, Column2` | The columns to be populated |
| `Value1, Value2` | The values for the defined columns — must match order and type |

### Example

```sql
-- Add a new Smartphone product to the Products table
INSERT INTO Products (ProductID, ProductName, CategoryID, Price, Stock)
VALUES (1, 'Smartphone', 1, 635.79, 50);
```

This adds a "Smartphone" belonging to the `Electronics` category (`CategoryID = 1`), priced at $635.79 with 50 units in stock.

### Alternative — Without Column Names

You can insert values without listing column names, but the values **must be in the exact same order** as the columns were defined in the table:

```sql
-- Values must match the table's column order exactly
INSERT INTO Products
VALUES (1, 'Smartphone', 1, 635.79, 50);
```

> ⚠️ This approach is risky — if the table structure changes (columns added, reordered, or removed), your insert will break or insert data into the wrong columns. **Always specify column names** for clarity and safety.

---

## 2️⃣ Bulk Inserts — Multiple Records at Once

When adding many records, inserting them one by one is slow and inefficient. **Bulk inserts** allow multiple rows to be added in a single statement, reducing the number of commands executed and saving time and resources.

### Syntax

```sql
INSERT INTO TableName (Column1, Column2, ...)
VALUES
    (Value1A, Value1B, ...),
    (Value2A, Value2B, ...),
    (Value3A, Value3B, ...);
```

### Example

```sql
-- Add three products at once
INSERT INTO Products (ProductID, ProductName, CategoryID, Price, Stock)
VALUES
    (2, 'Dumbbell Set',        3, 82.50,   15),
    (3, 'Laptop',              1, 1099.99, 30),
    (4, 'Smart Outdoor Light', 4, 75.62,   25);
```

### Single vs. Bulk Insert — Comparison

| Approach | When to Use | Efficiency |
|---|---|---|
| Single `INSERT` | Adding one new record | Fine for small, occasional inserts |
| Bulk `INSERT` | Adding multiple records at once | Much faster — fewer database transactions |

> ✅ **Always prefer bulk inserts** when loading multiple records. Each `INSERT` statement is a separate database transaction — fewer transactions means better performance.

---

## 3️⃣ Using `AUTO_INCREMENT` with `INSERT INTO`

When a column is set to `AUTO_INCREMENT`, the database **automatically generates a unique value** for that column on each insert. You simply skip that column in your `INSERT INTO` statement.

### Key Facts About `AUTO_INCREMENT`

| Rule | Detail |
|---|---|
| Default start | Begins at `1` |
| Increment step | Increases by `1` for each new row |
| Minimum value | Can never be lower than `0` |
| Per table | Only **one** `AUTO_INCREMENT` column allowed per table |
| Manual override | You can still manually insert a specific ID if needed |

### Example — Skipping the AUTO_INCREMENT Column

```sql
-- ProductID is AUTO_INCREMENT, so we skip it
INSERT INTO Products (ProductName, CategoryID, Price, Stock)
VALUES ('Knife Set', 2, 50.00, 25);
```

The database automatically assigns the next available `ProductID` — no manual input needed, no risk of duplicates.

### Example — Full Bulk Insert with AUTO_INCREMENT

```sql
-- Single record insert
INSERT INTO Products (ProductName, CategoryID, Price, Stock)
VALUES ('Smartphone', 1, 635.79, 50);

-- Bulk insert — all IDs assigned automatically
INSERT INTO Products (ProductName, CategoryID, Price, Stock)
VALUES
    ('Dumbbell Set',        3, 82.50,   15),
    ('Laptop',              1, 1099.99, 30),
    ('Smart Outdoor Light', 4, 75.62,   25),
    ('Knife Set',           2, 50.00,   25),
    ('Ballpoint Pen Set',   5, 5.00,    150),
    ('Bluetooth Speaker',   1, 135.50,  100),
    ('Blender',             2, 90.25,   40),
    ('Cookware Set',        2, 129.99,  20),
    ('Treadmill',           3, 1499.99, 10);

-- Verify all inserted records
SELECT * FROM Products;
```

### Expected Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 2 | Dumbbell Set | 3 | 82.50 | 15 |
| 3 | Laptop | 1 | 1099.99 | 30 |
| 4 | Smart Outdoor Light | 4 | 75.62 | 25 |
| 5 | Knife Set | 2 | 50.00 | 25 |
| 6 | Ballpoint Pen Set | 5 | 5.00 | 150 |
| 7 | Bluetooth Speaker | 1 | 135.50 | 100 |
| 8 | Blender | 2 | 90.25 | 40 |
| 9 | Cookware Set | 2 | 129.99 | 20 |
| 10 | Treadmill | 3 | 1499.99 | 10 |

> 💡 **Note:** If you manually insert a record with a specific `ProductID` (e.g., `ProductID = 50`), then `AUTO_INCREMENT` will assign the next ID as `51` — always one more than the highest existing value.

---

## 🧪 Complete Example — Populating the OnlineStore

Here's a full script that inserts data into all core tables of the `OnlineStore` database:

```sql
USE OnlineStore;

-- Insert categories
INSERT INTO Categories (CategoryName)
VALUES
    ('Electronics'),
    ('Kitchenware'),
    ('Fitness'),
    ('Outdoor'),
    ('Stationery');

-- Insert customers
INSERT INTO Customers (CustomerName, Email, Phone, Address)
VALUES
    ('John Doe',       'john.doe@smail.com',      '413-456-6862', '123 Elm Street, Springfield, 01103'),
    ('Jane Smith',     'jane.smith@inlook.com',   '708-567-5234', '456 Maple Avenue, Riverside, 60546'),
    ('Alice Johnson',  'alice.j@jmail.com',        '317-678-5717', '789 Oak Lane, Greenwood, 46142');

-- Insert products (CategoryID must exist in Categories)
INSERT INTO Products (ProductName, CategoryID, Price, Stock)
VALUES
    ('Smartphone',   1, 635.79,  50),
    ('Laptop',       1, 1099.99, 30),
    ('Blender',      2, 90.25,   40),
    ('Dumbbell Set', 3, 82.50,   15);

-- Insert orders (CustomerID must exist in Customers)
INSERT INTO Orders (CustomerID, OrderDate, TotalAmount)
VALUES
    (1, '2024-05-22', 235.50),
    (2, '2024-07-03', 2639.99),
    (3, '2024-11-07', 758.99);

-- Verify data
SELECT * FROM Categories;
SELECT * FROM Products;
SELECT * FROM Customers;
SELECT * FROM Orders;
```

---

## 🧩 Code Exercises

---

### Task 1 — Insert Without Specifying AUTO_INCREMENT

**Given the `Categories` table, insert a new category "Books" without specifying `CategoryID` (it is `AUTO_INCREMENT`).**

#### Starting Data

| CategoryID | CategoryName |
|---|---|
| 1 | Electronics |
| 2 | Kitchenware |
| 3 | Fitness |
| 4 | Outdoor |
| 5 | Stationery |

#### Solution

```sql
INSERT INTO Categories (CategoryName)
VALUES ('Books');
```

#### Expected Output

| CategoryID | CategoryName |
|---|---|
| 1 | Electronics |
| 2 | Kitchenware |
| 3 | Fitness |
| 4 | Outdoor |
| 5 | Stationery |
| 6 | Books |

> ✅ Since `CategoryID` is `AUTO_INCREMENT`, the database assigns `6` automatically — the next value after the existing highest ID of `5`.

---

### Task 2 — Bulk Insert with Manual IDs

**Add three new products to the `Products` table using a single `INSERT INTO` statement.**

> Note: In this task, `ProductID` is **not** set to `AUTO_INCREMENT`, so IDs must be provided manually.

#### Products to Add

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 56 | E-book Reader | 1 (Electronics) | $129.99 | 40 |
| 109 | Electric Cooker | 2 (Kitchenware) | $99.30 | 20 |
| 5 | Animal Farm | 4 (Outdoor) | $5.00 | 5 |

#### Starting Data

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Wireless Mouse | 1 | 20.99 | 50 |

#### Solution

```sql
INSERT INTO Products (ProductID, ProductName, CategoryID, Price, Stock)
VALUES
    (56,  'E-book Reader',  1, 129.99, 40),
    (109, 'Electric Cooker', 2, 99.30,  20),
    (5,   'Animal Farm',    4, 5.00,   5);
```

#### Expected Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Wireless Mouse | 1 | 20.99 | 50 |
| 56 | E-book Reader | 1 | 129.99 | 40 |
| 109 | Electric Cooker | 2 | 99.30 | 20 |
| 5 | Animal Farm | 4 | 5.00 | 5 |

---

## INSERT Methods — Quick Reference

| Method | Syntax | Best For |
|---|---|---|
| Single row, with columns | `INSERT INTO t (c1, c2) VALUES (v1, v2)` | One record, explicit and safe |
| Single row, without columns | `INSERT INTO t VALUES (v1, v2)` | Quick inserts (risky if schema changes) |
| Bulk insert | `INSERT INTO t (c1) VALUES (v1), (v2), (v3)` | Multiple records efficiently |
| With AUTO_INCREMENT | Skip the ID column entirely | Automatic unique ID generation |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always list column names** | Protects against table structure changes breaking your inserts |
| **Use `AUTO_INCREMENT` for primary keys** | Eliminates manual ID management and duplicate key errors |
| **Use bulk inserts for multiple rows** | Fewer database transactions = better performance |
| **Match data types to column definitions** | Mismatches cause insertion errors |
| **Verify with `SELECT *` after inserting** | Confirms data was inserted correctly |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Not specifying column names** in `INSERT INTO` — if the table structure changes, your values may go into the wrong columns or cause errors.

> ⚠️ **Data type mismatch** — inserting a string into an `INT` column or text into a `DATE` column will cause an error.

> ⚠️ **Inserting duplicate values into a `UNIQUE` or `PRIMARY KEY` column** — the database will reject the entire insert.

> ⚠️ **Violating a `FOREIGN KEY` constraint** — e.g., inserting an order with a `CustomerID` that doesn't exist in the `Customers` table.

> ⚠️ **Skipping `NOT NULL` columns** without providing a default value — the insert will fail.

> ⚠️ **Inserting rows one at a time in a loop** instead of using bulk insert — dramatically slower for large datasets.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `INSERT INTO ... VALUES` | Core statement for adding data to a table |
| Specifying column names | Always recommended for safety and clarity |
| Bulk insert | Add multiple rows in one statement for efficiency |
| `AUTO_INCREMENT` | Database auto-assigns the next unique ID — skip the column in your insert |
| Verify with `SELECT *` | Always confirm inserted data looks correct |

---

## What's Next?

Now that data is in our tables, it's time to update and manage it!

- ✏️ **Updating Records with `UPDATE`**
- 🗑️ **Deleting Records with `DELETE`**
- 🔍 **Filtering Data with `WHERE`**
- 🔗 **Joining Tables to Query Related Data**

> *Inserting data is like stocking the shelves — once they're full, the real work of managing and querying begins!* 🚀

---

