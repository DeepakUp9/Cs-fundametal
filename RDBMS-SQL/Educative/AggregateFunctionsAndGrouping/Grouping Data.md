# Grouping Data
> *Learn about grouping the data to get insightful results.*

---

## Introduction

Suppose we want to analyze the performance of our online store — like how many orders have been placed or which category has seen the highest sales. Aggregate functions like `COUNT()` can give us an overall row count, but to understand progress more deeply, we need to break the data down — for example, how many orders **each customer** has placed. This is where the `GROUP BY` clause becomes essential.

### 🎯 Learning Goals

- ✅ Understand what it means to group data
- ✅ Learn why grouping is essential for summarizing and analyzing data
- ✅ Explore how to use the `GROUP BY` clause to obtain aggregated results

---

## Why Do We Group Data?

Aggregate functions like `SUM()`, `COUNT()`, `AVG()`, `MIN()`, and `MAX()` provide a **bird's-eye view** of a dataset — for example, total revenue for the entire store. But what if we need to understand the factors driving that total? Grouping breaks data into smaller, meaningful subsets and applies aggregate functions **within each group**, helping us uncover the "why" behind patterns.

### Aggregate Functions vs. Grouping + Aggregate Functions

| Aggregate Function Question | Grouping + Aggregate Question |
|---|---|
| How many items were sold in total? | How many items were sold **in each category**? |
| What was the highest sale amount? | What was the highest sale amount **for each product**? |
| How many unique customers are there? | How many customers are there **in each region**? |
| How many total products were supplied? | How many products were supplied **by each vendor**? |
| How many total orders were placed? | How many orders were placed **by each customer per day**? |

> 💡 The left column gives one number for the whole table. The right column gives one number **per group** — far more insightful for analysis.

---

## The `GROUP BY` Statement

`GROUP BY` combines rows that share the same value(s) in specified columns into **summary rows**. It is almost always used with aggregate functions to perform calculations on each group rather than the entire dataset.

### Syntax

```sql
SELECT Column1, Column2, AGGREGATE_FUNCTION(Column3)
FROM TableName
GROUP BY Column1, Column2;
```

### Order of Execution

1. Rows are grouped by the specified column(s)
2. Aggregate function(s) are applied to each group
3. One result row is returned per group

> ⚠️ Every column in `SELECT` must either be in the `GROUP BY` clause or wrapped in an aggregate function — otherwise SQL will throw an error.

---

## 1️⃣ Basic `GROUP BY` Example

### Example — Total Stock Value per Category

```sql
SELECT CategoryID, SUM(Price * Stock) AS TotalStockValue
FROM Products
GROUP BY CategoryID;
```

#### How It Works

1. Products are grouped by `CategoryID` (e.g., all Electronics together, all Kitchenware together)
2. For each group, `Price * Stock` is calculated for every product in the group
3. `SUM()` adds those values together within the group
4. One result row is returned per category

#### Sample Output

| CategoryID | TotalStockValue |
|---|---|
| 1 | 65339.50 |
| 2 | 9899.70 |
| 3 | 16237.40 |
| 4 | 1890.50 |
| 5 | 13600.00 |

---

### Example — Number of Products per Category

```sql
SELECT CategoryID, COUNT(*) AS ProductCount
FROM Products
GROUP BY CategoryID;
```

#### Sample Output

| CategoryID | ProductCount |
|---|---|
| 1 | 6 |
| 2 | 4 |
| 3 | 3 |
| 4 | 2 |
| 5 | 3 |

---

### Example — Average Price per Category

```sql
SELECT CategoryID,
       AVG(Price) AS AveragePrice,
       MIN(Price) AS CheapestProduct,
       MAX(Price) AS MostExpensiveProduct
FROM Products
GROUP BY CategoryID;
```

#### Sample Output

| CategoryID | AveragePrice | CheapestProduct | MostExpensiveProduct |
|---|---|---|---|
| 1 | 501.71 | 135.50 | 1099.99 |
| 2 | 90.12 | 50.00 | 129.99 |
| 3 | 791.25 | 82.50 | 1499.99 |

---

## 2️⃣ `GROUP BY` with `WHERE` and `ORDER BY`

`GROUP BY` can be combined with `WHERE` to filter rows before grouping, and `ORDER BY` to sort the grouped results.

### Full Syntax

```sql
SELECT Column1, Column2, AGGREGATE_FUNCTION(Column3)
FROM TableName
WHERE Condition
GROUP BY Column1, Column2
ORDER BY Column1, Column2;
```

### Order of Execution

| Step | Clause | What Happens |
|---|---|---|
| 1 | `WHERE` | Filters rows based on the condition |
| 2 | `GROUP BY` | Groups the filtered rows |
| 3 | Aggregate | Applies aggregate functions to each group |
| 4 | `ORDER BY` | Sorts the grouped results |

---

### Example — Total Stock per Category (In-Stock Only, Sorted)

```sql
SELECT CategoryID, SUM(Stock) AS TotalStock
FROM Products
WHERE Stock > 0
GROUP BY CategoryID
ORDER BY CategoryID DESC;
```

#### How It Works

1. `WHERE Stock > 0` filters out any products with zero stock
2. `GROUP BY CategoryID` groups the remaining products by category
3. `SUM(Stock)` calculates total stock within each category
4. `ORDER BY CategoryID DESC` sorts results from highest to lowest `CategoryID`

#### Sample Output

| CategoryID | TotalStock |
|---|---|
| 5 | 153 |
| 4 | 25 |
| 3 | 25 |
| 2 | 125 |
| 1 | 230 |

---

### Example — Total Revenue per Customer (Minimum $100)

