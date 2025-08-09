# Uncovering-Ecommerce-Insights-with-SQL

## 📖 About The Project
This project involves a deep dive into the [Brazilian E-commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce?select=olist_customers_dataset.csv). The goal is to use SQL to analyze real-world e-commerce data to answer critical business questions related to sales performance, customer behavior, and operational efficiency. This repository contains the raw data and the SQL scripts used for the analysis.

***

### ❓ Key Business Questions Addressed
1.  What are the top 10 best-selling product categories?
2.  What is the overall sales trend by month?
3.  Which Brazilian states have the most customers?
4.  What is the average time it takes for a new customer to make their second purchase?
5.  What is the average processing time versus shipping time, and how does this vary by seller state?
6.  Which product categories are most frequently purchased together?

***

### 🛠️ Tools Used
* **Database:** SQLite
* **SQL Client:** DB Browser for SQLite / DBeaver
* **Version Control:** Git & GitHub

***

### 📊 SQL Analysis & Findings

#### 1. Top 10 Best-Selling Product Categories
This query identifies which product categories generate the most orders, helping the business focus marketing and inventory efforts.

**Finding:** 

```sql
SELECT
    pc.product_category_name_english AS 'product_category',
    COUNT(oi.order_id) AS number_of_orders
FROM order_items AS oi
JOIN products AS p ON oi.product_id = p.product_id
JOIN product_category AS pc ON p.product_category_name = pc.product_category_name
GROUP BY pc.product_category_name_english
ORDER BY number_of_orders DESC
LIMIT 10;
```

**Results:**

|      | product_category          | number_of_orders |
| :--- | :------------------------ | :--------------- |
| 1    | bed_bath_table            | 11115            |
| 2    | health_beauty             | 9670             |
| 3    | sports_leisure            | 8641             |
| 4    | furniture_decor           | 8334             |
| 5    | computers_accessories     | 7827             |
| 6    | housewares                | 6964             |
| 7    | watches_gifts             | 5991             |
| 8    | telephones                | 4545             |
| 9    | garden_tools              | 4347             |
| 10   | auto                      | 4235             |

#### 2. Overall Monthly Sales Trend
This query calculates total revenue per month to understand seasonality and growth over time.

**Finding:** 

```sql
SELECT
    strftime('%Y-%m', o.order_purchase_timestamp) AS sales_month,
    SUM(p.payment_value) AS total_revenue
FROM orders AS o
JOIN order_payments AS p ON o.order_id = p.order_id
WHERE o.order_status != 'canceled'
GROUP BY sales_month
ORDER BY sales_month;
```

**Results:**


#### 3. Customer Distribution by State
This query identifies the geographic concentration of the customer base.

**Finding:** 


```sql
SELECT
    customer_state,
    COUNT(DISTINCT customer_unique_id) AS customer_count
FROM customers
GROUP BY customer_state
ORDER BY customer_count DESC
LIMIT 10;
```

**Results:**

#### 4. Average Time to Second Purchase
This query analyzes customer retention by calculating how long it takes for a customer to return for a second purchase.

**Finding:** 

```sql
WITH RankedOrders AS (
    SELECT
        c.customer_unique_id,
        o.order_purchase_timestamp,
        ROW_NUMBER() OVER(PARTITION BY c.customer_unique_id ORDER BY o.order_purchase_timestamp) as order_rank
    FROM orders AS o
    JOIN customers AS c ON o.customer_id = c.customer_id
    WHERE o.order_status = 'delivered'
),
CustomerPurchaseTimes AS (
    SELECT
        customer_unique_id,
        MAX(CASE WHEN order_rank = 1 THEN order_purchase_timestamp END) as first_order_date,
        MAX(CASE WHEN order_rank = 2 THEN order_purchase_timestamp END) as second_order_date
    FROM RankedOrders
    GROUP BY customer_unique_id
)
SELECT
    AVG((strftime('%s', second_order_date) - strftime('%s', first_order_date)) / 86400.0) as avg_days_between_purchases
FROM CustomerPurchaseTimes
WHERE second_order_date IS NOT NULL;
```

**Results:**

#### 5. Logistics Funnel: Processing vs. Shipping Time
This query breaks down the delivery timeline to see if delays are caused by sellers (processing) or carriers (shipping).

**Finding:** 

```sql
WITH DeliveryTimes AS (
    SELECT
        s.seller_state,
        (strftime('%s', o.order_delivered_carrier_date) - strftime('%s', o.order_approved_at)) / 86400.0 as processing_days,
        (strftime('%s', o.order_delivered_customer_date) - strftime('%s', o.order_delivered_carrier_date)) / 86400.0 as shipping_days
    FROM orders AS o
    JOIN order_items AS oi ON o.order_id = oi.order_id
    JOIN sellers AS s ON oi.seller_id = s.seller_id
    WHERE o.order_status = 'delivered'
      AND o.order_approved_at IS NOT NULL
      AND o.order_delivered_carrier_date IS NOT NULL
      AND o.order_delivered_customer_date IS NOT NULL
)
SELECT
    seller_state,
    COUNT(*) as total_orders,
    AVG(processing_days) as avg_processing_days,
    AVG(shipping_days) as avg_shipping_days
FROM DeliveryTimes
GROUP BY seller_state
ORDER BY total_orders DESC;
```

**Results:**


#### 6. Market Basket Analysis: Products Bought Together
This query finds which product categories are most frequently purchased together, which can inform cross-selling and product bundling strategies.

**Finding:** 

```sql
WITH CategoryPairs AS (
    SELECT
        oi1.order_id,
        t1.product_category_name_english AS category_1,
        t2.product_category_name_english AS category_2
    FROM order_items AS oi1
    JOIN order_items AS oi2 ON oi1.order_id = oi2.order_id AND oi1.product_id < oi2.product_id
    JOIN products AS p1 ON oi1.product_id = p1.product_id
    JOIN product_category_name_translation AS t1 ON p1.product_category_name = t1.product_category_name
    JOIN products AS p2 ON oi2.product_id = p2.product_id
    JOIN product_category_name_translation AS t2 ON p2.product_category_name = t2.product_category_name
)
SELECT
    category_1,
    category_2,
    COUNT(*) as pair_count
FROM CategoryPairs
GROUP BY category_1, category_2
ORDER BY pair_count DESC
LIMIT 10;
```

**Results:**


### 🚀 Conclusions & Recommendations



