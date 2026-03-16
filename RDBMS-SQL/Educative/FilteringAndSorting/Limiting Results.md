# Limiting Results
> *Learn to apply limits on results and use pagination.*

---

## Introduction

Imagine managing an online store with thousands of products and needing to display only the top 10 best-sellers on the homepage. Fetching all products and manually filtering them is inefficient and time-consuming. By focusing on just a subset of data, we can improve performance, make data easier to browse, and create a smoother user experience. This is where `LIMIT` and `OFFSET` come in — they let us retrieve exactly the data we need without overloading the database or the user interface.

### 🎯 Learning Goals

- ✅ Use the `LIMIT` clause to restrict the number of rows returned by a query
- ✅ Use the `OFFSET` clause to skip a specific number of rows
- ✅ Combine `LIMIT` and `OFFSET` for paginating results in a user-friendly manner

---

## The `LIMIT` Clause

The `LIMIT` clause restricts how many rows a query returns. This makes result sets more manageable and improves query performance — especially with large datasets.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName
LIMIT n;
```

`n` = the number of rows to return.

### Example — First 5 Products

```sql
SELECT *
FROM Products
LIMIT 5;
```

#### Sample Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 2 | Dumbbell Set | 3 | 82.50 | 15 |
| 3 | Laptop | 1 | 1099.99 | 30 |
| 4 | Smart Outdoor Light | 4 | 75.62 | 25 |
| 5 | Knife Set | 2 | 50.00 | 25 |

> ⚠️ Without `ORDER BY`, `LIMIT` returns the first `n` rows in whatever order the database chooses — this can vary between runs. Always combine with `ORDER BY` for predictable results.

---

## Using `LIMIT` with `ORDER BY`

Combining `ORDER BY` with `LIMIT` ensures you're not just controlling *how many* rows to display, but *which* rows are shown first. Sort first, then limit.

### Clause Order

```sql
SELECT ...
FROM ...
WHERE ...       -- (optional) filter
ORDER BY ...    -- sort
LIMIT n;        -- then limit
```

### Example — 5 Most Affordable Products

```sql
SELECT ProductName, Price
FROM Products
ORDER BY Price ASC
LIMIT 5;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Ballpoint Pen Set | 5.00 |
| Knife Set | 50.00 |
| Smart Outdoor Light | 75.62 |
| Dumbbell Set | 82.50 |
| Blender | 90.25 |

### Example — Most Expensive Product

```sql
-- Get the single most expensive product
SELECT ProductName, Price
FROM Products
ORDER BY Price DESC
LIMIT 1;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Treadmill | 1499.99 |

### Example — Top 3 Most Recent Orders

```sql
SELECT OrderID, CustomerID, OrderDate, TotalAmount
FROM Orders
ORDER BY OrderDate DESC
LIMIT 3;
```

---

## The `OFFSET` Clause

The `OFFSET` clause works alongside `LIMIT`. While `LIMIT` controls how many rows to show, `OFFSET` controls **where to start** — by skipping a given number of rows. This is the foundation of **pagination**.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName
ORDER BY ColumnName
LIMIT n OFFSET x;
```

| Part | Description |
|---|---|
| `n` | Number of rows to return |
| `x` | Number of rows to skip before returning results |

> ⚠️ `OFFSET` alone does not limit results — it only skips rows. Always pair it with `LIMIT`.

---

### Example — Products 6th to 10th by Price

After displaying the 5 cheapest products, skip them and show the next 5:

```sql
SELECT ProductName, Price
FROM Products
ORDER BY Price ASC
LIMIT 5 OFFSET 5;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Cookware Set | 129.99 |
| Bluetooth Speaker | 135.50 |
| Smartphone | 635.79 |
| Laptop | 1099.99 |
| Treadmill | 1499.99 |

> 💡 `OFFSET 5` skips the first 5 rows (ranks 1–5), then `LIMIT 5` returns the next 5 rows (ranks 6–10).

---

### Example — Products Ranked 4th to 6th by Price

```sql
SELECT ProductName, Price
FROM Products
ORDER BY Price ASC
LIMIT 3 OFFSET 3;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Blender | 90.25 |
| Cookware Set | 129.99 |
| Bluetooth Speaker | 135.50 |

