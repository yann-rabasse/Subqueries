## Sources:

[gwz_sales_17](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_sales_17)

[gwz_ship_17](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_ship_17)

[gwz_campaign_17](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_campaign_17)

[gwz_nps_17](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_nps_17)


## 1) Using Subqueries instead of Saving Intermediate Aggregations into Tables

Previously, whenever we wanted to perform a simple data transformation step, we saved the results of this transformation into an intermediate table.

Subqueries prevent these issues by allowing you to insert an intermediate transformation step directly into another query without the need to create intermediate tables.

1) Are overpopulated with intermediate tables that take up space
2) Are challenging to navigate and have an overcomplex structure

Yesterday, you joined the sales and shipping tables yesterday to create a financial monitoring report. As a reminder, let’s create this report with the gwz tables you just copied into your new dataset.


## Greenweez Finance Monitoring

### 1.1) - Sales and Shipping Join
You joined the sales and shipping tables yesterday to create a financial monitoring report. We will repeat this process now with the gwz tables you copied into your new dataset.

#### STEP 1:

As a reminder, this was done through the following steps:

1) Aggregate gwz_sales_17 on the orders_id and date_date columns.
2) Save the results as a table called gwz_orders.

<img width="949" alt="pfs6r9m3dxqpu0cj5mkp254ow0b0" src="https://github.com/user-attachments/assets/804ab781-119f-4868-af16-be8368e44ecc" />


```
SELECT
  date_date,
  ### Key ###
  orders_id,
  ###########
  ROUND(SUM(turnover),2) AS turnover,
  ROUND(SUM(turnover-purchase_cost),2) AS margin
FROM `course17.gwz_sales_17`
GROUP BY
  date_date,
  orders_id
```


#### STEP 2:

1) Join gwz_orders and gwz_ship_17 on orders_id.
2) Save the results as a table called gwz_orders_operational.

<img width="1054" alt="v0x6ro17ab23quxu984vntk8mcj6" src="https://github.com/user-attachments/assets/aae15f04-cea5-41d3-9ab2-b6ef044f288d" />

```
SELECT
  date_date,
  ### Key ###
  o.orders_id,
  ###########
  -- orders infos --
  o.turnover,
  o.margin,
  -- ship infos --
  sh.shipping_fee,
  sh.log_cost+sh.ship_cost AS operational_cost
FROM `course17.gwz_orders` AS o
LEFT JOIN `course17.gwz_ship_17` AS sh
  ON o.orders_id = sh.orders_id
```

#### Using Subqueries

- Now, instead of running two separate queries and creating the intermediate gwz_orders table, use a WITH AS clause to perform this operation in one query.
- Save the results in a table called gwz_orders_join.

Note: Your SQL query will consist of 2 parts.
1) Use a WITH AS clause to create a temporary table with an alias name.
2) Reference this temporary table by its alias name when joining it with another table.

```
 WITH orders_join AS
   (SELECT
     date_date
     ### Key ###
     ,orders_id
     ###########
     ,ROUND(SUM(turnover),2) AS turnover
     ,ROUND(SUM(turnover-purchase_cost),2) AS margin
   FROM `course17.gwz_sales_17`
   GROUP BY
     date_date
     ,orders_id)

 SELECT
   date_date
   ### Key ###
   ,o.orders_id
   ###########
   -- orders infos --
   ,o.turnover
   ,o.margin
   -- ship infos --
   ,sh.shipping_fee
   ,sh.log_cost+s.ship_cost AS operational_cost
 FROM orders_join AS o
 LEFT JOIN `course17.gwz_ship_17` AS sh ON o.orders_id = sh.orders_id
```



### 1.2) Orders and Ads Join
Next, we will join the tables gwz_orders_join and gwz_campaign_17 to include ads cost in our financial report.

This will require three steps:

#### STEP 1:

1) Aggregate gwz_orders_join on date_date
2) Save this as an intermediate table called gwz_campaign_orders.



