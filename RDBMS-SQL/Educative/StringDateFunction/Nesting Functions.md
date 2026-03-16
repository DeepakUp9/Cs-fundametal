# Nesting Functions
> *Learn to apply nesting of SQL functions for effective query writing.*

---

## Introduction

Imagine creating a promotional message that includes a product's name formatted in uppercase and the month name of its most recent order date — all in one query. We'd need to combine string functions like `UPPER` and `SUBSTRING` with date functions like `MONTHNAME`. **Nesting functions** makes this possible, letting us produce clean, complex results without intermediate steps.

### 🎯 Learning Goals

- ✅ Understand what nesting functions means in SQL
- ✅ Recognize when and why to combine multiple SQL functions in a single query
- ✅ Learn best practices for writing nested function queries clearly and effectively

---

## What Is Nesting Functions?

**Nesting functions** means using one function's output as the **input** to another function within the same SQL statement.

```sql
-- Basic nesting: UPPER applied to the result of SUBSTRING
UPPER(SUBSTRING(ProductName, 1, 3))
--    └─── inner function runs first ──┘
-- └─── outer function receives the result ──┘
```

The **inner function** always executes first. Its result is passed directly as an argument to the **outer function**. You can nest as many levels deep as needed — though keeping it readable is important.

### Why Nesting Matters

| Benefit | Description |
|---|---|
| 🔁 **Fewer queries** | Multiple transformations in one statement |
| 🧹 **No temp tables** | No need for intermediate steps or stored results |
| 📐 **Concise code** | Express complex logic in a single expression |
| 🔧 **Flexible output** | Combine string, date, and numeric functions freely |

---

## How Nesting Works — Step by Step

Each nested function evaluates from the **innermost** outward:

```sql
CONCAT('Placed in ', MONTHNAME(OrderDate), ' of ', YEAR(OrderDate))
```

**Execution order:**
1. `MONTHNAME(OrderDate)` → `'November'`
2. `YEAR(OrderDate)` → `2024`
3. `CONCAT('Placed in ', 'November', ' of ', 2024)` → `'Placed in November of 2024'`

---

## Examples of Nesting Functions

### Example 1 — Uppercase Extracted Text

Get the first 3 letters of a product name and convert them to uppercase:

```sql
SELECT ProductName,
       UPPER(SUBSTRING(ProductName, 1, 3)) AS ShortCodeUpper
FROM Products;
```

#### Sample Output

| ProductName | ShortCodeUpper |
|---|---|
| Smartphone | SMA |
| Laptop | LAP |
| Bluetooth Speaker | BLU |
| Dumbbell Set | DUM |

---

### Example 2 — Formatted Order Summary (String + Date Functions)

Combine `MONTHNAME`, `YEAR`, `SUBSTRING`, `UPPER`, and `CONCAT` to build a descriptive order label:

```sql
SELECT OrderID,
       CONCAT(
           'Placed in ', MONTHNAME(OrderDate),
           ' of ', YEAR(OrderDate),
           ' by ', UPPER(SUBSTRING(CustomerName, 1, 3))
       ) AS OrderDetails
FROM Orders
JOIN Customers ON Orders.CustomerID = Customers.CustomerID;
```

#### How Each Function Contributes

| Function | Input | Output | Role |
|---|---|---|---|
| `MONTHNAME(OrderDate)` | `2024-05-22` | `May` | Extracts month name |
| `YEAR(OrderDate)` | `2024-05-22` | `2024` | Extracts year |
| `SUBSTRING(CustomerName, 1, 3)` | `John Doe` | `Joh` | Gets first 3 chars |
| `UPPER(...)` | `Joh` | `JOH` | Uppercases the result |
| `CONCAT(...)` | All above | `Placed in May of 2024 by JOH` | Merges everything |

#### Sample Output

| OrderID | OrderDetails |
|---|---|
| 1 | Placed in March of 2024 by CHA |
| 2 | Placed in May of 2024 by JOH |
| 3 | Placed in July of 2024 by JAN |
| 4 | Placed in August of 2024 by MIC |

---

### Example 3 — Title Case from Scratch

