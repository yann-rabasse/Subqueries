# Introduction: Greenweez Presentation

  
Greenweeez is a leading online marketplace for organic products.

It sells organic, ecological and sustainable produce to B2C customers with the aim to facilitate responsible consumption and a healthier lifestyle.
Greenweez operates only through its e-commerce website, meaning that it does not have a physical retail store. Therefore, to attract new customers online advertising is essential. This is the role of the media team at Greenweez.

### Media Team
The media team manages online advertising and various acquisition channels to maximize acquisition within limited budget.

### Mission

- Optimize client acquisition on different channels.
- Optimize ad campaigns to make sure that every cent spent generates as much value as possible.

### Goals & Activities
- Monitor various customer acquisition channels
- Optimize ad campaigns
- Analyze conversion funnel and conversion rate

### Main KPIs to track for the media team

- **Customers Acquisition Cost (CAC)**

    Measures the average cost of acquiring a new customer.
    CAC = sum of acquisition (marketing and sales) cost / number of new customers

- **Return on Ad Spend (ROAS)**

    Measures the return on investment for advertising spend. Often expressed in revenue or percentage. How much one $ spent on ads generates in revenue.
    ROAS = total revenue from ads / total ads costs

- **Cost Per Mille (CPM)**

    CPM is the cost an advertiser will pay to have their ads viewed 1000 times.
    CPM = ads cost / number of impressions * 1000

- **Cost Per Click (CPC)**

    CPC is the cost an advertiser will pay for 1 click on their ads.
  CPC = ads cost / number of clicks

- **Click Through Rate (CTR)**

    CTR is the conversion rate of impressions into clicks.
    CTR = number of clicks / number of impressions

## Objective
The objective of this exercise is to transform and analyze the Greenweez ads data to meet the needs of the media team.
In this exercise you will be able to:

- practice writing SQL queries with join and group by statements
- use BigQuery
- experience a complete data transformation process:
    - table exploration and primary key identification
    - links and relationships between tables summarized in an ER diagram
    - data transformation, analysis and testing

- better understand the needs of the media team

## Data Sources

[adwords](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_adwords)

[bing](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_bing)

[criteo](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_criteo)

[facebook](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_facebook)

[orders](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_orders)

[sessions](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse16!3sgz_sessions)


## 1) Table Exploration

1) First, to explore and understand the structure of the tables, click on the Schema and Preview tabs in BigQuery. How many columns are there? How many rows? Can you understand what data each of the columns contain?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

#### gz_adwords / bing / criteo / facebook
each row is a campaign on a given day:

- *_date_date - the date*_
- *_paid_source - the paid source platform associated with the campaign*_
- *_campaign_key - the unique id of the campaign*_
- *_campaign_name - the name of the campaign*_
- *_cost - the daily costs corresponding to the specific campaign_key and date_date*_
- *_impression - the daily impressions corresponding to the specific campaign_key and date_date*_
- *_click - the daily clicks corresponding to the specific campaign_key and date_date*_

#### gz_orders
Each row is an order. We can link the gz_orders table with the gz_sessions table by using the session_id:

- date_date - the date of purchase of the order
- orders_id - the unique identifier of the order
- news - 1 if the order corresponds to a new order and 0 otherwise
- turnover - the turnover of the order paid by the client
- session_id - the id of the session to link with the gz_sessions table

#### gz_sessions
Each row is a session. Each session contains information about the acquisition channel of the client:

- date_date - the date of the session
- session_id - the unique identifier of the session
- campaign_key - the unique id of the campaign
- campaign - the name of the campaign
- channel - the client acquisition channel
- channel_grouping - a broader category of the client acquisition channel

</details>

2) What are the primary keys for the different tables? To save time, create a primary key test only for the gz_orders and gz_sessions tables.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>
  
```
SELECT
  ### Key ###
  orders_id
  ###########
  ,COUNT(*) AS nb
FROM `course16.gz_orders`
GROUP BY
  orders_id
HAVING nb>=2
ORDER BY nb DESC
```

