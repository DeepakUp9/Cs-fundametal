# Filtering Groups
> *Learn about filtering groups using HAVING.*

---

## Introduction

Imagine managing an online store and needing to identify which product categories have enough products to justify launching a special promotion. Or creating a sales report that only shows customer groups whose total spending exceeded $100. We already know how to group data with `GROUP BY` — but how do we **filter those groups**? That's exactly what the `HAVING` clause is for.

### 🎯 Learning Goals

- ✅ Use the `HAVING` clause to filter groups that don't meet certain criteria
- ✅ Understand why `WHERE` cannot filter aggregated values — and why `HAVING` is needed instead
- ✅ Recognize the difference between filtering individual rows (`WHERE`) and filtering grouped data (`HAVING`)

---

## Why Filter Groups?

When we group rows and apply aggregate functions, we often only care about **groups that meet a specific threshold** — for example:

- Categories with **more than 3 products**
- Customers who have spent **more than $500 total**
- Suppliers who provide **more than 5 products**
- Products with **average rating above 4**

Without the ability to filter groups, we'd have to retrieve all groups and manually discard the irrelevant ones. `HAVING` lets SQL do this for us efficiently.

---

## The `HAVING` Clause

`HAVING` filters results **after** `GROUP BY` is applied. It works on aggregated values — things that don't exist until after grouping is complete.

### Syntax

```sql
SELECT Column1, Column2, AGGREGATE_FUNCTION(Column3)
FROM TableName
GROUP BY Column1, Column2
HAVING condition;
```

### Order of Execution

| Step | Clause | What Happens |
|---|---|---|
| 1 | `FROM` | Identifies the source table |
| 2 | `WHERE` | Filters individual rows (before grouping) |
| 3 | `GROUP BY` | Groups the filtered rows |
| 4 | Aggregate | Applies aggregate functions to each group |
| 5 | `HAVING` | Filters the groups based on aggregate results |
| 6 | `SELECT` | Returns the specified columns |
| 7 | `ORDER BY` | Sorts the final results |

> 💡 `HAVING` happens **after** grouping — that's why it can work with aggregate values. `WHERE` happens **before** grouping — which is why it **cannot** filter aggregate values.

---

## 1️⃣ Basic `HAVING` Example

### Example — Categories with More Than 3 Products

```sql
SELECT CategoryID, COUNT(*) AS ProductCount
FROM Products
GROUP BY CategoryID
HAVING ProductCount > 3;
```

#### How It Works

1. Products are grouped by `CategoryID`
2. `COUNT(*)` counts the number of products in each group
3. `HAVING ProductCount > 3` keeps only groups where the count exceeds 3

#### Sample Output

| CategoryID | ProductCount |
|---|---|
| 1 | 6 |
| 2 | 4 |

> 💡 Categories with 3 or fewer products are **silently excluded** from the result — `HAVING` filters them out after the group aggregation is done.

---

### Example — Categories with Total Stock Value Above $10,000

```sql
SELECT CategoryID, SUM(Price * Stock) AS TotalStockValue
FROM Products
GROUP BY CategoryID
HAVING SUM(Price * Stock) > 10000;
```

#### Sample Output

| CategoryID | TotalStockValue |
|---|---|
| 1 | 65339.50 |
| 3 | 16237.40 |

---

### Example — Customers Who Spent More Than $500

```sql
SELECT CustomerID, SUM(TotalAmount) AS TotalSpent
FROM Orders
GROUP BY CustomerID
HAVING SUM(TotalAmount) > 500;
```

#### Sample Output

| CustomerID | TotalSpent |
|---|---|
| 2 | 2639.99 |
| 4 | 2550.50 |
| 5 | 1286.58 |
| 3 | 758.99 |

---

## 2️⃣ Combining `WHERE` and `HAVING`

`WHERE` and `HAVING` can be used together in the same query. `WHERE` reduces the rows before grouping; `HAVING` then filters the resulting groups.

### Example — Expensive Products Only, Then Filter Groups

```sql
-- Step 1 (WHERE): Only consider products priced above $15
-- Step 2 (GROUP BY): Group remaining products by category
-- Step 3 (HAVING): Keep only categories with average price above $100

SELECT CategoryID,
       COUNT(*) AS ProductCount,
       AVG(Price) AS AveragePrice
FROM Products
WHERE Price > 15
GROUP BY CategoryID
HAVING AVG(Price) > 100;
```

#### Sample Output

| CategoryID | ProductCount | AveragePrice |
|---|---|---|
| 1 | 5 | 601.57 |
| 3 | 2 | 791.25 |

> ✅ `WHERE Price > 15` eliminates cheap products before grouping. `HAVING AVG(Price) > 100` then removes any remaining category groups with a low average price.

---

### Example — In-Stock Products, High-Value Categories

```sql
SELECT CategoryID, SUM(Stock) AS TotalStock
FROM Products
WHERE Stock > 0
GROUP BY CategoryID
HAVING SUM(Stock) > 50
ORDER BY TotalStock DESC;
```

---

## 3️⃣ `WHERE` vs. `HAVING` — Full Comparison

| Aspect | `WHERE` | `HAVING` |
|---|---|---|
| **Purpose** | Filters individual rows | Filters entire groups |
| **When it runs** | Before `GROUP BY` | After `GROUP BY` and aggregation |
| **Works on** | Raw column values | Aggregated results (`SUM`, `COUNT`, etc.) |
| **Used with** | Any `SELECT` statement | Requires `GROUP BY` (almost always) |
| **Can use aggregates?** | ❌ No — causes an error | ✅ Yes — designed for this |
| **Example** | `WHERE Price > 50` | `HAVING AVG(Price) > 50` |

### Why You Can't Use Aggregate Functions in `WHERE`

