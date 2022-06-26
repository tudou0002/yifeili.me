---
title: "Window Functions in SQL"
date: 2022-06-26T10:42:15-04:00
draft: false
tags:  ['sql']
author: Yifei Li
showToc: true
TocOpen: false
math: true
hidemeta: false
comments: false
description: ""
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/tudou0002/yifeili.me/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

Window function is an advanced topic in SQL and is asked quite often during the interviews. In this post, I will explain what is window function and provide you with some use cases that may help you understand this concept under context. 

# What are window functions?
SQL window functions come in handy when we want to make SQL statements about rows that are related to the current row during processing. We use the term "window" because a set of rows are sometimes called "window frame". For example, we may want to compute the average salary in each department and compare every employee's salary with their corresponding average salary. 

In SQL, there are two types of window functions--aggregate window functions & analytical window functions.
- Aggregate window functions: perform aggregate operations on a set of records. For example, `SUM`, `AVG`, `MAX`, `MIN`
- Analytical window functions: calculate some window based on the current row and then calculate the results based on that window of records. For example, `RANK`, `FIRST_VALUE`, `RANK`, `LEAD`, `LAG`, `ROW_NUMBER`, etc.

# What does a window function look like?
A simple window function template can be like this:
```sql
SELECT
window_function (col_name)
OVER ([PARTITION BY partition_list] [ORDER BY order_list] [window_frame_clause])
```

I'll then walk through the window functions I mentioned in the previous section with explanation and sql examples.
```sql
-- SUM
-- calculates the total of a numeric column in a given window
SELECT salary, SUM(salary) 
OVER (PARTITION BY department)
FROM Salary

-- AVG
-- find the average values of a numeric column
SELECT salary, AVG(salary) 
OVER (PARTITION BY department)
FROM Salary

-- MIN, MAX
-- calculate the minimum and the maximum values within a window
SELECT salary, MIN(salary) 
OVER (PARTITION BY department)
FROM Salary

-- ROW_NUMBER
-- assigns an incremental row number to each of the records within the selected window 
SELECT salary, 
ROW_NUMBER()
OVER (ORDER BY age)
FROM Salary

-- FIRST_VALUE
SELECT salary, first_value(salary) 
OVER (PARTITION BY department ORDER BY salary DESC) 
FROM Salary

-- RANK: return the record's rank based on the order standard
SELECT salary, rank() 
OVER (PARTITION BY department ORDER BY salary DESC)
FROM Salary

-- LAG and LEAD
-- LAG(): refers to the row that came before the currently processed row
SELECT salary, lag(salary) 
OVER (PARTITION BY department ORDER BY salary DESC)
FROM Salary
-- LEAD(): refers to the row that came after the currently processed row
SELECT salary, lag(salary) 
OVER (PARTITION BY department ORDER BY salary DESC)
FROM Salary

-- NTILE functions
-- split the column values into n percentile, assign the values in each percentile with
-- the percentile order. (bucket sort) e.g. 1,2,3,4 for ntile(4)
SELECT salary, ntile(4) 
OVER (PARTITION BY department ORDER BY salary DESC)
FROM Salary
```

# Examples

### Calculate Cumulative Sum of Products Sold by Day
Cumulative sum is sometimes called running total which requires the sum of all the values up to the current value. Given a sales record table, you are asked to calculate the cumulative sum of `products_sold` of each region. The cumulative sum should be ordered by the `date` column.

```sql
SELECT  date,
        products_sold,
        SUM(products_sold) OVER
            (PARTITION BY region ORDER BY date ROWS 
            BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) 
            AS cumulative_region
FROM sales
ORDER BY region, date;

```

### Calculate Moving Average of Covid Cases
Moving average, sometimes called rolling means, rolling averages, or running averages, are calculated as the mean of the current and a specified number of immediately preceding values for each point in time. The main idea is to examine how these averages behave over time instead of examining the behavior of the original or raw data points.

Given table `covid_cases` with following column:
- date
- country
- confirmed_day: positive case count per day

Compute the three-day moving average of positive cases

```sql
SELECT *,
      avg(confirmed_day) OVER(
          PARTITION BY country
          ORDER BY date
          ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
          AS 3day_moving_average
FROM covid_cases;
```

### Calculate Percent Change of Stock Price 
Given a table `price` with columns:
- date
- stock_price

You need to compute the percent change of stock price compared to that of the previous day.
```sql
SELECT  *,
        (stock_price/LAG(stock_price) OVER(ORDER BY date)) - 1 AS percent_change,
FROM price;
```