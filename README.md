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
- [How to Run](#-how-to-run)
- [Contributing](#-contributing)

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
├── 50_Easy_to_Medium_Level_Questions.md  -- 50 additional problems
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

### Full Schema

```sql
CREATE TABLE category (
  category_id   INT PRIMARY KEY,
  category_name VARCHAR(20)
);

CREATE TABLE customers (
  customer_id INT PRIMARY KEY,
  first_name  VARCHAR(20),
  last_name   VARCHAR(20),
  state       VARCHAR(20)
);

CREATE TABLE sellers (
  seller_id   INT PRIMARY KEY,
  seller_name VARCHAR(25),
  origin      VARCHAR(15)
);

CREATE TABLE products (
  product_id   INT PRIMARY KEY,
  product_name VARCHAR(50),
  price        FLOAT,
  cogs         FLOAT,
  category_id  INT,
  CONSTRAINT product_fk_category FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE orders (
  order_id     INT PRIMARY KEY,
  order_date   DATE,
  customer_id  INT,
  seller_id    INT,
  order_status VARCHAR(15),
  CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  CONSTRAINT orders_fk_sellers   FOREIGN KEY (seller_id)   REFERENCES sellers(seller_id)
);

CREATE TABLE order_items (
  order_item_id  INT PRIMARY KEY,
  order_id       INT,
  product_id     INT,
  quantity       INT,
  price_per_unit FLOAT,
  CONSTRAINT order_items_fk_orders   FOREIGN KEY (order_id)   REFERENCES orders(order_id),
  CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE payments (
  payment_id     INT PRIMARY KEY,
  order_id       INT,
  payment_date   DATE,
  payment_status VARCHAR(20),
  CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE shippings (
  shipping_id        INT PRIMARY KEY,
  order_id           INT,
  shipping_date      DATE,
  return_date        DATE,
  shipping_providers VARCHAR(15),
  delivery_status    VARCHAR(15),
  CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE inventory (
  inventory_id    INT PRIMARY KEY,
  product_id      INT,
  stock           INT,
  warehouse_id    INT,
  last_stock_date DATE,
  CONSTRAINT inventory_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);
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

### Advanced Problems

A full set of **22 advanced business problems** with solutions are in [`solutions.sql`](solutions.sql). Highlights include:

---

#### 1. 🏆 Top 10 Selling Products
```sql
SELECT 
    oi.product_id,
    p.product_name,
    SUM(oi.total_sale) AS total_sale,
    COUNT(o.order_id)  AS total_orders
FROM orders AS o
JOIN order_items AS oi ON oi.order_id = o.order_id
JOIN products    AS p  ON p.product_id = oi.product_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 10;
```

---

#### 2. 📊 Revenue by Category (with % Contribution)
```sql
SELECT 
    p.category_id,
    c.category_name,
    SUM(oi.total_sale) AS total_sale,
    ROUND(
        SUM(oi.total_sale) / (SELECT SUM(total_sale) FROM order_items) * 100, 2
    ) AS contribution_pct
FROM order_items AS oi
JOIN products  AS p ON p.product_id = oi.product_id
LEFT JOIN category AS c ON c.category_id = p.category_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```

---

#### 3. 📈 Monthly Sales Trend (with LAG)
```sql
SELECT 
    year, month,
    total_sale AS current_month_sale,
    LAG(total_sale, 1) OVER (ORDER BY year, month) AS last_month_sale
FROM (
    SELECT 
        EXTRACT(YEAR  FROM o.order_date) AS year,
        EXTRACT(MONTH FROM o.order_date) AS month,
        ROUND(SUM(oi.total_sale::numeric), 2) AS total_sale
    FROM orders AS o
    JOIN order_items AS oi ON oi.order_id = o.order_id
    WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY 1, 2
) AS t1;
```

---

#### 4. 🚚 Shipping Delays (> 3 Days)
```sql
SELECT 
    c.*,
    o.*,
    s.shipping_providers,
    s.shipping_date - o.order_date AS days_to_ship
FROM orders AS o
JOIN customers AS c ON c.customer_id = o.customer_id
JOIN shippings AS s ON o.order_id = s.order_id
WHERE s.shipping_date - o.order_date > 3;
```

---

#### 5. 🔁 Most Returned Products
```sql
SELECT 
    p.product_id,
    p.product_name,
    COUNT(*) AS total_units_sold,
    SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) AS total_returned,
    ROUND(
        SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END)::numeric
        / COUNT(*)::numeric * 100, 2
    ) AS return_pct
