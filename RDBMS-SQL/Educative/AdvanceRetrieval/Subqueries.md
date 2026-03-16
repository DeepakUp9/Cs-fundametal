# Subqueries
> *Learn about using subqueries in SQL.*

---

## Introduction

Imagine finding all customers who have placed orders totalling more than $100. This seems complex at first — but SQL **subqueries** let us break it into smaller, manageable steps. A subquery is a query nested inside another query, providing a flexible way to retrieve and manipulate data dynamically.

The problem above can be broken into three steps:
1. Find the total amount spent by each customer
2. Check if that total is greater than $100
3. Retrieve the customers who qualify

### 🎯 Learning Goals

- ✅ Understand what subqueries are and their importance
- ✅ Create subqueries with both aggregate and non-aggregate queries
- ✅ Use subqueries in `SELECT`, `FROM`, and `WHERE` clauses
- ✅ Use subqueries with `IN` and `DISTINCT`

---

## What Is a Subquery?

A **subquery** (also called an inner query or nested query) is a `SELECT` statement written inside another SQL statement. The outer statement is called the **main query** or **outer query**.

```sql
SELECT column
FROM table
WHERE column = (SELECT column FROM another_table WHERE condition);
--              ↑──────────────── subquery ────────────────────↑
```

### Key Rules

| Rule | Detail |
|---|---|
| **Always in parentheses** | `(SELECT ...)` — parentheses are required |
| **Executes first** | The inner query runs before the outer query uses its result |
| **Acts as temporary data** | Behaves like a temporary table or a single value |
| **Can be used in** | `SELECT`, `FROM`, `WHERE`, `HAVING`, `INSERT`, `UPDATE`, `DELETE` |

---

## Types of Subqueries

| Type | Returns | Common Usage |
|---|---|---|
| **Non-aggregate** | Raw values (no aggregation) | Compare field values between tables |
| **Aggregate** | A summarized value (`SUM`, `AVG`, etc.) | Compare against totals or averages |
| **Correlated** | A value per row of the outer query | Runs once per outer row |
| **Non-correlated** | One fixed result | Runs once; result is reused |

---

## 1️⃣ Non-Aggregate Subqueries

Non-aggregate subqueries return data without any aggregation. They are useful for comparing field values between tables.

### Example — Products in the "Electronics" Category

```sql
-- Inner query finds the CategoryID for "Electronics"
-- Outer query finds all products in that category

SELECT ProductName
FROM Products
WHERE CategoryID = (
    SELECT CategoryID
    FROM Categories
    WHERE CategoryName = 'Electronics'
);
```

#### How It Works

1. **Inner query** runs first: `SELECT CategoryID FROM Categories WHERE CategoryName = 'Electronics'` → returns `1`
2. **Outer query** uses that result: `WHERE CategoryID = 1`

#### Sample Output

| ProductName |
|---|
| Smartphone |
| Laptop |
| Bluetooth Speaker |
| Smart TV |
| Smartwatch |

> 💡 The subquery runs once and returns a single value — this is a **non-correlated** subquery.

---

## 2️⃣ Aggregate Subqueries

Aggregate subqueries use functions like `SUM`, `AVG`, `MIN`, `MAX`, or `COUNT`. They are useful for comparing records against calculated totals.

### Example — Products with Total Sales Over $50

```sql
SELECT ProductName
FROM Products
WHERE (
    SELECT SUM(TotalItemPrice)
    FROM Order_Details
    WHERE Order_Details.ProductID = Products.ProductID
) > 50;
```

#### How It Works

- The inner query calculates `SUM(TotalItemPrice)` for a **specific product** (correlated to each row in `Products`)
- The outer query keeps only products where that sum exceeds $50

> 💡 This is a **correlated subquery** — it references `Products.ProductID` from the outer query and runs **once per product row**.

#### Sample Output

| ProductName |
|---|
| Smartphone |
| Laptop |
| Dumbbell Set |

---

## 3️⃣ Subqueries in the `SELECT` Clause

Subqueries in `SELECT` create a **calculated derived column** for each row returned by the outer query.

### Example — Products with Their Total Sales

```sql
SELECT
    ProductName,
    Price,
    Stock,
    (
        SELECT SUM(TotalItemPrice)
        FROM Order_Details
        WHERE Order_Details.ProductID = Products.ProductID
    ) AS TotalSales
FROM Products;
```

#### Sample Output

| ProductName | Price | Stock | TotalSales |
|---|---|---|---|
| Smartphone | 635.79 | 50 | 1271.58 |
| Dumbbell Set | 82.50 | 15 | NULL |
| Laptop | 1099.99 | 30 | 1099.99 |
| Ballpoint Pen Set | 5.00 | 150 | 15.00 |

> 💡 Products with no orders show `NULL` for `TotalSales` — the subquery returns `NULL` when no matching rows exist in `Order_Details`.

---

## 4️⃣ Subqueries in the `WHERE` Clause

This is the most common use of subqueries — filtering data based on results from another table or calculation.

### Example — Products Priced Above the Average

