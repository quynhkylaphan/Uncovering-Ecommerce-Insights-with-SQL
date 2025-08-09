# Uncovering-Ecommerce-Insights-with-SQL

## 📖 About The Project
This project involves a deep dive into the [Brazilian E-commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce?select=olist_customers_dataset.csv). The goal is to use SQL to analyze real-world e-commerce data to answer critical business questions related to sales performance, customer behavior, and operational efficiency. This repository contains the raw data and the SQL scripts used for the analysis.

***

### 🛠️ Tools Used
* **Database:** SQLite
* **SQL Client:** DB Browser for SQLite

***

### 📊 SQL Analysis & Findings

#### 1. What are the top 10 best-selling product categories?
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


#### 2. What is the overall sales trend by month?
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

| sales_month | total_revenue      |
| :---------- | :----------------- |
| 2016-09     | 136.23             |
| 2016-10     | 53915.5            |
| 2016-12     | 19.62              |
| 2017-01     | 138119.76          |
| 2017-02     | 289081.01          |
| 2017-03     | 442406.37          |
| 2017-04     | 409846.01          |
| 2017-05     | 588529.96          |
| 2017-06     | 507302.62          |
| 2017-07     | 585331.36          |
| 2017-08     | 667224.47          |
| 2017-09     | 723781.27          |
| 2017-10     | 773104.35          |
| 2017-11     | 1187224.36         |
| 2017-12     | 874962.23          |
| 2018-01     | 1109464.0          |
| 2018-02     | 984790.19          |
| 2018-03     | 1156243.76         |
| 2018-04     | 1157336.33         |
| 2018-05     | 1149483.77         |
| 2018-06     | 1021220.02         |
| 2018-07     | 1047422.72         |
| 2018-08     | 998504.15          |
| 2018-09     | 166.46             |


#### 3. Which Brazilian states have the most customers?
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

| State | Customer Count |
| :---- | :------------- |
| SP    | 40302          |
| RJ    | 12384          |
| MG    | 11259          |
| RS    | 5277           |
| PR    | 4882           |
| SC    | 3534           |
| BA    | 3277           |
| DF    | 2075           |
| ES    | 1964           |
| GO    | 1952           |


#### 4. What is the average time it takes for a new customer to make their second purchase?
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

|    | avg_days_between_purchases |
| :--| :------------------------- |
| 1  | 81.0                       |


#### 5. What is the average processing time versus shipping time, and how does this vary by seller state?
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

| seller_state | total_orders | avg_processing_days | avg_shipping_days |
| :----------- | :----------- | :------------------ | :---------------- |
| SP           | 78585        | 3.0                 | 9.0               |
| MG           | 8601         | 3.0                 | 10.0              |
| PR           | 8485         | 3.0                 | 10.0              |
| RJ           | 4685         | 2.0                 | 9.0               |
| SC           | 3999         | 3.0                 | 11.0              |
| RS           | 2169         | 3.0                 | 8.0               |
| DF           | 883          | 3.0                 | 10.0              |
| BA           | 624          | 3.0                 | 10.0              |
| GO           | 508          | 2.0                 | 10.0              |
| PE           | 445          | 2.0                 | 11.0              |
| MA           | 402          | 5.0                 | 13.0              |
| ES           | 364          | 2.0                 | 10.0              |
| MT           | 144          | 3.0                 | 11.0              |
| CE           | 90           | 3.0                 | 15.0              |
| RN           | 56           | 4.0                 | 9.0               |
| MS           | 50           | 4.0                 | 8.0               |
| PB           | 37           | 3.0                 | 9.0               |
| RO           | 14           | 2.0                 | 15.0              |
| PI           | 11           | 2.0                 | 12.0              |
| SE           | 10           | 1.0                 | 10.0              |
| PA           | 8            | 3.0                 | 10.0              |
| AM           | 3            | 3.0                 | 44.0              |

#### 6. Which product categories are most frequently purchased together?
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

| Category 1            | Category 2                | Pair Count |
| :-------------------- | :------------------------ | :--------- |
| health_beauty         | bed_bath_table            | 213        |
| furniture_decor       | bed_bath_table            | 206        |
| housewares            | bed_bath_table            | 188        |
| sports_leisure        | bed_bath_table            | 149        |
| computers_accessories | bed_bath_table            | 145        |
| health_beauty         | furniture_decor           | 112        |
| watches_gifts         | bed_bath_table            | 109        |
| housewares            | furniture_decor           | 101        |
| health_beauty         | sports_leisure            | 98         |
| computers_accessories | health_beauty             | 97         |


### 🚀 Conclusions & Recommendations



