---
title: "Cool things you can do using Snowflake SQL"
categories:
  - Blog
tags:
  - SQL
  - Snowflake
  - QUALIFY
  - UNPIVOT
  - LATERAL JOIN
excerpt: "Snowflake SQL is awesome! Here's a couple cool things you can do with it."
---
# Table of Contents
1. [Introduction](#introduction)
2. [QUALIFY](#qualify)
3. [UNPIVOT](#unpivot)
4. [Conclusion](#conclusion)

# Introduction
Hello! It's been 5 months since my last post, so I'm excited to be back here writing again (it's the humanities major in me that gets me writing!). 

You probably have heard of Snowflake, but in case you haven't, it's an incredibly powerful cloud-based data warehouse. I could go on and on about what Snowflake allows you to do, but in short, Snowflake allows you to store all of your structured and semi-structured data in one place and query that data super, super fast. I mean super, super, super fast: not only does Snowflake optimize query performance like no other, you can also instantaneously scale up the computing power used for your queries to near-infinity. 

There's a reason why [Snowflake's IPO was incredibly successful](https://www.cnbc.com/2020/09/16/snowflake-snow-opening-trading-on-the-nyse.html), and the company counts Warren Buffett's Berkshire Hathaway and Salesforce as major investors. I know what you're thinking: I'm not affiliated with Snowflake in any way—I just think it's really incredible technology. :smile:

What I recently found out is Snowflake offers a [free 30-day trial](https://signup.snowflake.com/) where you don't provide any payment information, so you can test drive Snowflake and see all the neat things it can do. So that's what I did! 

In this post, I'll show you a couple cool Snowflake SQL things that you can't do in PostgreSQL/MySQL.  

# QUALIFY
The first SQL dialect I ever learned was PostgreSQL and that's because PostgreSQL was what was taught in my [Metis data science bootcamp](https://www.thisismetis.com/). I later learned MySQL when I started practicing SQL on LeetCode, and that's only because LeetCode doesn't offer PostgreSQL (:sad:). Between MySQL and PostgreSQL, PostgreSQL is definitely more powerful and popular these days. Then I tried Snowflake SQL, and Snowflake SQL is even better!

Let's look at the below employee table together. I created this table in Snowflake within minutes by defining the table columns and uploading a CSV file with dummy data. (I was really hungry when I created the dummy data, hence the food-based names.)

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | DEPARTMENT | ANNUAL_SALARY_AMOUNT_USD | IS_MANAGER | STARTED_WORK_ON |
|-------------|------------|-----------|------------|--------------------------|------------|-----------------|
| 1           | Binh       | Hoang     | Data       | 100000                   | FALSE      | 2021-01-11      |
| 2           | Lewis      | Coffee    | Finance    | 90000                    | FALSE      | 2021-03-21      |
| 3           | Bagel      | Soda      | Finance    | 200000                   | TRUE       | 2021-04-01      |
| 4           | Sun        | Moon      | Marketing  | 300000                   | TRUE       | 2020-02-22      |
| 5           | Water      | Bottle    | Marketing  | 150000                   | TRUE       | 2019-05-01      |
| 6           | Cactus     | Turnip    | Sales      | 90000                    | FALSE      | 2020-12-01      |
| 7           | Book       | Shelf     | Sales      | 160000                   | TRUE       | 2019-06-21      |
| 8           | Chicken    | Congee    | Sales      | 80000                    | FALSE      | 2021-03-26      |
| 9           | Soba       | Noodles   | Data       | 180000                   | TRUE       | 2020-07-15      |
| 10          | Dim        | Sum       | Marketing  | 90000                    | FALSE      | 2021-05-01      |

If I'm working in PostgreSQL and want to query the employees who have the highest salary in each department with all columns included (hence can't use GROUP BY), I'd have to use a combination of a CTE and a window function like DENSE_RANK() (RANK() would work as well, but not ROW_NUMBER() since ROW_NUMBER() wouldn't recognize ties). This is what the query below does. It works, but it's definitely verbose for what I'm trying to accomplish. 

### Query 1 (PostgreSQL)
```sql
WITH rank_employee_in_each_department_by_salary AS (
    SELECT 
        *,
        DENSE_RANK() OVER (PARTITION BY department ORDER BY annual_salary_amount_usd DESC) AS employee_rank
    FROM 
        employee    
)
SELECT
    employee_id,
    first_name,
    last_name,
    department,
    annual_salary_amount_usd,
    is_manager,
    started_work_on
FROM 
    rank_employee_in_each_department_by_salary
WHERE 
    employee_rank = 1
ORDER BY 
    department;
```

Enter Snowflake's [QUALIFY clause](https://docs.snowflake.com/en/sql-reference/constructs/qualify.html). 

From Snowflake's documentation, QUALIFY "does with window functions what HAVING does with aggregate functions and GROUP BY clauses." Using QUALIFY, I don't need to use a CTE and materialize an additional employee_rank column using a window function. I simply add a QUALIFY clause to filter my query using a window function. This is a much more elegant and performant query than what you could accomplish in most other SQL dialects. 

### Query 2 (Snowflake SQL)
```sql
SELECT 
    *
FROM 
    employee
QUALIFY 
    DENSE_RANK() OVER (PARTITION BY department ORDER BY annual_salary_amount_usd DESC) = 1
ORDER BY 
    department;
```

### Query 2 Results

| EMPLOYEE_ID | FIRST_NAME | LAST_NAME | DEPARTMENT | ANNUAL_SALARY_AMOUNT_USD | IS_MANAGER | STARTED_WORK_ON |
|-------------|------------|-----------|------------|--------------------------|------------|-----------------|
| 9           | Soba       | Noodles   | Data       | 180000                   | TRUE       | 2020-07-15      |
| 3           | Bagel      | Soda      | Finance    | 200000                   | TRUE       | 2021-04-01      |
| 4           | Sun        | Moon      | Marketing  | 300000                   | TRUE       | 2020-02-22      |
| 7           | Book       | Shelf     | Sales      | 160000                   | TRUE       | 2019-06-21      |

I don't believe Snowflake invented the QUALIFY clause since it's used in other SQL dialects such as Teradata, but I'm just glad it exists. If you know who was the original inventor of the QUALIFY clause, please comment below. Would love to know! 

# UNPIVOT

Let's say Human Resources wants to know the below bespoke metrics. Why? Who knows. 

1. Ratio of combined salary of Finance / total combined salary of Finance, Marketing, Sales
2. Ratio of combined salary of Data / total combined salary of Data, Finance, Sales
3. Ratio of combined salary of Marketing / total combined salary of Marketing, Data, Finance
4. Ratio of combined salary of Sales / total combined salary of Sales, Data, Marketing

What you do know is that you've been assigned to work on this request and even though the data request is strange, you have been assured that that's the data they actually want. 

You know that there's an Employee table that you can query to get this data, but you can't do a simple GROUP BY or window function since the groups of data vary across each denominator. 

You think about it some more and realize you can use a bunch of CASE WHENs, and you write the below query.

### Query 3 (Snowflake SQL)
```sql
SELECT
    ROUND(SUM(CASE WHEN department = 'Finance' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Finance', 'Marketing', 'Sales') THEN annual_salary_amount_usd END) * 100, 2) AS finance_ratio,
    ROUND(SUM(CASE WHEN department = 'Data' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Data', 'Finance', 'Sales') THEN annual_salary_amount_usd END) * 100, 2) AS data_ratio,
    ROUND(SUM(CASE WHEN department = 'Marketing' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Marketing', 'Data', 'Finance') THEN annual_salary_amount_usd END) * 100, 2) AS marketing_ratio,
    ROUND(SUM(CASE WHEN department = 'Sales' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Sales', 'Data', 'Marketing') THEN annual_salary_amount_usd END) * 100, 2) AS sales_ratio
FROM 
    employee;   
```

### Query 3 Results

| FINANCE_RATIO | DATA_RATIO | MARKETING_RATIO | SALES_RATIO |
|---------------|------------|-----------------|-------------|
| 25.00         | 31.11      | 48.65           | 28.70       |

Great! But what if you want to stack the data vertically (i.e. normalize the data) rather than horizontally. This becomes especially important if you had to calculate 10 of these bespoke metrics instead of 4: then you'd be scrolling from left to right to see all your data, which is not ideal. 

Enter Snowflake's [UNPIVOT operator](https://docs.snowflake.com/en/sql-reference/constructs/unpivot.html)! 

From Snowflake's documentation: *"Rotates a table by transforming columns into rows. UNPIVOT is a relational operator that accepts two columns (from a table or subquery), along with a list of columns, and generates a row for each column specified in the list. In a query, it is specified in the FROM clause after the table name or subquery."*

UNPIVOT is super useful for displaying query results vertically, which in most situations is much easier for you and others to read. And all it takes is wrapping your original query in a CTE and a few additional lines of code. 

### Query 4 (Snowflake SQL)
```sql
WITH bespoke_metrics AS (
    SELECT
        ROUND(SUM(CASE WHEN department = 'Finance' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Finance', 'Marketing', 'Sales') THEN annual_salary_amount_usd END) * 100, 2) AS finance_ratio,
        ROUND(SUM(CASE WHEN department = 'Data' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Data', 'Finance', 'Sales') THEN annual_salary_amount_usd END) * 100, 2) AS data_ratio,
        ROUND(SUM(CASE WHEN department = 'Marketing' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Marketing', 'Data', 'Finance') THEN annual_salary_amount_usd END) * 100, 2) AS marketing_ratio,
        ROUND(SUM(CASE WHEN department = 'Sales' THEN annual_salary_amount_usd END) / SUM(CASE WHEN department IN ('Sales', 'Data', 'Marketing') THEN annual_salary_amount_usd END) * 100, 2) AS sales_ratio
    FROM 
        employee    
)
SELECT 
    *
FROM 
    bespoke_metrics
    UNPIVOT (value 
             FOR metric IN (finance_ratio, data_ratio, marketing_ratio, sales_ratio));
```

### Query 4 Results

| METRICS         | VALUE |
|-----------------|-------|
| FINANCE_RATIO   | 25.00 |
| DATA_RATIO      | 31.11 |
| MARKETING_RATIO | 48.65 |
| SALES_RATIO     | 28.70 |

See how much easier this is to read? 

PostgreSQL does not have an UNPIVOT operator, but you can mimic the same functionality as UNPIVOT using a LATERAL JOIN as demonstrated in this [blog post](https://blog.sql-workbench.eu/post/unpivot-with-postgres/). However, LATERAL JOINs scare less seasoned SQL coders and are slightly less elegant, so UNPIVOT wins my heart. :heart:

Snowflake didn't invent UNPIVOT since UNPIVOT also exists in Oracle and SQL Server—I'm just glad the Snowflake people ported it over.

# Conclusion

I'm a writer. I spent four years at Yale honing my writing skills, and I've also been an editor for a [popular Tumblr](https://gaysiandiaries.com/). Coding, in my opinion, is very similar to writing. You can write messy code that no one understands, and you can write messy essays. On the flip side, you can write clear, concise, idiomatic code, and you can write clear, concise, expressive essays. 

I hope this post helps you write more idiomatic and concise Snowflake SQL—it'll save you and whoever reads your code from the feeling of dread people get when they look at bad code (or bad writing). 

![Young girl practicing coding on iPad](https://images.unsplash.com/photo-1599666505327-7758b44a9985?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=2850&q=80)
<span class="photo-credit">Photo by <a href="https://unsplash.com/@kellysikkema">Kelly Sikkema</a> on <a href="https://unsplash.com/">Unsplash</a></span>

Okay, for my next blog post, I plan on writing about LATERAL JOINs since I mentioned they tend to scare people. Let's make them less scary, for you and me lol.