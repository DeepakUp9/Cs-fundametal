# Introduction to Aggregate Functions
> *Learn about aggregate functions in SQL.*

---

## Introduction

Suppose our online store is doing great and receives hundreds of orders every day. As the year ends, we want to answer questions to identify sales trends:

- 💰 What is the total revenue generated this month?
- 📦 How many orders have been placed in total?
- 📊 What is the average price of products sold?

Instead of doing these calculations manually, **aggregate functions** allow us to get the answers in a straightforward and efficient way.

### 🎯 Learning Goals

- ✅ Understand what aggregate functions are and why they are useful
- ✅ Learn how to use `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX` to analyze data
- ✅ Learn how to combine aggregate functions in a single query
- ✅ Understand the importance of using aggregate functions on appropriate data types

---

## What Are Aggregate Functions?

Aggregate functions are special SQL functions that **operate on a group of rows and return a single summarizing value**. They allow us to quickly calculate totals, averages, counts, and extremes across an entire table or filtered subset.

Aggregate functions work seamlessly with `SELECT` and can be refined with `WHERE` for targeted analysis.

### General Syntax

```sql
SELECT AGGREGATE_FUNCTION(Column)
FROM TableName;
```

### All Aggregate Functions at a Glance

| Function | Returns | Works On |
|---|---|---|
| `COUNT()` | Number of rows | Any data type |
| `SUM()` | Total of all values | Numeric only |
| `AVG()` | Arithmetic mean | Numeric only |
| `MIN()` | Smallest value | Numbers, text, dates |
| `MAX()` | Largest value | Numbers, text, dates |

---

## 1️⃣ `COUNT()` — Count Rows

The `COUNT()` function returns the **number of rows** in a result set.

- `COUNT(*)` — counts **all rows**, including those with `NULL` values
- `COUNT(column)` — counts only rows where the column value is **not NULL**

### Example — Total Number of Products

```sql
SELECT COUNT(*) AS TotalProducts
FROM Products;
```

#### Sample Output

| TotalProducts |
|---|
| 20 |

> 💡 Always use `AS` to give aggregate results a meaningful name — without it, the column header shows the raw expression like `COUNT(*)`.

### Example — Count Products with a Price Listed

```sql
-- Only counts rows where Price is not NULL
SELECT COUNT(Price) AS ProductsWithPrice
FROM Products;
```

### Example — Count Orders After a Specific Date

```sql
SELECT COUNT(*) AS OrdersAfterJuly
FROM Orders
WHERE OrderDate > '2024-07-01';
```

---

## 2️⃣ `SUM()` — Total of Values

The `SUM()` function **adds up all values** in a numeric column. Useful for calculating total revenue, total stock value, and so on.

### Example — Total of All Product Prices

```sql
SELECT SUM(Price) AS TotalPrice
FROM Products;
```

#### Sample Output

| TotalPrice |
|---|
| 4283.48 |

### Example — Total Value of All Stock

```sql
-- Total inventory value across all products
SELECT SUM(Price * Stock) AS TotalInventoryValue
FROM Products;
```

### Example — Total Revenue from Filtered Orders

```sql
SELECT SUM(TotalAmount) AS RevenueAfterJuly
FROM Orders
WHERE OrderDate > '2024-07-01';
```

---

## 3️⃣ `AVG()` — Average Value

The `AVG()` function computes the **arithmetic mean** of values in a numeric column. It automatically ignores `NULL` values.

### Example — Average Product Price

```sql
SELECT AVG(Price) AS AveragePrice
FROM Products;
```

#### Sample Output

| AveragePrice |
|---|
| 428.349000 |

### Example — Average Order Value

```sql
SELECT AVG(TotalAmount) AS AverageOrderValue
FROM Orders;
```

### Example — Average Price of Electronics Only

```sql
SELECT AVG(Price) AS AvgElectronicsPrice
FROM Products
WHERE CategoryID = 1;
```

> 💡 `AVG()` ignores `NULL` values — it calculates the mean only from rows with an actual value in the column.

---

## 4️⃣ `MIN()` — Smallest Value

The `MIN()` function finds the **lowest value** in a column. Works on numbers, text (alphabetical), and dates (earliest).

### Example — Cheapest Product

```sql
SELECT MIN(Price) AS 'Cheapest Product'
FROM Products;
```

#### Sample Output

| Cheapest Product |
|---|
| 5.00 |

### Example — Earliest Order Date

