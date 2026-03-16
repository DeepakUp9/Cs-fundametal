# String Functions
> *Learn about string functions in SQL.*

---

## Introduction

Imagine creating a product catalog for an online store. Product names, categories, and descriptions often need formatting, concatenation, or modification. When building a customer report, we might need names in uppercase, or the first few letters of a product name to create a code. These tasks rely on **string functions** — tools that let us combine, modify, and analyze text fields, making data more meaningful and user-friendly.

### 🎯 Learning Goals

- ✅ Understand why string functions are important in real-world SQL scenarios
- ✅ Concatenate strings using `CONCAT`
- ✅ Extract parts of strings using `SUBSTRING`
- ✅ Replace text within strings using `REPLACE`
- ✅ Calculate the length of a string with `CHAR_LENGTH`
- ✅ Change text case with `UPPER` and `LOWER`
- ✅ Find the position of a substring with `LOCATE`

---

## String Functions at a Glance

| Function | Purpose | Example |
|---|---|---|
| `CONCAT` | Join multiple strings into one | `CONCAT('Hello', ' ', 'World')` → `Hello World` |
| `SUBSTRING` | Extract part of a string | `SUBSTRING('Smartphone', 1, 3)` → `Sma` |
| `REPLACE` | Swap all occurrences of a substring | `REPLACE('Street', 'Street', 'St')` → `St` |
| `CHAR_LENGTH` | Count characters in a string | `CHAR_LENGTH('Laptop')` → `6` |
| `UPPER` | Convert to uppercase | `UPPER('hello')` → `HELLO` |
| `LOWER` | Convert to lowercase | `LOWER('HELLO')` → `hello` |
| `LOCATE` | Find position of a substring | `LOCATE('@', 'user@mail.com')` → `5` |

---

## 1️⃣ `CONCAT` — Joining Strings

`CONCAT` joins two or more strings (or column values) into a single continuous string.

### Syntax

```sql
SELECT CONCAT(string1, string2, string3, ...) FROM TableName;
```

### Example — Customer Contact Info

Display each customer's name and email in one combined field:

```sql
SELECT CustomerName,
       Email,
       CONCAT(CustomerName, ' - ', Email) AS ContactInfo
FROM Customers;
```

#### Sample Output

| CustomerName | Email | ContactInfo |
|---|---|---|
| John Doe | john.doe@smail.com | John Doe - john.doe@smail.com |
| Jane Smith | jane.smith@inlook.com | Jane Smith - jane.smith@inlook.com |
| Alice Johnson | alice.j@jmail.com | Alice Johnson - alice.j@jmail.com |

> 💡 The separator `' - '` is just another string inside `CONCAT`. You can use any separator: a comma, a space, a slash — whatever makes sense.

### Example — Product Label with Price

```sql
SELECT CONCAT(ProductName, ' ($', Price, ')') AS ProductLabel
FROM Products;
```

#### Sample Output

| ProductLabel |
|---|
| Smartphone ($635.79) |
| Laptop ($1099.99) |
| Dumbbell Set ($82.50) |

> ⚠️ If **any** argument passed to `CONCAT` is `NULL`, the entire result is `NULL` in standard SQL. Use `COALESCE(column, '')` to substitute a default value for NULL fields before concatenating.

---

## 2️⃣ `SUBSTRING` — Extracting Part of a String

`SUBSTRING` extracts a specific portion of a string. You specify the starting position and optionally the length.

### Syntax

```sql
-- With length
SELECT SUBSTRING(string, start, length) FROM TableName;

-- Without length (extracts from start to end of string)
SELECT SUBSTRING(string, start) FROM TableName;
```

> ⚠️ **Positions start at 1**, not 0. `SUBSTRING('Smartphone', 1, 3)` returns `Sma`, not `mar`.

### Example — Three-Letter Product Code

```sql
SELECT ProductName,
       SUBSTRING(ProductName, 1, 3) AS ShortCode
FROM Products;
```

