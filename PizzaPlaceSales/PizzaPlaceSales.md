<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta property="og:title" content="Pizza Place Sales Project">
  <meta property="og:description" content="Exploring the Perfect Fusion of SQL and Pizza Analytics">
  <meta property="og:image" content="URL to an image representing your project">
  <meta property="og:url" content="URL to the project or documentation">

# Pizza Place Sales

**By Tal Kadosh**

**2024-01-17**

-----------------------------------------------------------------------
#### Table of Contents

1.  **Introduction**

    1.1 The business task

    1.2 The leading questions

2.  **Data background**

3.  **Data processing**

4.  **Data analysis**

5.  **Findings**

    5.1 Analysis Findings

    5.2 Recommendations based on the analysis

------------------------------------------------------------------------

### Introduction

#### 1.1: The business task

The business task is to conduct a comprehensive analysis of the pizza place's operations and customer interactions. This involves answering key questions
such as understanding the daily customer count, identifying peak hours to optimize staff and resources, evaluating the average number of pizzas per order,
and determining bestsellers to inform inventory and marketing strategies. Additionally, the financial performance over the year needs to be assessed to
identify any patterns or seasonality in sales. Finally, the analysis aims to uncover opportunities for menu optimization by identifying underperforming
pizzas and exploring potential promotions to drive customer engagement and boost sales.

The overarching goal is to gain actionable insights that will enhance operational efficiency, improve customer satisfaction, and contribute to the overall
success of the pizza business.

#### 1.2: The leading questions

1. How many customers do we have each day? Are there any peak hours?

2. How many pizzas are typically in an order? Do we have any bestsellers?

3. How much money did we make this year? Can we indentify any seasonality in the sales?

4. Are there any pizzas we should take of the menu, or any promotions we could leverage?

------------------------------------------------------------------------

### Data background

