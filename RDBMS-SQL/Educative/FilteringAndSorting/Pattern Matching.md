# Pattern Matching
> *Learn about pattern matching and wildcards in SQL.*

---

## Introduction

Imagine we need to find all products containing the word "book" in their name to run a promotional discount. Searching by exact matches won't work if we need to handle variations in spelling, partial terms, or different letter cases. This is where **pattern matching** becomes valuable.

Pattern matching in SQL allows us to find records that follow a defined pattern — particularly useful when searching for similar attributes like names starting with a certain letter or products containing specific keywords. It lets us retrieve rows that match **partial strings** instead of requiring exact matches — essential when data doesn't follow a strict or uniform naming convention.

### 🎯 Learning Goals

- ✅ Understand the purpose and importance of pattern matching in SQL
- ✅ Learn how to perform pattern matching with the `LIKE` operator
- ✅ Learn how to use wildcards for pattern matching

---

## The `LIKE` Operator

When working with user-generated content or inconsistently named products, we can't rely solely on equality operators (`=`). If a customer's search term might appear anywhere in a product name, `LIKE` helps us find it without knowing the exact string.

### Syntax

```sql
SELECT Column1, Column2, ...
FROM TableName
WHERE ColumnName LIKE 'pattern';
```

The `pattern` is built using **wildcards** — special characters that represent unknown or variable parts of a string.

---

## Wildcards

| Wildcard | Represents | Example Pattern | Matches |
|---|---|---|---|
| `%` | Zero, one, or more characters | `'Jo%'` | `John`, `Joanna`, `Jo` |
| `_` | Exactly one character | `'J_n'` | `Jon`, `Jan`, `Jin` |

### `%` — Percent Wildcard

Matches **any number of characters** (including none) at the position it is placed.

| Pattern | Meaning | Example Matches |
|---|---|---|
| `'Jo%'` | Starts with "Jo" | `John`, `Joanna`, `Jordan` |
| `'%book%'` | Contains "book" anywhere | `Notebook`, `Bookshelf`, `E-book Reader` |
| `'%set'` | Ends with "set" | `Dumbbell Set`, `Knife Set` |
| `'%smart%'` | Contains "smart" anywhere | `Smartphone`, `Smart TV`, `Smartwatch` |

### `_` — Underscore Wildcard

Matches **exactly one character** at the position it is placed.

| Pattern | Meaning | Example Matches |
|---|---|---|
| `'J_n'` | 3-letter word starting with J, ending with n | `Jon`, `Jan`, `Jin` |
| `'_at'` | 3-letter word ending with "at" | `Cat`, `Bat`, `Hat` |
| `'Sm___phone'` | Word where `___` = exactly 3 chars | `Smartphone` |
| `'___-___-____'` | Phone number format | `413-456-6862` |

---

## Using `LIKE` — Examples

### Example 1 — Match a Word Anywhere in a String

Find all customers living in **Greenwood** (city can appear anywhere in the address):

```sql
SELECT CustomerName, Address
FROM Customers
WHERE Address LIKE '%Greenwood%';
```

#### Sample Output

| CustomerName | Address |
|---|---|
| Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| Emma Watson | 303 Birch Road, Greenwood, 29646 |

> 💡 `'%Greenwood%'` uses `%` on both sides — "Greenwood" can appear at any position in the address. If the city always appears at the end, `'%Greenwood'` (no trailing `%`) is more specific and slightly faster.

---

### Example 2 — Match the Start of a String

Find all customers whose names start with **"Jo"**:

```sql
SELECT CustomerName, Email
FROM Customers
WHERE CustomerName LIKE 'Jo%';
```

#### Sample Output

| CustomerName | Email |
|---|---|
| John Doe | john.doe@smail.com |
| Joanna Peters | joanna.p@mail.com |

---

### Example 3 — Match the End of a String

Find all products whose names end with **"Set"**:

```sql
SELECT ProductName, Price
FROM Products
WHERE ProductName LIKE '%Set';
```