```sql
SELECT MIN(OrderDate) AS FirstOrderDate
FROM Orders;
```

### Example — Alphabetically First Product Name

```sql
SELECT MIN(ProductName) AS FirstAlphabetically
FROM Products;
```

---

## 5️⃣ `MAX()` — Largest Value

The `MAX()` function finds the **highest value** in a column. Works on numbers, text (reverse alphabetical), and dates (most recent).

### Example — Most Expensive Product

```sql
SELECT MAX(Price) AS 'Most Expensive Product'
FROM Products;
```

#### Sample Output

| Most Expensive Product |
|---|
| 1499.99 |

### Example — Most Recent Order

```sql
SELECT MAX(OrderDate) AS MostRecentOrder
FROM Orders;
```

### Example — Highest Stock Level

```sql
SELECT MAX(Stock) AS HighestStock
FROM Products;
```

---

## 6️⃣ Combining Multiple Aggregate Functions

Multiple aggregate functions can be used in a single query to get a comprehensive summary of data in one step — instead of writing and running multiple separate queries.

### Example — Full Product Summary

```sql
SELECT COUNT(*) AS TotalProducts,
       AVG(Price) AS AveragePrice,
       MAX(Stock) AS HighestStock
FROM Products;
```

#### Sample Output

| TotalProducts | AveragePrice | HighestStock |
|---|---|---|
| 20 | 428.349000 | 150 |

### Example — Comprehensive Price Analysis

```sql
SELECT COUNT(*) AS TotalProducts,
       MIN(Price) AS CheapestPrice,
       MAX(Price) AS MostExpensivePrice,
       AVG(Price) AS AveragePrice,
       SUM(Price) AS TotalPriceSum
FROM Products;
```

#### Sample Output

| TotalProducts | CheapestPrice | MostExpensivePrice | AveragePrice | TotalPriceSum |
|---|---|---|---|---|
| 20 | 5.00 | 1499.99 | 428.349000 | 4283.48 |

### Example — Order Revenue Summary

```sql
SELECT COUNT(*) AS TotalOrders,
       SUM(TotalAmount) AS TotalRevenue,
       AVG(TotalAmount) AS AverageOrderValue,
       MIN(TotalAmount) AS SmallestOrder,
       MAX(TotalAmount) AS LargestOrder
FROM Orders;
```

---

## ⚠️ Mixing Aggregate Functions with Regular Columns

You **cannot** mix aggregate functions with regular (non-aggregated) columns in the same `SELECT` without a `GROUP BY` clause:

```sql
-- ❌ This will throw an error
SELECT Price, COUNT(*) AS TotalProducts
FROM Products;
```

This fails because `COUNT(*)` returns one summarized value for the whole table, while `Price` represents individual rows — SQL cannot reconcile these without grouping logic.

```sql
-- ✅ Correct — use GROUP BY to group by the regular column
SELECT Price, COUNT(*) AS TotalProducts
FROM Products
GROUP BY Price;
```

> 💡 We will cover `GROUP BY` in detail in the next lesson. For now, remember: **aggregate functions alone = fine; aggregate + regular column = requires `GROUP BY`**.

---

## Using Aggregate Functions on Appropriate Data Types

| Function | Numeric | Text | Date | Notes |
|---|---|---|---|---|
| `COUNT()` | ✅ | ✅ | ✅ | Counts non-NULL values for a column; all rows for `*` |
| `SUM()` | ✅ | ⚠️ | ❌ | Numeric strings (e.g., `'100'`) auto-convert; non-numeric strings treated as 0 |
| `AVG()` | ✅ | ⚠️ | ❌ | Same as `SUM()` — non-numeric text becomes 0 |
| `MIN()` | ✅ | ✅ | ✅ | Text: alphabetical first; Date: earliest |
| `MAX()` | ✅ | ✅ | ✅ | Text: alphabetical last; Date: most recent |

### MySQL's Behaviour for `SUM()` / `AVG()` on Non-Numeric Data

```sql
-- Numeric strings: MySQL auto-converts
-- '100' + '200' → 300 ✅

-- Non-numeric strings: treated as 0
-- 'abc' + 'xyz' → 0 ⚠️ (no error, but incorrect result)

-- Always use numeric columns for SUM() and AVG()
-- to ensure accurate results
```

---

## NULL Values and Aggregate Functions