gz_sessions_pk

The primary key is session_id.

```
SELECT
  ### Key ###
  session_id
  ###########
  ,COUNT(*) AS nb
FROM `course16.gz_sessions`
GROUP BY
  session_id
HAVING nb>=2
ORDER BY nb DESC
```

</details>




## 2) Links and Relationships between Tables

### 2.1) Schema - Entity Relationship Diagram (ERD)

To better understand the relationship between relational databases, you should make an entity-relationship diagram (ERD).

The goal is to create the ERD for our 6 tables.

You could create it from scratch, but that would take too much time, therefore we have started to create the ERD for you. Please take the time to complete it.

![02-Greenweez-Ads-Monitoring-asset-3-Untitled](https://github.com/user-attachments/assets/16380072-4ccb-4054-98f8-a5df0689f52b)

![02-Greenweez-Ads-Monitoring-asset-4-Untitled](https://github.com/user-attachments/assets/686d7eef-fdd2-4f51-9b82-dfa6e7fd3852)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

#### Adwords / Bing / Criteo / Facebook
- _Key_
- Primary keys - date_date and campaign_key Join keys - date_date and campaign_key
- _Relationship_
- Zero to many with gz_session - a campaign ID with a very low cost and coverage could potentially create 0 sessions while most campaigns generate many sessions.

#### Session
- _Key_
- Primary key - session_id
- Join key - date_date and campaign_key for ads table & session_id for gz_orders

*Relationship*
- ads_tables - 1 with ads_table - if a session is generated on a date with a campaign_key, you should find a corresponding row in one of the 4 ad costs tables.
- gz_orders - 0 or 1 (we simplify it by considering that there can only be one order per session) - if the session does not generate an order, the session_id will not be present in orders_table. On the contrary, if the session generates an order, there will be a single row in orders_table with the corresponding session_id.

#### Orders
- _Key_
- Primary key - orders_id
- Join key - session_id for gz_session

_Relationship_

- One with gz_session if session_id IS NOT NULL and 0 if session_id IS NULL. If session_id is not null, it means that there is a corresponding row in gz_session.

![02-Greenweez-Ads-Monitoring-asset-5-Untitled](https://github.com/user-attachments/assets/60655ba5-f205-4d15-9d84-fc07fdb5fed7)

</details>


## 3) Table Transformation and Analysis - Join & Test

The objective is to set up a monitoring report of the daily turnover generated by ads for each paid_source and each campaign. Use the ERD you created above to inform your joins 🧩

### 3.1) Session and Orders Join
To do this, we first need to identify the turnover and the number of orders generated by each of the campaigns. Therefore, we will start by joining the gz_order and gz_session tables.



1) On what key would you join the gz_order and gz_session tables? Do you anticipate any issues with the joining procedure?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gz_order and gz_session can be joined on session_id in a straightforward way, we don’t anticipate any issues. We just need to choose the correct type of join.

</details>


2) Perform an inner join between the gz_order and gz_session tables.

Compare the number of rows in the gz_order and gz_session tables with the number of rows in the result.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

- gz_orders - 109 569 rows
- gz_sessions - 1 855 125 rows
- inner join request - 61 826 rows
- There are fewer rows in the inner join than in the two source tables. This means that:
    - There are many sessions that do not result in an order and are therefore not present in the gz_orders table.
    - There are many orders that are not tracked because of the use of cookies and ad block. They have no session_id and are not linked to gz_sessions.

```
SELECT
  -- orders_info --
  o.date_date
  ,o.orders_id
  ,o.turnover
  ,o.news
  -- session info --
  ,se.session_id
  ,se.campaign_key
  ,se.campaign
FROM `course16.gz_orders` AS o
INNER JOIN `course16.gz_sessions` AS se USING (session_id)
```

</details>


3) Perform a left join between the gz_order and gz_session tables.

