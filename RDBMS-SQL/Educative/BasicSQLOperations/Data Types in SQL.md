# Data Types in SQL
> *Learn about data types in SQL.*

---

## Introduction

Imagine you're at a store, deciding how to organize products on the shelves. Some items are electronics, while others are kitchenware or stationery. Similarly, in a database, every piece of data has a **type** that determines how it can be stored, manipulated, and queried. Just as it would be inefficient to use one kind of shelf for all product types, SQL data types help ensure data is handled in the most efficient way possible.

For example:
- 💰 Product **prices** are stored as numbers
- 👤 Customer **names** are stored as text
- 📅 Order **dates** are stored as dates

### 🎯 Learning Goals

- ✅ Understand various data types in SQL and their applications
- ✅ Identify and work with numeric, string, and date data types
- ✅ Learn best practices for choosing the right data types

---

## Why Data Types Matter

Data types in SQL define the **nature of data** a column can hold. They help in:

| Benefit | Description |
|---|---|
| 🔒 **Data Integrity** | Prevents invalid data from being stored (e.g., text in a price field) |
| ⚡ **Performance** | Correct types allow the database engine to process data efficiently |
| 💾 **Storage Optimization** | Right-sized types use less disk space |
| 🔢 **Accurate Operations** | Numeric types enable math; string types enable text search |

> 💡 **Example:** Storing prices as strings instead of numbers means you can't calculate totals or compare prices — you'd need expensive conversion steps every time.

---

## 🔢 Numeric Data Types

Numeric data types store numerical values and come in various types based on size and precision.

### `INT`

Stores **whole numbers** without decimal points. Ideal for counts, quantities, and identifiers.

```sql
-- Stock quantity must be a whole number
CREATE TABLE Products (
    ProductID   INT,
    ProductName VARCHAR(100),
    Stock       INT  -- e.g., 50, 15, 100 (not 0.5 or 1.5)
);
```

> ✅ Use `INT` when values will never have decimals — like item counts, IDs, or ages.

### `DECIMAL(p, s)`

Stores numbers with **fixed precision (`p`)** and **scale (`s`)**. Ideal for monetary values where exactness matters.

- `p` = total number of digits (both sides of decimal)
- `s` = number of digits after the decimal point

```sql
-- DECIMAL(10, 2) allows up to 99,999,999.99
CREATE TABLE Products (
    ProductID INT,
    Price     DECIMAL(10, 2)  -- e.g., 635.79, 1099.99
);
```

| Example Value | DECIMAL(10, 2) | Valid? |
|---|---|---|
| 635.79 | ✅ | Yes |
| 99999999.99 | ✅ | Yes (max value) |
| 1234567890.12 | ❌ | Exceeds precision |
| 19.999 | ⚠️ | Rounded to 20.00 |

> ✅ Always use `DECIMAL` for financial data — never `FLOAT` or `DOUBLE`, which can introduce rounding errors.

### Other Numeric Types

| Type | Size | Range | Use Case |
|---|---|---|---|
| `TINYINT` | 1 byte | -128 to 127 | Flags, small counts |
| `SMALLINT` | 2 bytes | -32,768 to 32,767 | Medium counts |
| `BIGINT` | 8 bytes | Very large range | Large IDs, populations |
| `FLOAT` | 4 bytes | Approximate decimal | Scientific data (not finance) |

---

## 🔤 String Data Types

String data types store sequences of characters — essential for names, emails, descriptions, and more.

### `CHAR(n)`

Stores **fixed-length** character strings. If the stored string is shorter than `n`, it is padded with spaces to fill the full length.

```sql
-- Country codes are always exactly 2 characters
CREATE TABLE Countries (
    CountryCode CHAR(2),   -- 'US', 'UK', 'IN'
    CountryName VARCHAR(100)
);
```

> 💡 Storing `'U'` in a `CHAR(2)` column saves it as `'U '` (with a trailing space) internally.

### `VARCHAR(n)`

Stores **variable-length** character strings up to `n` characters. Only uses as much space as the string actually needs (plus 1–2 bytes for length info).

