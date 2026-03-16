# The WHERE Clause
> *Learn to use the WHERE clause and logical operators in queries.*

---

## Introduction

Imagine our online store is offering a 70% discount on some shoes during a mega annual sale. A customer wants to find white shoes that are discounted and still in stock — instead of browsing through the entire database, it would be much faster to retrieve only the matching records. This is exactly what SQL's `WHERE` clause does — it filters data to return only the rows that meet specific criteria.

### 🎯 Learning Goals

- ✅ Understand the purpose and importance of the `WHERE` clause
- ✅ Use `WHERE` to filter records based on conditions
- ✅ Explore logical operators `AND`, `OR`, and `NOT`
- ✅ Combine multiple conditions for complex filters
- ✅ Understand the order of operations when multiple logical operators are involved

---

## The `WHERE` Clause

The `WHERE` clause filters rows in a table based on specific conditions — such as matching values, ranges, or patterns. It is always used alongside `SELECT`. Without it, a `SELECT` query retrieves **every row** from the table.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName
WHERE condition;
```

The `condition` specifies the criteria a row must meet to be included in the results.

### Comparison Operators Used in Conditions

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal to | `Price = 50.00` |
| `!=` or `<>` | Not equal to | `CategoryID != 1` |
| `<` | Less than | `Price < 20.00` |
| `>` | Greater than | `Stock > 0` |
| `<=` | Less than or equal to | `Price <= 100.00` |
| `>=` | Greater than or equal to | `Price >= 10.00` |

---

## 1️⃣ Filtering Based on a Single Condition

The simplest use of `WHERE` filters rows using one condition — a column, a comparison operator, and a value.

### Example — Products Priced Below $20

```sql
SELECT ProductName, Price
FROM Products
WHERE Price < 20.00;
```

### Sample Output

| ProductName | Price |
|---|---|
| Ballpoint Pen Set | 5.00 |

> 💡 Only rows where `Price < 20.00` are returned — all other products are excluded.

### More Single-Condition Examples

```sql
-- Find a product by exact name
SELECT *
FROM Products
WHERE ProductName = 'Laptop';

-- Find products with more than 100 units in stock
SELECT ProductName, Stock
FROM Products
WHERE Stock > 100;

-- Find orders placed on a specific date
SELECT OrderID, TotalAmount
FROM Orders
WHERE OrderDate = '2024-05-22';
```

---

## 2️⃣ Logical Operators for Multiple Conditions

When a single condition isn't enough, **logical operators** allow us to combine multiple filters for richer, more precise queries.

| Operator | Behaviour | Row is included when... |
|---|---|---|
| `AND` | All conditions must be true | Every condition is satisfied |
| `OR` | At least one condition must be true | Any one condition is satisfied |
| `NOT` | Negates a condition | The condition is **not** true |

---

### Using `AND`

`AND` requires **all conditions** to be true for a row to be selected.

```sql
-- Products that cost less than $50 AND are in stock
SELECT ProductName, Price, Stock
FROM Products
WHERE Price < 50.00 AND Stock > 0;
```

### Sample Output

| ProductName | Price | Stock |
|---|---|---|
| Knife Set | 50.00 | 25 |
| Ballpoint Pen Set | 5.00 | 150 |

> ✅ Both conditions must hold — a product priced below $50 but with zero stock would be excluded.

---

### Using `OR`

`OR` requires **at least one condition** to be true for a row to be selected.

```sql
-- Products in Electronics category OR priced above $100
SELECT ProductName, Price, CategoryID
FROM Products
WHERE CategoryID = 1 OR Price > 100;
```

### Sample Output

| ProductName | Price | CategoryID |
|---|---|---|
| Smartphone | 635.79 | 1 |
| Laptop | 1099.99 | 1 |
| Bluetooth Speaker | 135.50 | 1 |
| Cookware Set | 129.99 | 2 |
| Treadmill | 1499.99 | 3 |

> ✅ `Cookware Set` and `Treadmill` are included because their price exceeds $100, even though they're not in Electronics.

---

### Using `NOT`

`NOT` **negates** a condition — it selects rows where the condition is false.

```sql
-- Products that are NOT in the Electronics category
SELECT ProductName, CategoryID
FROM Products
WHERE NOT CategoryID = 1;
```

### Sample Output

| ProductName | CategoryID |
|---|---|
| Dumbbell Set | 3 |
| Smart Outdoor Light | 4 |
| Knife Set | 2 |
| Ballpoint Pen Set | 5 |
| Blender | 2 |
| Cookware Set | 2 |
| Treadmill | 3 |

> 💡 `NOT` can also be written as `CategoryID != 1` or `CategoryID <> 1` — all three are equivalent.

---

## 3️⃣ Combining Multiple Logical Operators

For complex business questions, we combine multiple operators. For example: *"Find products that either have fewer than 50 units in stock or cost more than $100, AND belong to the Fitness category."*

### Order of Operations

When multiple operators appear in a condition, SQL evaluates them in this order:

| Priority | Operator | Description |
|---|---|---|
| 1st | `()` | Parentheses — evaluated first |
| 2nd | `NOT` | Negation |
| 3rd | `AND` | All conditions must match |
| 4th | `OR` | At least one condition must match |

> ⚠️ This is just like math — multiplication before addition. `AND` is evaluated before `OR` unless you use parentheses to override it.

---

### Why Parentheses Matter

Consider this query — **without** parentheses:

```sql
-- ❌ Ambiguous — AND is evaluated before OR by default
SELECT ProductName, Price, Stock
FROM Products
WHERE Stock < 50 OR Price > 100 AND CategoryID = 3;
```

SQL internally reads this as:

```sql
WHERE Stock < 50 OR (Price > 100 AND CategoryID = 3)
```

This returns: products with `Stock < 50` from **any** category, PLUS products from CategoryID 3 with `Price > 100`. This is probably not what we wanted!

---

Now **with** parentheses:

```sql
-- ✅ Explicit grouping — exactly what we want
SELECT ProductName, Price, Stock
FROM Products
WHERE (Stock < 50 OR Price > 100) AND CategoryID = 3;
```

SQL evaluates:
1. First: `(Stock < 50 OR Price > 100)` — either condition inside the brackets
2. Then: `AND CategoryID = 3` — result must also be in Fitness category

### Sample Output

| ProductName | Price | Stock |
|---|---|---|
| Dumbbell Set | 82.50 | 15 |
| Treadmill | 1499.99 | 10 |

> ✅ Only Fitness products (`CategoryID = 3`) that have either less than 50 stock OR price above $100 are returned.

---

### Side-by-Side Comparison — With vs. Without Parentheses

```sql
-- Without parentheses (AND binds tighter than OR)
WHERE Stock < 50 OR Price > 100 AND CategoryID = 3
-- Interpreted as: Stock < 50 (any category) OR (Price > 100 AND CategoryID = 3)