#### Sample Output

| ProductName | Price |
|---|---|
| Dumbbell Set | 82.50 |
| Knife Set | 50.00 |
| Ballpoint Pen Set | 5.00 |
| Cookware Set | 129.99 |

---

### Example 4 — Using the Underscore Wildcard

Find all customers whose names are exactly **3 letters starting with "J" and ending with "n"**:

```sql
SELECT CustomerName
FROM Customers
WHERE CustomerName LIKE 'J_n';
```

> 💡 This is useful for validating data format — e.g., confirming that country codes are exactly 2 characters: `WHERE CountryCode LIKE '__'`.

---

### Example 5 — Match by Email Domain

Find all customers using a **@jmail.com** email address:

```sql
SELECT CustomerName, Email
FROM Customers
WHERE Email LIKE '%@jmail.com';
```

#### Sample Output

| CustomerName | Email |
|---|---|
| Alice Johnson | alice.j@jmail.com |
| Lucas Anderson | lucas.anderson@jmail.com |

---

## Combining `LIKE` with Logical Operators

`LIKE` can be combined with `AND`, `OR`, and `NOT` to build more complex patterns.

### Example — AND: Match Two Patterns Simultaneously

Find customers who live in **Greenwood** and have a **@service.org** email:

```sql
SELECT CustomerName, Address, Email
FROM Customers
WHERE Address LIKE '%Greenwood%'
  AND Email LIKE '%service.org%';
```

#### Sample Output

| CustomerName | Address | Email |
|---|---|---|
| Emma Watson | 303 Birch Road, Greenwood, 29646 | emma.w@service.org |

---

### Example — OR: Match Either Pattern

Find customers living in **Greenwood** or **Springfield**:

```sql
SELECT CustomerName, Address
FROM Customers
WHERE Address LIKE '%Greenwood%'
   OR Address LIKE '%Springfield%';
```

#### Sample Output

| CustomerName | Address |
|---|---|
| John Doe | 123 Elm Street, Springfield, 01103 |
| Alice Johnson | 789 Oak Lane, Greenwood, 46142 |
| Emma Watson | 303 Birch Road, Greenwood, 29646 |

---

### Example — NOT LIKE: Exclude a Pattern

Find all products that do **not** contain "Smart" in their name:

```sql
SELECT ProductName, Price
FROM Products
WHERE ProductName NOT LIKE '%Smart%';
```

> ✅ `NOT LIKE` is the clean way to exclude rows matching a pattern — the exact opposite of `LIKE`.

---

## Wildcard Placement — How Position Affects Results

The position of `%` significantly changes what gets matched:

```sql
-- Starts with "Smart"
WHERE ProductName LIKE 'Smart%'
-- Matches: Smartphone, Smart TV, Smartwatch, Smart Outdoor Light

-- Ends with "Set"
WHERE ProductName LIKE '%Set'
-- Matches: Dumbbell Set, Knife Set, Cookware Set

-- Contains "smart" anywhere (case-insensitive in MySQL)
WHERE ProductName LIKE '%smart%'
-- Matches: Smartphone, Smart TV, Smart Outdoor Light, Smartwatch

-- Exact match (no wildcards — same as =)
WHERE ProductName LIKE 'Laptop'
-- Matches: Laptop (only)
```

> ⚠️ Without `%` or `_`, `LIKE` behaves exactly like `=` — it only matches the exact string.

---

## `LIKE` vs. `=` — Key Difference

| Operator | Requires Exact Match? | Supports Wildcards? | Use When... |
|---|---|---|---|
| `=` | ✅ Yes | ❌ No | You know the full, exact value |
| `LIKE` | ❌ No | ✅ Yes | You need partial or pattern-based matching |

---

## 🧩 Code Exercises

---

### Task 1 — Match Phone Numbers by Prefix

**Find all customers with phone numbers starting with "708".**

```sql
SELECT CustomerName, Phone
FROM Customers
WHERE Phone LIKE '708%';
```

#### Expected Output

