# Date Functions
> *Learn to work with date and time in SQL.*

---

## Introduction

We often need to analyze how data changes over specific periods — how many orders are placed each month, when products become available, or what the estimated delivery date is for an order placed today. These operations require working with **date functions**, a critical aspect of database management.

### 🎯 Learning Goals

- ✅ Understand how to extract parts of a date
- ✅ Learn how to format dates for better readability
- ✅ Perform calculations with dates to solve practical problems

---

## Date Functions at a Glance

| Function | Purpose | Example Output |
|---|---|---|
| `CURDATE()` | Current date | `2024-11-07` |
| `CURTIME()` | Current time | `14:30:00` |
| `NOW()` | Current date and time | `2024-11-07 14:30:00` |
| `YEAR(date)` | Extract the year | `2024` |
| `MONTH(date)` | Extract the month (1–12) | `11` |
| `DAY(date)` | Extract the day (1–31) | `7` |
| `WEEK(date)` | Extract the week of year (1–52) | `45` |
| `DATE_FORMAT(date, fmt)` | Format date as a readable string | `Thursday, November 7th, 2024` |
| `DATE_ADD(date, INTERVAL n unit)` | Add a time interval to a date | `2024-11-14` (+ 7 days) |
| `DATE_SUB(date, INTERVAL n unit)` | Subtract a time interval from a date | `2024-10-31` (- 7 days) |
| `DATEDIFF(date1, date2)` | Days between two dates | `30` |

---

## 1️⃣ Getting Current Date and Time

Three functions return the current date and/or time from the server:

```sql
-- Current date only: YYYY-MM-DD
SELECT CURDATE();

-- Current date AND time: YYYY-MM-DD HH:MM:SS
SELECT NOW();

-- Current time only: HH:MM:SS
SELECT CURTIME();
```

#### Sample Output

| CURDATE() | NOW() | CURTIME() |
|---|---|---|
| 2024-11-07 | 2024-11-07 14:30:00 | 14:30:00 |

> 💡 These functions are especially useful in calculated columns — e.g., `DATEDIFF(CURDATE(), OrderDate)` to find how many days have passed since each order.

---

## 2️⃣ Extracting Date Parts

Extracting specific parts of a date lets us group, filter, and analyze time-based data at different granularities.

### Functions

| Function | Returns | Range |
|---|---|---|
| `YEAR(date)` | Four-digit year | e.g., 2024 |
| `MONTH(date)` | Month number | 1–12 |
| `DAY(date)` | Day of the month | 1–31 |
| `WEEK(date)` | Week of the year | 1–52 |

### Example — Extract Year, Month, and Day from Orders

```sql
SELECT OrderID,
       OrderDate,
       YEAR(OrderDate)  AS OrderYear,
       MONTH(OrderDate) AS OrderMonth,
       DAY(OrderDate)   AS OrderDay
FROM Orders;
```

#### Sample Output

| OrderID | OrderDate | OrderYear | OrderMonth | OrderDay |
|---|---|---|---|---|
| 1 | 2024-03-15 | 2024 | 3 | 15 |
| 2 | 2024-05-22 | 2024 | 5 | 22 |
| 3 | 2024-07-03 | 2024 | 7 | 3 |
| 4 | 2024-08-18 | 2024 | 8 | 18 |
| 5 | 2024-11-07 | 2024 | 11 | 7 |

---

### Using Extracted Parts in `WHERE` Clauses

```sql
-- Count orders placed in March (month 3)
SELECT COUNT(OrderID) AS MarchOrders
FROM Orders
WHERE MONTH(OrderDate) = 3;
```

```sql
-- Find all orders placed in 2024
SELECT OrderID, OrderDate, TotalAmount
FROM Orders
WHERE YEAR(OrderDate) = 2024;
```

---

### Using Extracted Parts in `GROUP BY`

```sql
-- Total revenue per month
SELECT MONTH(OrderDate) AS Month,
       COUNT(*) AS OrderCount,
       SUM(TotalAmount) AS MonthlyRevenue
FROM Orders
GROUP BY MONTH(OrderDate)
ORDER BY Month;
```

#### Sample Output

| Month | OrderCount | MonthlyRevenue |
|---|---|---|
| 3 | 2 | 1522.82 |
| 5 | 1 | 235.50 |
| 7 | 1 | 2639.99 |
| 8 | 1 | 2550.50 |
| 11 | 1 | 758.99 |

---

## 3️⃣ Formatting Dates

The raw `YYYY-MM-DD` format is precise but not always user-friendly. `DATE_FORMAT` lets us present dates in any structure we choose.

### Syntax

```sql
SELECT DATE_FORMAT(date_column, 'format_string') FROM TableName;
```

### Common Format Specifiers