FROM order_items AS oi
JOIN products AS p ON oi.product_id = p.product_id
JOIN orders   AS o ON o.order_id = oi.order_id
GROUP BY 1, 2
ORDER BY 5 DESC;
```

---

#### 6. 🏪 Top 5 Sellers (with Success Rate)
```sql
WITH top_sellers AS (
    SELECT s.seller_id, s.seller_name, SUM(oi.total_sale) AS total_sale
    FROM orders AS o
    JOIN sellers     AS s  ON o.seller_id = s.seller_id
    JOIN order_items AS oi ON oi.order_id = o.order_id
    GROUP BY 1, 2
    ORDER BY 3 DESC
    LIMIT 5
),
seller_reports AS (
    SELECT o.seller_id, ts.seller_name, o.order_status, COUNT(*) AS total_orders
    FROM orders AS o
    JOIN top_sellers AS ts ON ts.seller_id = o.seller_id
    WHERE o.order_status NOT IN ('Inprogress', 'Returned')
    GROUP BY 1, 2, 3
)
SELECT
    seller_id, seller_name,
    SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END) AS completed_orders,
    SUM(CASE WHEN order_status = 'Cancelled' THEN total_orders ELSE 0 END) AS cancelled_orders,
    SUM(total_orders) AS total_orders,
    ROUND(
        SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END)::numeric
        / SUM(total_orders)::numeric * 100, 2
    ) AS success_rate_pct
FROM seller_reports
GROUP BY 1, 2;
```

---

#### 7. 📦 Stored Procedure — Auto-Update Inventory on Sale

```sql
CREATE OR REPLACE PROCEDURE add_sales(
    p_order_id      INT,
    p_customer_id   INT,
    p_seller_id     INT,
    p_order_item_id INT,
    p_product_id    INT,
    p_quantity      INT
)
LANGUAGE plpgsql AS $$
DECLARE
    v_count   INT;
    v_price   FLOAT;
    v_product VARCHAR(50);
BEGIN
    SELECT price, product_name INTO v_price, v_product
    FROM products WHERE product_id = p_product_id;

    SELECT COUNT(*) INTO v_count
    FROM inventory
    WHERE product_id = p_product_id AND stock >= p_quantity;

    IF v_count > 0 THEN
        INSERT INTO orders(order_id, order_date, customer_id, seller_id)
        VALUES (p_order_id, CURRENT_DATE, p_customer_id, p_seller_id);

        INSERT INTO order_items(order_item_id, order_id, product_id, quantity, price_per_unit, total_sale)
        VALUES (p_order_item_id, p_order_id, p_product_id, p_quantity, v_price, v_price * p_quantity);

        UPDATE inventory SET stock = stock - p_quantity WHERE product_id = p_product_id;

        RAISE NOTICE 'Sale recorded for product: %. Inventory updated.', v_product;
    ELSE
        RAISE NOTICE 'Product: % is out of stock or insufficient quantity.', v_product;
    END IF;
END;
$$;

-- Usage
CALL add_sales(25005, 2, 5, 25004, 1, 14);
```

---

### Easy to Medium Problems

50 additional SQL exercises are documented in [`50_Easy_to_Medium_Level_Questions.md`](50_Easy_to_Medium_Level_Questions.md), covering:

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

---

## 🚀 How to Run

1. **Install PostgreSQL** (v14+ recommended)
2. **Create a database**:
   ```sql
   CREATE DATABASE amazon_sales;
   ```
3. **Run the schema**:
   ```bash
   psql -d amazon_sales -f Amazon_Schemas.sql
   ```
4. **Import CSV data** in the order specified in the [Data Import Order](#-data-import-order) section using pgAdmin or `\COPY`.
5. **Run solutions**:
   ```bash
   psql -d amazon_sales -f solutions.sql
   ```

---

## 🤝 Contributing

Contributions, improvements, and additional query solutions are welcome!

1. Fork the repository
2. Create a new branch (`git checkout -b feature/your-query`)
3. Commit your changes (`git commit -m 'Add solution for problem X'`)
4. Push to the branch (`git push origin feature/your-query`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

---

*Built with PostgreSQL | Dataset: Amazon USA E-Commerce Sales (Simulated)*
