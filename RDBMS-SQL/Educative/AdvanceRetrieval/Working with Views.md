# Working with Views
> *Learn about views in SQL.*

---

## Introduction

Imagine a complex query involving multiple joins and filters that needs to be executed repeatedly. Each time it's needed, it must be rewritten from scratch. Instead of recreating the same query every time, we can build a **virtual table called a view** — a stored query definition that can be used just like a regular table. Views make data access more secure, convenient, and organized.

### 🎯 Learning Goals

- ✅ Understand what views are and why they are useful
- ✅ Learn how to create a view
- ✅ Learn how to edit and update a view
- ✅ Learn how to use a view in joins
- ✅ Learn how to delete a view

---

## What Is a View?

A **view** is a virtual table that represents the result of a pre-written query. It does **not store data** — it stores the query definition. Every time the view is accessed, the underlying query runs and returns fresh results.

### Why Are Views Useful?

| Benefit | Description |
|---|---|
| 🔁 **Reusability** | Write a complex query once; use it everywhere by name |
| 🔒 **Security** | Grant users access to a view without exposing the underlying tables |
| 🧹 **Simplicity** | Reduce repetition; complex logic lives in one place |
| 🔧 **Maintainability** | Update logic in the view definition — all queries using it automatically benefit |
| 📐 **Consistency** | Every query using the view gets the same, centralized logic |

---

## Creating a View

When we create a view, we define a query and give it a name. That name can then be used anywhere a table name is used.

### Syntax

```sql
CREATE VIEW ViewName AS
SELECT Column1, Column2, ...
FROM TableName
WHERE condition;
```

### Example — High-Priced Electronics View

```sql
-- Create the view
CREATE VIEW HighPricedElectronics AS
SELECT ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price > 500;
```

Once created, the view can be queried **just like a table**:

```sql
-- Use the view
SELECT * FROM HighPricedElectronics;
```

#### Sample Output

| ProductName | Price | Stock |
|---|---|---|
| Laptop | 1099.99 | 30 |
| Smart TV | 2599.99 | 20 |
| Smartwatch | 500.00 | 50 |

> 💡 A view does **not physically store its data** in most SQL implementations — it refers back to the underlying tables each time it is accessed. This means the view always reflects the latest data.

---

## Editing and Updating Views

After creating a view, we may need to add or remove columns, or change the filter criteria. There are two approaches.

### Approach 1 — `CREATE OR REPLACE VIEW`

This redefines the view without dropping it first:

```sql
CREATE OR REPLACE VIEW HighPricedElectronics AS
SELECT ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price > 500 AND Stock > 20;

-- Verify the updated view
SELECT * FROM HighPricedElectronics;
```

> ✅ `CREATE OR REPLACE VIEW` is the cleanest approach — it updates the view in place without disrupting any queries that reference it.

---

### Approach 2 — `DROP` and `CREATE`

Drop the existing view, then recreate it with the updated definition:

```sql
-- Step 1: Drop the view
DROP VIEW HighPricedElectronics;

-- Step 2: Recreate with updated query (now includes ProductID)
CREATE VIEW HighPricedElectronics AS
SELECT ProductID, ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price > 500 AND Stock > 20;

-- Verify
SELECT * FROM HighPricedElectronics;
```

#### Sample Output

| ProductID | ProductName | Price | Stock |
|---|---|---|---|
| 3 | Laptop | 1099.99 | 30 |
| 19 | Smartwatch | 500.00 | 50 |

---

### Can We Update Data Through a View?

| Scenario | Updatable? |
|---|---|
| View references a **single table** with no aggregates | ✅ Usually yes |
| View references **multiple tables** | ❌ Typically not |
| View has **computed/aggregated columns** | ❌ No |
| View uses `GROUP BY`, `DISTINCT`, or `HAVING` | ❌ No |

> 💡 For clarity and simplicity, most developers prefer updating the **base table directly** rather than through a view.

---

## Joining with Views

Views can be used in joins exactly like regular tables. This is especially useful when the view simplifies complex data and you want to combine it with additional information.

### Example — Join a View with Another Table

Find suppliers for the high-priced electronics using the `HighPricedElectronics` view:

```sql
SELECT he.ProductName, ps.SupplierID
FROM HighPricedElectronics AS he
JOIN Product_Suppliers AS ps
ON he.ProductName = (
    SELECT ProductName
    FROM Products
    WHERE Products.ProductID = ps.ProductID
);
```

> ✅ The view acts exactly like a table in the `FROM` clause — making the join logic clean and readable.

### Example — Join Two Views

You can also join views with other views:

```sql
-- Create a second view
CREATE VIEW AffordableProducts AS
SELECT ProductID, ProductName, Price
FROM Products
WHERE Price < 100;

-- Join the two views
SELECT hp.ProductName AS HighEnd, ap.ProductName AS BudgetAlternative
FROM HighPricedElectronics hp, AffordableProducts ap
WHERE hp.CategoryID = ap.CategoryID;
```

---

## Deleting Views

If a view is no longer needed, use `DROP VIEW` to remove it. This **does not affect** any underlying data in the tables — it only removes the saved query definition.

### Syntax

```sql
DROP VIEW ViewName;
```

### Example

```sql
DROP VIEW HighPricedElectronics;
```

After dropping, any query referencing `HighPricedElectronics` will return an error — the view no longer exists.

---

## View Lifecycle — Full Example

Here's the complete lifecycle of a view from creation to deletion:

```sql
-- 1. Create the view
CREATE VIEW HighPricedElectronics AS
SELECT ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price > 500;

-- 2. Query it
SELECT * FROM HighPricedElectronics;

-- 3. Update it (add a stock filter)
CREATE OR REPLACE VIEW HighPricedElectronics AS
SELECT ProductID, ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price > 500 AND Stock > 20;

-- 4. Use it in a join
SELECT he.ProductName, ps.SupplierID
FROM HighPricedElectronics he
JOIN Product_Suppliers ps ON he.ProductID = ps.ProductID;

-- 5. Drop it when no longer needed
DROP VIEW HighPricedElectronics;
```

---

## Views vs. Tables — Key Differences

| Feature | Table | View |
|---|---|---|
| **Stores data** | ✅ Yes | ❌ No (stores the query) |
| **Created with** | `CREATE TABLE` | `CREATE VIEW` |
| **Updated with** | `ALTER TABLE` | `CREATE OR REPLACE VIEW` |
| **Queryable** | ✅ Yes | ✅ Yes (same syntax) |
| **Joinable** | ✅ Yes | ✅ Yes |
| **Data freshness** | Static until changed | Always reflects current underlying data |
| **Directly updatable** | ✅ Yes | ⚠️ Only under certain conditions |

---

## 🧩 Code Exercises

---

### Task 1 — Create a View for Low-Stock Products

**Create a view named `LowStockProducts` to list all products with stock below 20.**

```sql
CREATE VIEW LowStockProducts AS
SELECT ProductName, Stock
FROM Products
WHERE Stock < 20;

-- Query the view
SELECT * FROM LowStockProducts;
```

#### Expected Output

| ProductName | Stock |
|---|---|
| Dumbbell Set | 15 |
| Treadmill | 10 |
| Tent | 10 |

---

### Task 2 — Delete the View

**Drop the `LowStockProducts` view created in Task 1, then verify it's gone.**

Before deletion, the `OnlineStore` database contains:

| Tables_in_OnlineStore |
|---|
| Categories |
| Customers |
| LowStockProducts |
| Order_Details |
| Orders |
| Product_Suppliers |
| Products |
| Suppliers |

```sql
-- Delete the view
DROP VIEW LowStockProducts;

-- Confirm deletion
SHOW TABLES;
```

#### Expected Output

| Tables_in_OnlineStore |
|---|
| Categories |
| Customers |
| Order_Details |
| Orders |
| Product_Suppliers |
| Products |
| Suppliers |

> ✅ `LowStockProducts` is gone. The underlying `Products` table and all its data remain untouched.

---

### Task 3 — Create a View for Stationery Items

**Create a view named `StationeryItems` that selects `ProductID`, `ProductName`, and `Price` from the `Products` table where `CategoryID = 5` (Stationery).**

```sql
CREATE VIEW StationeryItems AS
SELECT ProductID, ProductName, Price
FROM Products
WHERE CategoryID = 5;

-- Query the view
SELECT * FROM StationeryItems;
```

#### Expected Output

| ProductID | ProductName | Price |
|---|---|---|
| 6 | Ballpoint Pen Set | 5.00 |
| 14 | Notebook | 250.50 |
| 15 | Desk Organizer | 85.99 |

---

## Views Quick Reference

| Operation | Syntax |
|---|---|
| Create a view | `CREATE VIEW name AS SELECT ...` |
| Query a view | `SELECT * FROM ViewName` |
| Update a view | `CREATE OR REPLACE VIEW name AS SELECT ...` |
| Drop and recreate | `DROP VIEW name; CREATE VIEW name AS ...` |
| Delete a view | `DROP VIEW ViewName` |
| Join with a view | `FROM ViewName AS alias JOIN Table ON ...` |
| Verify views exist | `SHOW TABLES` (views appear alongside tables) |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Name views descriptively** | `HighPricedElectronics` is clearer than `View1` |
| **Limit columns to only what's needed** | Especially important for security-focused views |
| **Use `CREATE OR REPLACE`** | Cleaner than dropping and recreating when updating |
| **Keep view logic simple** | Overly complex views hurt readability and performance |
| **Document each view** | Helps teammates understand its purpose and usage |
| **Combine views with proper privileges** | Views restrict what columns are visible, but users need the right permissions too |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Thinking views store data** — they don't. A view is just a saved query. Delete the rows from the underlying table and the view will reflect that immediately.

> ⚠️ **Using vague or redundant view names** — names like `MyView` or `TempView` don't communicate purpose. Always name views to reflect what they return.

> ⚠️ **Deeply nesting views** — a view built on a view built on a view becomes very hard to debug and maintain. Keep the chain shallow.

> ⚠️ **Assuming views guarantee security on their own** — a view restricts column/row visibility, but users with direct table access can still query the base table. Always combine views with proper user privileges.

> ⚠️ **Trying to update data through a complex view** — views with joins, aggregates, `GROUP BY`, or `DISTINCT` are not updatable. Update the base table directly.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| View | A virtual table — stores a query, not data |
| `CREATE VIEW` | Defines and names the view |
| `CREATE OR REPLACE VIEW` | Updates an existing view without dropping it |
| `DROP VIEW` | Removes the view definition (not the underlying data) |
| Joining with views | Views can be joined just like regular tables |
| Data freshness | Views always reflect the latest state of underlying tables |
| Updatability | Simple single-table views may be updatable; complex views are not |

---

## What's Next?

Now that you can simplify and secure data access with views, explore more advanced SQL features!

- 🪟 **Window functions for rankings and running totals**
- 🔄 **Common Table Expressions (CTEs)**
- ⚡ **Indexes for query performance optimization**
- 🔐 **User privileges and database security**

> *Views are your database's clean interface — they hide the complexity and show only what's needed!* 🚀

---