> `OFFSET 3` skips rows 1–3, then `LIMIT 3` returns rows 4, 5, and 6.

---

## Pagination

`LIMIT` and `OFFSET` together power **pagination** — displaying large datasets in manageable pages. Instead of showing hundreds of items at once, we show a fixed number per page and let users navigate between pages.

### Pagination Formula

```
OFFSET = (PageNumber - 1) × ItemsPerPage
LIMIT  = ItemsPerPage
```

### Example — 10 Products Per Page

```sql
-- Page 1: rows 1–10
SELECT ProductName, Price FROM Products ORDER BY ProductName ASC LIMIT 10 OFFSET 0;

-- Page 2: rows 11–20
SELECT ProductName, Price FROM Products ORDER BY ProductName ASC LIMIT 10 OFFSET 10;

-- Page 3: rows 21–30
SELECT ProductName, Price FROM Products ORDER BY ProductName ASC LIMIT 10 OFFSET 20;
```

### Pagination Reference Table

| Page | `LIMIT` | `OFFSET` | Rows Returned |
|---|---|---|---|
| 1 | 10 | 0 | Rows 1–10 |
| 2 | 10 | 10 | Rows 11–20 |
| 3 | 10 | 20 | Rows 21–30 |
| 4 | 10 | 30 | Rows 31–40 |
| *n* | 10 | (n-1) × 10 | Rows (10n-9)–10n |

> ✅ Pagination improves user experience by preventing information overload — users browse at their own pace through predictable, consistent pages of data.

---

## How `LIMIT` and `OFFSET` Work Together

```
Full sorted dataset:
┌────┬──────────────────────┬─────────┐
│ #  │ ProductName          │ Price   │
├────┼──────────────────────┼─────────┤
│ 1  │ Ballpoint Pen Set    │   5.00  │  ← OFFSET 3 skips rows 1–3
│ 2  │ Knife Set            │  50.00  │  ← skipped
│ 3  │ Smart Outdoor Light  │  75.62  │  ← skipped
│ 4  │ Dumbbell Set         │  82.50  │  ← LIMIT 3 starts here  ✓
│ 5  │ Blender              │  90.25  │                          ✓
│ 6  │ Cookware Set         │ 129.99  │                          ✓
│ 7  │ Bluetooth Speaker    │ 135.50  │  ← not included
│ 8  │ Smartphone           │ 635.79  │  ← not included
└────┴──────────────────────┴─────────┘
Result: rows 4, 5, 6 (LIMIT 3 OFFSET 3)
```

---

## Database Compatibility Note

> 📌 `LIMIT` is **not part of the SQL standard** — it is specific to **MySQL** and some other databases (e.g., PostgreSQL, SQLite). Some databases use different syntax:

| Database | Equivalent Syntax |
|---|---|
| MySQL, PostgreSQL, SQLite | `LIMIT n OFFSET x` |
| SQL Server, MS Access | `SELECT TOP n ...` |
| Oracle | `WHERE ROWNUM <= n` or `FETCH FIRST n ROWS ONLY` |

---

## 🧩 Code Exercises

---

### Task 1 — Retrieve the First 10 Customers

**Retrieve the names of the first ten customers from the `Customers` table.**

```sql
SELECT CustomerName
FROM Customers
LIMIT 10;
```

#### Expected Output

| CustomerName |
|---|
| John Doe |
| Jane Smith |
| Alice Johnson |
| Michael Carter |
| Charlie Davis |
| Sophia Martin |
| Ethan Williams |
| Emma Watson |
| John Doe |
| Emily Davis |

> 💡 No `ORDER BY` is used here — the first 10 rows in insertion order are returned. In a real application, always add `ORDER BY CustomerID ASC` for consistent results.

---

### Task 2 — Orders 6th to 10th by Date

**Fetch the 6th to 10th orders by `OrderDate`. Return only `OrderID` and `OrderDate`.**

```sql
SELECT OrderID, OrderDate
FROM Orders
ORDER BY OrderDate ASC
LIMIT 5 OFFSET 5;
```