```sql
SELECT ProductName, Price
FROM Products
WHERE Price > (
    SELECT AVG(Price)
    FROM Products
);
```

#### Sample Output

| ProductName | Price |
|---|---|
| Laptop | 1099.99 |
| Treadmill | 1499.99 |
| Smart TV | 2599.99 |
| Smartwatch | 500.00 |

---

### Example — Customers Who Spent More Than $100

```sql
SELECT CustomerName,
       (
           SELECT SUM(TotalAmount)
           FROM Orders
           WHERE Orders.CustomerID = Customers.CustomerID
       ) AS TotalSpent
FROM Customers
WHERE (
    SELECT SUM(TotalAmount)
    FROM Orders
    WHERE Orders.CustomerID = Customers.CustomerID
) > 100;
```

#### How It Works

| Part | Purpose |
|---|---|
| First subquery (in `SELECT`) | Calculates total spent per customer for display |
| Second subquery (in `WHERE`) | Filters to only customers who spent more than $100 |

#### Sample Output

| CustomerName | TotalSpent |
|---|---|
| Charlie Davis | 1286.58 |
| John Doe | 235.50 |
| Jane Smith | 2639.99 |
| Michael Carter | 2550.50 |
| Alice Johnson | 758.99 |

---

## 5️⃣ Subqueries in the `FROM` Clause

Subqueries in `FROM` create a **derived table** (a temporary named result set) that the outer query treats like a real table.

### Example — Average Price per Category

```sql
SELECT CategoryName, AVG(Price) AS AveragePrice
FROM (
    SELECT p.Price, c.CategoryName
    FROM Products p, Categories c
    WHERE p.CategoryID = c.CategoryID
) AS CategoryProducts
GROUP BY CategoryName;
```

#### How It Works

1. The **subquery** joins `Products` and `Categories`, producing a temporary table named `CategoryProducts`
2. The **outer query** groups by `CategoryName` and calculates the average price from that temporary table

#### Sample Output

| CategoryName | AveragePrice |
|---|---|
| Electronics | 501.71 |
| Fitness | 791.25 |
| Kitchenware | 90.12 |
| Outdoor | 170.87 |
| Stationery | 45.50 |

> ✅ Derived table subqueries **must be given an alias** (`AS CategoryProducts`) — SQL requires it.

---

## 6️⃣ Subqueries with `IN`

`IN` with a subquery filters rows where a column's value matches **any value** in the subquery's result set.

### Example — Customers Who Spent More Than $100 (Using `IN`)

```sql
SELECT CustomerName
FROM Customers
WHERE CustomerID IN (
    SELECT CustomerID
    FROM Orders
    GROUP BY CustomerID
    HAVING SUM(TotalAmount) > 100
);
```

#### How It Works

1. Inner query groups orders by customer and finds those with total > $100
2. Outer query returns names of customers whose `CustomerID` appears in that list

---

### Example — Products Ordered More Than 2 Units

```sql
SELECT ProductName
FROM Products
WHERE ProductID IN (
    SELECT ProductID
    FROM Order_Details
    WHERE Quantity > 2
);
```

#### Sample Output

| ProductName |
|---|
| Ballpoint Pen Set |
| Cookware Set |
| Smart Outdoor Light |

---

## 7️⃣ Subqueries with `DISTINCT`

Adding `DISTINCT` to a subquery ensures only **unique values** are returned.

### Example — Categories of Products That Have Been Ordered

```sql
SELECT DISTINCT CategoryName
FROM Categories
WHERE CategoryID IN (
    SELECT DISTINCT CategoryID
    FROM Products
    WHERE ProductID IN (
        SELECT DISTINCT ProductID
        FROM Order_Details
    )
);
```

#### How It Works (Layer by Layer)

| Layer | Query | Returns |
|---|---|---|
| **Innermost** | `SELECT DISTINCT ProductID FROM Order_Details` | Unique product IDs that have been ordered |
| **Middle** | `SELECT DISTINCT CategoryID FROM Products WHERE ProductID IN (...)` | Unique category IDs for those products |
| **Outer** | `SELECT DISTINCT CategoryName FROM Categories WHERE CategoryID IN (...)` | Unique category names |

#### Sample Output

| CategoryName |
|---|
| Electronics |
| Fitness |
| Kitchenware |
| Stationery |

---

## Correlated vs. Non-Correlated Subqueries

| Type | Description | Runs | Example |
|---|---|---|---|
| **Non-correlated** | Independent of the outer query — runs once | Once | `WHERE Price > (SELECT AVG(Price) FROM Products)` |
| **Correlated** | References the outer query — runs per outer row | Once per outer row | `WHERE (SELECT SUM(...) WHERE ProductID = Products.ProductID) > 50` |

> ⚠️ Correlated subqueries are powerful but can be **slow on large tables** — they re-execute for every row of the outer query. Consider using `JOIN` with `GROUP BY` as a performance alternative when possible.

---

## Subquery Locations — Summary