#### STEP 2:

Aggregate gwz_campaign_17 on date_date and save it in the gwz_campaign_date intermediate table.



#### STEP 3:

1) Join the gwz_campaign_orders and gwz_campaign_date tables on date_date to gather all order and ads statistics.
2) Instead of running three separate queries and creating the intermediate gwz_campaign_orders and gwz_campaign_date tables, use a WITH AS clause to perform this operation in one query. Save the results in the gwz_campaign_join table.
  - aggregate gwz_orders_join on date_date in a subquery using the WITH AS clause and give it an alias orders_date
  - aggregate gwz_campaign_17 on date_date using a second WITH AS clause and give it an alias campaign_date

3) join orders_date and campaign_date on date_date to aggregate all order and ad statistics for each day

```
WITH orders_date AS
  (SELECT
    ### Key ###
    date_date
    ###########
    ,SUM(turnover) AS turnover
    ,SUM(margin) AS margin
    ,SUM(shipping_fee) AS shipping_fee
    ,SUM(operationnal_cost) AS operational_cost
  FROM `course17.gwz_orders_join`
  GROUP BY
    date_date)

,campaign_date AS
  (SELECT
    ### Key ###
    date_date
    ###########
    ,SUM(ads_cost) AS ads_cost
  FROM `course17.gwz_campaign_17`
  GROUP BY
    date_date)

SELECT
  ### Key ###
  o.date_date
  ###########
  -- orders infos --
  ,o.turnover
  ,o.margin
  ,o.shipping_fee
  ,o.operational_cost
  -- ship infos --
  ,c.ads_cost
FROM orders_date AS o
LEFT JOIN campaign_date AS c USING (date_date)
ORDER BY date_date DESC
```



## 2) Using Subqueries instead of Saving Simple Intermediate Transformations into Tables

Often, when we perform simple successive transformations, we store the results of each transformation in a single table. If the transformations are not too complex, it is better to use subqueries instead.

#### 2.1) Margin Calculation

We want to calculate sales margin metrics from the gwz_sales_17 table. We are interested in three metrics: margin, margin_percent, and margin_level.
- margin_level identifies products with a low or high margin, i.e., less than 5% or more than 40%. The metrics need to be calculated successively. Instead of using successive tables, we will use subqueries.
- calculate the three metrics margin, margin_percent and margin_level from the gwz_sales_17 table. Instead of creating intermediate tables, use the WITH AS clause to perform this operation in one query.

Calculate the margin_percent based on the margin and the turnover.

To define the margin_level, use the following:

- If margin_percent ≥40% , then “High”.
- If margin_percent <5%, then “Low”.
- If margin_percent ≥5% AND <40%, then “Medium”.


```
WITH margin_table AS (
    SELECT
      ### Key ###
      orders_id
      ,products_id
      ###########
      ,turnover
      ,turnover-purchase_cost AS margin
    FROM `course17.gwz_sales_17`
  )

  ,margin_percent_table AS (
    SELECT
      ### Key ###
      orders_id
      ,products_id
      ###########
      ,turnover
      ,margin
      ,ROUND(SAFE_DIVIDE(margin,turnover),2) AS margin_percent
    FROM margin_table
  )

SELECT
    ### Key ###
    orders_id
    ,products_id
    ###########
    ,turnover
    ,margin
    ,margin_percent
    ,CASE
      WHEN margin_percent < 0.05 THEN "low"
      WHEN margin_percent > 0.4 THEN "high"
      WHEN margin_percent BETWEEN 0.05 AND 0.4 THEN "medium"
      ELSE NULL
    END AS margin_level
  FROM margin_percent_table
```

Note: The two firsts steps could potentially be gathered in only one step.


#### 2.2) Categorisation of Promotions

From the gwz_sales_17 table, we want to calculate promo_percent and sort the results by the promo_type column. This can be achieved in 3 steps:

- Calculate the promo_value based on turnover_before_promo and turnover in the first step.
- In the second step, calculate the promo_percent based on the promo_value and the turnover.
- In the third step, add a promo_type column based on the values in promo_percent.

To define the promo_type column, please use the following information:

- If promo_name contains “DLC” (”Date Limite de Consommation” in 🇫🇷 or “Best Before Date” in 🇬🇧) or “DLUO” in lower or upper case, then “short-lived”.
- If promo_percent ≥30% and NOT “short-lived”, then “High promo”. 
- If promo_percent <10% then “Low promo”. 
- If promo_percent ≥10% AND <30% then “Medium promo”. 


Instead of using 3 intermediate tables, use WITH AS clause to perform this operation in one query.

Target:
![01-Subqueries-asset-5-Untitled](https://github.com/user-attachments/assets/085d7454-74dc-4b5c-8d1c-ecd71bd4ff5e)

```
WITH promo_table AS (
    SELECT
      ### Key ###
      orders_id
      ,products_id
      ###########
      ,turnover
      ,promo_name
      ,turnover_before_promo - turnover AS promo
    FROM `course17.gwz_sales_17`
  )

  ,promo_percent_table AS (
    SELECT
      ### Key ###
      orders_id
      ,products_id
      ###########
      ,turnover
      ,promo_name
      ,promo
      ,ROUND(SAFE_DIVIDE(promo,turnover),2) AS promo_percent
    FROM promo_table
  )

SELECT
  ### Key ###
  orders_id
  ,products_id
  ###########
  ,turnover
  ,promo_name
  ,promo
  ,promo_percent
  ,CASE
    WHEN UPPER(promo_name) LIKE "%DLC%" OR UPPER(promo_name) LIKE "%DLUO%" THEN "short-lived"
    WHEN promo_percent >= 0.30 THEN "High promotion"
    WHEN promo_percent < 0.10 THEN "Low promotion"
    WHEN promo_percent >= 0.10 AND promo_percent < 0.30 THEN "Medium promotion"
    ELSE NULL
  END AS promo_type
FROM promo_percent_table
```

Note: Same as previously, the two firsts steps could potentially be gathered in only one step.


## 3) Using Subqueries instead of Join

#### !!! This is bad practice !!!
It is cleaner and more efficient (in terms of speed and computational cost) to use JOIN instead of a subquery.

For example, let’s assume we want to add the transporter, log_cost and ship_cost columns from the gwz_ship_17 to gwz_sales_17. We could achieve this using subqueries as below:
```
   SELECT
     ### Key ###
     orders_id,
     products_id,
     ###########
     -- sales table --
     category_1,
     turnover,
     qty,
     -- ship table --
     (SELECT transporter FROM `course17.gwz_ship_17` t WHERE t.orders_id = s.orders_id) AS transporter,
     (SELECT log_cost FROM `course17.gwz_ship_17` t WHERE t.orders_id = s.orders_id) AS log_cost,
     (SELECT ship_cost FROM `course17.gwz_ship_17` t WHERE t.orders_id = s.orders_id) AS ship_cost
   FROM `course17.gwz_sales_17` s
```


However, in the example below we can see how using JOIN allows us to build a better structured, cleaner and a more efficient query than using subqueries:

```
   SELECT
     ### Key ###
     s.orders_id
     ,s.products_id
     ###########
     -- sales table
     ,s.category_1
     ,s.turnover
     ,s.qty
     -- ship table
     ,t.transporter
     ,t.log_cost
     ,t.ship_cost
   FROM `course17.gwz_sales_17` s
   INNER JOIN `course17.gwz_ship_17` t USING (orders_id)
```


## To conclude: 

When combining columns from multiple tables, JOIN is generally preferable to subqueries due to its performance and maintainability advantages. In other words, “KISS” your queries.

#### Note: KISS = "Keep it simple, stupid"