Compare the number of rows in the gz_order and gz_session tables with the number of rows in the result.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

- gz_orders - 109 569 rows
- gz_sessions - 1 855 125 rows
- left  join request - 109569  rows
- There are fewer rows in the inner join than in the two source tables. This means that:
- We have the same number of rows in gz_orders and the joined table.
- Note - since this is a left join, we use o instead of s for the session_id join key.

```
SELECT
  -- orders_info --
  o.date_date
  ,o.orders_id
  ,o.turnover
  ,o.news
  -- session info --
  ,o.session_id
  ,se.campaign_key
  ,se.campaign
FROM `course16.gz_orders` AS o
LEFT JOIN `course16.gz_sessions` AS se USING (session_id)
```

</details>


4) Perform a right join between the gz_order and gz_session tables.

Compare the number of rows in the gz_order and gz_session tables with the number of rows in the result.



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>
  
gz_orders - 109 569 rows

gz_sessions - 1 855 125 rows

right join request - 1 855 125 rows

We have the same number of rows in the right join and in the session tables.

```
SELECT
  -- orders_info --
  o.date_date
  ,o.orders_id
  ,o.turnover
  ,o.news
  -- session info --
  ,se.session_id
  ,se.campaign_key
  ,se.campaign
FROM `course16.gz_orders` AS o
RIGHT JOIN `course16.gz_sessions` AS se USING (session_id)
```

</details>


5) Perform a full outer join between the gz_order and gz_session tables.

Compare the number of rows in the gz_order and gz_session tables with the number of rows in the result.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>
  
gz_orders - 109 569 rows

gz_sessions - 1 855 125 rows

full outer join request - 1 902 868 rows

There are more rows in the full outer join than in the two source tables.

Note - since this is a full_outer_join IFNULL(s.session_id,o.session_id) for the session_id join key.

```
SELECT
  -- orders_info --
  o.date_date
  ,o.orders_id
  ,o.turnover
  ,o.news
  -- session info --
  ,IFNULL(se.session_id,o.session_id) AS session_id
  ,se.campaign_key
  ,se.campaign
FROM `course16.gz_orders` AS o
FULL OUTER JOIN `course16.gz_sessions` AS se USING (session_id)
```

</details>


6) The different join methods return different of results. We need to choose the one that best meets our needs. The purpose of joining gz_order and gz_session tables is to add information to each order about their corresponding campaign to ultimately calculate KPIs for the different ad campaigns. So which JOIN method do you think is the most appropriate in this case?


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

We only want to keep orders that have a session_id and the associated campaign information in the gz_session table. Therefore, the appropriate join method is INNER JOIN.

</details>


7) Save the results of the INNER JOIN query in a gz_orders_ga table. Create a primary key test for it.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gz_orders_ga

```
SELECT
  o.date_date
  ### Key ###
  ,o.orders_id
  ###########
  -- orders_info --
  ,o.turnover
  ,o.news
  -- session info --
  ,se.session_id
  ,se.campaign_key
  ,se.campaign
FROM `course16.gz_orders` AS o
INNER JOIN `course16.gz_sessions` AS se USING (session_id)
```

gz_orders_ga_pk

```
SELECT
  ### Key ###
  orders_id
  ###########
  ,COUNT(*) AS nb
FROM `course16.gz_orders_ga`
GROUP BY
  orders_id
HAVING nb>=2
ORDER BY nb DESC
```

</details>


### 3.2) Campaign Aggregation
The campaign information is distributed across 4 different tables according to different data sources: gz_adwords, gz_bing, gz_criteo, gz_facebook. The 4 tables follow the same structure. However, for our analysis it would be better to put all the campaign information together in a single gz_campaign table.



Use UNION ALL to combine all 4 ad data sources into a single table named gz_campaign.