#### Expected Output

| OrderID | OrderDate |
|---|---|
| 3 | 2024-07-03 |
| 4 | 2024-08-18 |
| 9 | 2024-09-10 |
| 10 | 2024-10-05 |
| 5 | 2024-11-07 |

> 💡 `OFFSET 5` skips the 5 earliest orders; `LIMIT 5` then returns the next 5 — orders ranked 6th through 10th by date.

---

### Task 3 — Pagination: Second Page of 3 Products

**Return the second set of 3 products (rows 4–6), ordered alphabetically by `ProductName`. Return only `ProductName` and `Price`.**

```sql
SELECT ProductName, Price
FROM Products
ORDER BY ProductName ASC
LIMIT 3 OFFSET 3;
```

#### Expected Output

| ProductName | Price |
|---|---|
| Camping Chair | 100.00 |
| Cookware Set | 129.99 |
| Desk Organizer | 85.99 |

> 💡 This is page 2 of a 3-items-per-page layout. Page 1 would use `LIMIT 3 OFFSET 0`, page 3 would use `LIMIT 3 OFFSET 6`, and so on.

---

## `LIMIT` and `OFFSET` — Quick Reference

| Goal | Syntax | Example |
|---|---|---|
| First `n` rows | `LIMIT n` | `LIMIT 5` |
| Top `n` after sorting | `ORDER BY col LIMIT n` | `ORDER BY Price ASC LIMIT 5` |
| Skip first `x` rows | `LIMIT n OFFSET x` | `LIMIT 5 OFFSET 5` |
| Page `p` with `n` rows/page | `LIMIT n OFFSET (p-1)*n` | `LIMIT 10 OFFSET 20` (page 3) |
| Single top record | `ORDER BY col DESC LIMIT 1` | `ORDER BY Price DESC LIMIT 1` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always use `ORDER BY` with `LIMIT`** | Without sorting, returned rows can vary between runs — unpredictable |
| **Use pagination for large datasets** | Prevents overloading the UI and improves response time |
| **Track `OFFSET` and `LIMIT` values** | Ensures users can navigate pages consistently |
| **Start `OFFSET` at 0 for page 1** | `OFFSET 0` returns from the very first row — don't start at 1 |
| **Use the formula `(page - 1) × items_per_page`** | Consistent, reliable pagination calculation |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Using `LIMIT` without `ORDER BY`** — the returned subset is not guaranteed to be consistent across multiple runs. Always sort first.

> ⚠️ **Using `OFFSET` without `LIMIT`** — `OFFSET` only skips rows; without `LIMIT`, the database still returns all remaining rows after the skip.

> ⚠️ **Confusing `OFFSET` and `LIMIT`** — remember: `OFFSET` = rows to **skip**, `LIMIT` = rows to **return**.

> ⚠️ **Starting page 1 with `OFFSET 1`** — this skips the very first row. Page 1 should always use `OFFSET 0`.

> ⚠️ **Using `LIMIT` in non-MySQL databases without checking syntax** — `LIMIT` is MySQL-specific. SQL Server uses `TOP`, Oracle uses `FETCH FIRST n ROWS ONLY`.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `LIMIT n` | Returns only the first `n` rows |
| `ORDER BY` + `LIMIT` | Sorts first, then limits — ensures meaningful results |
| `OFFSET x` | Skips the first `x` rows before returning results |
| `LIMIT n OFFSET x` | The foundation of pagination |
| Pagination formula | `OFFSET = (page - 1) × items_per_page` |
| Database note | `LIMIT` is MySQL-specific — other databases use `TOP` or `FETCH FIRST` |

---

## What's Next?

Now that you can control exactly which rows to retrieve, let's explore more powerful data analysis tools!

- 🔢 **Aggregating data with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`**
- 📊 **Grouping data with `GROUP BY`**
- 🔎 **Filtering groups with `HAVING`**
- 🔗 **Combining tables with `JOIN`**

> *`LIMIT` and `OFFSET` turn an ocean of data into manageable, navigable pages — your users will thank you!* 🚀

---