| Specifier | Output | Example |
|---|---|---|
| `%Y` | 4-digit year | `2024` |
| `%y` | 2-digit year | `24` |
| `%M` | Full month name | `November` |
| `%m` | 2-digit month | `11` |
| `%D` | Day with suffix | `7th`, `22nd` |
| `%d` | 2-digit day | `07` |
| `%W` | Full weekday name | `Thursday` |
| `%a` | Abbreviated weekday | `Thu` |
| `%H` | Hour (24-hour) | `14` |
| `%h` | Hour (12-hour) | `02` |
| `%i` | Minutes | `30` |
| `%s` | Seconds | `00` |

### Example — Human-Readable Full Date

```sql
SELECT OrderID,
       OrderDate,
       DATE_FORMAT(OrderDate, '%W, %M %D, %Y') AS FormattedDate
FROM Orders;
```

#### Sample Output

| OrderID | OrderDate | FormattedDate |
|---|---|---|
| 1 | 2024-03-15 | Friday, March 15th, 2024 |
| 2 | 2024-05-22 | Wednesday, May 22nd, 2024 |
| 3 | 2024-07-03 | Wednesday, July 3rd, 2024 |

### Example — US Format (MM/DD/YYYY)

```sql
SELECT OrderID,
       OrderDate,
       DATE_FORMAT(OrderDate, '%m/%d/%Y') AS FormattedOrderDate
FROM Orders;
```

#### Sample Output

| OrderID | OrderDate | FormattedOrderDate |
|---|---|---|
| 1 | 2024-03-15 | 03/15/2024 |
| 2 | 2024-05-22 | 05/22/2024 |
| 3 | 2024-07-03 | 07/03/2024 |

### Example — Month and Year Only (for Monthly Reports)

```sql
SELECT DATE_FORMAT(OrderDate, '%M %Y') AS MonthYear,
       COUNT(*) AS OrderCount,
       SUM(TotalAmount) AS Revenue
FROM Orders
GROUP BY DATE_FORMAT(OrderDate, '%M %Y')
ORDER BY MIN(OrderDate);
```

#### Sample Output

| MonthYear | OrderCount | Revenue |
|---|---|---|
| March 2024 | 1 | 1286.58 |
| May 2024 | 1 | 235.50 |
| July 2024 | 1 | 2639.99 |

---

## 4️⃣ Date Calculations

We often need to add or subtract time from a date, or find the gap between two dates.

### `DATE_ADD` and `DATE_SUB`

```sql
-- Add a time interval
DATE_ADD(date_column, INTERVAL value unit)

-- Subtract a time interval
DATE_SUB(date_column, INTERVAL value unit)
```

Supported units include: `DAY`, `WEEK`, `MONTH`, `YEAR`, `HOUR`, `MINUTE`, `SECOND`

### Example — Estimated Delivery Date (+7 days)

```sql
SELECT OrderID,
       OrderDate,
       DATE_ADD(OrderDate, INTERVAL 7 DAY) AS EstimatedDeliveryDate
FROM Orders;
```

#### Sample Output

| OrderID | OrderDate | EstimatedDeliveryDate |
|---|---|---|
| 1 | 2024-03-15 | 2024-03-22 |
| 2 | 2024-05-22 | 2024-05-29 |
| 3 | 2024-07-03 | 2024-07-10 |

### Example — Orders Placed Within the Last 30 Days

```sql
SELECT OrderID, CustomerID, OrderDate
FROM Orders
WHERE OrderDate >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
```

### Example — Orders Placed in the Next 6 Months (Future Planning)

```sql
SELECT OrderID, OrderDate
FROM Orders
WHERE OrderDate BETWEEN CURDATE() AND DATE_ADD(CURDATE(), INTERVAL 6 MONTH);
```

---

### `DATEDIFF` — Days Between Two Dates

```sql
DATEDIFF(date1, date2)
-- Returns: date1 - date2 (in days)
-- Positive if date1 is later, negative if date1 is earlier
```

### Example — Days Since Each Order Was Placed

```sql
SELECT OrderID,
       OrderDate,
       DATEDIFF(CURDATE(), OrderDate) AS DaysSinceOrder
FROM Orders;
```

#### Sample Output

| OrderID | OrderDate | DaysSinceOrder |
|---|---|---|
| 1 | 2024-03-15 | 237 |
| 2 | 2024-05-22 | 169 |
| 3 | 2024-07-03 | 127 |

### Example — Orders Older Than 90 Days

```sql
SELECT OrderID, CustomerID, OrderDate
FROM Orders
WHERE DATEDIFF(CURDATE(), OrderDate) > 90;
```

---

## Date Functions — Combined Example

Here's a query that uses multiple date functions together for a comprehensive order report:

```sql
SELECT OrderID,
       CustomerID,
       OrderDate,
       DATE_FORMAT(OrderDate, '%W, %M %D, %Y')           AS FormattedDate,
       DATE_ADD(OrderDate, INTERVAL 7 DAY)               AS EstimatedDelivery,
       DATEDIFF(CURDATE(), OrderDate)                    AS DaysAgo,
       CONCAT(YEAR(OrderDate), '-W', WEEK(OrderDate))    AS WeekReference
FROM Orders
ORDER BY OrderDate ASC;
```

