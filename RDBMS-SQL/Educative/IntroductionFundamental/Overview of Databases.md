# Database Information
> *Learn about the structure of a database for an online store.*

---

## Introduction

Imagine you visit an online store with a well-organized system. The homepage categorizes items by types such as electronics, clothing, and books. Each category further organizes products by brand, price, and ratings — making it easy to find what you're looking for without browsing through unrelated items. This requires a very well planned and thought-out strategy to maintain and display all the necessary information in a concise manner.

In this course, we will work on a model that simulates an online store database. This will help us learn SQL concepts with confidence and relate them to real-life scenarios.

### 🎯 Learning Goals

- ✅ Understand the scenario of the online store database
- ✅ Get introduced to the main tables: `Categories`, `Products`, `Customers`, `Suppliers`, and `Orders`
- ✅ Recognize how these tables are interconnected to represent real-world data relationships

---

## Overview of the Database

Our database — named **OnlineStore** — is modeled on an online retail store. It represents a typical e-commerce platform where customers can browse products, place orders, and receive shipments from various suppliers.

The `OnlineStore` database is designed to handle:

| Area | Description |
|---|---|
| 🛍️ **Product Catalog** | A collection of products organized into categories for easy navigation |
| 👤 **Customer Information** | Details about customers who register and make purchases |
| 🏭 **Supplier Details** | Information about suppliers providing products to the store |
| 📦 **Order Management** | Records of customer orders and transaction details |

---

## Main Tables in the Database

The `OnlineStore` database maintains data using the following **five main tables**:

1. `Categories`
2. `Products`
3. `Customers`
4. `Suppliers`
5. `Orders`

Let's dive into each table and its specific role.

---

## 📁 Categories

The `Categories` table organizes products into manageable groups. This helps customers find products easily and assists the admin with inventory management.

### Fields

| Field | Description |
|---|---|
| `CategoryID` | Unique identifier for each category |
| `CategoryName` | Name of each category |

### Sample Data

| CategoryID | CategoryName |
|---|---|
| 1 | Electronics |
| 2 | Kitchenware |
| 3 | Fitness |
| 4 | Outdoor |
| 5 | Stationery |

---

## 📦 Products

The `Products` table stores details of items available for sale. It plays a vital role in maintaining store operations by tracking all purchasable items.

### Fields

| Field | Description |
|---|---|
| `ProductID` | Unique identifier for each product |
| `ProductName` | Name of each product |
| `CategoryID` | The product's category (references the `Categories` table) |
| `Price` | Cost of the product |
| `Stock` | Number of items currently available |

### Sample Data

| ProductID | ProductName | CategoryID | Price | Stock |
|---|---|---|---|---|
| 1 | Smartphone | 1 | $635.79 | 50 |
| 2 | Dumbbell Set | 3 | $82.50 | 15 |
| 3 | Laptop | 1 | $1,099.99 | 30 |
| 4 | Smart Outdoor Light | 4 | $75.62 | 25 |
| 5 | Knife Set | 2 | $50.00 | 25 |
| 6 | Ballpoint Pen Set | 5 | $5.00 | 150 |
| 7 | Bluetooth Speaker | 1 | $135.50 | 100 |
| 8 | Blender | 2 | $90.25 | 40 |
| 9 | Cookware Set | 2 | $129.99 | 20 |
| 10 | Treadmill | 3 | $1,499.99 | 10 |

> 💡 Products with `CategoryID = 1` belong to **Electronics**, `CategoryID = 2` to **Kitchenware**, and so on (see Categories table above).

---

## 👤 Customers

The `Customers` table holds information about registered customers. Tracking customer information is important for processing orders and managing customer relationships.

### Fields

| Field | Description |
|---|---|
| `CustomerID` | Unique identifier for each customer |
| `CustomerName` | Full name of each customer |
| `Email` | Contact email address |
| `Phone` | Contact phone number |
| `Address` | Shipping address |

### Sample Data

| CustomerID | CustomerName | Email | Phone | Address |
|---|---|---|---|---|
| 1 | John Doe | john.doe@smail.com | 413-456-6862 | 123 Elm Street, Springfield, 01103 |
| 2 | Jane Smith | jane.smith@inlook.com | 708-567-5234 | 456 Maple Avenue, Riverside, 60546 |
| 3 | Alice Johnson | alice.j@jmail.com | 317-678-5717 | 789 Oak Lane, Greenwood, 46142 |
| 4 | Michael Carter | michael.c@domain.com | 205-789-4593 | 101 Pine Court, Meadowbrook, 35242 |
| 5 | Charlie Davis | charlie.d@provider.net | 541-890-1900 | 202 Birch Drive, Lakeview, 97630 |

---

## 🏭 Suppliers

The `Suppliers` table stores details of product suppliers. Keeping track of supplier information is important for restocking inventory and communicating with suppliers.

### Fields

| Field | Description |
|---|---|
| `SupplierID` | Unique identifier for each supplier |
| `SupplierName` | Name of the supplier company |
| `Email` | Email address of each supplier |
| `Phone` | Contact phone number of each supplier |
| `Address` | Business address of each supplier |

### Sample Data