```sql
-- Product names vary in length
CREATE TABLE Products (
    ProductID   INT,
    ProductName VARCHAR(100),  -- 'Phone' uses 5 chars, not 100
    Email       VARCHAR(255)
);
```

### CHAR vs. VARCHAR — Quick Comparison

| Feature | `CHAR(n)` | `VARCHAR(n)` |
|---|---|---|
| **Storage** | Always uses defined space | Only uses space needed |
| **Padding** | Pads with spaces | No padding |
| **Max Length** | Up to 255 characters | Up to 65,535 characters |
| **Speed** | Faster for short, fixed data | More efficient for varying data |
| **Best For** | Country codes, fixed IDs | Names, emails, descriptions |

```sql
-- Good use of CHAR vs VARCHAR
CREATE TABLE Customers (
    CustomerID   INT,
    CustomerName VARCHAR(100),   -- Variable length name
    CountryCode  CHAR(2),        -- Always 2 characters: 'US', 'IN'
    Email        VARCHAR(255)    -- Variable length email
);
```

---

## 📅 Date and Time Data Types

Date and time types are crucial for tracking when transactions occur, orders are placed, or products are added.

### `DATE`

Stores dates in **`YYYY-MM-DD`** format.

```sql
-- Order placed on December 9, 2024
INSERT INTO Orders (OrderDate) VALUES ('2024-12-09');
```

### `TIME`

Stores time in **`HH:MM:SS`** format.

```sql
-- Order placed at 2:30 PM
INSERT INTO OrderLog (OrderTime) VALUES ('14:30:00');
```

### `DATETIME`

Combines **date and time** in a single field using **`YYYY-MM-DD HH:MM:SS`** format.

```sql
-- Full timestamp: December 9, 2024 at 2:30 PM
INSERT INTO Orders (OrderDateTime) VALUES ('2024-12-09 14:30:00');
```

### Date/Time Types — Summary

| Type | Format | Example | Use Case |
|---|---|---|---|
| `DATE` | `YYYY-MM-DD` | `2024-12-09` | Birthdays, order dates |
| `TIME` | `HH:MM:SS` | `14:30:00` | Shift times, log entries |
| `DATETIME` | `YYYY-MM-DD HH:MM:SS` | `2024-12-09 14:30:00` | Timestamps, order history |

```sql
-- Querying orders placed after a specific date
SELECT OrderID, OrderDate
FROM Orders
WHERE OrderDate > '2024-06-01';
```

---

## 🧩 Other Specialized Data Types

SQL also offers specialized types for unique use cases.

### `BOOLEAN`

Stores `TRUE` or `FALSE` values. In MySQL, `BOOLEAN` is a synonym for `TINYINT(1)`.

```sql
CREATE TABLE Orders (
    OrderID     INT,
    IsDelivered BOOLEAN  -- TRUE = delivered, FALSE = pending
);

-- Query only delivered orders
SELECT * FROM Orders WHERE IsDelivered = TRUE;
```

### `TINYINT(M)`

Stores small integers using just 1 byte. The `M` parameter sets the display width, not the storage size.

```sql
-- Rating between 1 and 5
CREATE TABLE Reviews (
    ReviewID INT,
    Rating   TINYINT  -- Values: 1, 2, 3, 4, 5
);
```

### `BLOB`

Stores large **binary objects** like images or files.

```sql
CREATE TABLE ProductImages (
    ProductID  INT,
    ImageData  BLOB  -- Raw binary image data
);
```

### `TEXT`

Stores large **text content** such as blog posts, product descriptions, or user comments.

```sql
CREATE TABLE Products (
    ProductID   INT,
    Description TEXT  -- Long product description paragraph
);
```

### `ENUM`

Stores **one value** from a predefined list. Great for status fields with a fixed set of options.

```sql
CREATE TABLE Orders (
    OrderID INT,
    Status  ENUM('Pending', 'Shipped', 'Delivered', 'Cancelled')
);

-- Insert a new order with status
INSERT INTO Orders (OrderID, Status) VALUES (101, 'Pending');
```