| CustomerName | Phone |
|---|---|
| Jane Smith | 708-567-5234 |
| Emily Davis | 708-345-4980 |
| Olivia Thompson | 708-123-1561 |
| Lucas Anderson | 708-345-9809 |

> 💡 `'708%'` anchors the match to the start — only phone numbers **beginning with** 708 are returned.

---

### Task 2 — Match a Substring in Product Names

**Find all products that contain the word "smart" anywhere in their name.**

```sql
SELECT ProductID, ProductName, Price, Stock
FROM Products
WHERE ProductName LIKE '%smart%';
```

#### Expected Output

| ProductID | ProductName | Price | Stock |
|---|---|---|---|
| 1 | Smartphone | 635.79 | 50 |
| 4 | Smart Outdoor Light | 75.62 | 25 |
| 11 | Smart Kitchen Scale | 75.65 | 40 |
| 16 | Smart TV | 2599.99 | 20 |
| 19 | Smartwatch | 500.00 | 50 |
| 20 | Smart Fitness Band | 350.99 | 60 |

> 💡 MySQL's `LIKE` is **case-insensitive** by default — `'%smart%'` matches `Smart`, `SMART`, and `smart` equally.

---

## Pattern Quick Reference

| Goal | Pattern | Example |
|---|---|---|
| Starts with "Jo" | `'Jo%'` | `John`, `Joanna` |
| Ends with "Set" | `'%Set'` | `Knife Set`, `Dumbbell Set` |
| Contains "smart" | `'%smart%'` | `Smartphone`, `Smart TV` |
| Exactly 3 characters | `'___'` | `Jon`, `Jan`, `USB` |
| Starts with "J", 3 chars, ends with "n" | `'J_n'` | `Jon`, `Jan` |
| Specific domain | `'%@jmail.com'` | `alice.j@jmail.com` |
| Exclude a pattern | `NOT LIKE '%smart%'` | Everything except Smart products |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Avoid overly generic patterns** | `'%a%'` matches almost everything — slow and noisy |
| **Anchor patterns when possible** | `'Smart%'` is faster than `'%Smart%'` on large tables |
| **Use `NOT LIKE` to exclude patterns** | Cleaner than complex `WHERE` workarounds |
| **Sanitize user inputs used in patterns** | Prevents SQL injection in dynamic queries |
| **Test patterns on sample data first** | Ensures the pattern captures exactly what you intend |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting wildcards** — `WHERE ProductName LIKE 'Smart'` is the same as `= 'Smart'`. Add `%` for partial matching.

> ⚠️ **Assuming case sensitivity** — MySQL `LIKE` is case-insensitive by default, but some databases (e.g., PostgreSQL) are case-sensitive. Use `ILIKE` in PostgreSQL for case-insensitive matching.

> ⚠️ **Leading wildcards on large tables** — `'%term%'` cannot use an index and scans every row. Use sparingly on large datasets.

> ⚠️ **Overly broad patterns** — `'%e%'` will match almost every name. Be as specific as possible.

> ⚠️ **Confusing `_` and `%`** — `_` is exactly one character; `%` is any number. `'Jo_'` won't match `John` but `'Jo%'` will.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `LIKE` | Matches partial strings using patterns |
| `%` wildcard | Matches zero or more characters |
| `_` wildcard | Matches exactly one character |
| `NOT LIKE` | Excludes rows matching a pattern |
| Combined with `AND` / `OR` | Builds complex multi-pattern filters |
| No wildcards | `LIKE` behaves identically to `=` |

---

## What's Next?

Now that you can match patterns, explore more powerful filtering tools!

- 📋 **Filtering lists with `IN` and `BETWEEN`**
- 📊 **Sorting results with `ORDER BY`**
- 🔢 **Aggregating data with `COUNT`, `SUM`, `AVG`**
- 🔗 **Combining tables with `JOIN`**

> *Pattern matching turns vague searches into precise queries — master wildcards and your data always stays within reach!* 🚀

---

