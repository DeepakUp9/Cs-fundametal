# Range and List Filtering
> *Learn to use range and list filtering in SQL.*

---

## Introduction

Imagine we want to find products in a certain price range or belonging to a specific set of categories from the store inventory. For instance, products priced between $10 and $30, or only those in Electronics or Kitchenware. Filtering data this way helps us focus on what matters most and saves time from scrolling through irrelevant results. These tasks are efficiently handled in SQL using the `BETWEEN` and `IN` operators.

### 🎯 Learning Goals

- ✅ Use the `BETWEEN` operator to filter rows within a numeric or date range
- ✅ Use the `IN` operator to filter rows matching values in a specified list
- ✅ Understand scenarios where these operators make queries more concise and readable

---

## Filtering Data with `BETWEEN`

Working with ranges is common when dealing with numeric fields like prices or date fields like order dates. Instead of chaining multiple comparison operators, `BETWEEN` writes the same condition more clearly and concisely.

### Syntax

```sql
ColumnName BETWEEN LowerBound AND UpperBound
```

| Part | Description |
|---|---|
| `LowerBound` | The starting value of the range (inclusive) |
| `UpperBound` | The ending value of the range (inclusive) |

> ✅ `BETWEEN` is **inclusive** — both boundary values are included in the results.

### Without `BETWEEN` vs. With `BETWEEN`

```sql
-- Without BETWEEN (verbose)
WHERE Price >= 10 AND Price <= 30

-- With BETWEEN (cleaner, same result)
WHERE Price BETWEEN 10 AND 30
```

Both produce identical results — `BETWEEN` is simply a more readable shorthand.

---

### Example 1 — Filter Products by Price Range

Retrieve all products priced between $500 and $1,500:

```sql
SELECT *
FROM Products
WHERE Price BETWEEN 500 AND 1500;
```

#### Sample Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 10 | Treadmill | 3 | 1499.99 | 10 |
| 19 | Smartwatch | 1 | 500.00 | 50 |

> 💡 `Price = 500.00` and `Price = 1499.99` are both included — `BETWEEN` always includes the boundary values.

---

### Example 2 — Filter Orders by Date Range

Retrieve all orders placed in the first half of 2024:

```sql
SELECT OrderID, CustomerID, OrderDate, TotalAmount
FROM Orders
WHERE OrderDate BETWEEN '2024-01-01' AND '2024-06-30';
```

#### Sample Output

| OrderID | CustomerID | OrderDate | TotalAmount |
|---|---|---|---|
| 1 | 5 | 2024-03-15 | 1286.58 |
| 2 | 1 | 2024-05-22 | 235.50 |

> 💡 Always use the `'YYYY-MM-DD'` format for date values in SQL — this is the standard format recognised across all SQL environments.

---

### Example 3 — Exclusive Range (Without `BETWEEN`)

`BETWEEN` is always inclusive. To filter an **exclusive** range (not including boundary values), use comparison operators instead:

```sql
-- Include $10 and $30 (inclusive)
WHERE Price BETWEEN 10 AND 30

-- Exclude $10 and $30 (exclusive)
WHERE Price > 10 AND Price < 30

-- Include $10, exclude $30 (half-open range)
WHERE Price >= 10 AND Price < 30
```

---

### When to Use `BETWEEN`

| Use Case | Example |
|---|---|
| Price range | `Price BETWEEN 10 AND 100` |
| Date range | `OrderDate BETWEEN '2024-01-01' AND '2024-12-31'` |
| ID range | `CustomerID BETWEEN 1 AND 50` |
| Stock range | `Stock BETWEEN 10 AND 50` |

---

## Filtering Data with `IN`

Sometimes we don't need a continuous range — we need results that match a **known set of specific values**. The `IN` operator is the perfect tool for this. Instead of writing multiple `OR` conditions, `IN` keeps the query clean and concise.

### Syntax

```sql
ColumnName IN (Value1, Value2, ..., ValueN)
```

This checks if the value in `ColumnName` matches **any** value in the list.

### Without `IN` vs. With `IN`

```sql
-- Without IN (repetitive)
WHERE CategoryID = 1 OR CategoryID = 3 OR CategoryID = 5

-- With IN (cleaner, same result)
WHERE CategoryID IN (1, 3, 5)
```

