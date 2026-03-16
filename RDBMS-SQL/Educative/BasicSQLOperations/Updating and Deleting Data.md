# Updating and Deleting Data
> *Learn to update data in the tables and delete rows from the tables.*

---

## Introduction

Imagine we are managing our `OnlineStore` database and need to update product prices, remove discontinued items, or reflect a customer's new address. To keep the database accurate, we need tools to modify or remove data effectively. Keeping data current is essential for accurate reporting and customer satisfaction.

### 🎯 Learning Goals

- ✅ Understand when and why to use the `UPDATE` statement
- ✅ Learn how to modify existing records effectively
- ✅ Understand when and why to use the `DELETE` statement
- ✅ Learn how to safely remove records from a table

---

## Updating Data with `UPDATE`

Databases must adapt as data evolves over time. In an online store, that means correcting errors, updating prices, or editing stock levels. The `UPDATE` statement modifies one or more fields of an existing record — **without** having to delete and re-insert the entire row.

### Syntax

```sql
UPDATE TableName
SET Column1 = Value1, Column2 = Value2, ...
WHERE condition;
```

| Part | Description |
|---|---|
| `TableName` | The table containing the record(s) to update |
| `SET` | Specifies the column(s) and their new value(s) |
| `WHERE` | Identifies which row(s) to update — **always include this!** |

> ⚠️ **Without a `WHERE` clause, every row in the table will be updated** — one of the most dangerous mistakes in SQL.

---

### Example 1 — Update a Single Column

Increase the price of the `Smartphone` from $635.79 to $850.00 due to rising costs:

```sql
-- Preview before update
SELECT ProductName, Price
FROM Products
WHERE ProductName = 'Smartphone';

-- Perform the update
UPDATE Products
SET Price = 850.00
WHERE ProductName = 'Smartphone';

-- Verify after update
SELECT ProductName, Price
FROM Products
WHERE ProductName = 'Smartphone';
```

**Before:**

| ProductName | Price |
|---|---|
| Smartphone | 635.79 |

**After:**

| ProductName | Price |
|---|---|
| Smartphone | 850.00 |

> ✅ Always run a `SELECT` before and after an `UPDATE` to confirm the change was applied correctly.

---

### Example 2 — Update Multiple Columns at Once

Change the price **and** stock of the `Dumbbell Set` simultaneously:

```sql
-- Preview before update
SELECT ProductName, Price, Stock
FROM Products
WHERE ProductName = 'Dumbbell Set';

-- Update both Price and Stock in one statement
UPDATE Products
SET Price = 50.00, Stock = 100
WHERE ProductName = 'Dumbbell Set';

-- Verify after update
SELECT ProductName, Price, Stock
FROM Products
WHERE ProductName = 'Dumbbell Set';
```

**Before:**

| ProductName | Price | Stock |
|---|---|---|
| Dumbbell Set | 82.50 | 15 |

**After:**

| ProductName | Price | Stock |
|---|---|---|
| Dumbbell Set | 50.00 | 100 |

---

### Example 3 — Update Using Relative Values

Instead of setting an absolute value, you can update relative to the current value using arithmetic:

```sql
-- Increase Yoga Mat price by $5.00 (current: $25.00 → new: $30.00)
UPDATE Products
SET Price = Price + 5.00
WHERE ProductName = 'Yoga Mat';

-- Decrease Laptop stock by 10 (units sold)
UPDATE Products
SET Stock = Stock - 10
WHERE ProductName = 'Laptop';
```

This is far safer than manually calculating the new value — the database handles it, and the math is always based on the current stored value.

---

## Deleting Data with `DELETE`

Sometimes records are no longer relevant — discontinued products, inactive customers, or incorrect entries. The `DELETE` statement removes one or more rows from a table.

### Syntax

```sql
DELETE FROM TableName
WHERE condition;
```

> ⚠️ **Without a `WHERE` clause, ALL rows in the table are permanently deleted.** This action cannot be undone.

---

### Example 1 — Delete a Single Record

Remove the `Smartphone` from the `Products` table (discontinued):

