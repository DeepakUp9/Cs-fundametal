# Sorting Results
> *Learn to apply sorting on SQL queries.*

---

## Introduction

In many real-world scenarios — such as displaying a product catalog or listing customer orders — we want results to appear in a clear, logical order. Imagine a large set of products with varying prices, stock levels, and names. If displayed randomly, it would be hard to find the most expensive item, the cheapest product, or scan through them alphabetically. By sorting query results, we bring structure to what might otherwise be a confusing set of data.

Sorting is an essential feature of SQL that helps us:
- 📋 Organize data for reports
- 🏆 Quickly identify top-performing or underperforming items
- 👁️ Improve data readability and presentation

### 🎯 Learning Goals

- ✅ Use the `ORDER BY` clause to sort results by one or more columns
- ✅ Organize data by sorting across multiple columns
- ✅ Recognize the default sorting order and specify both ascending and descending orders

---

## The `ORDER BY` Clause

The `ORDER BY` clause is placed at the **end** of a `SELECT` statement. It lets us pick one or more columns to determine the sequence of rows in the output.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName
ORDER BY Column1 [ASC|DESC], Column2 [ASC|DESC], ...;
```

### Default Sorting Behaviour

By default, `ORDER BY` sorts in **ascending order (`ASC`)**:

| Column Type | Ascending (`ASC`) Order |
|---|---|
| **Numeric** | Smallest → Largest (e.g., 5, 10, 50, 100) |
| **Text** | Alphabetical A → Z |
| **Date** | Earliest → Latest |

To reverse the order, add `DESC` after the column name.

---

## 1️⃣ Sorting by a Single Column

### Example — Sort Products by Price (Ascending)

```sql
SELECT ProductName, Price
FROM Products
ORDER BY Price;
-- Same as: ORDER BY Price ASC
```

#### Sample Output

| ProductName | Price |
|---|---|
| Ballpoint Pen Set | 5.00 |
| Knife Set | 50.00 |
| Smart Outdoor Light | 75.62 |
| Dumbbell Set | 82.50 |
| Blender | 90.25 |
| Bluetooth Speaker | 135.50 |
| Smartphone | 635.79 |
| Laptop | 1099.99 |
| Treadmill | 1499.99 |

### Example — Sort Products by Price (Descending)

```sql
SELECT ProductName, Price
FROM Products
ORDER BY Price DESC;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Treadmill | 1499.99 |
| Laptop | 1099.99 |
| Smartphone | 635.79 |
| Bluetooth Speaker | 135.50 |
| Blender | 90.25 |
| Dumbbell Set | 82.50 |
| Smart Outdoor Light | 75.62 |
| Knife Set | 50.00 |
| Ballpoint Pen Set | 5.00 |

### Example — Sort Customers Alphabetically

```sql
SELECT CustomerName, Email
FROM Customers
ORDER BY CustomerName ASC;
```

### Example — Sort Orders by Most Recent Date

```sql
SELECT OrderID, CustomerID, OrderDate, TotalAmount
FROM Orders
ORDER BY OrderDate DESC;
```

---

## 2️⃣ Sorting by Multiple Columns

Sometimes a single column isn't enough. For instance, we might want to sort products first by category, then by price within each category. SQL applies sorting in the order the columns are listed — the first column is the **primary sort**, and subsequent columns break ties within the primary sort.

### Understanding Ties

A **tie** occurs when two or more rows have the same value in the primary sort column. For example:

| ProductName | Price |
|---|---|
| Smartphone | 50.00 |
| Laptop | 50.00 |
| Knife Set | 50.00 |
| Dumbbell Set | 82.00 |
| Ballpoint Pen Set | 5.00 |

If we sort only by `Price`, the three products at $50.00 are tied — their relative order is undefined. Adding a second sort column (like `ProductName`) breaks the tie with a clear, predictable rule.

---

### Example — Sort by Category then Price

