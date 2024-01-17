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

4.  **Data analysis and visualization**

5.  **Findings**

    5.1 Conclusion of my analysis

    5.2 Recommendations based on my data analysis

------------------------------------------------------------------------

### Introduction

#### 1.1: The business task


#### 1.2: The leading questions


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
);
```

------------------------------------------------------------------------

### Data analysis and visualization


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
