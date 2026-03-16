# Joining Tables
> *Learn about different types of joins in SQL.*

---

## Introduction

Imagine generating a report of our online store showing customer names alongside the products they ordered. Customer details live in the `Customers` table and product details in the `Products` table — we need a way to **combine data from multiple tables** into a single result. This is exactly what **SQL joins** do.

### 🎯 Learning Goals

- ✅ Understand what joins are and why they are important in SQL
- ✅ Learn the four types of SQL joins: `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, and `FULL JOIN`
- ✅ Perform basic and multiple-table joins using the course database

---

## Why Joins Are Essential

Joins link tables through a **shared column** — typically a primary key / foreign key relationship:

- The **primary key** uniquely identifies each record in a table
- The **foreign key** references that primary key in another table, creating a relationship

Without joins, we would need to store all data in one giant table — inefficient, redundant, and against the principles of relational databases. Joins let us keep data organized in separate, focused tables and **combine it only when needed**.

### Example: The Shared Column

In the `OnlineStore` database:

```
Customers table          Orders table
┌────────────┬────────┐  ┌─────────┬────────────┬───────────┐
│ CustomerID │  Name  │  │ OrderID │ CustomerID │ OrderDate │
├────────────┼────────┤  ├─────────┼────────────┼───────────┤
│     1      │  John  │  │    1    │     3      │ 2024-03   │
│     2      │  Jane  │◄─┼────────┼─────2──────┤ 2024-05   │
│     3      │ Alice  │  │    3    │     1      │ 2024-07   │
└────────────┴────────┘  └─────────┴────────────┴───────────┘
                  ▲                       │
                  └───── JOIN ON ─────────┘
                     CustomerID = CustomerID
```

`CustomerID` exists in both tables. Using it as the join condition, we can combine order data with customer names.

---

## Types of SQL Joins

| Join Type | Returns |
|---|---|
| `INNER JOIN` | Only rows with a match in **both** tables |
| `LEFT JOIN` | All rows from the **left** table + matching rows from right (NULL if no match) |
| `RIGHT JOIN` | All rows from the **right** table + matching rows from left (NULL if no match) |
| `FULL JOIN` | All rows from **both** tables (NULL where no match exists on either side) |

---

## 1️⃣ `INNER JOIN` — Matching Rows Only

`INNER JOIN` returns only rows that have a **matching value in both tables**. Non-matching rows from either table are excluded.

### Syntax

```sql
SELECT Column1, ...
FROM Table1
INNER JOIN Table2
ON Table1.ColumnName = Table2.ColumnName;
```

### Visual

```
Customers:  [1] [2] [3] [4] [5]
Orders:         [2] [3] [4]

INNER JOIN result: [2] [3] [4]
(only the overlapping records)
```

### Example — Orders with Customer Names

```sql
SELECT o.OrderID, c.CustomerName, o.OrderDate, o.TotalAmount
FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.CustomerID;
```

#### Sample Output

| OrderID | CustomerName | OrderDate | TotalAmount |
|---|---|---|---|
| 1 | Charlie Davis | 2024-03-15 | 1286.58 |
| 2 | John Doe | 2024-05-22 | 235.50 |
| 3 | Jane Smith | 2024-07-03 | 2639.99 |

> 💡 Any order without a matching `CustomerID` in `Customers` is **excluded** from the result. Any customer who never placed an order is also excluded.

---

## 2️⃣ `LEFT JOIN` — All Left Rows + Matching Right Rows

`LEFT JOIN` returns **all rows from the left (first) table**, plus matching rows from the right table. If no match exists in the right table, those columns are shown as `NULL`.

### Syntax

```sql
SELECT Column1, ...
FROM Table1
LEFT JOIN Table2
ON Table1.ColumnName = Table2.ColumnName;
```

### Visual

```
Customers:  [1] [2] [3] [4] [5]
Orders:         [2] [3] [4]

