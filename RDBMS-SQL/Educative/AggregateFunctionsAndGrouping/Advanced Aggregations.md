# Advanced Aggregations
> *Learn about advanced aggregation in SQL.*

---

## Introduction

In today's dynamic retail environment, we often need to summarize data to get both detailed insights and an overall picture. Imagine generating a sales report that shows the total sales for each product **and** includes a grand total for all products combined — in a single query. This is where **advanced aggregations** shine.

### 🎯 Learning Goals

- ✅ Understand why advanced aggregation techniques like `WITH ROLLUP` are useful
- ✅ Understand how to perform advanced aggregations using `WITH ROLLUP`
- ✅ Perform grouping with automatic subtotals and grand totals

---

## What Is Advanced Aggregation?

**Advanced aggregation** refers to techniques in SQL that go beyond basic summaries (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) to generate **multiple layers of summary** — such as subtotals per group and a grand total across all groups.

### Basic vs. Advanced Aggregation

| Basic Aggregation | Advanced Aggregation |
|---|---|
| Total sales across all products | Total sales per product + grand total |
| Average price per category | Average per category + overall average |
| Count of orders per customer | Count per customer + total order count |
| Sum of items per order | Sum per order + sum per product + overall total |

Without advanced aggregation, generating both per-group totals and an overall total would require multiple separate queries. Techniques like `WITH ROLLUP` derive these multiple levels of insight **in a single query**.

---

## `WITH ROLLUP`

`WITH ROLLUP` is an extension to `GROUP BY` that automatically adds **subtotal** and **grand total rows** to the grouped results.

### How It Works

- Normal `GROUP BY` returns one summary row per group
- `WITH ROLLUP` returns those same rows **plus** additional rows at each grouping level
- The extra rows contain `NULL` in the grouped columns — these `NULL` values represent the **subtotal or grand total** for that level

### Syntax

```sql
SELECT Column1, Column2, AGGREGATE_FUNCTION(Column3)
FROM TableName
GROUP BY Column1, Column2 WITH ROLLUP;
```

> 💡 `WITH ROLLUP` is placed at the end of the `GROUP BY` clause, after all the column names.

---

## 1️⃣ Single-Level Rollup — Grand Total

### Example — Total Sales per Customer + Grand Total

```sql
SELECT CustomerID, SUM(TotalAmount) AS TotalSales
FROM Orders
GROUP BY CustomerID WITH ROLLUP;
```

#### Sample Output

| CustomerID | TotalSales |
|---|---|
| 1 | 235.50 |
| 2 | 2639.99 |
| 3 | 758.99 |
| 4 | 2550.50 |
| 5 | 1286.58 |
| **NULL** | **7471.56** |

> 💡 The row where `CustomerID = NULL` is the **grand total** — the sum of all sales across every customer. `NULL` here does not mean missing data; it is `WITH ROLLUP`'s way of marking a summary row.

---

### Example — Product Count per Category + Grand Total

```sql
SELECT CategoryID, COUNT(*) AS ProductCount
FROM Products
GROUP BY CategoryID WITH ROLLUP;
```

#### Sample Output

| CategoryID | ProductCount |
|---|---|
| 1 | 6 |
| 2 | 4 |
| 3 | 3 |
| 4 | 2 |
| 5 | 5 |
| **NULL** | **20** |

> The `NULL` row shows there are **20 products in total** across all categories.

---

## 2️⃣ Multi-Level Rollup — Subtotals and Grand Total

When `GROUP BY` has multiple columns, `WITH ROLLUP` generates a row for **each grouping level** — one subtotal row per first-level group, and one grand total row at the very end.

### Example — Quantity Sold per Product per Order + Subtotals + Grand Total

```sql
SELECT OrderID, ProductID, SUM(Quantity) AS TotalQuantitySold
FROM Order_Details
GROUP BY OrderID, ProductID WITH ROLLUP;
```

#### Sample Output (partial)

| OrderID | ProductID | TotalQuantitySold |
|---|---|---|
| 1 | 1 | 2 |
| 1 | 6 | 3 |
| **1** | **NULL** | **5** |
| 2 | 7 | 1 |
| 2 | 5 | 2 |
| **2** | **NULL** | **3** |
| 3 | 3 | 1 |
| 3 | 12 | 2 |
| **3** | **NULL** | **3** |
| **NULL** | **NULL** | **34** |

#### Reading the Output

| Row Type | `OrderID` | `ProductID` | Meaning |
|---|---|---|---|
| Detail row | 1 | 1 | Order 1 sold 2 units of Product 1 |
| Subtotal row | 1 | NULL | Order 1 total: 5 items across all products |
| Grand total row | NULL | NULL | All orders total: 34 items across everything |

> ✅ Multi-level `WITH ROLLUP` adds one subtotal per top-level group (per `OrderID`) plus one final grand total row — all in a single query.

---

## Understanding `NULL` in Rollup Rows

`NULL` values in `WITH ROLLUP` output have a special meaning — they are **not missing data**. They mark summary rows:

