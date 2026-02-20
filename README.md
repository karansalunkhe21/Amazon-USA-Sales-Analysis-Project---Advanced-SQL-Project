# 🛒 Amazon USA Sales Analysis — Advanced SQL Project

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue?logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/Language-SQL-orange)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-red)
![Records](https://img.shields.io/badge/Dataset-20K%2B%20Records-green)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Database Schema & ERD](#-database-schema--erd)
- [Project Structure](#-project-structure)
- [Database Setup](#-database-setup)
- [Data Import Order](#-data-import-order)
- [Data Cleaning](#-data-cleaning)
- [Business Problems Solved](#-business-problems-solved)
  - [Advanced Problems (22)](#advanced-problems)
  - [Easy to Medium Problems (50)](#easy-to-medium-problems)
- [Key SQL Concepts Used](#-key-sql-concepts-used)
- [Learning Outcomes](#-learning-outcomes)

---

## 📖 Project Overview

This project involves a comprehensive analysis of **20,000+ sales records** from an Amazon-like e-commerce platform using **PostgreSQL**. It is structured around real-world business challenges spanning customer behavior, product performance, revenue trends, inventory management, and logistics.

The project is split into two tiers:
- **22 Advanced Business Problems** — complex analytical queries using CTEs, window functions, subqueries, and stored procedures.
- **50 Easy-to-Medium Problems** — covering foundational to intermediate SQL including joins, aggregation, date functions, and subqueries.

> 💡 An ERD diagram is included to visualize the database schema and table relationships.

---

## 🗂 Database Schema & ERD

### ERD Diagram (Design Sketch)
![ERD Sketch](erd2.png)

### ERD Diagram (Final)
![ERD](erd.png)

The database consists of **9 relational tables**:

| Table         | Description                                      |
|---------------|--------------------------------------------------|
| `category`    | Product categories                               |
| `customers`   | Customer details including state                 |
| `sellers`     | Seller information and origin                    |
| `products`    | Product catalog with price and COGS              |
| `orders`      | Order header — date, customer, seller, status    |
| `order_items` | Line items per order with quantity and price     |
| `payments`    | Payment status per order                         |
| `shippings`   | Shipping & delivery details with return dates    |
| `inventory`   | Stock levels per product and warehouse           |

---

## 📁 Project Structure

```
amazon-sql-project/
│
├── data/
│   ├── category.csv
│   ├── customers.csv
│   ├── sellers.csv
│   ├── products.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── payments.csv
│   ├── shipping.csv
│   └── inventory.csv
│
├── Amazon_Schemas.sql               -- Table creation scripts
├── Problems_Statements.sql          -- 22 advanced problem statements
├── solutions.sql                    -- Full solutions with explanations
├── erd.png                          -- Final ERD
├── erd2.png                         -- ERD design sketch
└── README.md
```

---

## ⚙️ Database Setup

Run the schema file to create all tables:

```sql
-- Run this first
\i Amazon_Schemas.sql
```
---

## 📥 Data Import Order

> ⚠️ Follow this exact sequence to respect foreign key constraints:

| Step | Table         |
|------|---------------|
| 1    | `category`    |
| 2    | `customers`   |
| 3    | `sellers`     |
| 4    | `products`    |
| 5    | `orders`      |
| 6    | `order_items` |
| 7    | `payments`    |
| 8    | `shippings`   |
| 9    | `inventory`   |

---

## 🧹 Data Cleaning

Before analysis, the following cleaning steps were performed:

- **Duplicate Removal** — Identified and removed duplicate records in `customers` and `orders`.
- **Null Handling** — Missing addresses defaulted to `'xxxx'`; null payment statuses treated as `'Pending'`; null `return_date` in shippings left as-is (valid — not all orders are returned).
- **Computed Column** — Added `total_sale` column to `order_items`:

```sql
ALTER TABLE order_items ADD COLUMN total_sale FLOAT;
UPDATE order_items SET total_sale = quantity * price_per_unit;
```

---

## 🧩 Business Problems Solved

### 22 Advanced Problems
Challenging queries that reflect real business decision-making:

- 🏆 Top 10 best-selling products by revenue
- 📊 Revenue breakdown by category with percentage contribution
- 📈 Monthly sales trend with month-over-month comparison
- 👥 Customer Lifetime Value (CLTV) with ranking
- 📦 Inventory stock alerts for low-stock products
- 🚚 Identifying shipping delays beyond 3 days
- 💳 Payment success rate breakdown by status
- 🏪 Top 5 sellers with order success rate
- 🔁 Most returned products with return rate percentage
- 🛑 Inactive sellers with no sales in the last 6 months
- 📍 Top 5 customers per state by order volume
- 📉 Products with highest revenue decline (2022 vs 2023)
- ⚙️ Stored procedure that auto-updates inventory on every sale

---

### Easy to Medium Problems

| Category | Topics |
|---|---|
| Basic Queries & Joins | SELECT, INNER/LEFT/RIGHT/CROSS JOIN, multi-table joins |
| Aggregation & Grouping | COUNT, SUM, AVG, GROUP BY, HAVING, CASE |
| Subqueries | WHERE/SELECT subqueries, EXISTS, IN, NOT IN, correlated |
| Window Functions | RANK, DENSE_RANK, ROW_NUMBER, NTILE, LEAD, LAG, running totals |
| Date Functions | EXTRACT, DATEDIFF, DATE_TRUNC, CURRENT_DATE, AGE |

---

## 🛠 Key SQL Concepts Used

- **Joins** — INNER, LEFT, multi-table
- **Aggregations** — SUM, COUNT, AVG with GROUP BY / HAVING
- **CTEs** — `WITH` clauses for multi-step logic
- **Window Functions** — `RANK()`, `DENSE_RANK()`, `LAG()`, `LEAD()`, `PARTITION BY`
- **Subqueries** — correlated, scalar, EXISTS, NOT IN
- **Date Functions** — `EXTRACT()`, `DATE_TRUNC()`, `INTERVAL`, `CURRENT_DATE`
- **CASE Statements** — conditional aggregation and segmentation
- **Stored Procedures** — `plpgsql` procedure with transaction logic
- **DDL & DML** — `ALTER TABLE`, `UPDATE`, `INSERT`, constraint management

---

## 📚 Learning Outcomes

By completing this project, the following skills were developed and demonstrated:

- Designing and implementing a normalized relational database schema
- Cleaning and preprocessing real-world datasets
- Writing advanced SQL queries for business intelligence
- Using window functions for ranking, trends, and period-over-period comparisons
- Building stored procedures with conditional logic and inventory automation
- Optimizing query structure for readability and performance