LEFT JOIN result: [1] [2] [3] [4] [5]
(all customers; NULL for those without orders)
```

### Example — All Orders Including Unmatched Customers

```sql
SELECT o.OrderID, c.CustomerName, o.OrderDate, o.TotalAmount
FROM Orders o
LEFT JOIN Customers c ON o.CustomerID = c.CustomerID;
```

> 💡 If an order has no matching `CustomerID` in `Customers`, the customer columns (`CustomerName`, etc.) will show `NULL`.

### Example — All Customers, Even Those Without Orders

```sql
SELECT c.CustomerName, o.OrderID, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID;
```

#### Sample Output

| CustomerName | OrderID | TotalAmount |
|---|---|---|
| John Doe | 2 | 235.50 |
| Jane Smith | 3 | 2639.99 |
| Alice Johnson | 7 | 156.24 |
| Michael Carter | NULL | NULL |
| Charlie Davis | 1 | 1286.58 |

> ✅ Michael Carter has no orders — his row still appears but with `NULL` in the order columns.

---

## 3️⃣ `RIGHT JOIN` — All Right Rows + Matching Left Rows

`RIGHT JOIN` is the **mirror image of `LEFT JOIN`**. It returns all rows from the right (second) table, plus matching rows from the left. If no match exists in the left table, those columns show `NULL`.

### Syntax

```sql
SELECT Column1, ...
FROM Table1
RIGHT JOIN Table2
ON Table1.ColumnName = Table2.ColumnName;
```

### Visual

```
Orders:         [2] [3] [4]
Customers:  [1] [2] [3] [4] [5]

RIGHT JOIN result: [1] [2] [3] [4] [5]
(all customers; NULL for those without orders)
```

### Example — All Customers and Their Orders (if any)

```sql
SELECT c.CustomerName, o.OrderID, o.OrderDate, o.TotalAmount
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerID = c.CustomerID;
```

#### Sample Output

| CustomerName | OrderID | OrderDate | TotalAmount |
|---|---|---|---|
| John Doe | 2 | 2024-05-22 | 235.50 |
| Jane Smith | 3 | 2024-07-03 | 2639.99 |
| Michael Carter | NULL | NULL | NULL |
| Emma Watson | NULL | NULL | NULL |

> 💡 Customers who have never placed an order still appear — with `NULL` for all order columns.

### `LEFT JOIN` vs `RIGHT JOIN`

Most queries can be written with either join — it just depends on which table you put first:

```sql
-- These two queries produce identical results:

-- Option 1: RIGHT JOIN
SELECT c.CustomerName, o.OrderID
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerID = c.CustomerID;

-- Option 2: LEFT JOIN (swap table order)
SELECT c.CustomerName, o.OrderID
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID;
```

> ✅ Most developers prefer `LEFT JOIN` by convention — it's easier to read as "start with this table, then add from that one."

---

## 4️⃣ `FULL JOIN` — All Rows from Both Tables

`FULL JOIN` (also called `FULL OUTER JOIN`) returns **all rows from both tables**, matching them where possible and filling in `NULL` where no match exists.

### Syntax

```sql
SELECT Column1, ...
FROM Table1
FULL JOIN Table2
ON Table1.ColumnName = Table2.ColumnName;
```

### Visual

```
Customers:  [1] [2] [3] [4] [5]
Orders:     [1]     [3]     [5] [6] [7]

FULL JOIN: [1] [2] [3] [4] [5] [6] [7]
(everything from both sides; NULL where unmatched)
```

### Example

```sql
SELECT c.CustomerName, o.OrderID, o.OrderDate, o.TotalAmount
FROM Orders o
FULL JOIN Customers c ON o.CustomerID = c.CustomerID;
```

> ⚠️ **MySQL does not natively support `FULL OUTER JOIN`.** It is supported in PostgreSQL, SQL Server, and Oracle. In MySQL, simulate it by combining `LEFT JOIN` and `RIGHT JOIN` with `UNION`:

```sql
-- MySQL equivalent of FULL JOIN
SELECT c.CustomerName, o.OrderID, o.TotalAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID

UNION

SELECT c.CustomerName, o.OrderID, o.TotalAmount
FROM Customers c
RIGHT JOIN Orders o ON c.CustomerID = o.CustomerID;
```

---

## 5️⃣ Joining Multiple Tables

We can chain multiple `JOIN` clauses to combine more than two tables. SQL processes them one at a time from left to right.

### Example — 4-Table Join: Orders + Customers + Order_Details + Products

```sql
SELECT o.OrderID,
       c.CustomerName,
       p.ProductName,
       od.Quantity,
       o.OrderDate
FROM Orders o
INNER JOIN Customers c     ON o.CustomerID  = c.CustomerID
INNER JOIN Order_Details od ON o.OrderID    = od.OrderID
INNER JOIN Products p      ON od.ProductID  = p.ProductID;
```

#### How It Works

| Line | What It Does |
|---|---|
| `FROM Orders o` | Start with the Orders table |
| `JOIN Customers c` | Add matching customer for each order |
| `JOIN Order_Details od` | Add each product line within each order |
| `JOIN Products p` | Add product name and details for each line item |

#### Sample Output

| OrderID | CustomerName | ProductName | Quantity | OrderDate |
|---|---|---|---|---|
| 1 | Charlie Davis | Smartphone | 2 | 2024-03-15 |
| 1 | Charlie Davis | Ballpoint Pen Set | 3 | 2024-03-15 |
| 2 | John Doe | Bluetooth Speaker | 1 | 2024-05-22 |
| 2 | John Doe | Knife Set | 2 | 2024-05-22 |

---

## All Join Types — Visual Summary

```
Table A:  [ 1 ][ 2 ][ 3 ][ 4 ]
Table B:       [ 2 ][ 3 ][ 4 ][ 5 ]