#### Sample Output

| ProductName | ShortCode |
|---|---|
| Smartphone | Sma |
| Laptop | Lap |
| Dumbbell Set | Dum |
| Bluetooth Speaker | Blu |
| Treadmill | Tre |

### Example — Extract City from Address

```sql
-- Address format: "123 Elm Street, Springfield, 01103"
-- Extract from position 20 onwards (after the street)
SELECT CustomerName,
       SUBSTRING(Address, 1, 20) AS StreetPreview
FROM Customers;
```

### Example — Extract Domain from Email (Combined with `LOCATE`)

```sql
-- Get everything after the @ symbol
SELECT Email,
       SUBSTRING(Email, LOCATE('@', Email) + 1) AS Domain
FROM Customers;
```

> 💡 Combining `SUBSTRING` with `LOCATE` is a powerful pattern for parsing structured strings like emails, URLs, and phone numbers.

---

## 3️⃣ `REPLACE` — Swapping Text

`REPLACE` finds all occurrences of a substring within a string and replaces them with a new substring.

### Syntax

```sql
SELECT REPLACE(string, from_string, new_string) FROM TableName;
```

### Example — Standardize "Street" to "St"

```sql
SELECT CustomerName,
       Address,
       REPLACE(Address, 'Street', 'St') AS UpdatedAddress
FROM Customers;
```

#### Sample Output

| CustomerName | Address | UpdatedAddress |
|---|---|---|
| John Doe | 123 Elm Street, Springfield, 01103 | 123 Elm St, Springfield, 01103 |
| Alice Johnson | 789 Oak Lane, Greenwood, 46142 | 789 Oak Lane, Greenwood, 46142 |

> 💡 If "Street" doesn't appear in the address, `REPLACE` simply returns the original string unchanged — no error is thrown.

### Example — Remove Spaces from Product Names

```sql
-- Create a URL-safe slug from product names
SELECT ProductName,
       REPLACE(ProductName, ' ', '-') AS ProductSlug
FROM Products;
```

#### Sample Output

| ProductName | ProductSlug |
|---|---|
| Bluetooth Speaker | Bluetooth-Speaker |
| Smart Outdoor Light | Smart-Outdoor-Light |
| Dumbbell Set | Dumbbell-Set |

> ⚠️ `REPLACE` is **case-sensitive** in most SQL databases. `REPLACE('Street', 'street', 'St')` would NOT replace "Street" — the case must match.

---

## 4️⃣ `CHAR_LENGTH` — Measuring String Length

`CHAR_LENGTH` returns the number of **characters** in a string.

### Syntax

```sql
SELECT CHAR_LENGTH(string) FROM TableName;
```

### Example — Product Name Lengths

```sql
SELECT ProductName,
       CHAR_LENGTH(ProductName) AS NameLength
FROM Products;
```

#### Sample Output

| ProductName | NameLength |
|---|---|
| Smartphone | 10 |
| Laptop | 6 |
| Bluetooth Speaker | 18 |
| Smart Outdoor Light | 19 |
| Ballpoint Pen Set | 18 |

### `LENGTH` vs. `CHAR_LENGTH`

| Function | Measures | Use When |
|---|---|---|
| `CHAR_LENGTH()` | Number of **characters** | Standard text — counts characters regardless of encoding |
| `LENGTH()` | Number of **bytes** | Multi-byte character sets (e.g., UTF-8 emoji, Asian scripts) |

> 💡 For standard ASCII text, both return the same value. For multi-byte characters, `CHAR_LENGTH` is the more reliable choice for counting visible characters.

### Example — Find Products with Long Names

```sql
-- Find products with names longer than 15 characters
SELECT ProductName, CHAR_LENGTH(ProductName) AS NameLength
FROM Products
WHERE CHAR_LENGTH(ProductName) > 15
ORDER BY NameLength DESC;
```

---

## 5️⃣ `UPPER` and `LOWER` — Changing Text Case