```sql
-- Preview before deletion
SELECT ProductName, Price
FROM Products;

-- Delete the Smartphone
DELETE FROM Products
WHERE ProductName = 'Smartphone';

-- Verify after deletion
SELECT ProductName, Price
FROM Products;
```

> ✅ Always run `SELECT` after a `DELETE` to confirm the record is gone.

---

### Example 2 — Delete Multiple Records Using `OR`

Remove several discontinued products in one statement:

```sql
-- Delete Smart TV, Notebook, and Dumbbell Set
DELETE FROM Products
WHERE ProductName = 'Smart TV'
   OR ProductName = 'Notebook'
   OR ProductName = 'Dumbbell Set';
```

---

## 🧪 Full Example — Combined UPDATE and DELETE Script

Here's a real-world script that updates pricing and removes discontinued products from the `Products` table:

```sql
-- View the table before any changes
SELECT * FROM Products;

-- Increase the price of the 'Yoga Mat' by $5.00 (25.00 → 30.00)
UPDATE Products
SET Price = Price + 5.00
WHERE ProductName = 'Yoga Mat';

-- Decrease the stock of the 'Laptop' by 10 (units sold)
UPDATE Products
SET Stock = Stock - 10
WHERE ProductName = 'Laptop';

-- Remove three discontinued products in one statement
DELETE FROM Products
WHERE ProductName = 'Smart TV'
   OR ProductName = 'Notebook'
   OR ProductName = 'Dumbbell Set';

-- View the table after all changes
SELECT * FROM Products;
```

The `SELECT` statements at the beginning and end let you compare the table state before and after all modifications.

---

## UPDATE vs. DELETE — Quick Comparison

| Feature | `UPDATE` | `DELETE` |
|---|---|---|
| **Purpose** | Modifies existing row data | Removes entire rows |
| **`WHERE` required?** | Strongly recommended | Strongly recommended |
| **Without `WHERE`** | Updates ALL rows | Deletes ALL rows |
| **Reversible?** | Only if you have a backup | Only if you have a backup |
| **Affects structure?** | No — only data values | No — only rows |

---

## 🧩 Code Exercises

Both exercises use the `Customers` table:

```sql
CREATE TABLE Customers (
    CustomerID   INT PRIMARY KEY AUTO_INCREMENT,
    CustomerName VARCHAR(50) NOT NULL,
    Email        VARCHAR(50),
    Phone        VARCHAR(15),
    Address      VARCHAR(100)
);
```

---

### Task 1 — Update a Customer's Address

**Alice Johnson has moved from Greenwood to Springfield. Update her address to `"127 Rose St, Springfield, 62701"`.**

#### Current Record

| CustomerID | CustomerName | Email | Phone | Address |
|---|---|---|---|---|
| 3 | Alice Johnson | alice.j@jmail.com | 317-678-5717 | 789 Oak Lane, Greenwood, 46142 |

#### Solution

```sql
UPDATE Customers
SET Address = '127 Rose St, Springfield, 62701'
WHERE CustomerName = 'Alice Johnson';

-- Verify the change
SELECT *
FROM Customers
WHERE CustomerName = 'Alice Johnson';
```

#### Expected Output

| CustomerID | CustomerName | Email | Phone | Address |
|---|---|---|---|---|
| 3 | Alice Johnson | alice.j@jmail.com | 317-678-5717 | 127 Rose St, Springfield, 62701 |

> 💡 Using `CustomerName` in the `WHERE` clause works here, but using the `PRIMARY KEY` (`CustomerID = 3`) is safer — names can have duplicates, IDs cannot.

---

### Task 2 — Delete Inactive Customers

**Remove customers who haven't ordered in a year: John Doe, Michael Carter, Charlotte Miller, and Lucas Anderson.**

#### Starting Table (sample)

| CustomerID | CustomerName | Address |
|---|---|---|
| 1 | John Doe | 123 Elm Street, Springfield, 01103 |
| 2 | Jane Smith | 456 Maple Avenue, Riverside, 60546 |
| 3 | Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| 4 | Michael Carter | 101 Pine Court, Meadowbrook, 35242 |
| 5 | Charlie Davis | 202 Birch Drive, Lakeview, 97630 |
| … | … | … |
| 21 | Charlotte Miller | 66 Pine Trail, Sunset Valley, 78745 |
| 22 | Lucas Anderson | 77 Walnut Place, Lakeview, 48850 |
| 23 | Emma Watson | 303 Birch Road, Greenwood, 29646 |