Both are equivalent — `IN` just removes the repetition.

---

### Example 1 — Filter Products by Category

Retrieve all products in Electronics, Fitness, and Stationery:

```sql
SELECT *
FROM Products
WHERE CategoryID IN (1, 3, 5);
```

#### Sample Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 2 | Dumbbell Set | 3 | 82.50 | 15 |
| 6 | Ballpoint Pen Set | 5 | 5.00 | 150 |
| 7 | Bluetooth Speaker | 1 | 135.50 | 100 |
| 10 | Treadmill | 3 | 1499.99 | 10 |

---

### Example 2 — Filter by Customer Names

Retrieve orders for specific customers by name:

```sql
SELECT CustomerID, CustomerName, Email
FROM Customers
WHERE CustomerName IN ('John Doe', 'Alice Johnson', 'Emma Watson');
```

#### Sample Output

| CustomerID | CustomerName | Email |
|---|---|---|
| 1 | John Doe | john.doe@smail.com |
| 3 | Alice Johnson | alice.j@jmail.com |
| 23 | Emma Watson | emma.w@service.org |

---

### Example 3 — Using `NOT IN`

Retrieve all products that do **not** belong to Electronics, Fitness, or Stationery:

```sql
SELECT ProductName, CategoryID
FROM Products
WHERE CategoryID NOT IN (1, 3, 5);
```

> ✅ `NOT IN` is the clean inverse of `IN` — excludes all rows matching any value in the list.

---

### When to Use `IN`

| Use Case | Example |
|---|---|
| Specific category IDs | `CategoryID IN (1, 3, 5)` |
| Selected customer IDs | `CustomerID IN (3, 5, 12)` |
| Specific product names | `ProductName IN ('Laptop', 'Smartphone')` |
| Multiple status values | `Status IN ('Pending', 'Shipped')` |

---

## Combining `BETWEEN` and `IN`

`BETWEEN` and `IN` can be combined with `AND` / `OR` to create highly specific filters in a single, readable query.

### Example — Price Range AND Specific Categories

Find products priced between $500 and $1,500 that also belong to Electronics, Fitness, or Stationery:

```sql
SELECT *
FROM Products
WHERE (Price BETWEEN 500 AND 1500)
  AND (CategoryID IN (1, 3, 5));
```

#### Sample Output

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | 635.79 | 50 |
| 10 | Treadmill | 3 | 1499.99 | 10 |
| 19 | Smartwatch | 1 | 500.00 | 50 |

> 💡 Parentheses make the grouping explicit and easier to read — use them whenever combining `BETWEEN` and `IN` with other operators.

---

### Example — Price Range OR Specific Categories

Find products priced between $500 and $1,500 **or** belonging to Electronics, Fitness, or Stationery:

```sql
SELECT ProductName, Price, CategoryID
FROM Products
WHERE (Price BETWEEN 500 AND 1500)
   OR (CategoryID IN (1, 3, 5));
```

> 💡 With `OR`, a product only needs to satisfy **one** of the two conditions to appear in results.

---

## `BETWEEN` vs. `IN` — Quick Comparison

| Feature | `BETWEEN` | `IN` |
|---|---|---|
| **Best for** | Continuous ranges (numbers, dates) | Discrete, specific values |
| **Inclusive?** | ✅ Yes — includes both boundaries | ✅ Yes — includes all listed values |
| **Negation** | `NOT BETWEEN` | `NOT IN` |
| **Equivalent to** | `col >= low AND col <= high` | `col = v1 OR col = v2 OR ...` |
| **Works with dates?** | ✅ Yes | ✅ Yes (exact date matches) |
| **Example** | `Price BETWEEN 10 AND 100` | `CategoryID IN (1, 3, 5)` |

---

## 🧩 Code Exercises

---

### Task 1 — Combine `IN` and `BETWEEN`

**Find all products in the "Kitchenware" (`CategoryID = 2`) and "Stationery" (`CategoryID = 5`) categories priced between $5 and $100.**

```sql
SELECT ProductName, Price, CategoryID
FROM Products
WHERE CategoryID IN (2, 5)
  AND Price BETWEEN 5 AND 100;
```