| Function | Behaviour with NULL |
|---|---|
| `COUNT(*)` | Counts all rows — **includes** NULLs |
| `COUNT(col)` | Counts only non-NULL values in the column |
| `SUM(col)` | Ignores NULLs — adds up only non-NULL values |
| `AVG(col)` | Ignores NULLs — calculates mean of non-NULL values only |
| `MIN(col)` | Ignores NULLs |
| `MAX(col)` | Ignores NULLs |

```sql
-- Example: if 3 out of 10 rows have NULL in TotalAmount
COUNT(*)           → 10  (counts all rows)
COUNT(TotalAmount) → 7   (counts only non-NULL)
AVG(TotalAmount)   → average of the 7 non-NULL values
```

---

## 🧩 Code Exercises

---

### Task 1 — Total Revenue

**Find the total amount spent on all orders from the `Orders` table.**

```sql
SELECT SUM(TotalAmount) AS TotalSales
FROM Orders;
```

#### Expected Output

| TotalSales |
|---|
| 673.43 |

---

### Task 2 — Combined Order Summary

**Find the total number of orders, total revenue, and average order value from the `Orders` table.**

```sql
SELECT COUNT(*) AS TotalOrders,
       SUM(TotalAmount) AS TotalRevenue,
       AVG(TotalAmount) AS AverageOrderValue
FROM Orders;
```

#### Expected Output

| TotalOrders | TotalRevenue | AverageOrderValue |
|---|---|---|
| 10 | 673.43 | 67.343000 |

> 💡 Three different metrics — count, total, and average — all retrieved in a single efficient query.

---

## Aggregate Functions — Quick Reference

| Function | Syntax | Example | Returns |
|---|---|---|---|
| Count all rows | `COUNT(*)` | `SELECT COUNT(*) FROM Products` | Total row count |
| Count non-null | `COUNT(col)` | `SELECT COUNT(Price) FROM Products` | Non-null values |
| Total | `SUM(col)` | `SELECT SUM(TotalAmount) FROM Orders` | Sum of all values |
| Average | `AVG(col)` | `SELECT AVG(Price) FROM Products` | Arithmetic mean |
| Minimum | `MIN(col)` | `SELECT MIN(Price) FROM Products` | Smallest value |
| Maximum | `MAX(col)` | `SELECT MAX(Price) FROM Products` | Largest value |
| Combined | Multiple functions | `SELECT COUNT(*), AVG(Price), MAX(Stock)` | All in one row |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always alias aggregate results with `AS`** | Raw expressions like `COUNT(*)` are hard to read in output |
| **Use the right data type** | `SUM()` and `AVG()` should only be used on numeric columns |
| **Check for NULL values** | `COUNT(col)` vs `COUNT(*)` behave differently when NULLs exist |
| **Combine functions in one query** | More efficient than running multiple separate queries |
| **Don't mix aggregates and regular columns without `GROUP BY`** | Causes errors — always group when combining both |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Calling an aggregate function without a column or `*`** — `COUNT()` with nothing inside is a syntax error.

> ⚠️ **Mixing aggregate functions with regular columns without `GROUP BY`** — causes an error; use `GROUP BY` to group by the regular column.

> ⚠️ **Using `SUM()` or `AVG()` on text columns** — produces 0 for non-numeric strings in MySQL, which looks like a valid result but is incorrect.

> ⚠️ **Forgetting that `COUNT(col)` ignores NULLs** — if you need to count all rows, always use `COUNT(*)`.

> ⚠️ **Not aliasing aggregate columns** — the raw function name (`AVG(Price)`) is displayed as the column header, making output hard to read.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Aggregate functions | Operate on multiple rows and return a single summary value |
| `COUNT()` | Counts rows — `COUNT(*)` includes NULLs, `COUNT(col)` excludes them |
| `SUM()` | Totals numeric values — ignores NULLs |
| `AVG()` | Calculates mean of numeric values — ignores NULLs |
| `MIN()` / `MAX()` | Finds smallest/largest value — works on numbers, text, and dates |
| Combining functions | Multiple aggregates in one `SELECT` for efficient summaries |
| Mixing with columns | Requires `GROUP BY` — covered in the next lesson |

---

## What's Next?

Now that you can summarize entire tables, it's time to summarize **groups** within tables!

- 📊 **Grouping data with `GROUP BY`**
- 🔎 **Filtering groups with `HAVING`**
- 🔗 **Using aggregates across joined tables**

> *Aggregate functions turn thousands of rows into a single insight — master them and your data analysis becomes effortless!* 🚀

---