`UPPER` converts all letters to uppercase. `LOWER` converts all letters to lowercase.

### Syntax

```sql
SELECT UPPER(string) FROM TableName;
SELECT LOWER(string) FROM TableName;
```

### Example — Customer Names in Uppercase

```sql
SELECT CustomerName,
       UPPER(CustomerName) AS NameInUpperCase
FROM Customers;
```

#### Sample Output

| CustomerName | NameInUpperCase |
|---|---|
| John Doe | JOHN DOE |
| Jane Smith | JANE SMITH |
| Alice Johnson | ALICE JOHNSON |

### Example — Emails in Lowercase (for Consistent Comparison)

```sql
-- Ensure email comparison is case-insensitive
SELECT CustomerName, LOWER(Email) AS NormalizedEmail
FROM Customers
WHERE LOWER(Email) LIKE '%jmail.com%';
```

> ✅ Normalizing to lowercase before comparison is a common best practice to avoid case-sensitivity issues in searches.

---

## 6️⃣ `LOCATE` — Finding a Substring's Position

`LOCATE` returns the **position** (index) of the first occurrence of a substring within a string. Returns `0` if not found.

### Syntax

```sql
-- Full syntax (start position is optional; default is 1)
SELECT LOCATE(substring, string, start) FROM TableName;

-- Without start position
SELECT LOCATE(substring, string) FROM TableName;
```

### Example — Find Space Position in Customer Names

```sql
SELECT CustomerName,
       LOCATE(' ', CustomerName) AS SpacePosition
FROM Customers;
```

#### Sample Output

| CustomerName | SpacePosition |
|---|---|
| John Doe | 5 |
| Jane Smith | 5 |
| Alice Johnson | 6 |
| Michael Carter | 8 |

> 💡 `SpacePosition` tells us where the first name ends and the last name begins — useful for splitting full names.

### Example — Extract First Name Using `LOCATE` + `SUBSTRING`

```sql
SELECT CustomerName,
       SUBSTRING(CustomerName, 1, LOCATE(' ', CustomerName) - 1) AS FirstName
FROM Customers;
```

#### Sample Output

| CustomerName | FirstName |
|---|---|
| John Doe | John |
| Jane Smith | Jane |
| Alice Johnson | Alice |
| Michael Carter | Michael |

> ✅ Combining `LOCATE` and `SUBSTRING` is a powerful pattern for parsing structured text fields.

---

## 🧩 Code Exercises

---

### Task 1 — Create Product Codes from Category + Product Names

**Create a product code by concatenating the first 3 letters of the category name and the first 3 letters of the product name.**

```sql
SELECT CONCAT(
    SUBSTRING(CategoryName, 1, 3),
    SUBSTRING(ProductName, 1, 3)
) AS ProductCode
FROM Products
JOIN Categories ON Products.CategoryID = Categories.CategoryID;
```

#### Expected Output (sample)

| ProductCode |
|---|
| EleSma |
| FitDum |
| EleLap |
| … |
| OutHik |
| EleSma |
| FitSma |

> 💡 This combines `CONCAT`, `SUBSTRING`, and `JOIN` — three separate concepts working together in one query.

---

### Task 2 — Convert Product Names to Title Case

**Write a query to display product names in title case (first letter uppercase, rest lowercase).**

```sql
SELECT ProductName,
       CONCAT(
           UPPER(SUBSTRING(ProductName, 1, 1)),
           LOWER(SUBSTRING(ProductName, 2))
       ) AS TitleCaseName
FROM Products;
```

#### Expected Output (sample)

| TitleCaseName |
|---|
| Bluetooth speaker |
| Camping chair |
| Cookware set |
| … |
| Tent |
| Treadmill |
| Yoga mat |

> 💡 `SUBSTRING(ProductName, 1, 1)` gets the first character; `SUBSTRING(ProductName, 2)` gets everything from the second character onwards.

---

### Task 3 — Extract Email Domain

**Extract the domain (e.g., `mail.com`) from customer email addresses.**

