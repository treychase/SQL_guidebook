# SQL Guidebook - Complete Documentation

[![SQL Guidebook CI/CD](https://github.com/treychase/SQL_guidebook/actions/workflows/main.yml/badge.svg)](https://github.com/treychase/SQL_guidebook/actions/workflows/main.yml)

## 📖 Overview

This SQL Guidebook is a comprehensive personal reference guide demonstrating advanced SQL queries using a real-world e-commerce database scenario. It covers essential SQL operations from basic queries to advanced window functions, CTEs, and complex joins.

**Purpose:** To serve as a reusable resource for SQL interviews, data analysis tasks, and daily database operations.

**Technology:** SQLite3 with Python integration for demonstration and testing.

---

## 🗄️ Database Schema

The guidebook uses a normalized e-commerce database with four interconnected tables:

### **Entity Relationship Diagram (ERD) Overview:**

```
customers (1) ----< orders (M)
                     |
                     |
                     V
              order_items (M) >---- (M) products
```

### **Table Structures:**

#### **1. customers**
Stores customer information and signup details.

| Column | Type | Description |
|--------|------|-------------|
| customer_id | INTEGER (PK) | Unique customer identifier |
| first_name | TEXT | Customer's first name |
| last_name | TEXT | Customer's last name |
| email | TEXT (UNIQUE) | Customer's email address |
| city | TEXT | Customer's city |
| state | TEXT | Customer's state |
| signup_date | DATE | Account creation date |

#### **2. products**
Contains product catalog information.

| Column | Type | Description |
|--------|------|-------------|
| product_id | INTEGER (PK) | Unique product identifier |
| product_name | TEXT | Product name |
| category | TEXT | Product category |
| price | DECIMAL(10,2) | Product price |
| stock_quantity | INTEGER | Current stock level |

#### **3. orders**
Records customer order transactions.

| Column | Type | Description |
|--------|------|-------------|
| order_id | INTEGER (PK) | Unique order identifier |
| customer_id | INTEGER (FK) | Reference to customer |
| order_date | DATE | Date order was placed |
| total_amount | DECIMAL(10,2) | Total order value |
| status | TEXT | Order status (Completed, Shipped, Processing) |

#### **4. order_items**
Tracks individual items within each order (junction table).

| Column | Type | Description |
|--------|------|-------------|
| item_id | INTEGER (PK) | Unique item identifier |
| order_id | INTEGER (FK) | Reference to order |
| product_id | INTEGER (FK) | Reference to product |
| quantity | INTEGER | Quantity purchased |
| unit_price | DECIMAL(10,2) | Price per unit at time of sale |

---

## 📝 Query Documentation

### **Query 1: Basic SELECT with WHERE, ORDER BY, and LIMIT**

**Title:** Basic Product Filtering and Sorting

**SQL Concepts:** `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`

**How It Works:**
```sql
SELECT 
    product_name,
    category,
    price,
    stock_quantity
FROM products
WHERE category = 'Electronics'
ORDER BY price DESC
LIMIT 5
```

**Explanation:**
- **SELECT**: Specifies which columns to retrieve
- **WHERE**: Filters rows to only Electronics category
- **ORDER BY DESC**: Sorts results by price from highest to lowest
- **LIMIT 5**: Returns only the top 5 results

**Use Cases:**
- Finding premium products in a specific category
- Product catalog displays showing "Featured" or "Top" items
- Quick market research on pricing within categories
- Inventory management for high-value items

**Expected Output:**
Returns the 5 most expensive electronics products with their stock levels.

---

### **Query 2: Aggregate Functions with GROUP BY and HAVING**

**Title:** Sales Analysis by Customer with Aggregates

**SQL Concepts:** `COUNT()`, `SUM()`, `AVG()`, `MAX()`, `GROUP BY`, `HAVING`

**How It Works:**
```sql
SELECT 
    customer_id,
    COUNT(*) as total_orders,
    SUM(total_amount) as total_spent,
    AVG(total_amount) as avg_order_value,
    MAX(total_amount) as largest_order
FROM orders
GROUP BY customer_id
HAVING SUM(total_amount) > 500
ORDER BY total_spent DESC
```

**Explanation:**
- **GROUP BY**: Combines all orders for each unique customer
- **COUNT(*)**: Counts number of orders per customer
- **SUM(total_amount)**: Calculates total spending per customer
- **AVG(total_amount)**: Computes average order value
- **HAVING**: Filters groups (customers) with total spending > $500
- **ORDER BY**: Ranks customers by spending (highest first)

**Use Cases:**
- Identifying high-value customers (VIP programs)
- Customer segmentation for targeted marketing
- Loyalty program eligibility determination
- Sales performance analysis
- Customer lifetime value (CLV) calculations

**Expected Output:**
List of customers who spent over $500, with their order statistics ranked by total spending.

---

### **Query 3: INNER JOIN - Combining Multiple Tables**

**Title:** Customer Order Details with INNER JOIN

**SQL Concepts:** `INNER JOIN`, Table Aliases, String Concatenation (`||`)

**How It Works:**
```sql
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name as customer_name,
    c.city,
    c.state,
    o.order_id,
    o.order_date,
    o.total_amount,
    o.status
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date
```

**Explanation:**
- **Table Aliases** (c, o): Shorthand references for table names
- **INNER JOIN**: Returns only rows where customer_id exists in BOTH tables
- **ON clause**: Specifies the matching condition between tables
- **String concatenation (||)**: Combines first and last names
- Only customers who have placed orders appear in results

**Use Cases:**
- Customer order history reports
- Order fulfillment dashboards
- Customer service inquiries (looking up specific customer orders)
- Sales reports showing customer details
- Geographic sales analysis by customer location

**Expected Output:**
All orders with associated customer information, excluding customers who haven't ordered yet.

---

### **Query 4: LEFT JOIN - Including All Records from Left Table**

**Title:** All Customers with Order Count (LEFT JOIN)

**SQL Concepts:** `LEFT JOIN`, `COALESCE()`, NULL handling, `GROUP BY`

**How It Works:**
```sql
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name as customer_name,
    c.email,
    COUNT(o.order_id) as order_count,
    COALESCE(SUM(o.total_amount), 0) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name, c.email
ORDER BY order_count DESC, total_spent DESC
```

**Explanation:**
- **LEFT JOIN**: Returns ALL customers, even those without orders
- **COUNT(o.order_id)**: Counts orders (returns 0 for customers without orders)
- **COALESCE()**: Replaces NULL values with 0 for customers with no orders
- **GROUP BY**: Required when using aggregates with joined tables

**Key Difference from INNER JOIN:**
- INNER JOIN: Only customers WITH orders
- LEFT JOIN: ALL customers (with or without orders)

**Use Cases:**
- Customer engagement analysis (finding inactive customers)
- Marketing campaigns to re-engage dormant customers
- Complete customer lists with activity metrics
- Customer acquisition reports
- Identifying customers who signed up but never purchased

**Expected Output:**
Complete customer list showing order count and spending, with 0 values for customers who haven't ordered.

---

### **Query 5: Multiple JOINs - Complete Order Details**

**Title:** Complete Order Information with Multiple JOINs

**SQL Concepts:** Multiple `INNER JOIN`s, Calculated Fields

**How It Works:**
```sql
SELECT 
    o.order_id,
    c.first_name || ' ' || c.last_name as customer_name,
    p.product_name,
    p.category,
    oi.quantity,
    oi.unit_price,
    (oi.quantity * oi.unit_price) as line_total,
    o.order_date,
    o.status
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
ORDER BY o.order_date DESC, o.order_id
```

**Explanation:**
- **Four tables joined**: orders → customers, order_items, products
- **Sequential joins**: Each JOIN builds on previous table
- **Calculated field**: `(oi.quantity * oi.unit_price)` computes line item total
- **Result**: Denormalized view showing complete order information

**Join Chain:**
1. orders → customers (get customer info)
2. orders → order_items (get items in each order)
3. order_items → products (get product details)

**Use Cases:**
- Detailed order reports for accounting
- Order fulfillment sheets for warehouse
- Customer invoice generation
- Sales analysis by product and customer
- Inventory tracking (what's selling to whom)
- Product popularity reports

**Expected Output:**
Each row represents one item in an order, with full customer and product details.

---

### **Query 6: CASE WHEN - Data Transformation**

**Title:** Customer Segmentation with CASE WHEN

**SQL Concepts:** `CASE WHEN`, Conditional Logic, Customer Classification

**How It Works:**
```sql
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name as customer_name,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    COUNT(o.order_id) as order_count,
    CASE 
        WHEN COALESCE(SUM(o.total_amount), 0) >= 1500 THEN 'VIP'
        WHEN COALESCE(SUM(o.total_amount), 0) >= 500 THEN 'Regular'
        ELSE 'New'
    END as customer_segment
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC
```

**Explanation:**
- **CASE WHEN**: SQL's if-then-else logic
- **Evaluation order**: Checks conditions from top to bottom
- **First match wins**: Stops at first TRUE condition
- **ELSE**: Default value if no conditions match
- **Segmentation rules**:
  - VIP: $1,500+ total spending
  - Regular: $500-$1,499 spending
  - New: < $500 or no orders

**Use Cases:**
- Customer segmentation for marketing tiers
- Dynamic pricing strategies (VIP discounts)
- Personalized email campaigns
- Loyalty program management
- Customer service prioritization
- Sales rep territory assignment
- Risk assessment (credit limits)

**Expected Output:**
All customers with their segment classification based on spending patterns.

---

### **Query 7: Window Functions - RANK and ROW_NUMBER**

**Title:** Product Ranking by Category with Window Functions

**SQL Concepts:** `RANK()`, `ROW_NUMBER()`, `OVER`, `PARTITION BY`

**How It Works:**
```sql
SELECT 
    product_name,
    category,
    price,
    stock_quantity,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as price_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as row_num
FROM products
ORDER BY category, price_rank
```

**Explanation:**
- **Window Functions**: Perform calculations across sets of rows
- **PARTITION BY category**: Creates separate "windows" for each category
- **ORDER BY price DESC**: Within each category, ranks by price (high to low)
- **RANK()**: Assigns rank, allows ties (1, 2, 2, 4...)
- **ROW_NUMBER()**: Assigns unique sequential numbers (1, 2, 3, 4...)

**Difference between RANK() and ROW_NUMBER():**
- **RANK()**: If two products cost $99, both get rank 2, next is rank 4
- **ROW_NUMBER()**: Always sequential (1, 2, 3), even for ties

**Use Cases:**
- Product catalog organization (show top 3 per category)
- Competitive pricing analysis
- Featured product selection
- Inventory prioritization
- Marketing material generation (top products)
- Price positioning reports
- E-commerce "Best Sellers" lists

**Expected Output:**
Products with their rank within each category, allowing easy identification of premium vs. budget items.

---

### **Query 8: Common Table Expressions (CTE) with WITH**

**Title:** Monthly Sales Analysis using CTE

**SQL Concepts:** `WITH` (CTE), `STRFTIME()`, Date Functions

**How It Works:**
```sql
WITH monthly_sales AS (
    SELECT 
        STRFTIME('%Y-%m', order_date) as month,
        order_id,
        total_amount,
        status
    FROM orders
)
SELECT 
    month,
    COUNT(order_id) as total_orders,
    SUM(total_amount) as monthly_revenue,
    AVG(total_amount) as avg_order_value,
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) as completed_orders
FROM monthly_sales
GROUP BY month
ORDER BY month
```

**Explanation:**
- **CTE (Common Table Expression)**: Named temporary result set
- **WITH clause**: Defines the CTE before main query
- **STRFTIME('%Y-%m', date)**: Extracts year-month from date (2024-05)
- **Two-step process**:
  1. CTE organizes data by month
  2. Main query aggregates monthly statistics
- **Benefits**: More readable than nested subqueries

**Use Cases:**
- Monthly/quarterly revenue reports
- Trend analysis over time
- Business performance dashboards
- Budget vs. actual comparisons
- Seasonal pattern identification
- Year-over-year growth calculations
- Financial reporting

**Expected Output:**
Monthly summary showing order volume, revenue, and completion rates.

---

### **Query 9: Advanced Window Functions - Running Total**

**Title:** Running Total of Sales with Window Functions

**SQL Concepts:** `SUM() OVER`, Window Frames, `ROWS BETWEEN`

**How It Works:**
```sql
SELECT 
    order_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY order_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total,
    AVG(total_amount) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3
FROM orders
ORDER BY order_date
```

**Explanation:**
- **Window Frames**: Define which rows to include in calculation
- **UNBOUNDED PRECEDING**: From first row to current
- **CURRENT ROW**: The row being calculated
- **2 PRECEDING**: Two rows before current
- **Running total**: Cumulative sum from start to current row
- **Moving average**: Average of last 3 orders (smooths fluctuations)

**Frame Types Explained:**
- `UNBOUNDED PRECEDING`: Start of partition
- `N PRECEDING`: N rows before current
- `CURRENT ROW`: The current row
- `N FOLLOWING`: N rows after current
- `UNBOUNDED FOLLOWING`: End of partition

**Use Cases:**
- Cumulative sales tracking
- Progress toward goals (running total vs. target)
- Trend smoothing with moving averages
- Financial dashboards showing growth
- Stock price analysis (moving averages)
- Website traffic trends
- Performance tracking over time

**Expected Output:**
Each order shows its individual amount, cumulative total, and 3-order moving average.

---

### **Query 10: UPDATE Operation**

**Title:** UPDATE Product Stock After Sale

**SQL Concepts:** `UPDATE`, `SET`, `WHERE`

**How It Works:**
```sql
UPDATE products
SET stock_quantity = stock_quantity - 5
WHERE product_id = 2
```

**Explanation:**
- **UPDATE**: Modifies existing records
- **SET**: Specifies which column(s) to change
- **Arithmetic in SET**: Can perform calculations (subtract 5 from current value)
- **WHERE**: Critical! Limits which rows are updated
- **Without WHERE**: ALL rows would be updated (dangerous!)

**Safety Best Practices:**
1. Always use WHERE clause
2. Test with SELECT first: `SELECT * FROM products WHERE product_id = 2`
3. Backup data before bulk updates
4. Use transactions for complex updates

**Use Cases:**
- Inventory management (adjusting stock after sales)
- Price updates
- Status changes (order fulfillment)
- Customer information updates
- Batch corrections (fixing data errors)
- Scheduled maintenance tasks

**Expected Output:**
Reduces stock_quantity by 5 for product_id = 2, showing before/after comparison.

---

### **Query 11: UNION - Combining Results**

**Title:** Inventory Alerts using UNION

**SQL Concepts:** `UNION`, Combining Multiple SELECT Statements

**How It Works:**
```sql
SELECT 
    product_name,
    'Low Stock' as alert_type,
    stock_quantity as metric
FROM products
WHERE stock_quantity < 100

UNION

SELECT 
    product_name,
    'High Value' as alert_type,
    price as metric
FROM products
WHERE price > 200
ORDER BY alert_type, metric DESC
```

**Explanation:**
- **UNION**: Combines results from multiple SELECT statements
- **Requirements**: Same number of columns, compatible data types
- **Removes duplicates**: Use `UNION ALL` to keep duplicates
- **Different criteria**: First query finds low stock, second finds expensive items
- **Single result set**: Both alerts in one report

**UNION vs. UNION ALL:**
- **UNION**: Removes duplicate rows (slower)
- **UNION ALL**: Keeps all rows including duplicates (faster)

**Use Cases:**
- Alert/notification systems (combining different alert types)
- Reporting from multiple similar tables
- Combining data from different time periods
- Multi-criteria searches
- Dashboard widgets (different KPIs in one view)
- Email digest compilations

**Expected Output:**
Single list showing both low-stock items and high-value products for management review.

---

## 🎯 SQL Concepts Summary

### **Basic Operations**
- ✅ CREATE TABLE, INSERT, UPDATE
- ✅ SELECT, FROM, WHERE, ORDER BY, LIMIT

### **Intermediate**
- ✅ GROUP BY, HAVING
- ✅ Aggregate functions (COUNT, SUM, AVG, MAX)
- ✅ INNER JOIN, LEFT JOIN
- ✅ CASE WHEN (conditional logic)

### **Advanced**
- ✅ Window Functions (RANK, ROW_NUMBER, SUM OVER)
- ✅ PARTITION BY
- ✅ Common Table Expressions (WITH)
- ✅ UNION

### **Special Features**
- ✅ COALESCE (NULL handling)
- ✅ STRFTIME (date functions)
- ✅ String concatenation (||)
- ✅ Window frames (ROWS BETWEEN)
- ✅ Calculated fields
- ✅ Multiple table joins

---

## 🚀 Getting Started

### **Prerequisites**
- Python 3.7+
- SQLite3 (included with Python)
- Pandas library
- Jupyter Notebook

### **Installation**
```bash
pip install pandas jupyter
```

### **Running the Guidebook**
```bash
jupyter notebook SQL_Guidebook.ipynb
```

---

## 📊 Sample Outputs

### Database Summary
- **Total Customers:** 8
- **Total Products:** 10
- **Total Orders:** 10
- **Total Revenue:** $5,028.80
- **Average Order Value:** $502.88

---

## 💡 Tips for SQL Success

1. **Always test with SELECT before UPDATE/DELETE**
2. **Use table aliases for readability**
3. **Understand JOIN types** (INNER vs. LEFT vs. RIGHT)
4. **HAVING filters groups, WHERE filters rows**
5. **Window functions don't reduce rows** (unlike GROUP BY)
6. **CTEs improve query readability** for complex logic
7. **Index frequently queried columns** for performance
8. **Comment your queries** for future reference

---

## 🎓 Learning Path

**Beginner → Intermediate → Advanced**

1. Start with Queries 1-2 (basics)
2. Progress to 3-5 (joins)
3. Master 6 (conditional logic)
4. Tackle 7-9 (window functions)
5. Explore 10-11 (data manipulation)

---

## 📚 Additional Resources

- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [PostgreSQL Tutorial](https://www.postgresql.org/docs/)
- [Mode Analytics SQL Tutorial](https://mode.com/sql-tutorial/)
- [LeetCode SQL Practice](https://leetcode.com/problemset/database/)

---



