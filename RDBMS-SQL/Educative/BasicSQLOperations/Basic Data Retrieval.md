# Basic Data Retrieval
> *Learn about retrieving data from databases.*

---

## Introduction

Retrieving information from a database is like searching for items in a well-organized warehouse. Each table — such as `Products` or `Categories` — represents a shelf where specific types of items are stored. To find what we need, we use a precise map that leads us to the exact shelf and item we're looking for.

SQL's data retrieval capabilities keep operations running smoothly — whether displaying available products to customers, analyzing sales trends, or managing inventory effectively.

### 🎯 Learning Goals

- ✅ Use the `SELECT` statement to retrieve data
- ✅ Select specific columns from a table
- ✅ Select data based on calculated columns
- ✅ Apply aliases to tables and columns for clarity
- ✅ Use the `DISTINCT` clause to eliminate duplicate records

---

## The `SELECT` Statement

The `SELECT` statement is the **foundation of data retrieval in SQL**. It specifies which columns to retrieve and from which table.

### General Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName;
```

---

## 1️⃣ Selecting All Columns

Use the wildcard `*` to retrieve **every column** from a table without listing them individually. This is useful for getting a full overview of data — for example, an admin viewing the complete product list to analyze inventory.

### Syntax

```sql
SELECT *
FROM TableName;
```

### Example

```sql
-- Retrieve everything from the Products table
SELECT *
FROM Products;
```

> ⚠️ While `SELECT *` is convenient, it retrieves all columns in the order they were defined during table creation — you have no control over column order in the output.

---

## 2️⃣ Selecting Specific Columns

When you only need a few details, listing specific columns is far more efficient. For example, a customer browsing books only needs the name and price — not the `ProductID`, `Stock`, or `CategoryID`.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName;
```

### Example

```sql
-- Retrieve only ProductName and Price from the Products table
SELECT ProductName, Price
FROM Products;
```

### Sample Output

| ProductName | Price |
|---|---|
| Smartphone | 635.79 |
| Dumbbell Set | 82.50 |
| Laptop | 1099.99 |
| Smart Outdoor Light | 75.62 |
| Knife Set | 50.00 |

> ✅ **Advantage over `SELECT *`:** Using `SELECT col1, col2` lets you control the **order** columns appear in your output — they display in the order you list them, not the order they were created in the table.

---

## 3️⃣ Using Aliases

Aliases let you **temporarily rename** tables or columns in a query for clarity and brevity. They exist only for the duration of the query and don't change the actual table or column names.

Aliases are especially helpful when:
- Column or table names are long or unclear
- Multiple tables share the same column name
- You want output labels to match reporting standards

### Aliases for Tables

Assign using the `AS` keyword or simply a space:

```sql
-- Using AS keyword
SELECT p.ProductName, p.Price
FROM Products AS p;

-- Using a space (same result)
SELECT p.ProductName, p.Price
FROM Products p;
```

Here, `Products` is aliased as `p` — you can then reference its columns as `p.ProductName`, `p.Price`, etc.

### Aliases for Columns

Rename output columns to make results clearer:

```sql
-- Rename ProductName → Name, Price → Cost in the output
SELECT ProductName AS Name, Price AS Cost
FROM Products;
```

### Sample Output

| Name | Cost |
|---|---|
| Smartphone | 635.79 |
| Laptop | 1099.99 |
| Bluetooth Speaker | 135.50 |

### Aliases Across Multiple Tables

When two tables share the same column name, aliases eliminate ambiguity:

```sql
-- Without aliases, "name" is ambiguous — which table does it come from?
SELECT e.name AS EmployeeName, m.name AS ManagerName
FROM Employees AS e, Managers AS m;
```

> 💡 **Best Practice:** Choose aliases that clearly represent what the table or column means — `p` for `Products`, `c` for `Customers`, etc.

---

## 4️⃣ Calculated Columns

Calculated columns are **derived on the fly** from existing data using expressions or formulas. They appear in the result set but are **not stored** in the database — they exist only for that query.