```sql
SELECT CustomerID, SUM(TotalAmount) AS TotalSpent
FROM Orders
WHERE TotalAmount >= 100
GROUP BY CustomerID
ORDER BY TotalSpent DESC;
```

---

### Example — Count of Orders per Customer per Month

```sql
SELECT CustomerID,
       MONTH(OrderDate) AS OrderMonth,
       COUNT(*) AS OrderCount
FROM Orders
GROUP BY CustomerID, MONTH(OrderDate)
ORDER BY CustomerID, OrderMonth;
```

> 💡 You can group by multiple columns — here, grouping by both `CustomerID` and `OrderMonth` gives one row per **customer per month**.

---

## How `GROUP BY` Changes the Result

Let's compare queries without and with `GROUP BY`:

### Without `GROUP BY` — Single Summary Row

```sql
-- Returns ONE row for the entire table
SELECT COUNT(*) AS TotalOrders, SUM(TotalAmount) AS TotalRevenue
FROM Orders;
```

| TotalOrders | TotalRevenue |
|---|---|
| 10 | 9470.55 |

### With `GROUP BY` — One Row per Group

```sql
-- Returns ONE row per customer
SELECT CustomerID, COUNT(*) AS OrderCount, SUM(TotalAmount) AS TotalSpent
FROM Orders
GROUP BY CustomerID;
```

| CustomerID | OrderCount | TotalSpent |
|---|---|---|
| 1 | 1 | 235.50 |
| 2 | 1 | 2639.99 |
| 3 | 1 | 758.99 |
| 4 | 1 | 2550.50 |
| 5 | 1 | 1286.58 |

> ✅ `GROUP BY` transforms a single-row summary into a per-group breakdown — this is the key difference.

---

## 🧩 Code Exercise

### Task — Orders Placed by Each Customer

**Retrieve the count of orders placed by each customer. Return `CustomerID` and the count as `OrderCount` from the `Orders` table.**

```sql
SELECT CustomerID, COUNT(OrderID) AS OrderCount
FROM Orders
GROUP BY CustomerID;
```

#### Expected Output (sample)

| CustomerID | OrderCount |
|---|---|
| 1 | 1 |
| 2 | 1 |
| 3 | 1 |
| … | … |
| 8 | 1 |
| 9 | 1 |
| 10 | 1 |

> 💡 Each `CustomerID` is a separate group. `COUNT(OrderID)` counts the number of order rows within each customer's group.

---

## `GROUP BY` Rules — What Goes in `SELECT`?

| Column Type | In `SELECT`? | In `GROUP BY`? | Example |
|---|---|---|---|
| Grouping column | ✅ Yes | ✅ Yes | `CategoryID` |
| Aggregate result | ✅ Yes | ❌ No | `SUM(Price)`, `COUNT(*)` |
| Regular (non-grouped) column | ❌ Error | ❌ — | `ProductName` without grouping |

```sql
-- ✅ Correct: CategoryID is in both SELECT and GROUP BY
SELECT CategoryID, SUM(Price) AS TotalPrice
FROM Products
GROUP BY CategoryID;

-- ❌ Error: ProductName is in SELECT but not in GROUP BY or an aggregate
SELECT ProductName, SUM(Price) AS TotalPrice
FROM Products
GROUP BY CategoryID;
```

---

## Quick Reference — Clause Order

```sql
SELECT   col, AGGREGATE(col)   -- What to show
FROM     TableName              -- Where to get it
WHERE    condition              -- Filter rows (before grouping)
GROUP BY col                    -- Group the filtered rows
ORDER BY col;                   -- Sort the grouped results
```

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Only include grouped or aggregated columns in `SELECT`** | Mixing non-grouped columns causes errors or ambiguous results |
| **Use meaningful aliases for aggregates** | `TotalStock` is clearer than `SUM(Stock)` in the output header |
| **Filter with `WHERE` before grouping when possible** | Reduces the number of rows being grouped — better performance |
| **Keep `GROUP BY` columns logically consistent** | Group by columns that make real-world sense together |
| **Use `ORDER BY` to make grouped results readable** | Sorted output is much easier to analyze and present |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting to include non-aggregate columns in `GROUP BY`** — every column in `SELECT` that isn't inside an aggregate function must appear in `GROUP BY`, or SQL will throw an error.

> ⚠️ **Using `GROUP BY` without any aggregate function** — while technically valid in some cases, it usually signals a misunderstanding of what grouping is for. `GROUP BY` is most valuable when paired with aggregates.

> ⚠️ **Filtering groups with `WHERE` instead of `HAVING`** — `WHERE` filters individual rows before grouping. To filter the groups themselves (e.g., "only categories with more than 3 products"), use `HAVING` — covered in the next lesson.

> ⚠️ **Grouping by too many columns** — over-grouping creates too many tiny groups, making the summary less useful. Group only by columns that are meaningfully distinct.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `GROUP BY` | Groups rows with the same value(s) into summary rows |
| With aggregate functions | Calculates `SUM`, `COUNT`, `AVG`, `MIN`, `MAX` within each group |
| With `WHERE` | Filters rows before they are grouped |
| With `ORDER BY` | Sorts the final grouped results |
| Execution order | `WHERE` → `GROUP BY` → Aggregate → `ORDER BY` |
| SELECT rule | Columns must be in `GROUP BY` or wrapped in an aggregate |

---

## What's Next?

Now that you can group data, learn how to filter those groups!

- 🔎 **Filtering grouped results with `HAVING`**
- 🔗 **Using `GROUP BY` with `JOIN` across multiple tables**
- 📊 **Advanced aggregations and subqueries**

> *`GROUP BY` transforms raw data into meaningful categories — it's the foundation of almost every real-world SQL report!* 🚀

---