Convert a product name to title case (first letter uppercase, rest lowercase) using nested `CONCAT`, `UPPER`, `LOWER`, and `SUBSTRING`:

```sql
SELECT ProductName,
       CONCAT(
           UPPER(SUBSTRING(ProductName, 1, 1)),
           LOWER(SUBSTRING(ProductName, 2))
       ) AS TitleCase
FROM Products;
```

#### Sample Output

| ProductName | TitleCase |
|---|---|
| LAPTOP | Laptop |
| bluetooth speaker | Bluetooth speaker |
| DUMBBELL SET | Dumbbell set |

---

### Example 4 — Domain Extraction (LOCATE + SUBSTRING)

Combine `LOCATE` and `SUBSTRING` to extract the email domain:

```sql
SELECT Email,
       SUBSTRING(Email, LOCATE('@', Email) + 1) AS Domain
FROM Customers;
```

#### How It Works

1. `LOCATE('@', Email)` → finds position of `@` (e.g., position `5`)
2. `+ 1` → moves past the `@` to position `6`
3. `SUBSTRING(Email, 6)` → returns everything from position 6 onwards

---

### Example 5 — Formatted Price Label

Combine `CONCAT`, `UPPER`, `SUBSTRING`, and column values to build a product label:

```sql
SELECT CONCAT(
           UPPER(SUBSTRING(ProductName, 1, 3)),
           ' | ',
           ProductName,
           ' | $',
           Price
       ) AS ProductLabel
FROM Products;
```

#### Sample Output

| ProductLabel |
|---|
| SMA \| Smartphone \| $635.79 |
| LAP \| Laptop \| $1099.99 |
| DUM \| Dumbbell Set \| $82.50 |

---

## Nesting Depth — What to Watch For

When nesting goes 3+ levels deep, formatting becomes critical:

```sql
-- Hard to read (all on one line)
SELECT CONCAT(UPPER(SUBSTRING(CustomerName,1,1)),LOWER(SUBSTRING(CustomerName,2))) AS Name FROM Customers;

-- Easy to read (properly indented)
SELECT CONCAT(
           UPPER(SUBSTRING(CustomerName, 1, 1)),
           LOWER(SUBSTRING(CustomerName, 2))
       ) AS ProperCaseName
FROM Customers;
```

> ✅ **Indent inner functions** to visually show the nesting hierarchy. Use consistent spacing and always alias the result.

---

## 🧩 Code Exercises

---

### Task 1 — Format Product Name with Two-Part Code

**Return each product's `ProductID` and a `FormattedName` built from the first 2 letters in uppercase, a dash, and the remaining letters in lowercase.**

```sql
SELECT ProductID,
       CONCAT(
           UPPER(SUBSTRING(ProductName, 1, 2)),
           '-',
           LOWER(SUBSTRING(ProductName, 3, CHAR_LENGTH(ProductName) - 2))
       ) AS FormattedName
FROM Products;
```

#### Execution Breakdown

| Step | Function | Input | Output |
|---|---|---|---|
| 1 | `SUBSTRING(ProductName, 1, 2)` | `Bluetooth Speaker` | `Bl` |
| 2 | `UPPER(...)` | `Bl` | `BL` |
| 3 | `CHAR_LENGTH(ProductName) - 2` | `18 - 2` | `16` |
| 4 | `SUBSTRING(ProductName, 3, 16)` | `Bluetooth Speaker` | `uetooth Speaker` |
| 5 | `LOWER(...)` | `uetooth Speaker` | `uetooth speaker` |
| 6 | `CONCAT(...)` | `BL`, `-`, `uetooth speaker` | `BL-uetooth speaker` |

#### Expected Output (sample)

| ProductID | FormattedName |
|---|---|
| 6 | BA-llpoint pen set |
| 8 | BL-ender |
| 7 | BL-uetooth speaker |
| … | … |
| 13 | TE-nt |
| 10 | TR-eadmill |
| 12 | YO-ga mat |

---

### Task 2 — Order Summary with Year and Amount

**Display each order's ID alongside a `OrderSummary` showing the year in brackets followed by the total amount with a dollar sign.**