-- With parentheses (your intended logic)
WHERE (Stock < 50 OR Price > 100) AND CategoryID = 3
-- Interpreted as: (Stock < 50 OR Price > 100) AND must be in CategoryID = 3
```

---

## 4️⃣ More Practical Examples

### Range Filter — Price Between Two Values

```sql
-- Products priced between $50 and $200 (inclusive)
SELECT ProductName, Price
FROM Products
WHERE Price >= 50.00 AND Price <= 200.00;
```

### Exclude Multiple Categories

```sql
-- Products that are not Kitchenware or Stationery
SELECT ProductName, CategoryID
FROM Products
WHERE NOT CategoryID = 2 AND NOT CategoryID = 5;
```

### Combined Filter — Category, Price, and Stock

```sql
-- Electronics products under $700 that are in stock
SELECT ProductName, Price, Stock
FROM Products
WHERE CategoryID = 1 AND Price < 700 AND Stock > 0;
```

### Filter on Dates

```sql
-- Orders placed after July 1, 2024
SELECT OrderID, CustomerID, OrderDate, TotalAmount
FROM Orders
WHERE OrderDate > '2024-07-01';
```

---

## 🧩 Code Exercise

### Task

Write a query to find products **priced between $10 and $1000**, belonging to the **Electronics category** (`CategoryID = 1`), and **currently in stock**.

### Solution

```sql
SELECT ProductName, Price, Stock
FROM Products
WHERE Price >= 10
  AND Price <= 1000
  AND CategoryID = 1
  AND Stock > 0;
```

### Expected Output

| ProductName | Price | Stock |
|---|---|---|
| Smartphone | 635.79 | 50 |
| Bluetooth Speaker | 135.50 | 100 |
| Smartwatch | 500.00 | 50 |

> 💡 All four conditions must be satisfied simultaneously — `AND` ensures every filter is applied.

---

## WHERE Clause — Quick Reference

| Goal | Syntax Example |
|---|---|
| Single condition | `WHERE Price < 50` |
| Both conditions true | `WHERE Price < 50 AND Stock > 0` |
| Either condition true | `WHERE CategoryID = 1 OR Price > 100` |
| Exclude a value | `WHERE NOT CategoryID = 1` |
| Range filter | `WHERE Price >= 10 AND Price <= 1000` |
| Complex grouping | `WHERE (Stock < 50 OR Price > 100) AND CategoryID = 3` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Start simple, then add complexity** | Test one condition first before layering `AND` / `OR` |
| **Always use parentheses for complex conditions** | Prevents unexpected results due to operator precedence |
| **Comment complex `WHERE` clauses** | Makes logic clear for future maintenance |
| **Test with varied data** | Ensure the filter works for edge cases, not just happy path |
| **Use meaningful comparisons** | `Price > 0` is clearer than relying on implicit truthy values |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting operator precedence** — `AND` always binds tighter than `OR`. Without parentheses, `A OR B AND C` means `A OR (B AND C)`, not `(A OR B) AND C`.

> ⚠️ **Omitting parentheses in complex queries** — leads to subtly wrong results that can be hard to debug.

> ⚠️ **Not testing queries with varied data** — a query may appear to work correctly on limited data but fail on edge cases.

> ⚠️ **Using `=` with NULL values** — `WHERE Column = NULL` does not work in SQL. Use `WHERE Column IS NULL` instead.

> ⚠️ **Filtering on non-indexed columns in large tables** — can cause slow queries. Consider indexing frequently filtered columns.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `WHERE` | Filters rows based on conditions — without it, all rows are returned |
| Comparison operators | `=`, `!=`, `<`, `>`, `<=`, `>=` for building conditions |
| `AND` | All conditions must be true |
| `OR` | At least one condition must be true |
| `NOT` | Negates a condition — selects rows where condition is false |
| Order of operations | `()` → `NOT` → `AND` → `OR` |
| Parentheses | Always use them to make complex conditions explicit and correct |

---

## What's Next?

Now that you can filter rows precisely, it's time to learn more powerful ways to work with data!

- 📊 **Sorting results with `ORDER BY`**
- 🔢 **Limiting results with `LIMIT`**
- 🔎 **Pattern matching with `LIKE`**
- 📋 **Filtering lists with `IN` and `BETWEEN`**

> *The `WHERE` clause is your precision tool — the more accurately you write conditions, the more meaningful your results become!* 🚀

---