#### Expected Output

| ProductName | Price | CategoryID |
|---|---|---|
| Knife Set | 50.00 | 2 |
| Blender | 90.25 | 2 |
| Smart Kitchen Scale | 75.65 | 2 |
| Ballpoint Pen Set | 5.00 | 5 |
| Desk Organizer | 85.99 | 5 |

> 💡 `AND` ensures both conditions are true — a product must be in the right category **and** within the price range to appear.

---

### Task 2 — Date Range and Specific Customers

**Retrieve orders placed between `2024-02-01` and `2024-11-07` by customers with IDs 3, 5, and 12.**

```sql
SELECT OrderID, OrderDate, CustomerID, TotalAmount
FROM Orders
WHERE (OrderDate BETWEEN '2024-02-01' AND '2024-11-07')
  AND (CustomerID IN (3, 5, 12));
```

#### Expected Output

| OrderID | OrderDate | CustomerID | TotalAmount |
|---|---|---|---|
| 7 | 2024-04-12 | 3 | 156.24 |
| 1 | 2024-03-15 | 5 | 1286.58 |
| 6 | 2024-02-01 | 12 | 1590.49 |

> 💡 `2024-02-01` is included in results because `BETWEEN` is inclusive on both ends.

---

## Filtering Operators — Full Quick Reference

| Goal | Syntax | Example |
|---|---|---|
| Within a range (inclusive) | `col BETWEEN low AND high` | `Price BETWEEN 10 AND 100` |
| Outside a range | `col NOT BETWEEN low AND high` | `Price NOT BETWEEN 10 AND 100` |
| Matches any in a list | `col IN (v1, v2, v3)` | `CategoryID IN (1, 3, 5)` |
| Matches none in a list | `col NOT IN (v1, v2, v3)` | `CategoryID NOT IN (1, 3, 5)` |
| Range + list combined | `BETWEEN ... AND ... IN (...)` | See Task examples above |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use `BETWEEN` for continuous ranges** | Cleaner and more readable than chaining `>=` and `<=` |
| **Use `IN` for discrete value sets** | Avoids long chains of `OR` conditions |
| **Always specify lower bound first in `BETWEEN`** | `BETWEEN 50 AND 10` produces no results — lower must come first |
| **Use parentheses when combining operators** | Makes logic explicit and prevents operator precedence surprises |
| **Use `NOT BETWEEN` and `NOT IN` for exclusions** | Clean and readable alternative to complex `WHERE` conditions |
| **Validate input data types** | Ensure values in `IN` lists and `BETWEEN` bounds match the column's data type |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Reversed `BETWEEN` bounds** — `BETWEEN 50 AND 10` returns no results because the lower bound must be smaller than the upper bound.

> ⚠️ **Forgetting `BETWEEN` is inclusive** — if you need to exclude boundary values, use `>` and `<` instead.

> ⚠️ **Missing commas or parentheses in `IN`** — `IN (1 3 5)` is a syntax error; values must be comma-separated: `IN (1, 3, 5)`.

> ⚠️ **Using `IN` with a very large list** — `IN (1, 2, 3, ..., 1000)` can be slow. For large sets, consider using a subquery or a temporary table instead.

> ⚠️ **Wrong date format in `BETWEEN`** — always use `'YYYY-MM-DD'` for date values to avoid unexpected results across different SQL environments.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `BETWEEN low AND high` | Filters a continuous inclusive range of numbers or dates |
| `NOT BETWEEN` | Excludes the specified range |
| `IN (v1, v2, ...)` | Filters rows matching any value in the list |
| `NOT IN (v1, v2, ...)` | Excludes rows matching any value in the list |
| Combining both | Use `AND` / `OR` with parentheses for complex, multi-condition filters |

---

## What's Next?

Now that you can filter by ranges and lists, let's explore more powerful data retrieval tools!

- 📊 **Sorting results with `ORDER BY`**
- 🔢 **Aggregating data with `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`**
- 🔗 **Combining tables with `JOIN`**
- 🔎 **Grouping data with `GROUP BY` and `HAVING`**

> *`BETWEEN` handles the spectrum, `IN` handles the list — together they cover almost any filtering scenario you'll face!* 🚀

---