| SupplierID | SupplierName | Email | Phone | Address |
|---|---|---|---|---|
| 1 | TechSolutions | contact@techsolutions.com | 415-123-4567 | 123 Silicon Ave, San Francisco, 94105 |
| 2 | GadgetWorks | support@gadgetworks.com | 408-987-6543 | 456 Tech Drive, San Jose, 95101 |
| 3 | HomeEssentials | sales@homeessentials.com | 713-111-2233 | 789 Comfort St, Houston, 77002 |
| 4 | SportyGear | info@sportygear.com | 312-321-6540 | 321 Fit Lane, Chicago, 60611 |
| 5 | StationaryWorld | help@stationaryworld.com | 212-543-8765 | 234 Paper Rd, New York, 10001 |

---

## 🧾 Orders

The `Orders` table records customer orders and details such as order date and total amount. Storing order details helps track sales and understand purchasing patterns.

### Fields

| Field | Description |
|---|---|
| `OrderID` | Unique identifier for each order |
| `CustomerID` | The customer who placed the order (references `Customers` table) |
| `OrderDate` | Date when the order was placed |
| `TotalAmount` | Total cost of the order |

### Sample Data

| OrderID | CustomerID | OrderDate | TotalAmount |
|---|---|---|---|
| 1 | 5 | 2024-03-15 | $1,286.58 |
| 2 | 1 | 2024-05-22 | $235.50 |
| 3 | 2 | 2024-07-03 | $2,639.99 |
| 4 | 4 | 2024-08-18 | $2,550.50 |
| 5 | 3 | 2024-11-07 | $758.99 |

> 💡 Order 1 was placed by **Charlie Davis** (`CustomerID = 5`), Order 2 by **John Doe** (`CustomerID = 1`), and so on (see Customers table above).

---

## 🔗 Other (Supporting) Tables

### Order_Details

Ensures detailed tracking of items within each order — enabling accurate order fulfilment and inventory analysis. It references both `Orders` and `Products`.

| OrderItemID | OrderID | ProductID | Quantity | TotalItemPrice |
|---|---|---|---|---|
| 1 | 1 | 1 | 2 | $1,271.58 |
| 2 | 1 | 6 | 3 | $15.00 |
| 3 | 2 | 7 | 1 | $135.50 |
| 4 | 2 | 5 | 2 | $100.00 |
| 5 | 3 | 3 | 1 | $1,099.99 |
| 6 | 3 | 12 | 2 | $40.00 |
| 7 | 3 | 19 | 1 | $500.00 |

> 💡 **Example:** Order `OrderID = 1` contains 2 Smartphones ($1,271.58) + 3 Ballpoint Pen Sets ($15.00) = **$1,286.58 total**.

### Product_Suppliers

Links products to their respective suppliers, facilitating effective supply chain management and restocking processes.

| ProductID | SupplierID |
|---|---|
| 1 | 1 |
| 2 | 3 |
| 1 | 2 |
| 3 | 2 |
| 5 | 3 |

> 💡 The **Smartphone** (`ProductID = 1`) has two suppliers: **TechSolutions** (`SupplierID = 1`) and **GadgetWorks** (`SupplierID = 2`).

---

## 🔄 Interrelationships Between Tables

Understanding how tables are linked is crucial for managing the database effectively.

### Table Dependency Summary

| Relationship | Rule |
|---|---|
| **Products → Categories** | A product's `CategoryID` must exist in the `Categories` table first |
| **Orders → Customers** | An order's `CustomerID` must exist in the `Customers` table first |
| **Order_Details → Orders + Products** | Both the order and product must exist before adding order items |
| **Product_Suppliers → Products + Suppliers** | Both the product and supplier must exist before linking them |

### Primary vs. Supporting Tables

| Type | Tables | Purpose |
|---|---|---|
| **Primary** | `Categories`, `Products`, `Customers`, `Orders`, `Suppliers` | Store essential data for core store operations |
| **Supporting** | `Order_Details`, `Product_Suppliers` | Provide detailed data that enhances core table functionality |

---

## 🧪 Example Workflow: Adding a New Product

Here's a step-by-step example of how data flows through the `OnlineStore` database when adding a new product:

### Step 1 — Check/Add Category

```sql
-- Check if the category exists
SELECT * FROM Categories WHERE CategoryName = 'Electronics';

-- If not, add it
INSERT INTO Categories (CategoryName) VALUES ('Electronics');
```

### Step 2 — Insert the New Product

```sql
-- Add the product, referencing the correct CategoryID
INSERT INTO Products (ProductName, CategoryID, Price, Stock)
VALUES ('Wireless Headphones', 1, 199.99, 75);
```

### Step 3 — Link to a Supplier

```sql
-- Associate the new product with a supplier
INSERT INTO Product_Suppliers (ProductID, SupplierID)
VALUES (11, 2); -- ProductID 11, SupplierID 2 (GadgetWorks)
```

> ⚠️ **Always follow the dependency chain:** Categories → Products → Product_Suppliers. Skipping steps will cause errors due to missing references.

---

## Summary

| Table | Role |
|---|---|
| `Categories` | Groups products into browsable sections |
| `Products` | Lists all items available for sale |
| `Customers` | Stores registered customer details |
| `Suppliers` | Tracks supplier contact and business info |
| `Orders` | Records each customer's purchase |
| `Order_Details` | Breaks down each order into individual products |
| `Product_Suppliers` | Maps which supplier provides which product |

---

## What's Next?

Now that you understand the structure of the `OnlineStore` database, we're ready to start writing SQL queries to interact with it!

- 🔍 **Querying data with SELECT**
- 🔗 **Joining tables to combine information**
- ✏️ **Inserting, updating, and deleting records**

> *Every table tells part of the story — SQL lets you read them all together!* 🚀

---

*Made with ❤️ for SQL learners everywhere.*