The data can be accessed [Here](https://www.mavenanalytics.io/data-playground?page=4&pageSize=5), "Pizza Place Sales". My focus will be on the data spanning
from January 2015 to December 2015.

------------------------------------------------------------------------

### Data processing

This dataset contains four tables in CSV format that are connected to each other by primary keys.

1. **order_details:**

    order_details_id: A unique identifier for each order detail entry.

    order_id: A unique identifier for each order.
   
    pizza_id: A unique identifier for each pizza.
   
    quantity: Represents the quantity of the specified pizza in the order detail.
   
2. **orders:**

   order_id: A unique identifier for each order.

   date: The data when the order was placed.

   time: The time when the order was placed.
   
3. **pizza_prices:**

   pizza_id: A unique identifier for each pizza.

   pizza_type_id: Identifies the type of pizza.

   size: Represents the size of the pizza.
   
   price: Indicates the price associated with the specific pizza size.

4. **menu:**
   
   pizza_type_id: Identifies the type of pizza.
   
   name: The name of the pizza.
   
   category: Specifies the category to which the pizza belongs.
   
   ingredients: Lists the ingredients used in the pizza.
   

### Data Cleaning

1. **order_details:**
   
The initial query retrieves all distinct rows to identify any duplications, while the second query checks for null values. Upon examining both, I determined
that the data in this table is suitable for use, maintaining consistent row counts before and after these checks — 48,620 rows.

```sql         
SELECT
  DISTINCT *
FROM
  `PizzaPlaceSales.order_details`
```
```sql 
SELECT *
FROM `PizzaPlaceSales.order_details`
WHERE order_details_id IS NULL
   OR order_id IS NULL
   OR pizza_id IS NULL
   OR quantity IS NULL;
```

2. **orders:**

The identical process was applied to the table as previously, producing consistent results. The data remains deemed suitable, with an unaltered number of
rows both before and after the checks—21,350 rows.

```sql         
SELECT
  DISTINCT *
FROM
  `PizzaPlaceSales.orders`
```
```sql 
SELECT *
FROM `PizzaPlaceSales.orders`
WHERE order_id IS NULL
   OR date IS NULL
   OR time IS NULL
```

3. **pizza_prices:**

In this table, there are only 96 rows. Upon inspection, I confirmed the absence of any null values. Additionally, I executed a query to check for
duplications, and the result remained consistent.

```sql         
SELECT
  DISTINCT *
FROM
  `PizzaPlaceSales.pizza_prices`
```

4. **menu:**

In this table, there are 33 rows. Upon inspection, I verified the absence of any null values or duplications. Following this, I created a new table, modified
its name, and renamed each column header to a meaningful identifier. Additionally, I eliminated the first row, which contained random values that could
potentially interfere with subsequent analyses.

```sql         
CREATE TABLE PizzaPlaceSales.menu AS
SELECT
  string_field_0 AS pizza_type_id,
  string_field_1 AS name,
  string_field_2 AS category,
  string_field_3 AS ingredients
FROM (
  SELECT
    * EXCEPT(row_number)
  FROM (
    SELECT
      *,
      ROW_NUMBER() OVER () AS row_number
    FROM
      `PizzaPlaceSales.pizza_types`
  )
  WHERE
    row_number > 1
)
```

------------------------------------------------------------------------

### Data analysis

The initial inquiry from the customer is, **"How many customers do we have each day? Are there any peak hours?"** To address these questions, I formulated the
following two queries:

```sql
SELECT
   COUNT(date) AS total_orders,
   ROUND(COUNT(date) / COUNT(DISTINCT date), 2) AS daily_orders_avg
FROM
   `PizzaPlaceSales.orders`
```
We recorded a total of 21,350 orders over the entire year, averaging 59.64 orders per day.

```sql
SELECT
  EXTRACT(HOUR FROM time) AS hour_of_day,
  COUNT(*) AS orders_count
FROM
  `PizzaPlaceSales.orders`
GROUP BY
  hour_of_day
ORDER BY
  orders_count DESC
LIMIT 5;
```
```SQL
SELECT
  EXTRACT(HOUR FROM time) AS hour_of_day,
  COUNT(*) AS orders_count
FROM
  `PizzaPlaceSales.orders`
GROUP BY
  hour_of_day
ORDER BY
  orders_count ASC
LIMIT 5;
```
The result of the first query reveal the top 5 hours as follows: 12:00, 13:00, 18:00, 17:00, and 19:00.
The result of the second query reveal the least busy 5 hours as follows: 09:00, 10:00, 23:00, 22:00, and 21:00.

In order to answer the second question, **"How many pizzas are typically in an order? Do we have any bestsellers?"**, I wrote two another queris: 

```sql
SELECT
  ROUND (AVG(quantity), 2) AS average_pizzas_per_order
FROM
  `PizzaPlaceSales.order_details`
```
The results of this query show us that the average number of pizzas for each delivery is 1.02.

```sql
SELECT
  pizza_id,
  SUM(quantity) AS total_sold
FROM
  `PizzaPlaceSales.order_details`
GROUP BY
  pizza_id
ORDER BY
  total_sold DESC
LIMIT 10;
```
After executing this query, we discovered the best-selling pizzas to be:
1. big_meat_s - 1914
2. thai_ckn_l - 1410
3. five_cheese_l - 1409
4. four_cheese_l - 1316
5. classic_dlx_m -1181
6. spicy_ital_l - 1109
7. hawaiian_s - 1020
8. southw_ckn_l - 1016
9. bbq_ckn_l - 992
10. bbq_ckn_m - 956

The next question posed was, **"How much money did we make this year? Can we identify any seasonality in the sales?"**. To address this inquiry, I formulated
the following queries:

```sql
SELECT
  ROUND(SUM(od.quantity * pp.price),2) AS total_revenue
FROM
  `PizzaPlaceSales.pizza_prices` pp
JOIN
  `PizzaPlaceSales.order_details` od
ON
  od.pizza_id = pp.pizza_id
```
The total revenue for the entire year is $817,860.05.

```sql
SELECT
  EXTRACT(MONTH FROM date) AS month,
  EXTRACT(YEAR FROM date) AS year,
  COUNT(*) AS monthly_orders
FROM
  `PizzaPlaceSales.orders`
GROUP BY
  year, month
ORDER BY
  year, month;
```
After running this query, we receive the following results:
1. January - 1845
2. February - 1685
3. March - 1840
4. April - 1799
5. May - 1853
6. June - 1773
7. July - 1935
8. August - 1841
9. September - 1661
10. October - 1646
11. November - 1792
12. December -1680

In addressing the final question, **"Are there any pizzas we should remove from the menu, or any promotions we could leverage?"**, I executed the following
query:

```sql
SELECT
  pizza_id,
  SUM(quantity) AS total_sold
FROM
  `PizzaPlaceSales.order_details`
GROUP BY
  pizza_id
ORDER BY
  total_sold ASC
LIMIT 5;
```
After executing this query, it was determined that the pizzas with the lowest order volumes are:
1. the_greek_xxl - 28
2. green_garden_l - 95
3. ckn_alfredo_s - 96
4. calabrese_s - 99
5. mexicana_s _ 162

To better understand the market needs, I analyzed additional ordering patterns.

```sql
WITH PizzaSizeCounts AS (
  SELECT
    pp.size,
    COUNT(od.pizza_id) AS size_count
  FROM
    `PizzaPlaceSales.pizza_prices` pp
  LEFT JOIN
    `PizzaPlaceSales.order_details` od
  ON
    pp.pizza_id = od.pizza_id
  GROUP BY
    pp.size
),
TotalPizzaCount AS (
  SELECT
    COUNT(*) AS total_count
  FROM
    `PizzaPlaceSales.order_details`
)
SELECT
  psc.size,
  psc.size_count,
  ROUND((psc.size_count / tpc.total_count) * 100, 2) AS percentage
FROM
  PizzaSizeCounts psc
CROSS JOIN
  TotalPizzaCount tpc;
```
This query provides insights into customer preferences concerning the size of their pizzas:
1. L, 18526 Units, 38.1%
2. M, 15385 Units, 31.64%
3. S, 14137 Units, 29.08%
4. XL, 544 Units, 1.12%
5. XXL, 28 Units, 0.06%

```sql
WITH PizzaCounts AS (
  SELECT
    m.category,
    COUNT(od.pizza_id) AS pizza_count
  FROM
    `PizzaPlaceSales.menu` m
  LEFT JOIN
    (
      SELECT
        LEFT(od.pizza_id, LENGTH(od.pizza_id) - 2) AS pizza_id,
        od.quantity
      FROM
        `PizzaPlaceSales.order_details` od
    ) od
  ON
    m.pizza_type_id = od.pizza_id
  GROUP BY
    m.category
)

SELECT
  category,
  pizza_count,
  ROUND(pizza_count / NULLIF((SELECT COUNT(*) FROM `PizzaPlaceSales.order_details`), 0), 4) * 100 AS percentage
FROM
  PizzaCounts;
  ```
This query provides insights into customer preferences for pizza types:
1. Veggie, 11449 Units, 23.55%
2. Classic, 14007 Units, 28.81%
3. Supreme, 11777 Units, 24.22%
4. Chicken, 10815 Units, 22.24%

------------------------------------------------------------------------

### Findings

#### 5.1: Analysis Findings

Throughout the analysis, several key findings emerge:

1. **Order and Sales Pattern-** The business observes a total of 21,350 orders throughout the year, averaging approximately 59.64 orders per day.
The top 5 peak hours for orders are identified as 12:00, 13:00, 18:00, 17:00, and 19:00.
Conversely, the least busy 5 hours are at 09:00, 10:00, 23:00, 22:00, and 21:00.

2. **Bestsellers and Revenue-** The best-selling pizzas include "big_meat_s", "thai_ckn_l", "five_cheese_l", and others. The total revenue for the entire
year amounted to $817,860.05.

3. **Monthly Sales-** Monthly sales fluctuate, with July having the highest pizza sales (1935 units) and September having the lowest (1661 units).

4. **Least Ordered Pizzas-** The pizzas with the least orders include "the_greek_xxl," "green_garden_l," "ckn_alfredo_s," "calabrese_s," and "mexicana_s."

5. **Customer Preferences-** Size Preferences: Large (L) pizzas are the most popular, accounting for 38.1% of sales, followed by Medium (M) at 31.64%, Small
(S) at 29.08%, XL at 1.12%, and XXL at 0.06%.

6. **Type Preferences-** The most preferred pizza types by the following order are "Classic", "Supreme," "Veggie" and "Chicken".

#### 5.2: Recommendations based on the analysis
After presenting the conclusions, here are some operational suggestions that can aid in their implementation:

1. **Happy Hour Promotion:**
   Introduce a Happy Hour promotion during the identified least busy hours in the late evening and in the early morning to attract more customers. Offer
   discounted prices, bundled deals, or special promotions to incentivize orders during traditionally slower times.

2. **Seasonal Specials:**
   Capitalize on the observed increase in sales during the summer months by introducing seasonal specials or summer-themed pizzas. Highlight these offerings
   through targeted marketing to boost sales during peak seasons.

3. **Fall Sale Campaign:**
   Implement a fall sale campaign during the weaker months from September to December. Offer promotions, discounts, or exclusive deals to stimulate customer
   interest and increase sales during traditionally slower periods.

4. **Menu Optimization:**
   Based on the analysis of bestsellers and least ordered pizzas, optimize the menu by emphasizing popular items and potentially removing or revamping less
   popular ones. This can streamline operations and improve overall customer satisfaction.

5. **Efficiency during Peak Hours:**
   Optimize operations during peak hours to maximize efficiency. Consider streamlining processes, ensuring sufficient staff during busy times, and
   implementing technologies to speed up order processing.