[Union All Doc](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax#set_operators)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT * FROM `course16.gz_adwords`
 UNION ALL (SELECT * FROM `course16.gz_bing`)
 UNION ALL (SELECT * FROM `course16.gz_criteo`)
 UNION ALL (SELECT * FROM `course16.gz_facebook`)
```


</details>

### 3.3) Orders and Campaign Join
We have updated the ER Diagram with the 2 new tables, but it is incomplete.

![02-Greenweez-Ads-Monitoring-asset-7-Untitled](https://github.com/user-attachments/assets/8b435f52-6963-495e-9a9e-04338bebc9e9)



1) As before, fill it in, being sure to specify the:

- PRIMARY KEY
- JOINING KEYS
- RELATIONSHIP BTW TABLES

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

#### gz_campaign

- Primary keys - date_date and campaign_key
- Join keys - date_date and campaign_key
- Relationships:
    - Zero to many with gz_orders_ga - a campaign with a very low cost and coverage could potentially create 0 orders while most campaigns generate many orders in gz_orders_ga.

#### gz_orders_ga

- Primary keys - orders_id
- Join keys - date_date and campaign_key for gz_campaign
- Relationships:
    - One with gz_campaign when campaign_key IS NOT NULL, 0 if campaign_key IS NULL. If campaign_key is not NULL on a date, an ad on a specific date caused an order.

![02-Greenweez-Ads-Monitoring-asset-7-Untitled](https://github.com/user-attachments/assets/0e79379c-463b-4bce-900f-937878519ecb)


</details>



2) Now that we have completed the ERD let’s join the two tables.
On what key would you join the gz_campaign and gz_orders_ga tables to calculate the relevant ad metrics? Do you anticipate any issues?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

We can join gz_campaign and gz_orders_ga on date_date + campaign_key. However, this is a 1:N relationship as many orders may have the same date_date and campaign_key values. This would result in many duplicate values that would not be reliable or usable for analysis.

If we want to join the ad and order information together, **we must first group the orders by the date_date and campaign_id columns. The relationship will then be 1:1 instead of 1:N, and we will be able to perform the join without issues.

</details>


3) Aggregate gz_orders_ga on date_date and campaign_key. Calculate the following aggregated metrics:

- number of transactions (orders)
- sum of turnover
- sum of new customers (news)

Save the result as gz_campaign_orders. Create a primary key test for the table called gz_campaign_orders_pk.

We need to perform a join between gz_campaign and gz_campaign_orders to gather all the campaign and order information in one table.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

  **gz_campaign_orders**

  ```
 SELECT
  ### Key ###
  date_date
  ,campaign_key
  ###########
   -- orders metrics --
  ,COUNT(DISTINCT orders_id) AS nb_transactions
  ,SUM(turnover) AS turnover
  ,SUM(news) AS news
 FROM `course16.gz_orders_ga`
 GROUP BY
    date_date
    ,campaign_key
```

**gz_campaign_orders_pk**

```
 SELECT
   ### Key ###
   date_date
   ,campaign_key
   ###########
   ,COUNT(*) AS nb
 FROM `course16.gz_campaign_orders`
 GROUP BY
  date_date
  ,campaign_key
 HAVING nb>=2
 ORDER BY nb DESC
```

</details>


4) Perform an INNER join between the gz_campaign and gz_campaign_orders tables. Compare the number of rows in gz_campaign and gz_campaign_orders tables with the returned result from your join query. Try different join methods and compare the different results that you get from them.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

gz_campaign - 5607 rows gz_campaign_orders - 8176 rows inner join query result - 3605 rows

There are fewer rows in the inner join than in the two source tables. This means that:

- There are campaigns that on some days did not generate orders and therefore do not appear in the gz_campaign_order table, but they have a cost row in gz_campaign for the same date.
- There are many orders that are related to a free campaign and not to a paid campaign. Thus, they have no row in gz_campaign.

```
SELECT
 ### Key ###
 c.date_date
 ,c.campaign_key
 ###########
 -- campaign table --
 ,c.paid_source
 ,c.campaign_name
 ,c.cost
 ,c.click
 ,c.impression
 -- orders table --
 ,nb_transactions
 ,turnover
 ,news
FROM `course16.gz_campaign` AS c
INNER JOIN `course17.gz_campaign_orders` AS o
 ON c.date_date = o.date_date
 AND c.campaign_key = o.campaign_key
```

</details>


5) Our goal is to analyze the performance and profitability of our paid ad campaigns. We don’t want to focus on campaigns that don’t have an associated cost and rank and therefore are not present in the gz_campaign table. Furthermore, we need to access the information about the revenue generated by these ad campaigns, and this information can only be found in the gz_campaign_orders table. So which JOIN method would you choose?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

We want all the information about the statistics of the ads. Therefore, we need to keep all the rows from the gz_campaign table. We can therefore only use LEFT join.

</details>

6) Join the gz_campaign and gz_campaign_orders tables with a LEFT JOIN. You will notice that there are some NULL values in the turnover and nb_transaction columns. This occurs when a campaign has cost money, but has not generated any orders. Set the values of turnover and nb_transaction to be “0” instead of NULL for these cases. Save your result in the gz_campaign_join table.

Note: Use IFNULL or CASE WHEN

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

We use IFNULL() function to affect 0 instead of null in orders KPI columns

**gz_campaign_join**

```
SELECT
 ### Key ###
 c.date_date
 ,c.campaign_key
 ###########
 -- campaign table --
 ,c.paid_source
 ,c.campaign_name
 ,c.cost
 ,c.click
 ,c.impression
 -- orders table --
 ,IFNULL(nb_transactions,0) AS nb_transactions
 ,IFNULL(turnover,0) AS turnover
 ,IFNULL(news,0) AS news
FROM `course17.gz_campaign` AS c
LEFT JOIN `course16.gz_campaign_orders` AS o
 ON c.date_date = o.date_date
 AND c.campaign_key = o.campaign_key
```

</details>

### 3.4) Aggregation & Analysis

For our final report, the metrics to be calculated for each paid_source and each campaign are:

- turnover generated
- number of orders
- number of new orders (news)
- cost
- impressions
- clicks
- associated KPIs - ROAS, CAC, CPM, CPC, CTR



1) In the gz_campaign_join table, aggregate the above-mentioned order and ad metrics (not the KPIs yet) by the paid_source column. Instead of saving the result in a new table, you will use this query as a subquery for the next step and alias it as gz_campaign_ps (ps stands for paid_source).

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
 ### Key ###
 paid_source
 ###########
 -- orders metrics --
 ,SUM(nb_transactions) AS nb_transactions
 ,SUM(news) AS news
 ,ROUND(SUM(turnover),0) AS turnover
 -- campaign metrics  --
 ,ROUND(SUM(cost),0) AS cost
 ,ROUND(SUM(click),0) AS click
 ,ROUND(SUM(impression),0) AS impression
FROM `course16.gz_campaign_join`
GROUP BY paid_source
```

</details>

2) Use your previously written subquery gz_campaign_ps to calculate the KPIs listed below:

- ROAS, CAC new, CAC orders, CPM, CPC, CTR

Run the query and answer the following questions:

- What conclusions can you draw from these results?
- What could explain the differences in the ad and revenue metrics between the different paid sources?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH gz_campaign_ps AS (
SELECT
 ### Key ###
 paid_source
 ###########
 -- orders metrics --
 ,SUM(nb_transactions) AS nb_transactions
 ,SUM(news) AS news
 ,ROUND(SUM(turnover),0) AS turnover
 -- campaign metrics  --
 ,ROUND(SUM(cost),0) AS cost
 ,ROUND(SUM(click),0) AS click
 ,ROUND(SUM(impression),0) AS impression
FROM `course16.gz_campaign_join`
GROUP BY paid_source
)

SELECT
 ### Key ###
 paid_source
 ###########
 -- orders metrics --
 ,nb_transactions
 ,turnover
 -- campaign metrics  --
 ,cost
 ,click
 ,impression
 -- KPI
 ,ROUND(SAFE_DIVIDE(turnover,cost),2) AS ROAS
 ,ROUND(SAFE_DIVIDE(cost,news),2) AS CAC
 ,ROUND(SAFE_DIVIDE(cost,nb_transactions),2) AS CAC_orders
 ,ROUND(SAFE_DIVIDE(cost,impression)*1000,2) AS CPM
 ,ROUND(SAFE_DIVIDE(cost,click),2) AS CPC
 ,ROUND(SAFE_DIVIDE(click,impression)*100,2) AS CTR