INNER JOIN:        [ 2 ][ 3 ][ 4 ]
LEFT JOIN:  [ 1 ][ 2 ][ 3 ][ 4 ]
RIGHT JOIN:        [ 2 ][ 3 ][ 4 ][ 5 ]
FULL JOIN:  [ 1 ][ 2 ][ 3 ][ 4 ][ 5 ]
```

---

## 🧩 Code Exercises

---

### Task 1 — All Customers and Their Orders (Including No Orders)

**Retrieve all customers and their orders, including customers who haven't placed any orders.**

```sql
SELECT Customers.CustomerName, Orders.OrderID, Orders.TotalAmount
FROM Customers
LEFT JOIN Orders ON Customers.CustomerID = Orders.CustomerID;
```

#### Expected Output (sample)

| CustomerName | OrderID | TotalAmount |
|---|---|---|
| John Doe | NULL | NULL |
| Jane Smith | NULL | NULL |
| Alice Johnson | 7 | 156.24 |
| … | … | … |
| Olivia Thompson | 6 | 1590.69 |
| James Walker | 10 | 751.50 |
| Grace Taylor | 4 | 2550.50 |
| … | … | … |
| Emma Watson | NULL | NULL |

> 💡 `LEFT JOIN` ensures all customers appear. Those without orders show `NULL` for `OrderID` and `TotalAmount`.

---

### Task 2 — Products with Their Categories

**Show all product names and their categories. If a product has no matching category, still display the product name.**

```sql
SELECT p.ProductName, c.CategoryName
FROM Products p
LEFT JOIN Categories c ON p.CategoryID = c.CategoryID;
```

#### Expected Output (sample)

| ProductName | CategoryName |
|---|---|
| Smartphone | Electronics |
| Dumbbell Set | Fitness |
| Laptop | Electronics |
| Smart Outdoor Light | Outdoor |
| … | … |
| Smartwatch | Electronics |
| Smart Fitness Band | Fitness |

---

### Task 3 — Suppliers and the Products They Supply

**List suppliers and the products they supply. Include suppliers who don't currently supply any products.**

```sql
SELECT s.SupplierName, p.ProductName
FROM Suppliers s
LEFT JOIN Product_Suppliers ps ON s.SupplierID = ps.SupplierID
LEFT JOIN Products p ON ps.ProductID = p.ProductID;
```

#### Expected Output (sample)

| SupplierName | ProductName |
|---|---|
| TechSolutions | Smartphone |
| TechSolutions | Laptop |
| TechSolutions | Smart TV |
| … | … |
| SportyGear | Smart Outdoor Light |
| SportyGear | Tent |
| … | … |
| TechNexus | Bluetooth Speaker |
| GourmetKitchen | Notebook |

> 💡 This is a **chained LEFT JOIN** — we first join `Suppliers` to the bridge table `Product_Suppliers`, then join that to `Products`. Any supplier not present in `Product_Suppliers` will still appear with `NULL` for `ProductName`.

---

## Join Quick Reference

| Join Type | Syntax | Returns | Use When |
|---|---|---|---|
| `INNER JOIN` | `FROM A INNER JOIN B ON ...` | Only matched rows | You need rows with data from both sides |
| `LEFT JOIN` | `FROM A LEFT JOIN B ON ...` | All of A + matched B | You need all left rows, even without a match |
| `RIGHT JOIN` | `FROM A RIGHT JOIN B ON ...` | All of B + matched A | You need all right rows, even without a match |
| `FULL JOIN` | `FROM A FULL JOIN B ON ...` | All rows from both | You need everything from both sides |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Always use explicit `ON` conditions** | Avoids accidental cartesian products (every row × every row) |
| **Use table aliases** | `Orders o` and `Customers c` keep multi-join queries readable |
| **Start with two tables and test first** | Add more joins incrementally to catch errors early |
| **Filter with `WHERE` after joining** | Reduces the final result set without affecting the join logic |
| **Choose the right join type** | Using `INNER JOIN` when `LEFT JOIN` is needed silently drops valid rows |
| **Qualify ambiguous column names** | `o.CustomerID` not just `CustomerID` when multiple tables share a name |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Listing multiple tables in `FROM` without `JOIN`** — `FROM Orders, Customers` without a join condition creates a **cartesian product** (every order paired with every customer — thousands of nonsensical rows).

> ⚠️ **Confusing `LEFT JOIN` and `RIGHT JOIN` direction** — the "left" table is the one in `FROM`; the "right" is the one after `JOIN`. Swap them if needed or just rewrite as a `LEFT JOIN` with the tables reversed.

> ⚠️ **Using ambiguous column names** — if both tables have a `CustomerID` column, write `o.CustomerID` and `c.CustomerID` to avoid errors.

> ⚠️ **Choosing `INNER JOIN` when `LEFT JOIN` is needed** — this silently drops rows (e.g., customers without orders disappear). Always consider whether you need to preserve non-matching rows.

> ⚠️ **Joining on unrelated columns** — always verify that the columns used in the `ON` condition have a real logical relationship (ideally a primary key / foreign key pair).

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Joins | Combine data from multiple tables via a shared column |
| `INNER JOIN` | Only rows matching in both tables |
| `LEFT JOIN` | All rows from left + matching right (NULL if no match) |
| `RIGHT JOIN` | All rows from right + matching left (NULL if no match) |
| `FULL JOIN` | All rows from both (NULL wherever no match); not in MySQL natively |
| Multiple joins | Chain `JOIN` clauses to combine 3+ tables |
| Aliases | Always use table aliases in multi-join queries for clarity |

---

## What's Next?

Now that you can join tables, unlock even more powerful data retrieval techniques!

- 📊 **Subqueries and nested `SELECT`**
- 🔎 **Self-joins for hierarchical data**
- 🔢 **Aggregating data across joined tables**
- 🪟 **Window functions for advanced analytics**

> *Joins are the superpower of relational databases — they let you ask questions that span the entire data model!* 🚀

---

