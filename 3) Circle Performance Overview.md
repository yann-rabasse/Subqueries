# Introduction

## Circle Presentation

Circle is a French brand of eco-responsible and ethical sportswear.
- They produce high performance and comfortable clothing for running, yoga and training.
- They focus on circularity by selling clothing made from recycled materials that is collected and processed 100% in Europe.
- They sell their products either to B2C customers through their website or to B2B customers through their sales team.

It is essential to properly monitor stocks to avoid shortages, prioritise production and avoid selling out-of-stock products to B2B customers.

# Objective

The objective of this exercise is to query Circle’s stock data in SQL to perform relevant analyses. In this exercise, you will be able to:
- improve your SQL skills
- answer specific business questions by joining information from different tables and performing intermediate calculations

## Sources

[Import circle_stock](https://docs.google.com/spreadsheets/d/19cDDybWRQrWkGpGJfL6Yp63zkHXEtQRn_SNXySUm6sI/edit#gid=0)

[Import circle_sales](https://docs.google.com/spreadsheets/d/19cDDybWRQrWkGpGJfL6Yp63zkHXEtQRn_SNXySUm6sI/edit#gid=1009765988)

[Import circle_parcel_product](https://docs.google.com/spreadsheets/d/1qhHVVdi6Z8PnD62QJtbnEjyX8d35sho573XwoasrYBk/edit#gid=1276048679)

[Import circle_parcel](https://docs.google.com/spreadsheets/d/1qhHVVdi6Z8PnD62QJtbnEjyX8d35sho573XwoasrYBk/edit#gid=0)


## 1) Table Exploration

1) First, explore the tables and remind yourself how they are structured. What data do each of the columns contain?

2) Data Analysis

  2.1) Find the models (‘model_name’) that have never been sold but are still listed in the inventory catalogue (‘circle_stock’).

Note: You will need to combine information from circle_sales and circle_stock tables.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH stock_product_id AS (
SELECT
  *,
  CASE
    WHEN size IS NULL AND color IS NOT NULL THEN CONCAT(model,"_",color)
    WHEN size IS NULL AND color IS NULL THEN model
    WHEN size IS NOT NULL AND color IS NULL THEN CONCAT(model,"_",size)
    ELSE CONCAT(model,"_",color,"_",size)
  END AS product_id
FROM `course15.circle_stock`
)

, sales_stock_table AS (
SELECT
  stock_table.*,
  SUM(qty) AS total_sold
FROM stock_product_id AS stock_table
LEFT JOIN `course15.circle_sales` AS sale
  USING (product_id)
GROUP BY
  stock_table.product_id,
  sale.product_id,
  model,
  model_name,
  color,
  color_name,
  size,
  `new`,
  forecast_stock,
  stock,
  price
)

SELECT
  DISTINCT model_name
FROM sales_stock_table
WHERE total_sold IS NULL
```


</details>


  2.2) For the different priority levels and transporters, evaluate the monthly parcel delivery performance in terms of:

- Average time between purchase and delivery
- The shortest and the longest purchase to delivery times
- Number of products in parcel

Note: 

You can follow these steps:

- Join parcel and parcel_product tables together to retrieve the relevant information from each table
- Transform the date columns into date format to be able to perform calculations on them
- Calculate the difference between date_purchase and date_delivery columns
- Extract month from date_purchase to be able to group by month later on
- Transform the priority field by adding a number as prefix i.e. “1 - High”, “2 - Medium”, “3 - Low” to be able to sort the information by priority level in the final results
- Calculate the delivery time KPIs and the number of products in parcel per month, priority level and transporter


What would you recommend to improve the delivery performance? Are there some transporters that perform better? Is there a considerable difference between the different priority levels?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH parcel_product_merge AS (
SELECT
  parcel.parcel_id,
  transporter,
  priority,
  PARSE_DATE("%B %e, %Y", date_purchase) AS date_purchase,
  PARSE_DATE("%B %e, %Y", date_shipping) AS date_shipping,
  PARSE_DATE("%B %e, %Y", date_delivery) AS date_delivery,
  parcel_product.model_name,
  qty
FROM `course15.cc_parcel` AS parcel
LEFT JOIN `course15.cc_parcel_product` AS parcel_product
  USING (parcel_id)
WHERE date_cancelled IS NULL
)

, time_calculation AS (
SELECT
  *,
  EXTRACT(MONTH FROM date_purchase) AS purchase_month,
  DATE_DIFF(date_delivery,date_purchase,DAY) AS purchase_to_delivery_time
FROM parcel_product_merge
)

SELECT
  purchase_month,
  CASE
    WHEN priority = "Low" THEN "3 - Low"
    WHEN priority = "Medium" THEN "2 - Medium"
    WHEN priority = "High" THEN "1 - High"
  END AS priority,
  transporter, --to remove if we want to analyze only the priority granularity
  ROUND(AVG(purchase_to_delivery_time),1) AS average_delivery_time,
  MIN(purchase_to_delivery_time) AS minimum_delivery_time,
  MAX(purchase_to_delivery_time) AS maximum_delivery_time,
  COUNT(parcel_id) AS number_parcels,
  SUM(qty) AS total_products,
  ROUND(SAFE_DIVIDE(SUM(qty), COUNT(parcel_id)),1) AS number_products_per_parcel
FROM time_calculation
GROUP BY
  purchase_month,
  priority,
  transporter --to remove if we want to analyze only the priority granularity
ORDER BY purchase_month, priority
```


</details>


3) Calculate the total revenue generated from sales and estimate if the average quantity of products sold per day varies from one month to another.

Note: 

The difficulty here is that the price information is not in the sales table, but in the stock table. You need to:

- Create the product_id column in stock table by concatenating model, size and color as
- Perform a JOIN between the sales and stock tables on product_id
- Extract month from date_date to be able to group by month later on
- Calculate the total revenue and the number of products sold per day across several months



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH stock_product_id AS (
SELECT
  price,
  CASE
    WHEN size IS NULL AND color IS NOT NULL THEN CONCAT(model,"_",color)
    WHEN size IS NULL AND color IS NULL THEN model
    WHEN size IS NOT NULL AND color IS NULL THEN CONCAT(model,"_",size)
    ELSE CONCAT(model,"_",color,"_",size)
  END AS product_id
FROM `course15.circle_stock`
)

, detailed_prices AS (
SELECT
  date_date AS purchase_date,
  EXTRACT(MONTH FROM date_date) AS purchase_month,
  product_id,
  qty,
  price
FROM `course15.circle_sales`
LEFT JOIN stock_product_id
  USING (product_id)
)

SELECT
 purchase_month,
  SUM(qty) AS total_products_sold,
  SUM(price) AS total_revenue,
  ROUND(SAFE_DIVIDE(SUM(price), SUM(qty)), 1) AS average_basket,
  ROUND(SAFE_DIVIDE(SUM(qty), COUNT(DISTINCT(purchase_date))),0) AS average_products_sold_per_active_day
FROM detailed_prices
GROUP BY purchase_month
ORDER BY purchase_month
```


</details>