FROM gz_campaign_ps
ORDER BY paid_source
```

Stats are very different from one paid source to another. It is because there are different steps of the acquisition funnel.

![Screenshot_1](https://github.com/user-attachments/assets/5ef54133-a1fc-4821-bcc7-e0365663fe69)


Facebook is a social media platform rather high in the acquisition funnel. Facebook users don’t have the intent to buy an organic product from Greenweez when navigating through facebook. Therefore, the displayed ads don’t lead to high conversion rates and the associated metrics like ROAS and CAC are lower as compared to other paid sources.

Criteo targets users who have already visited the website by displaying ads on other websites after they have left the Greenweez platform. ROAS and CAC values are better than for facebook but worst than for bing and adwords.

Bing and adwords have relatively similar KPIs. However, Greenweez spent less money on Bing than on Adwords. Therefore, the less expensive campaign seems to be more effective.

However, most of the paid acquisition is generated by adwords which is the main channel by far in term of conversions.

</details>



3) Finally, compare the performance of each paid source across several months. To achieve this, you will need to edit your gz_campaign_ps subquery and group by both paid_source and month.

Analyze how the metrics change for each paid source over time.

- What conclusions can you draw from these results?
- Which paid sources are performing well and which ones are underperforming?

Target:
![02-Greenweez-Ads-Monitoring-asset-9-Untitled](https://github.com/user-attachments/assets/f757c1ea-fbb2-4c3e-9bf3-c9a9fc6e1f67)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

  ```
WITH gz_campaign_ps AS (
SELECT
 ### Key ###
 paid_source
 , EXTRACT(month FROM date_date) AS month
 ###########
 -- orders metrics --
 ,SUM(nb_transactions) AS nb_transactions
 ,SUM(news) AS news
 ,ROUND(SUM(turnover),0) AS turnover
 -- campaign metrics  --
 ,ROUND(SUM(cost),0) AS cost
 ,ROUND(SUM(click),0) AS click
 ,ROUND(SUM(impression),0) AS impression
FROM `course16.gz_campaign_join`
GROUP BY
 paid_source,
 month
)

SELECT
 ### Key ###
 paid_source
 , month
 ###########
 -- orders metrics --
 ,nb_transactions
 ,turnover
 -- campaign metrics  --
 ,cost
 ,click
 ,impression
 -- KPI
 ,ROUND(SAFE_DIVIDE(turnover,cost),2) AS ROAS
 ,ROUND(SAFE_DIVIDE(cost,news),2) AS CAC
 ,ROUND(SAFE_DIVIDE(cost,nb_transactions),2) AS CAC_orders
 ,ROUND(SAFE_DIVIDE(cost,impression)*1000,2) AS CPM
 ,ROUND(SAFE_DIVIDE(cost,click),2) AS CPC
 ,ROUND(SAFE_DIVIDE(click,impression)*100,2) AS CTR
FROM gz_campaign_ps
ORDER BY
 paid_source,
 month
```

For Bing and Ads, the turnover decreased in July and August after a higher sales period in June. It increased a little bit again in September, but not enough to compensate for the increasing ad costs. Therefore, ROAS and CAC values have considerably diminished for these 2 paid sources. The situation needs to be monitored more closely.

For Criteo and Facebook, costs have increased but so has the turnover, therefore it is less worrying. ROAS and CAC are quite stable.

</details>