```sql
SELECT OrderID,
       CONCAT(
           '[', YEAR(OrderDate), '] ',
           '$', TotalAmount
       ) AS OrderSummary
FROM Orders;
```

#### Expected Output (sample)

| OrderID | OrderSummary |
|---|---|
| 1 | [2024] $1286.58 |
| 2 | [2024] $235.50 |
| 3 | [2024] $2639.99 |
| … | … |
| 8 | [2024] $229.99 |
| 9 | [2024] $1701.00 |
| 10 | [2024] $145.25 |

---

### Task 3 — Proper Case Customer Names

**Display customer names with the first character uppercase and the rest lowercase.**

```sql
SELECT CONCAT(
           UPPER(SUBSTRING(CustomerName, 1, 1)),
           LOWER(SUBSTRING(CustomerName, 2))
       ) AS Proper_Case_Name
FROM Customers;
```

#### Expected Output (sample)

| Proper_Case_Name |
|---|
| John doe |
| Jane smith |
| Alice johnson |
| … |
| Charlotte miller |
| Lucas anderson |
| Emma watson |

> 💡 `SUBSTRING(CustomerName, 2)` without a length argument extracts from position 2 all the way to the end of the string.

---

## Nesting Functions — Pattern Library

Here are the most common nested function patterns you'll use:

| Pattern | Code | Use Case |
|---|---|---|
| Uppercase first N chars | `UPPER(SUBSTRING(col, 1, n))` | Short codes, prefixes |
| Title case | `CONCAT(UPPER(SUBSTRING(col,1,1)), LOWER(SUBSTRING(col,2)))` | Display names |
| Two-part formatted name | `CONCAT(UPPER(SUBSTRING(col,1,2)), '-', LOWER(SUBSTRING(col,3)))` | Product codes |
| Month + Year label | `CONCAT(MONTHNAME(date), ' ', YEAR(date))` | Report headers |
| Email domain | `SUBSTRING(email, LOCATE('@', email) + 1)` | Domain extraction |
| Order label | `CONCAT('[', YEAR(date), '] $', amount)` | Summary strings |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Test inner functions first** | Run `SELECT SUBSTRING(col, 1, 3)` alone before nesting inside `UPPER` |
| **Use proper indentation** | Makes nesting depth clear and queries easier to debug |
| **Always alias nested expressions** | `AS FormattedName` is essential — otherwise the header shows the raw expression |
| **Keep nesting to 3–4 levels max** | Deeper nesting is harder to debug; consider breaking it into steps or a view |
| **Validate edge cases** | Test with `NULL` values and very short strings to catch unexpected results |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Mismatched parentheses** — each function needs its own opening and closing parenthesis. `UPPER(SUBSTRING(col, 1, 3)` missing the closing `)` will cause a syntax error.

> ⚠️ **Forgetting evaluation order** — the **innermost** function always runs first. If you expect `UPPER` to run before `SUBSTRING`, you'll get unexpected results.

> ⚠️ **No alias on complex expressions** — without `AS`, the column header displays the entire nested expression, making results unreadable.

> ⚠️ **Not testing pieces independently** — nested functions are hard to debug all at once. Always test the innermost function alone first, then build outward.

> ⚠️ **Ignoring `NULL` inputs** — if any input column is `NULL`, most functions return `NULL`. Use `COALESCE(col, '')` to guard against this.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| Nesting functions | Using one function's output as another's input |
| Execution order | Innermost function runs first, then outward |
| Common combinations | `UPPER(SUBSTRING(...))`, `CONCAT(MONTHNAME(...), YEAR(...))` |
| Readability | Indent nested expressions and always alias results |
| Testing strategy | Build from inside out — test each layer before adding the next |

---

## What's Next?

Now that you can combine functions powerfully, explore more advanced SQL capabilities!

- 🔄 **Conditional logic with `CASE` expressions**
- 🪟 **Window functions for rankings and running totals**
- 🔄 **Common Table Expressions (CTEs)**
- ⚡ **Query optimization and indexing**

> *Nesting functions is like building sentences — each word (function) adds meaning, and together they form a complete, powerful expression!* 🚀

---