| `NULL` position | Represents |
|---|---|
| Only the last grouped column is `NULL` | **Subtotal** for the current higher-level group |
| All grouped columns are `NULL` | **Grand total** across the entire table |

### Labelling `NULL` Rows with `COALESCE()`

To make rollup output more readable in reports, replace `NULL` with a label using `COALESCE()`:

```sql
SELECT
  COALESCE(CAST(CustomerID AS CHAR), 'Grand Total') AS CustomerID,
  SUM(TotalAmount) AS TotalSales
FROM Orders
GROUP BY CustomerID WITH ROLLUP;
```

#### Output

| CustomerID | TotalSales |
|---|---|
| 1 | 235.50 |
| 2 | 2639.99 |
| 3 | 758.99 |
| 4 | 2550.50 |
| 5 | 1286.58 |
| **Grand Total** | **7471.56** |

> ✅ `COALESCE()` returns the first non-NULL value — so when `CustomerID` is `NULL` (a rollup row), it shows `'Grand Total'` instead.

---

## `WITH ROLLUP` vs. Manual Grand Total

### Without `WITH ROLLUP` — Two Separate Queries

```sql
-- Query 1: Sales per customer
SELECT CustomerID, SUM(TotalAmount) AS TotalSales
FROM Orders
GROUP BY CustomerID;

-- Query 2: Grand total (a separate query)
SELECT SUM(TotalAmount) AS GrandTotal
FROM Orders;
```

### With `WITH ROLLUP` — One Query Does Both

```sql
-- Single query handles per-group totals AND grand total
SELECT CustomerID, SUM(TotalAmount) AS TotalSales
FROM Orders
GROUP BY CustomerID WITH ROLLUP;
```

> ✅ `WITH ROLLUP` consolidates multiple summary levels into one efficient query — less code, same results.

---

## 🧩 Code Exercise

### Task — Total Quantity Sold per Product + Grand Total

**Write a query to calculate:**
1. Total quantity sold for each product
2. A grand total of all quantities sold

```sql
SELECT ProductID, SUM(Quantity) AS TotalQuantitySold
FROM Order_Details
GROUP BY ProductID WITH ROLLUP;
```

#### Expected Output

| ProductID | TotalQuantitySold |
|---|---|
| 1 | 2 |
| 3 | 1 |
| 4 | 2 |
| 5 | 6 |
| 6 | 4 |
| 7 | 1 |
| 9 | 1 |
| 10 | 1 |
| 12 | 2 |
| 13 | 2 |
| 14 | 3 |
| 15 | 3 |
| 17 | 1 |
| 18 | 3 |
| 19 | 2 |
| **NULL** | **34** |

> 💡 The final `NULL` row confirms that **34 total units** were sold across all products — the grand total automatically added by `WITH ROLLUP`.

---

## `WITH ROLLUP` Quick Reference

| Grouping Columns | Extra Rows Added by `WITH ROLLUP` |
|---|---|
| 1 column | 1 grand total row |
| 2 columns | 1 subtotal per group of col1 + 1 grand total |
| 3 columns | Subtotals at each level + 1 grand total |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use `COALESCE()` to label `NULL` rows** | Makes rollup output clear and readable in reports |
| **Always verify the grand total row** | Cross-check against a simple `SELECT SUM(col)` for accuracy |
| **Use `WITH ROLLUP` cautiously on large datasets** | Extra summary computations add overhead |
| **Include all grouping columns in `SELECT`** | Omitting them causes errors or ambiguous output |
| **Understand each `NULL` level in multi-column rollups** | `NULL` at different positions means subtotal vs. grand total |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Treating rollup `NULL` as missing data** — `NULL` in a `WITH ROLLUP` result marks a summary row, not an absent value. Always interpret it in context.

> ⚠️ **Forgetting to include grouping columns in `SELECT`** — if you group by `CategoryID` but don't include it in `SELECT`, the output won't show which group each row belongs to.

> ⚠️ **Not verifying grand totals** — always sanity-check the grand total row against `SELECT SUM(col) FROM table` to confirm accuracy.

> ⚠️ **Confusing subtotal and grand total rows in multi-level rollups** — in a two-column `GROUP BY`, a row with only the second column as `NULL` is a subtotal; a row with both columns as `NULL` is the grand total.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Advanced aggregation | Generates multiple layers of summary (subtotals + grand totals) |
| `WITH ROLLUP` | Extends `GROUP BY` to automatically add rollup summary rows |
| Single-level rollup | Adds one grand total row (`NULL` in the grouping column) |
| Multi-level rollup | Adds subtotals per group + one final grand total |
| `NULL` in rollup rows | Marks summary rows — not missing data |
| `COALESCE()` | Replaces rollup `NULL` markers with readable labels |

---

## What's Next?

Now that you can generate comprehensive multi-level summaries, it's time to combine data from multiple tables!

- 🔗 **Combining tables with `JOIN`**
- 📊 **Subqueries and nested `SELECT`**
- 🔎 **Window functions for advanced analytics**

> *`WITH ROLLUP` turns a grouped query into a full report — per-group totals, subtotals, and grand totals all in one go!* 🚀

---

