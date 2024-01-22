# Pizza Place Sales

**By Tal Kadosh**

**2024-01-17**

-----------------------------------------------------------------------
#### Table of Contents

1.  **Introduction**

    1.1 The business task

    1.2 The leading questions

2.  **Data background**

    2.1 Data source

    2.2 Data credibility by "ROCCC" model

    2.3 Data License

3.  **Data processing**

4.  **Data analysis**

5.  **Data visualization**

6.  **Findings**

    5.1 Conclusion of my analysis

    5.2 Recommendations based on my data analysis

------------------------------------------------------------------------

### Introduction

#### 1.1: The business task


#### 1.2: The leading questions

1. How many customers do we have each day? Are there any peak hours?

2. How many pizzas are typically in an order? Do we have any bestsellers?

3. How much money did we make this year? Can we indentify any seasonality in the sales?

4. Are there any pizzas we should take of the menu, or any promotions we could leverage?

------------------------------------------------------------------------

### Data background

#### 2.1: Data source

I will utilize Cyclistic's publicly accessible historical data stored on their cloud servers for this project. The data can be accessed [Here](https://divvy
tripdata.s3.amazonaws.com/index.html). My focus will be on the data spanning from December 2022 to November 2023, organized into 12 separate CSV files, each
representing a specific month of the year.

#### 2.2: Data credibility by "ROCCC" model

1.  **Reliability:** 

2.  **Original:** The data is verified by its original source.

3.  **Comprehensiveness:** It encompasses all essential information to address the queries.

4.  **Current:** The data is pertinent to the previous year.

5.  **Cited:** The data is trustworthy and referenced from the source in the preceding paragraph.

#### 2.3: Data License

The data is maintained and made available by [Motivate International Inc](https://divvybikes.com/data-license-agreement).

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
   
The first query retrieves all distinct rows to identify any duplications. The second query checks for null values. After examining both, I discovered that
the data in this table is suitable for use, with consistent row counts before and after these checks — 48,620 rows.

```sql         
SELECT
  DISTINCT *
FROM
  `tangential-box-405116.PizzaPlaceSales.order_details`
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

The same process was applied to the table as before, yielding consistent results. The data is deemed suitable, maintaining an unchanged number of rows both
before and after the checks — 21,350 rows.

```sql         
SELECT
  DISTINCT *
FROM
  `tangential-box-405116.PizzaPlaceSales.orders`
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
  `tangential-box-405116.PizzaPlaceSales.pizza_prices`
```

4. **menu:**

In this table, there are 33 rows. Upon inspection, I confirmed the absence of any null values or duplications. Subsequently, I created a new table and
modified its name. Each column header was changed to a meaningful name, and I removed the first row containing random values that could potentially disrupt
subsequent analyses.

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
We observe a total of 21,350 orders throughout the entire year, resulting in an average of 59.64 orders per day.

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
The results of this query reveal the top 5 peak hours as follows: 12:00, 13:00, 18:00, 17:00, and 19:00, listed in ascending order.

In order to answer the second question, **"How many pizzas are typically in an order? Do we have any bestsellers?"**, I wrote two another queris: 

```sql
SELECT
  ROUND (AVG(quantity), 2) AS average_pizzas_per_order
FROM
  `PizzaPlaceSales.order_details`
```
The results to this query show us that the average number of pizzas for each delivery are 1.02.

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
LIMIT 5;
```
After running this query, we found out that the bestsellers pizzas are:
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

The next question was **"How much money did we make this year? Can we indentify any seasonality in the sales?"**. in order to answer it, I wrote those querys:

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
The total revenue of the entire year is 817,860.05$.

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
After running this query, we recive those results:
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

In order to answer the last question, **"Are there any pizzas we should take of the menu, or any promotions we could leverage?"**, I ran this query:

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
After running this query, we found out that the least ordered pizzas are:
1. the_greek_xxl - 28
2. green_garden_l - 95
3. ckn_alfredo_s - 96
4. calabrese_s - 99
5. mexicana_s _ 162





------------------------------------------------------------------------

### Data visualization


------------------------------------------------------------------------

### Findings

#### 5.1: Conclusion of my analysis

Throughout the analysis of bike-sharing data in Chicago, several key conclusions emerge:

1. **Ride Duration Differences:**
Casual riders, with an average ride time of 25 minutes, showcase a more leisure-oriented and variable usage pattern compared to members, who have an average
ride time of 12.5 minutes, suggesting utilitarian and routine-based behavior.

2. **Weekly Riding Patterns:**
Casual riders exhibit a substantial increase in average ride time during weekends, with an almost 8-minute difference from midweek, while members maintain a
more consistent pattern, indicating potential recreational usage by casual riders during weekends.

3. **Total Ride Counts:**
Members significantly outnumber casual riders in total ride counts (596,389 vs. 410,549), highlighting the higher frequency of bike-sharing usage among
members, likely for commuting or routine activities.

4. **Seasonal Trends:**
Both casual and member riders show a preference for bike-sharing during the warmer months, with a larger seasonal gap in casual riders' usage patterns,
indicating heightened sensitivity to seasonal variations and recreational activities.

5. **Preferred Riding Days:**
Members favor weekdays, particularly Monday through Thursday, suggesting utilitarian usage for commuting, while casual riders prefer the latter part of the
week, especially weekends, indicating a more leisure-oriented and recreational approach.

6. **Station Preferences:**
Top stations for casual riders, such as "Streeter Dr & Grand Ave", may be strategically located near tourist attractions, cultural landmarks, or recreational
areas, while the more uniform distribution of top stations for members suggests even utilization across various neighborhoods for routine activities.

These conclusions provide valuable insights for targeted marketing, service enhancements, and resource allocation to better meet the distinct needs and
behaviors of casual and member riders in the Chicago bike-sharing system.

#### 5.2: Recommendations based on my data analysis

1. **How do annual members and casual riders use Cyclistic bikes dierently?**
Annual members exhibit more consistent and utilitarian usage, likely for commuting or routine activities, with shorter average ride times (12.5 minutes).
Casual riders, with longer average ride times (25 minutes), show a more variable and leisure-oriented pattern.

2. **Why would casual riders buy Cyclistic annual?**
Position Cyclistic annual memberships as a cost-effective and convenient option for casual riders who frequent the service, especially during the summer
peak. Emphasize benefits like unlimited rides and discounts for longer-term commitments.

3. **How can Cyclistic use digital media to infuence casual riders to become members?**
Launch targeted digital media campaigns highlighting the advantages of Cyclistic annual memberships, such as cost savings and seamless access. Leverage
social media, online advertising, and influencers to reach casual riders during the summer peak, emphasizing the value and convenience of becoming annual
members.