```sql
SELECT Email,
       SUBSTRING(Email, LOCATE('@', Email) + 1) AS Domain
FROM Customers;
```

#### Expected Output (sample)

| Email | Domain |
|---|---|
| john.doe@smail.com | smail.com |
| jane.smith@inlook.com | inlook.com |
| alice.j@jmail.com | jmail.com |
| … | … |
| charlotte.m@hotmail.com | hotmail.com |
| lucas.anderson@jmail.com | jmail.com |
| emma.w@service.org | service.org |

> 💡 `LOCATE('@', Email)` finds the position of `@`. Adding `+ 1` moves one character past it. `SUBSTRING` then extracts from that point to the end.

---

## Combining String Functions

String functions become most powerful when combined:

```sql
-- Product summary: "LAP | Laptop | $1,099.99"
SELECT CONCAT(
    UPPER(SUBSTRING(ProductName, 1, 3)),
    ' | ',
    ProductName,
    ' | $',
    Price
) AS ProductSummary
FROM Products;
```

| ProductSummary |
|---|
| SMA \| Smartphone \| $635.79 |
| LAP \| Laptop \| $1099.99 |
| DUM \| Dumbbell Set \| $82.50 |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use consistent separators in `CONCAT`** | Makes combined strings readable and parseable |
| **Remember `SUBSTRING` starts at position 1** | Starting at 0 gives unexpected results |
| **Preview `REPLACE` with `SELECT` first** | Avoid unintended mass replacements — verify before using in `UPDATE` |
| **Use `CHAR_LENGTH` for character counts** | More reliable than `LENGTH` for multi-byte character sets |
| **Normalize with `LOWER`/`UPPER` before comparison** | Prevents case-sensitivity bugs in searches and joins |
| **Handle `NULL` in `CONCAT` with `COALESCE`** | Prevents entire results becoming `NULL` when one field is empty |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **`SUBSTRING` positions start at 1, not 0** — `SUBSTRING('Hello', 0, 3)` returns `Hel` in some systems but behaves inconsistently. Always start at 1.

> ⚠️ **`CONCAT` with `NULL` returns `NULL`** — if any argument is `NULL`, the result is `NULL`. Use `COALESCE(column, '')` to replace NULLs with empty strings.

> ⚠️ **`REPLACE` is case-sensitive** — `REPLACE(col, 'street', 'St')` won't replace "Street". Match the exact case of the target text.

> ⚠️ **No separator in `CONCAT`** — `CONCAT('John', 'Doe')` → `JohnDoe`. Always add a space or separator between values.

> ⚠️ **Using `LENGTH` instead of `CHAR_LENGTH` for character counting** — `LENGTH` counts bytes, not characters. They differ for multi-byte encodings like UTF-8.

---

## Summary

| Function | Syntax | Returns |
|---|---|---|
| `CONCAT` | `CONCAT(s1, s2, ...)` | All strings joined into one |
| `SUBSTRING` | `SUBSTRING(s, start, length)` | Portion of the string |
| `REPLACE` | `REPLACE(s, old, new)` | String with replacements applied |
| `CHAR_LENGTH` | `CHAR_LENGTH(s)` | Number of characters |
| `UPPER` | `UPPER(s)` | All letters uppercased |
| `LOWER` | `LOWER(s)` | All letters lowercased |
| `LOCATE` | `LOCATE(sub, s, start)` | Position of first match (0 if not found) |

---

## What's Next?

Now that you can manipulate text, explore more data transformation tools!

- 🔢 **Numeric functions (`ROUND`, `FLOOR`, `CEIL`, `ABS`)**
- 📅 **Date and time functions (`NOW`, `DATE_FORMAT`, `DATEDIFF`)**
- 🔄 **Conditional logic with `CASE` expressions**
- 🔗 **Using string functions in `WHERE` filters for advanced searches**

> *String functions are your text editor inside the database — format, combine, and extract exactly what you need!* 🚀

---