---

## 🧩 Code Exercises

---

### Task 1 — Extract Month and Day

**Extract the month and day from `OrderDate`, and display them alongside `OrderID`.**

```sql
SELECT OrderID,
       OrderDate,
       MONTH(OrderDate) AS OrderMonth,
       DAY(OrderDate)   AS OrderDay
FROM Orders;
```

#### Expected Output (sample)

| OrderID | OrderDate | OrderMonth | OrderDay |
|---|---|---|---|
| 1 | 2024-03-15 | 3 | 15 |
| 2 | 2024-05-22 | 5 | 22 |
| 3 | 2024-07-03 | 7 | 3 |
| … | … | … | … |
| 8 | 2024-06-27 | 6 | 27 |
| 9 | 2024-09-10 | 9 | 10 |
| 10 | 2024-10-05 | 10 | 5 |

---

### Task 2 — Format Dates as MM/DD/YYYY

**Show each `OrderID`, `OrderDate`, and a `FormattedOrderDate` column in `MM/DD/YYYY` format.**

```sql
SELECT OrderID,
       OrderDate,
       DATE_FORMAT(OrderDate, '%m/%d/%Y') AS FormattedOrderDate
FROM Orders;
```

#### Expected Output (sample)

| OrderID | OrderDate | FormattedOrderDate |
|---|---|---|
| 1 | 2024-03-15 | 03/15/2024 |
| 2 | 2024-05-22 | 05/22/2024 |
| 3 | 2024-07-03 | 07/03/2024 |
| … | … | … |
| 8 | 2024-06-27 | 06/27/2024 |
| 9 | 2024-09-10 | 09/10/2024 |
| 10 | 2024-10-05 | 10/05/2024 |

---

## Date Functions Quick Reference

| Task | Function/Syntax |
|---|---|
| Get today's date | `CURDATE()` |
| Get date + time | `NOW()` |
| Get current time | `CURTIME()` |
| Extract year | `YEAR(date)` |
| Extract month | `MONTH(date)` |
| Extract day | `DAY(date)` |
| Extract week number | `WEEK(date)` |
| Format a date | `DATE_FORMAT(date, '%W, %M %D, %Y')` |
| Add days | `DATE_ADD(date, INTERVAL 7 DAY)` |
| Subtract months | `DATE_SUB(date, INTERVAL 3 MONTH)` |
| Days between dates | `DATEDIFF(date1, date2)` |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Store dates in `DATE` or `DATETIME` columns** | Prevents string comparison issues and enables date functions to work correctly |
| **Use ISO format (`YYYY-MM-DD`) for date literals** | Standard format recognized across all SQL environments |
| **Double-check interval units** | `INTERVAL 1 MONTH` vs `INTERVAL 30 DAY` gives different results in some months |
| **Be mindful of time zones** | `NOW()` runs on the server — client and server may be in different time zones |
| **Use `DATE_FORMAT` for display only** | Formatted dates as strings can't be used in date arithmetic — keep raw dates for calculations |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Storing dates as text strings** — `'2024-11-07'` stored in a `VARCHAR` column breaks all date functions. Always use `DATE` or `DATETIME` column types.

> ⚠️ **Confusing `MONTH()` (returns a number) with `MONTHNAME()` (returns a name)** — `MONTH('2024-11-07')` → `11`, `MONTHNAME('2024-11-07')` → `November`.

> ⚠️ **Wrong format specifiers in `DATE_FORMAT`** — `%M` is the full month name, `%m` is the two-digit number. Mixing them gives unexpected results.

> ⚠️ **Forgetting interval syntax** — `INTERVAL 7 DAYS` is wrong; the correct syntax is `INTERVAL 7 DAY` (no `S`).

> ⚠️ **`DATEDIFF` order matters** — `DATEDIFF(date1, date2)` = `date1 - date2`. Swap them for a positive vs. negative result.

---

## Summary

| Concept | Key Takeaway |
|---|---|
| `CURDATE()` / `NOW()` / `CURTIME()` | Get the current date, date+time, or time from the server |
| `YEAR()` / `MONTH()` / `DAY()` / `WEEK()` | Extract specific parts of a date for filtering or grouping |
| `DATE_FORMAT()` | Convert a date into a human-readable string format |
| `DATE_ADD()` / `DATE_SUB()` | Add or subtract a time interval from a date |
| `DATEDIFF()` | Calculate the number of days between two dates |
| ISO format | Always use `YYYY-MM-DD` for date literals in queries |

---

## What's Next?

Now that you can work with dates, explore more powerful analytical SQL tools!

- 🔢 **Numeric functions (`ROUND`, `FLOOR`, `CEIL`, `ABS`)**
- 🔄 **Conditional logic with `CASE` expressions**
- 🪟 **Window functions for time-based running totals and rankings**
- 🔗 **Combining date functions with `GROUP BY` for trend analysis**

> *Date functions turn timestamps into insights — track trends, calculate deadlines, and understand your data's timeline!* 🚀

---

