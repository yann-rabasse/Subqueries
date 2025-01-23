# Takeaways
Subqueries is a way to organise long queries into smaller steps

The output of each step is a table, that is given a temporary name so that they can be queried from later steps

Breaking long queries into smaller steps will improve query’s readability, hence also ease to debug them

# Case Study

Today, we’re going to review the queries that were written on the previous day to use subqueries.

Your task is to understand how foot traffic affects the store revenue.

Today, you’ve been given additional data to test a hypothesis whether sales are driven by morning flights, i.e. passengers trying to sneak in breakfast prior to their departure.

# Data: Flights

[flights.csv](https://docs.google.com/spreadsheets/d/1d4KfqLYX5a2JH0dtBLhD959XNoJrzRDN32INE3u1NJs/edit?gid=2045233871#gid=2045233871)


You’ve been provided new data that records each flight departing your airport for the month of November.

Each row in this data represents an outbound flight, including their departure time and destination.

#### Sample

![01-Flights](https://github.com/user-attachments/assets/be5a1be3-767b-4a15-94bc-d3fe120a4115)



### Task 1: Reviewing SQL queries

Yesterday, we saved steps in our analysis as temporary tables, so that they can be queried at a later step.

For example, our aggregation of daily sales were saved into a temporary table called tmp_sales.

While this method worked, updating table based on an updated query was troublesome. We’ve had to delete the temporary table, re-run the query, and re-export them.

Your first task is to take the queries from yesterday, and re-write them using subqueries.

#### SQL queries from yesterday

**The goal here is to review the syntax of subqueries using familiar data, and familiar queries from the day before.**

While the queries are provided in the collapse section above, it is a good idea to try writing this query from scratch as a Livecode exercise.

That way, students that are still unsure about the syntax of subqueries have an opportunity to see how to break down the steps to write a subquery.*


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH tmp_sales AS (
  SELECT
    DATE(date) AS `date`
    ,ROUND(SUM(amount),2) AS `sales`
  FROM `recapjoin.sales`
  GROUP BY DATE(date)
), tmp_traffic AS (
  SELECT
    date
    ,SUM(traffic) AS `traffic`
  FROM `recapjoin.traffic`
  GROUP BY date
)
SELECT
  sales.date
  ,sales.sales
  ,visitors.traffic
FROM tmp_sales sales
LEFT JOIN tmp_traffic visitors ON sales.date = visitors.date
ORDER BY date
```

</details>


### Task 2: Analyse additional data

Your hypothesis is that the general store does well particularly on days when there are more morning flights.

As some people may not have enough time to get breakfast on their way to the airport, they may be more likely to purchase breakfast items from the store.

Similar hypotheses relating to flights around lunch and dinner times can also be tested.

#### Step 1: Understanding the outcome

Prior to writing your analysis, it is always good practice to first understand your data, and the final outcome you’re trying to achieve.

To test your hypothesis, the final table you’re trying to achieve look like the following.

![02-Analyse](https://github.com/user-attachments/assets/b9b92525-d030-4048-b38c-703b411e4789)


#### Step 2: Write an analysis plan

Now, let’s spend time to understand the flights data and write an analysis plan, prior to writing queries.

**The goal here is to get students to think through how to use subqueries to adapt the shape of the raw data into the final output specified in Step 1.**

**Analysis plan**

Step 1. …

Step 2. …

Step 3. …

Example output:

1) Identify each flight as either morning or evening flight
2) Summarise the number of morning and evening flights per day
3) Summarise sales by day
4) Combine results from items 2 and 3


#### Step 3: Write subqueries

Let’s write the query implementing your analysis plan by using subqueries.

**The goal here is to guide the students in build subqueries with an increased level of complexity, as well as using concepts learned from the Aggregation day.**

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH tmp_sales AS (
  SELECT
    DATE(date) AS `date`
    ,ROUND(SUM(amount),2) AS `sales`
  FROM `recapjoin.sales`
  GROUP BY DATE(date)
), tmp_flights AS (
  SELECT
    DATE(time) AS `date`
    ,SUM(CASE WHEN TIME(time) < TIME(8, 0, 0) THEN 1 ELSE 0 END) AS `morning_flights`,
    ,COUNT(*) AS `num_flights`
  FROM `recapjoin.flights`
  GROUP BY DATE(time)
)
SELECT
  sales.date
  ,sales.sales
  ,flights.morning_flights
  ,flights.num_flights
FROM tmp_sales sales
LEFT JOIN tmp_flights flights ON sales.date = flights.date
ORDER BY date
```

**Result**

<img width="558" alt="8m5xg3t1lxlesdkmzqy7a1kkeycd" src="https://github.com/user-attachments/assets/8901fa34-0ebc-4dd9-b4ad-e26283e7eccd" />


</details>


#### Step 4: Interpreting the result

Remember, our hypothesis is that number of morning flights leaving the airport influence revenue.

We’re going to look at interpreting the result returned from our queries.

While Data Visualization is a future topic, we’re going to use BigQuery’s convenient feature to quickly interpret our result.

Instructions

**Chart**
Following the instructions above will generate a chart similar to the following.

![28ww0x6dfzxmq0vqn40ljtyq41c5](https://github.com/user-attachments/assets/f1b88006-34e9-4f71-a440-e1f73a70e09a)



**Interpretation**

**The goal here is to encourage students to not stop at writing queries, and to start analysing the results they get from their queries.**

Ideas for class prompts: • Is there an upward trend in sales as the number of morning flights increases? • Does it look like there are other factors impacting sales?*

#### Step 5: Triangulate

As a responsible analyst, you’ll always check, double check, and triple check that the results you have written are correct.

Trust us, that will save you the risks of putting the wrong numbers in important reports.

In our final table though, we have already triangulated the sales figures from yesterday’s recap. How can we triangulate the number of flights?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH tmp_sales AS (
  SELECT
    DATE(date) AS `date`
    ,ROUND(SUM(amount),2) AS `sales`
  FROM `recapjoin.sales`
  GROUP BY DATE(date)
), tmp_flights AS (
  SELECT
    DATE(time) AS `date`
    ,SUM(CASE WHEN TIME(time) < TIME(8, 0, 0) THEN 1 ELSE 0 END) AS `morning_flights`,
    ,COUNT(*) AS `num_flights`
  FROM `recapjoin.flights`
  GROUP BY DATE(time)
), tmp_final AS (
  SELECT
    sales.date
    ,sales.sales
    ,flights.morning_flights
    ,flights.num_flights
  FROM tmp_sales sales
  LEFT JOIN tmp_flights flights ON sales.date = flights.date
  ORDER BY date
)
SELECT
  SUM(num_flights)
FROM tmp_final

# returns 886
```

```
SELECT
  COUNT(*)
FROM `recapjoin.flights`

# returns 886
```

</details>
