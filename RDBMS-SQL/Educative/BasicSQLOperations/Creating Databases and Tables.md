# Creating Databases and Tables
> *Learn to create databases and tables using MySQL.*

---

## Introduction

Imagine you are tasked with building an online store to sell products globally. To manage your inventory, track orders, and store customer information, you need a structured system to organize all this data. When your store becomes popular and attracts many customers, the amount of data that needs to be managed grows enormously. This is where **creating databases and tables** becomes essential.

### 🎯 Learning Goals

- ✅ Use the `CREATE DATABASE` statement to create a new database
- ✅ Use the `CREATE TABLE` statement to define new tables
- ✅ Define columns with appropriate data types
- ✅ Use `SHOW` and `DESCRIBE` commands to inspect databases and tables

---

## Step 1 — Create a Database with `CREATE DATABASE`

Before storing any data, we need a **container** to hold our tables and other database objects. In SQL, we create one using the `CREATE DATABASE` statement.

### Syntax

```sql
CREATE DATABASE DatabaseName;
```

- `DatabaseName` must be **unique** within the database server
- Think of it as creating a new folder to hold all related files

### Example

```sql
CREATE DATABASE OnlineStore;
```

This initializes a new database where we can begin creating tables for products, customers, orders, and more.

> 💡 **Naming Tip:** Use descriptive, PascalCase names like `OnlineStore` or `InventorySystem` — avoid vague names like `DB1` or `TestDB`.

---

## Step 2 — Select the Database with `USE`

Once a database is created, we need to **select it** before performing any operations inside it.

### Syntax

```sql
USE DatabaseName;
```

### Example

```sql
USE OnlineStore;
```

This tells the SQL server to set `OnlineStore` as the **current working database**. All subsequent commands will apply to this database until another is selected.

> ⚠️ **Important:** Always run `USE DatabaseName;` before creating tables or querying data — otherwise you'll get an error or accidentally modify the wrong database.

---

## Step 3 — Create Tables with `CREATE TABLE`

Tables are where the actual data lives. Each table holds data in **rows and columns**, and each column has a specific **data type** that controls what kind of data it can store.

### Syntax

```sql
CREATE TABLE TableName (
    Column1 DataType [constraints],
    Column2 DataType [constraints],
    ...
);
```

| Part | Description |
|---|---|
| `TableName` | The name of the table to create |
| `Column1`, `Column2` | Names of the columns |
| `DataType` | The type of data the column stores (`INT`, `VARCHAR`, `DATE`, etc.) |
| `[constraints]` | Optional rules like `NOT NULL`, `UNIQUE`, `PRIMARY KEY` |

> ⚠️ `CREATE TABLE` will only work **after** selecting a database with `USE`. If no database is selected, an error will occur.

---

## Step 4 — Define Columns and Data Types

Choosing the right data types for each column ensures data is stored **efficiently and accurately**, and helps maintain data integrity.

### Example: Creating the `Products` Table

```sql
CREATE TABLE Products (
    ProductID   INT,            -- Unique identifier for each product
    ProductName VARCHAR(50),    -- Name of the product (up to 50 characters)
    CategoryID  INT,            -- Links the product to a category
    Price       DECIMAL(10, 2), -- Price with two decimal places
    Stock       INT             -- Available quantity
);
```

### Column Breakdown

| Column | Data Type | Reason |
|---|---|---|
| `ProductID` | `INT` | Whole number identifier — no decimals needed |
| `ProductName` | `VARCHAR(50)` | Text of varying length, max 50 characters |
| `CategoryID` | `INT` | Foreign key reference — whole number |
| `Price` | `DECIMAL(10,2)` | Exact monetary value with 2 decimal places |
| `Stock` | `INT` | Whole number — you can't have 0.5 items |

---

## Step 5 — Use `SHOW` and `DESCRIBE` Commands

After creating databases and tables, use these commands to **review and inspect** their structures.

| Command | What It Does |
|---|---|
| `SHOW DATABASES;` | Lists all databases on the current SQL server |
| `SHOW TABLES;` | Lists all tables in the currently selected database |
| `DESCRIBE TableName;` | Displays the structure (columns, types, constraints) of a specific table |

### Examples

```sql
-- See all databases on the server
SHOW DATABASES;
```

```sql
-- See all tables in the current database
USE OnlineStore;
SHOW TABLES;
```

```sql
-- Inspect the structure of the Products table
USE OnlineStore;
DESCRIBE Products;
```

### Sample Output of `DESCRIBE Products`

| Field | Type | Null | Key | Default | Extra |
|---|---|---|---|---|---|
| ProductID | int | YES | | NULL | |
| ProductName | varchar(50) | YES | | NULL | |
| CategoryID | int | YES | | NULL | |
| Price | decimal(10,2) | YES | | NULL | |
| Stock | int | YES | | NULL | |

---

## 🧪 Full Example — Building the OnlineStore Database

Here's a complete script that creates the `OnlineStore` database and all its core tables:

```sql
-- Step 1: Create the database
CREATE DATABASE OnlineStore;

-- Step 2: Select the database
USE OnlineStore;

-- Step 3: Create the Categories table
CREATE TABLE Categories (
    CategoryID   INT,          -- Unique identifier for each category
    CategoryName VARCHAR(50)   -- Name of the category
);

-- Step 4: Create the Products table
CREATE TABLE Products (
    ProductID   INT,            -- Unique identifier for each product
    ProductName VARCHAR(50),    -- Name of the product
    CategoryID  INT,            -- Links the product to a category
    Price       DECIMAL(10, 2), -- Product price with two decimal places
    Stock       INT             -- Quantity of product available
);

-- Step 5: Create the Customers table
CREATE TABLE Customers (
    CustomerID   INT,           -- Unique identifier for each customer
    CustomerName VARCHAR(50),   -- Full name of the customer
    Email        VARCHAR(50),   -- Email address of the customer
    Phone        VARCHAR(15),   -- Contact phone number
    Address      VARCHAR(100)   -- Mailing address
);

-- Step 6: Create the Orders table
CREATE TABLE Orders (
    OrderID     INT,            -- Unique identifier for each order
    CustomerID  INT,            -- Links the order to a customer
    OrderDate   DATE,           -- Date when the order was placed
    TotalAmount DECIMAL(10, 2)  -- Total cost of the order
);

-- Inspect: List all tables in the database
SHOW TABLES;

-- Inspect: View the structure of the Products table
DESCRIBE Products;
```

---

## 🧩 Code Exercise — Create the `Suppliers` Table

### Task

Using the `OnlineStore` database, create a table named `Suppliers` to store information about suppliers with the following columns:

| Column | Data Type |
|---|---|
| `SupplierID` | `INT` |
| `SupplierName` | `VARCHAR(50)` |
| `Email` | `VARCHAR(50)` |
| `Phone` | `VARCHAR(15)` |
| `Address` | `VARCHAR(100)` |

### Solution

```sql
USE OnlineStore;

CREATE TABLE Suppliers (
    SupplierID   INT,           -- Unique identifier for each supplier
    SupplierName VARCHAR(50),   -- Name of the supplier company
    Email        VARCHAR(50),   -- Supplier email address
    Phone        VARCHAR(15),   -- Supplier contact number
    Address      VARCHAR(100)   -- Supplier business address
);

-- Verify the table was created correctly
DESCRIBE Suppliers;
```

### Expected Output of `DESCRIBE Suppliers`

| Field | Type |
|---|---|
| SupplierID | int |
| SupplierName | varchar(50) |
| Email | varchar(50) |
| Phone | varchar(15) |
| Address | varchar(100) |

---

## Complete OnlineStore Table Overview

Once all tables are created, here's what the full database structure looks like:

| Table | Key Columns | Purpose |
|---|---|---|
| `Categories` | `CategoryID`, `CategoryName` | Groups products into sections |
| `Products` | `ProductID`, `ProductName`, `CategoryID`, `Price`, `Stock` | Lists all items for sale |
| `Customers` | `CustomerID`, `CustomerName`, `Email`, `Phone`, `Address` | Stores buyer information |
| `Orders` | `OrderID`, `CustomerID`, `OrderDate`, `TotalAmount` | Records purchase transactions |
| `Suppliers` | `SupplierID`, `SupplierName`, `Email`, `Phone`, `Address` | Tracks product suppliers |

---

## ✅ Best Practices

| Practice | Why It Matters |
|---|---|
| **Use clear, consistent names** | `Customers` and `CustomerName` are better than `Tbl1` or `Name1` |
| **Always run `USE` before `CREATE TABLE`** | Ensures you're building tables in the right database |
| **Choose data types carefully** | Right types save space and prevent invalid data |
| **Comment your SQL code** | Makes scripts easier to read and maintain |
| **Use `DESCRIBE` after creating tables** | Confirms the structure is exactly as intended |
| **Group related tables in one database** | Keeps data organized and relationships clean |

---

## ❌ Common Mistakes to Avoid

> ⚠️ **Forgetting `USE DatabaseName;`** before creating tables causes an error or builds the table in the wrong database.

> ⚠️ **Using vague column names** like `Col1` or `Field2` makes queries hard to read and maintain.

> ⚠️ **Wrong data types** — e.g., using `VARCHAR` for prices instead of `DECIMAL` — breaks math operations and wastes storage.

> ⚠️ **Making `VARCHAR` too short** — if `ProductName VARCHAR(10)` is defined but a product name has 15 characters, the insert will fail or truncate.

---

## Summary

| Concept | Statement / Command | Purpose |
|---|---|---|
| **Create a database** | `CREATE DATABASE name;` | Initializes a new database container |
| **Select a database** | `USE name;` | Sets the active working database |
| **Create a table** | `CREATE TABLE name (...);` | Defines a new table with columns |
| **List databases** | `SHOW DATABASES;` | Shows all databases on the server |
| **List tables** | `SHOW TABLES;` | Shows all tables in current database |
| **Inspect a table** | `DESCRIBE TableName;` | Displays column structure of a table |

---

## What's Next?

Now that you can create databases and tables, the next step is to fill them with data and start querying!

- ➕ **Inserting Data with `INSERT INTO`**
- 🔍 **Retrieving Data with `SELECT`**
- 🔒 **Adding Constraints like `PRIMARY KEY` and `NOT NULL`**

> *A well-structured database is the foundation of every great application — you've just laid yours!* 🚀

---

*Made with ❤️ for SQL learners everywhere.*