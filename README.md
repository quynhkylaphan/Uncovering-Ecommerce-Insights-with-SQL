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