Sort products by `CategoryID` ascending, then by `Price` descending within each category:

```sql
SELECT ProductName, CategoryID, Price
FROM Products
ORDER BY CategoryID ASC, Price DESC;
```

#### Sample Output

| ProductName | CategoryID | Price |
|---|---|---|
| Laptop | 1 | 1099.99 |
| Smartphone | 1 | 635.79 |
| Bluetooth Speaker | 1 | 135.50 |
| Cookware Set | 2 | 129.99 |
| Blender | 2 | 90.25 |
| Knife Set | 2 | 50.00 |
| Treadmill | 3 | 1499.99 |
| Dumbbell Set | 3 | 82.50 |

> 💡 Within each `CategoryID` group, products are ordered from most expensive to least expensive. The first sort column groups them; the second sort column orders within those groups.

---

### Example — Sort by Multiple Columns with Mixed Directions

Sort customers by city (from the address, ascending) and then by name (descending) within each city:

```sql
SELECT CustomerName, Address
FROM Customers
ORDER BY Address ASC, CustomerName DESC;
```

> ✅ Each column can have its own `ASC` or `DESC` direction independently.

---

## 3️⃣ Using `ORDER BY` with `WHERE`

`ORDER BY` can be combined with `WHERE` to first filter results and then sort them. The clause order in the query must be: `WHERE` before `ORDER BY`.

### Clause Order

```sql
SELECT ...
FROM ...
WHERE ...      -- Filter first
ORDER BY ...;  -- Then sort
```

### Example — Filter Under $500, Sort by Price Descending

```sql
SELECT ProductName, Price
FROM Products
WHERE Price < 500
ORDER BY Price DESC;
```

#### Sample Output

| ProductName | Price |
|---|---|
| Smartwatch | 350.99 |
| Bluetooth Speaker | 135.50 |
| Cookware Set | 129.99 |
| Blender | 90.25 |
| Dumbbell Set | 82.50 |
| Smart Outdoor Light | 75.62 |
| Knife Set | 50.00 |
| Ballpoint Pen Set | 5.00 |

### Example — Filter by Category and Sort by Stock

```sql
SELECT ProductName, CategoryID, Stock
FROM Products
WHERE CategoryID IN (1, 3)
ORDER BY Stock ASC;
```

---

## 4️⃣ Using Column Numbers in `ORDER BY`

Instead of column names, you can use the **positional number** of a column from the `SELECT` clause in `ORDER BY`. Position `1` = first column in `SELECT`, position `2` = second, and so on.

### Example

```sql
SELECT ProductName, Price   -- Price is column 2
FROM Products
ORDER BY 2;                  -- Sort by Price (same as ORDER BY Price ASC)
```

### When It's Useful

| Scenario | Example |
|---|---|
| Long column names | Avoid retyping a long alias |
| Calculated columns | `ORDER BY 3` instead of repeating `(Price * Stock)` |

> ⚠️ **Use sparingly.** Column numbers refer to the `SELECT` list position — not the table structure. If someone reorders the `SELECT` columns, the sort will silently change. Always prefer column names for readability and safety in complex or shared queries.

---

## `ASC` vs. `DESC` — Quick Reference

| Keyword | Full Name | Numeric Order | Text Order | Date Order |
|---|---|---|---|---|
| `ASC` | Ascending (default) | 1, 2, 3... | A → Z | Oldest → Newest |
| `DESC` | Descending | ...3, 2, 1 | Z → A | Newest → Oldest |

---

## 🧩 Code Exercises

---

### Task 1 — Sort by Multiple Columns

**Retrieve all customers sorted first by `Address` (ascending) and then by `CustomerName` (descending).**

```sql
SELECT CustomerName, Address
FROM Customers
ORDER BY Address ASC, CustomerName DESC;
```

#### Expected Output (sample)