```sql
-- ❌ This throws an error
SELECT CategoryID, AVG(Price) AS AvgPrice
FROM Products
WHERE AVG(Price) > 100       -- ERROR: aggregate not allowed in WHERE
GROUP BY CategoryID;

-- ✅ Correct: use HAVING for aggregate conditions
SELECT CategoryID, AVG(Price) AS AvgPrice
FROM Products
GROUP BY CategoryID
HAVING AVG(Price) > 100;
```

> The reason is simple: at the time `WHERE` runs, the data hasn't been grouped yet — there are no "average prices per category" to filter on.

---

## Visualising the Execution Flow

```
Full Products Table
        │
        ▼
┌─────────────┐
│   WHERE     │  Filter rows: e.g. WHERE Price > 15
│  (row-level)│  Removes individual rows before grouping
└─────────────┘
        │
        ▼
┌─────────────┐
│  GROUP BY   │  Group remaining rows by CategoryID
│             │  Forms groups: {Cat 1: 5 rows}, {Cat 2: 3 rows}, ...
└─────────────┘
        │
        ▼
┌─────────────┐
│  Aggregate  │  Apply SUM, COUNT, AVG, etc. within each group
│  Functions  │  One summary value per group
└─────────────┘
        │
        ▼
┌─────────────┐
│   HAVING    │  Filter groups: e.g. HAVING AVG(Price) > 100
│(group-level)│  Removes entire groups that don't meet the condition
└─────────────┘
        │
        ▼
┌─────────────┐
│  ORDER BY   │  Sort the final result
└─────────────┘
        │
        ▼
   Final Result
```

---

## 🧩 Code Exercise

### Task — Categories with Total Stock Exceeding 100

**Find categories (by `CategoryID`) where the total stock of all products exceeds 100.**

```sql
SELECT CategoryID, SUM(Stock) AS TotalStock
FROM Products
GROUP BY CategoryID
HAVING SUM(Stock) > 100;
```

#### Expected Output

| CategoryID | TotalStock |
|---|---|
| 1 | 250 |
| 2 | 125 |
| 3 | 160 |
| 5 | 385 |

> 💡 Categories 4 has a total stock of 100 or less, so it doesn't appear. `HAVING` removes it after summing all stocks per category.

---

## Full Query Template — `WHERE` + `GROUP BY` + `HAVING` + `ORDER BY`

```sql
SELECT   Column1, AGGREGATE_FUNCTION(Column2) AS AliasName
FROM     TableName
WHERE    row_level_condition          -- Filter rows before grouping
GROUP BY Column1                      -- Group the filtered rows
HAVING   group_level_condition        -- Filter groups after aggregation
ORDER BY AliasName DESC;              -- Sort the final results
```

### Real-World Example

```sql
-- Find Electronics and Fitness categories where average price > $200
-- Sort by highest average price first

SELECT CategoryID,
       COUNT(*) AS ProductCount,
       AVG(Price) AS AveragePrice
FROM Products
WHERE CategoryID IN (1, 3)       -- Only Electronics and Fitness
GROUP BY CategoryID
HAVING AVG(Price) > 200          -- Only categories with avg price > $200
ORDER BY AveragePrice DESC;
```

---

## `HAVING` Quick Reference

| Goal | Syntax |
|---|---|
| Groups with count > n | `HAVING COUNT(*) > n` |
| Groups with total > amount | `HAVING SUM(col) > amount` |
| Groups with average in range | `HAVING AVG(col) BETWEEN low AND high` |
| Groups where min > threshold | `HAVING MIN(col) > threshold` |
| Groups where max < threshold | `HAVING MAX(col) < threshold` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use `WHERE` to filter rows before grouping** | Reduces dataset size before aggregation — better performance |
| **Use `HAVING` only for aggregate conditions** | Conditions on non-aggregated columns belong in `WHERE` |
| **Keep `HAVING` conditions simple and clear** | Complex conditions are harder to debug and maintain |
| **Always pair `HAVING` with `GROUP BY`** | `HAVING` without `GROUP BY` is technically valid but rarely meaningful |
| **Use aliases in `HAVING` when supported** | `HAVING ProductCount > 3` is more readable than `HAVING COUNT(*) > 3` |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Using aggregate functions in `WHERE`** — this causes a syntax error. Aggregates can only appear in `HAVING`, `SELECT`, or `ORDER BY`.

> ⚠️ **Using `HAVING` for non-aggregate conditions** — `HAVING CategoryID = 1` works but is inefficient; `WHERE CategoryID = 1` is the correct and faster approach.

> ⚠️ **Mixing up `WHERE` and `HAVING`** — `WHERE` is row-level (before grouping), `HAVING` is group-level (after aggregation).

> ⚠️ **Forgetting that `HAVING` filters groups, not rows** — if a group doesn't pass the `HAVING` condition, the entire group disappears from results, not just some rows.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `HAVING` | Filters grouped data based on aggregate conditions |
| `WHERE` vs `HAVING` | `WHERE` = row-level (before grouping), `HAVING` = group-level (after grouping) |
| Why not `WHERE`? | Aggregate values don't exist at `WHERE` stage — grouping hasn't happened yet |
| Combined usage | `WHERE` + `GROUP BY` + `HAVING` + `ORDER BY` for full precision |
| Execution order | `WHERE` → `GROUP BY` → Aggregate → `HAVING` → `ORDER BY` |

---

## What's Next?

Now that you can group and filter grouped data, explore how to combine data from multiple tables!

- 🔗 **Combining tables with `JOIN`**
- 📊 **Subqueries and nested SELECT**
- 🔎 **Advanced aggregations across joined tables**

> *`WHERE` cleans the rows, `GROUP BY` organizes them, and `HAVING` refines the groups — together they make your analysis surgical!* 🚀

---