### Example — Without Alias

```sql
-- Calculate total stock value per product
SELECT ProductName, Price, Stock, (Price * Stock)
FROM Products;
```

The output will show a column literally named `(Price * Stock)` — which is hard to read.

### Example — With Alias

```sql
-- Give the calculated column a meaningful name
SELECT ProductName, Price, Stock, (Price * Stock) AS TotalValue
FROM Products;
```

### Sample Output

| ProductName | Price | Stock | TotalValue |
|---|---|---|---|
| Smartphone | 635.79 | 50 | 31789.50 |
| Dumbbell Set | 82.50 | 15 | 1237.50 |
| Laptop | 1099.99 | 30 | 32999.70 |
| Bluetooth Speaker | 135.50 | 100 | 13550.00 |
| Treadmill | 1499.99 | 10 | 14999.90 |

### More Calculated Column Examples

```sql
-- Apply a 10% discount and show discounted price
SELECT ProductName,
       Price,
       Price * 0.10 AS Discount,
       Price - (Price * 0.10) AS DiscountedPrice
FROM Products;
```

```sql
-- Calculate how many orders it would take to sell all stock
-- (assuming average order quantity of 2)
SELECT ProductName,
       Stock,
       Stock / 2 AS EstimatedOrdersToSellOut
FROM Products;
```

> ✅ Always use an alias on calculated columns — raw expressions like `(Price * Stock)` are hard to reference or read in reports.

---

## 5️⃣ The `DISTINCT` Clause

The `DISTINCT` clause **removes duplicate values** from the result set, returning only unique entries. It's useful for understanding diversity or groupings in your data.

### Syntax

```sql
SELECT DISTINCT ColumnName
FROM TableName;
```

### Example — Without DISTINCT

```sql
-- Returns all CategoryID values, including duplicates
SELECT CategoryID
FROM Products;
```

| CategoryID |
|---|
| 1 |
| 3 |
| 1 |
| 4 |
| 2 |
| 5 |
| 1 |
| 2 |
| 2 |
| 3 |

### Example — With DISTINCT

```sql
-- Returns only unique CategoryID values
SELECT DISTINCT CategoryID
FROM Products;
```

| CategoryID |
|---|
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |

> ⚠️ Use `DISTINCT` only when necessary — it requires the database to compare all rows and can slow down queries on large datasets.

### DISTINCT on Multiple Columns

```sql
-- Get unique combinations of CategoryID and Stock status
SELECT DISTINCT CategoryID, Stock
FROM Products;
```

This returns unique **pairs** of values — two rows are considered duplicates only if **both** columns match.

---

## 🧪 Putting It All Together

Here's a query that combines specific columns, a table alias, column aliases, and a calculated column:

```sql
SELECT p.ProductName  AS Name,
       p.Price        AS UnitPrice,
       p.Stock        AS AvailableStock,
       (p.Price * p.Stock) AS TotalInventoryValue
FROM Products AS p;
```

### Sample Output

| Name | UnitPrice | AvailableStock | TotalInventoryValue |
|---|---|---|---|
| Smartphone | 635.79 | 50 | 31789.50 |
| Dumbbell Set | 82.50 | 15 | 1237.50 |
| Laptop | 1099.99 | 30 | 32999.70 |
| Smart Outdoor Light | 75.62 | 25 | 1890.50 |
| Knife Set | 50.00 | 25 | 1250.00 |

---

## 🧩 Code Exercises

All exercises use the `Customers` table with the following definition:

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

### Task 1 — Select All Columns

**Write a query to retrieve data from all columns of `Customers`.**

```sql
SELECT *
FROM Customers;
```

#### Expected Output (sample)