| CustomerName | Address |
|---|---|
| Emma Watson | 101 Maple Street, Springfield, 62701 |
| Michael Carter | 101 Pine Court, Meadowbrook, 35242 |
| Liam Scott | 11 Birch Circle, Sunset Valley, 78745 |
| … | … |
| James Walker | 78 Fir Avenue, Pineville, 28134 |
| Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| Grace Taylor | 90 Cedar Crescent, Springfield, 62701 |

> 💡 Rows with the same `Address` are further sorted by `CustomerName` in Z → A order — the second column breaks the tie.

---

### Task 2 — Sort by a Single Column

**Retrieve all products sorted by `Stock` in ascending order.**

```sql
SELECT ProductName, Stock
FROM Products
ORDER BY Stock ASC;
```

#### Expected Output (sample)

| ProductName | Stock |
|---|---|
| Treadmill | 10 |
| Tent | 10 |
| Dumbbell Set | 15 |
| … | … |
| Bluetooth Speaker | 100 |
| Ballpoint Pen Set | 150 |
| Notebook | 200 |

> 💡 `Treadmill` and `Tent` are both tied at `Stock = 10` — their relative order within the tie is not guaranteed without a second sort column.

---

## `ORDER BY` Quick Reference

| Goal | Syntax |
|---|---|
| Sort ascending (default) | `ORDER BY Price` or `ORDER BY Price ASC` |
| Sort descending | `ORDER BY Price DESC` |
| Sort by multiple columns | `ORDER BY CategoryID ASC, Price DESC` |
| Sort with filter | `WHERE Price < 500 ORDER BY Price DESC` |
| Sort by column position | `ORDER BY 2` (2nd column in SELECT) |
| Sort text alphabetically | `ORDER BY CustomerName ASC` |
| Sort dates newest first | `ORDER BY OrderDate DESC` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always specify `ASC` or `DESC` explicitly** | Prevents ambiguity — don't rely on the default being understood |
| **Use column names, not numbers** | Column positions can shift when queries are edited; names are stable |
| **Put `ORDER BY` after `WHERE`** | SQL requires this specific clause order |
| **Use a secondary sort to break ties** | Ensures a predictable, repeatable result order |
| **Avoid excessive sorting on large tables** | Sorting is computationally expensive on millions of rows — use indexes |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Assuming results have a default order without `ORDER BY`** — SQL makes no guarantee about row order unless `ORDER BY` is explicitly specified.

> ⚠️ **Forgetting to specify `ASC` or `DESC`** — while ascending is the default, always write it explicitly for clarity, especially in multi-column sorts.

> ⚠️ **Sorting by a column not in the `SELECT` list when using column numbers** — `ORDER BY 5` when only 3 columns are selected will cause an error.

> ⚠️ **Overusing `ORDER BY` in large production queries** — sorting millions of rows without an index is slow. Consider indexed columns for frequently sorted data.

> ⚠️ **Putting `ORDER BY` before `WHERE`** — this causes a syntax error. The correct order is always `WHERE` → `ORDER BY`.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `ORDER BY col ASC` | Sort ascending (default) — smallest/A/oldest first |
| `ORDER BY col DESC` | Sort descending — largest/Z/newest first |
| Multiple columns | First column is primary sort; subsequent columns break ties |
| Combined with `WHERE` | Filter first, then sort — `WHERE` always comes before `ORDER BY` |
| Column numbers | `ORDER BY 2` sorts by the second SELECT column — use sparingly |
| Ties | When values are equal in the primary sort, a secondary column defines order |

---

## What's Next?

Now that you can sort data, explore how to aggregate and group it!

- 🔢 **Aggregating data with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`**
- 📊 **Grouping data with `GROUP BY`**
- 🔎 **Filtering groups with `HAVING`**
- 🔗 **Combining tables with `JOIN`**

> *Sorting brings order to chaos — with `ORDER BY`, your data always tells its story in the right sequence!* 🚀

---