### `SET`

Stores **multiple values** from a predefined list in a single field.

```sql
CREATE TABLE UserProfiles (
    UserID  INT,
    Hobbies SET('Reading', 'Traveling', 'Gaming', 'Cooking')
);

-- A user with multiple hobbies
INSERT INTO UserProfiles (UserID, Hobbies)
VALUES (1, 'Reading,Gaming');
```

### All Specialized Types — Summary

| Type | Stores | Example Use |
|---|---|---|
| `BOOLEAN` | `TRUE` / `FALSE` | Is order delivered? |
| `TINYINT` | Small integers (-128 to 127) | Ratings, flags |
| `BLOB` | Binary data (images, files) | Product images |
| `TEXT` | Long text | Product descriptions |
| `ENUM` | One of a fixed list | Order status |
| `SET` | Multiple from a fixed list | User hobbies/tags |

---

## 🧪 Putting It All Together — A Real Table Example

Here's how multiple data types work together in the `Products` table:

```sql
CREATE TABLE Products (
    ProductID    INT,                                      -- Whole number ID
    ProductName  VARCHAR(100),                            -- Variable text
    CategoryID   INT,                                     -- Foreign key (whole number)
    Price        DECIMAL(10, 2),                         -- Exact monetary value
    Stock        INT,                                     -- Whole number quantity
    Description  TEXT,                                    -- Long text
    AddedOn      DATETIME,                                -- Full timestamp
    IsAvailable  BOOLEAN,                                 -- True/False flag
    Status       ENUM('Active', 'Discontinued', 'Draft') -- Fixed options
);
```

### Sample Insert

```sql
INSERT INTO Products
    (ProductID, ProductName, CategoryID, Price, Stock, Description, AddedOn, IsAvailable, Status)
VALUES
    (1, 'Smartphone', 1, 635.79, 50,
     'Latest model with 5G support and 128GB storage.',
     '2024-01-15 09:00:00', TRUE, 'Active');
```

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use `VARCHAR` for variable text** | Saves storage space compared to fixed `CHAR` |
| **Use `CHAR` for fixed-length codes** | Faster lookups for consistent-length data |
| **Use `DECIMAL` for money, not `FLOAT`** | Avoids floating-point rounding errors |
| **Use `INT` for IDs and counts** | Efficient storage and fast indexing |
| **Use `ENUM` for known, fixed options** | Prevents invalid values and saves space |
| **Use `DATETIME` for full timestamps** | Captures both date and time in one field |
| **Always comment complex type choices** | Helps collaborators understand your design |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Storing prices as `FLOAT`** can cause subtle rounding errors — always use `DECIMAL` for financial data.

> ⚠️ **Using `TEXT` for short strings** wastes overhead — use `VARCHAR` for anything under a few thousand characters.

> ⚠️ **Using `CHAR` for variable-length data** wastes storage with unnecessary space padding.

> ⚠️ **Storing dates as strings** (`'09-12-2024'`) breaks date comparisons and sorting — always use proper `DATE` or `DATETIME` types.

---

## Summary

| Category | Types | Key Use |
|---|---|---|
| **Numeric** | `INT`, `DECIMAL`, `TINYINT`, `BIGINT` | Counts, prices, IDs |
| **String** | `CHAR`, `VARCHAR`, `TEXT` | Names, emails, descriptions |
| **Date/Time** | `DATE`, `TIME`, `DATETIME` | Timestamps, order dates |
| **Specialized** | `BOOLEAN`, `BLOB`, `ENUM`, `SET` | Flags, images, fixed-option fields |

---

## What's Next?

Now that you understand SQL data types, you're ready to start building and querying real database tables!

- 🏗️ **Creating Tables with the Right Data Types**
- 🔍 **Writing SELECT Queries to Retrieve Data**
- 🔗 **Using Constraints like PRIMARY KEY and FOREIGN KEY**

> *Choosing the right data type is like choosing the right tool — it makes everything else easier!* 🚀

---