#### Solution

```sql
-- Use PRIMARY KEY (CustomerID) for precise, safe deletion
DELETE FROM Customers
WHERE CustomerID = 1
   OR CustomerID = 4
   OR CustomerID = 21
   OR CustomerID = 22;

-- Verify the remaining customers
SELECT CustomerID, CustomerName, Address
FROM Customers;
```

#### Expected Output

| CustomerID | CustomerName | Address |
|---|---|---|
| 2 | Jane Smith | 456 Maple Avenue, Riverside, 60546 |
| 3 | Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| 5 | Charlie Davis | 202 Birch Drive, Lakeview, 97630 |
| … | … | … |
| 20 | Benjamin Wilson | 55 Oak Path, Meadowbrook, 35242 |
| 23 | Emma Watson | 303 Birch Road, Greenwood, 29646 |

> ✅ Using `CustomerID` (primary key) in the `WHERE` clause is the safest approach — it guarantees exactly the right rows are deleted, even if two customers share the same name.

---

## Safe UPDATE & DELETE Workflow

Follow this workflow every time you update or delete data:

```
Step 1 ── Run SELECT with your WHERE condition
         Confirm which rows will be affected

Step 2 ── Run the UPDATE or DELETE
         Apply the change to only those rows

Step 3 ── Run SELECT again
         Verify the result looks exactly as expected
```

```sql
-- Step 1: Preview
SELECT * FROM Products WHERE ProductName = 'Laptop';

-- Step 2: Update
UPDATE Products SET Price = 1199.99 WHERE ProductName = 'Laptop';

-- Step 3: Verify
SELECT * FROM Products WHERE ProductName = 'Laptop';
```

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always use `WHERE` with `UPDATE` and `DELETE`** | Without it, all rows are affected — potentially catastrophic |
| **Preview with `SELECT` before modifying** | Confirms exactly which rows will be changed or removed |
| **Use primary keys in `WHERE` clauses** | More precise and reliable than using names or other non-unique values |
| **Back up the database before bulk changes** | Provides a recovery option if something goes wrong |
| **Test on a backup/test database first** | Safe environment to validate queries before production |
| **Use relative updates (`Price = Price + 5`)** | Avoids manual calculation errors on current values |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting `WHERE` in `UPDATE` or `DELETE`** — updates or deletes **every row** in the table. Always double-check before executing.

> ⚠️ **Wrong condition in `WHERE`** — incorrect filters lead to updating or deleting the wrong records. Preview first with `SELECT`.

> ⚠️ **Updating a primary key column** — if other tables reference it via `FOREIGN KEY`, this breaks those relationships. Update child table foreign keys first, then the parent.

> ⚠️ **Deleting a parent record before its children** — e.g., deleting a `Customer` while they still have rows in `Orders` will violate the `FOREIGN KEY` constraint. Delete child records first.

> ⚠️ **Not verifying after the change** — always run `SELECT` after `UPDATE` or `DELETE` to confirm the result.

---

## Summary

| Statement | Purpose | Key Clause |
|---|---|---|
| `UPDATE ... SET ...` | Modifies values in existing rows | `WHERE` — targets specific rows |
| `DELETE FROM ...` | Removes entire rows from a table | `WHERE` — targets specific rows |
| `SELECT` (before/after) | Previews and verifies changes | `WHERE` — same condition as UPDATE/DELETE |

---

## What's Next?

Now that you can create, read, update, and delete data — you have the full **CRUD** toolkit!

- 📊 **Sorting results with `ORDER BY`**
- 🔢 **Aggregating data with `COUNT`, `SUM`, `AVG`**
- 🔗 **Combining tables with `JOIN`**
- 🔎 **Pattern matching with `LIKE` and `BETWEEN`**

> *With `UPDATE` and `DELETE`, you control your data's lifecycle — always use them carefully and intentionally!* 🚀

---