| CustomerID | CustomerName | Email | Phone | Address |
|---|---|---|---|---|
| 1 | John Doe | john.doe@smail.com | 413-456-6862 | 123 Elm Street, Springfield, 01103 |
| 2 | Jane Smith | jane.smith@inlook.com | 708-567-5234 | 456 Maple Avenue, Riverside, 60546 |
| 3 | Alice Johnson | alice.j@jmail.com | 317-678-5717 | 789 Oak Lane, Greenwood, 46142 |
| … | … | … | … | … |
| 23 | Emma Watson | emma.w@service.org | 864-789-4731 | 303 Birch Road, Greenwood, 29646 |

---

### Task 2 — Select Specific Columns

**Write a query to select the `CustomerName` and `Address` from the `Customers` table.**

```sql
SELECT CustomerName, Address
FROM Customers;
```

#### Expected Output (sample)

| CustomerName | Address |
|---|---|
| John Doe | 123 Elm Street, Springfield, 01103 |
| Jane Smith | 456 Maple Avenue, Riverside, 60546 |
| Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| … | … |
| Emma Watson | 303 Birch Road, Greenwood, 29646 |

---

### Task 3 — Table and Column Aliases

**Use table and column aliases to select `Email` and `Phone` from `Customers`, displayed as `CustomerEmail` and `CustomerPhone`.**

*(This distinguishes them from the `Email` and `Phone` columns in the `Suppliers` table.)*

```sql
SELECT c.Email AS CustomerEmail,
       c.Phone AS CustomerPhone
FROM Customers AS c;
```

#### Expected Output (sample)

| CustomerEmail | CustomerPhone |
|---|---|
| john.doe@smail.com | 413-456-6862 |
| jane.smith@inlook.com | 708-567-5234 |
| alice.j@jmail.com | 317-678-5717 |
| … | … |
| emma.w@service.org | 864-789-4731 |

---

## SELECT Features — Quick Reference

| Feature | Syntax | Purpose |
|---|---|---|
| **All columns** | `SELECT *` | Retrieve every column |
| **Specific columns** | `SELECT col1, col2` | Retrieve only named columns |
| **Table alias** | `FROM Table AS t` | Shorten table name in query |
| **Column alias** | `col AS NewName` | Rename column in output |
| **Calculated column** | `(col1 * col2) AS Total` | Derive new values on the fly |
| **Distinct values** | `SELECT DISTINCT col` | Remove duplicate rows |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Avoid `SELECT *` in production** | Retrieves unnecessary data, slows queries, complicates maintenance |
| **Name aliases clearly** | `p` for Products, `c` for Customers — not `x` or `tmp` |
| **Always alias calculated columns** | Raw expressions like `(Price * Stock)` are unreadable in reports |
| **Use `DISTINCT` sparingly** | It sorts through all rows and can slow down large queries |
| **Alias shared column names** | Prevents ambiguity when querying multiple tables |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Using `SELECT *` carelessly** pulls all columns — slow, wasteful, and may expose sensitive data.

> ⚠️ **Forgetting aliases on shared column names** (e.g., both `Customers` and `Suppliers` have `Email`) causes confusion in output.

> ⚠️ **No alias on calculated columns** — the output header becomes the raw expression, which is hard to read and reference.

> ⚠️ **Overusing `DISTINCT`** on large tables significantly increases query processing time.

> ⚠️ **Confusing aliases with actual renaming** — aliases only apply within that query; the real column/table names remain unchanged.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `SELECT *` | Retrieves all columns; convenient but use sparingly |
| `SELECT col1, col2` | Retrieves specific columns in your chosen order |
| Table aliases | Shorten table names for cleaner, more readable queries |
| Column aliases | Rename output columns for clarity or reporting |
| Calculated columns | Derive new values from existing columns on the fly |
| `DISTINCT` | Removes duplicate rows from the result set |

---

## What's Next?

Now that you can retrieve data, the next step is learning to **filter and sort** it!

- 🔍 **Filtering rows with `WHERE`**
- 📊 **Sorting results with `ORDER BY`**
- 🔢 **Limiting results with `LIMIT`**
- 🔗 **Combining tables with `JOIN`**

> *`SELECT` is your window into the database — learn to use it precisely and everything becomes clearer!* 🚀

---