| Location | Syntax | Purpose |
|---|---|---|
| **`WHERE` clause** | `WHERE col = (SELECT ...)` | Filter rows based on a calculated value |
| **`SELECT` clause** | `SELECT (SELECT ...) AS alias` | Add a calculated column to each output row |
| **`FROM` clause** | `FROM (SELECT ...) AS alias` | Treat a query result as a derived table |
| **`IN` operator** | `WHERE col IN (SELECT ...)` | Filter rows matching a set of values |
| **`HAVING` clause** | `HAVING col > (SELECT ...)` | Filter groups based on a calculated value |

---

## 🧩 Code Exercises

---

### Task 1 — Find the Cheapest Product

**Retrieve product IDs and names for products with the minimum price.**

```sql
SELECT ProductID, ProductName
FROM Products
WHERE Price = (
    SELECT MIN(Price)
    FROM Products
);
```

#### Expected Output

| ProductID | ProductName |
|---|---|
| 6 | Ballpoint Pen Set |

> 💡 The subquery calculates the global minimum price; the outer query finds all products matching that price.

---

### Task 2 — Orders Below Average

**Find all orders with a `TotalAmount` below the average of all orders.**

```sql
SELECT OrderID, CustomerID, TotalAmount
FROM Orders
WHERE TotalAmount < (
    SELECT AVG(TotalAmount)
    FROM Orders
);
```

#### Expected Output

| OrderID | CustomerID | TotalAmount |
|---|---|---|
| 2 | 10 | 235.50 |
| 5 | 16 | 758.99 |
| 7 | 3 | 156.24 |
| 8 | 8 | 230.99 |
| 10 | 13 | 751.50 |

---

### Task 3 — Suppliers of "Outdoor" Products (Nested Subqueries)

**Find the names of suppliers who provide products in the "Outdoor" category.**

```sql
SELECT SupplierName
FROM Suppliers
WHERE SupplierID IN (
    SELECT SupplierID
    FROM Product_Suppliers
    WHERE ProductID IN (
        SELECT ProductID
        FROM Products
        WHERE CategoryID = (
            SELECT CategoryID
            FROM Categories
            WHERE CategoryName = 'Outdoor'
        )
    )
);
```

#### Expected Output

| SupplierName |
|---|
| SportyGear |
| ActiveLifestyle |
| StationaryWorld |
| FitPower |
| OutdoorExplorers |
| TechNexus |
| KitchenMasters |

> 💡 This chains **4 nested subqueries**: Category name → CategoryID → ProductIDs → SupplierIDs → Supplier names. Each layer feeds the next.

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always enclose subqueries in parentheses** | Required syntax — SQL won't parse without them |
| **Alias derived tables in `FROM`** | `FROM (SELECT ...) AS alias` — required by SQL |
| **Keep subqueries focused** | Only select the columns you actually need |
| **Consider `JOIN` as an alternative** | For correlated subqueries on large tables, `JOIN + GROUP BY` is often faster |
| **Test subqueries independently** | Run the inner query alone first to verify it returns the expected result |
| **Use `=` when subquery returns one value** | `IN` is for sets; `=` is for a single value — use the right one |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting parentheses** — `WHERE col = SELECT ...` is a syntax error. Always wrap subqueries: `WHERE col = (SELECT ...)`.

> ⚠️ **Returning multiple columns in a single-value context** — `WHERE col = (SELECT col1, col2 FROM ...)` fails. A scalar subquery must return exactly one column.

> ⚠️ **Using `=` when the subquery returns multiple rows** — `WHERE col = (SELECT ...)` fails if the inner query returns more than one row. Use `IN` for multi-row results.

> ⚠️ **Using `IN` when only one value is returned** — `WHERE col IN (SELECT ...)` works, but `WHERE col = (SELECT ...)` is cleaner and more explicit when one value is guaranteed.

> ⚠️ **Not aliasing derived tables in `FROM`** — `FROM (SELECT ...) ` without an alias causes an error. Always add `AS tablename`.

> ⚠️ **Unoptimized correlated subqueries** — a correlated subquery in `SELECT` or `WHERE` that runs once per row can be very slow on millions of rows. Benchmark and consider rewrites with `JOIN`.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Subquery | A query nested inside another — always in parentheses |
| Non-aggregate | Compares field values; no aggregation |
| Aggregate | Uses `SUM`, `AVG`, `MIN`, `MAX`, `COUNT` on a group |
| In `WHERE` | Filters rows based on a calculated or compared value |
| In `SELECT` | Adds a derived column to each output row |
| In `FROM` | Creates a temporary derived table (must be aliased) |
| With `IN` | Filters rows matching any value in a result set |
| With `DISTINCT` | Returns only unique values from the subquery |
| Correlated | References outer query — runs per row; can be slow |
| Non-correlated | Independent of outer query — runs once; efficient |

---

## What's Next?

Now that you can nest queries, explore more advanced SQL techniques!

- 🪟 **Window functions for running totals and rankings**
- 🔄 **CTEs (Common Table Expressions) — a cleaner alternative to subqueries**
- 🔗 **Self-joins for hierarchical and recursive data**

> *Subqueries let you ask questions about questions — they're the building blocks of every complex SQL report!* 🚀

---

