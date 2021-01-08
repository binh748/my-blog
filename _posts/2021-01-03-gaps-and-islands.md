---
title: "Smarter SQL solution to gaps & islands problem"
categories:
  - Blog
tags:
  - SQL
  - leetcode
  - debugging
  - gaps and islands
excerpt: "I first encountered the \"gaps and islands\" problem while doing SQL problems on Leetcode."
---
# Table of Contents
1. [Introduction](#introduction)
2. [Solving Gaps & Islands Problem](#solving-gaps--islands-problem)
3. [Fixing Gaps & Islands Bug](#fixing-gaps--islands-bug)
4. [Conclusion](#conclusion)

# Introduction
Up to this point, my posts have been about my data science projects. With 2021 starting, I want to add new types of posts where I discuss a specific data science/coding problem, its nuances, and how I solved it. Let’s jump right in!

I first encountered the "gaps and islands" problem while doing SQL problems on LeetCode. The problem occurs when you want to identify groups of consecutive data that meet certain criteria (“islands”) and the gaps that exist in between those groups (“gaps”). The gaps could be due to missing data or consecutive data that do not meet certain criteria.

I learned how to solve this problem using a grouping technique that’s explained in a number of online resources, but what I found in doing more of these problems is that the resources don’t address a corner case where the groupings for the islands and gaps can overlap, causing a bug in the solution.

In this post, I’ll explain the conventional way to solve the problem, the corner case that causes the bug, and how I fix it.

# Solving Gaps & Islands Problem

## LeetCode SQL Problem 1454. Active Users
From September to December 2020, I was practicing SQL to prepare for data analyst job interviews using resources like LeetCode, Mode, and pgexercises.com. Pounding SQL exercises day and night may sound dreary, but the fun thing is that I’d occasionally encounter a problem that stretched my thinking, for example, LeetCode problem [1454. Active Users](https://leetcode.com/problems/active-users/). The instructions for the query are

*Write an SQL query to find the id and the name of active users. Active users are those who logged in to their accounts for 5 or more consecutive days. Return the result table ordered by the id.*

The tricky thing about this problem is figuring out how to identify the active users: how do you use SQL to keep track of consecutive days?

I wasn’t able to figure it out on my own, so I read the following resources to familiarize myself with how to tackle this type of problem, which as you can guess is a gaps and islands problem:

1. [LeetCode user adrianlievano1993’s explanation](https://leetcode.com/problems/active-users/discuss/644070/100-Faster-than-Submissions-Window-Functions-Documentation-Linked)
2. [Matt Boegner’s blog post explanation](https://mattboegner.com/improve-your-sql-skills-master-the-gaps-islands-problem/)
3. [StackOverflow user GarethD’s explanation](https://stackoverflow.com/questions/26117179/sql-count-consecutive-days)

Here’s the basic outline of the solution, which I’ll go into more detail on later:

1. CTE #1: Use DENSE_RANK() and PARTITION BY the columns that tell you if your consecutive data meets certain criteria and ORDER BY the column that tells you if your data is consecutive (e.g. a date or id column).
2. CTE #2: Create a column of groupings by subtracting the DENSE_RANK() column created in CTE #1 from the column that tells you if your data is consecutive (e.g. a date or id column).
3. CTE #3: Perform a GROUP BY aggregation using the groupings created in CTE #2 and filter for the groups that meet the solution criteria.
4. Main query: Write the query to solve the problem using the filtered groups in CTE #3.

Implementing this solution for [1454. Active Users](https://leetcode.com/problems/active-users/) worked well. Full solution code below:

```sql
WITH dense_rank_cte AS (
    SELECT id, login_date,
           DENSE_RANK() OVER(PARTITION BY id ORDER BY login_date) AS dense_rank_num
      FROM Logins
),
grouping_cte AS (
    SELECT id, login_date, dense_rank_num,
           DATE_ADD(login_date, INTERVAL -dense_rank_num DAY) AS groupings
      FROM dense_rank_cte
),
grouping_info_cte AS (
    SELECT id, MIN(login_date) AS start_date, MAX(login_date) AS end_date,
           dense_rank_num, groupings, COUNT(*),
           DATEDIFF(MAX(login_date), MIN(login_date)) + 1 AS duration
      FROM grouping_cte
     GROUP BY id, groupings
    HAVING DATEDIFF(MAX(login_date), MIN(login_date)) + 1 >= 5
     ORDER BY id, start_date
)
SELECT DISTINCT g.id, a.name
  FROM grouping_info_cte AS g
  JOIN Accounts AS a
 USING(id)
 ORDER BY g.id;
```

## LeetCode Problem 601. Human Traffic of Stadium
Fast forward a week. I stumbled upon another gaps and islands problem with LeetCode problem [601. Human Traffic of Stadium](https://leetcode.com/problems/human-traffic-of-stadium/). The instructions for the query are

*Write an SQL query to display the records [in the Stadium table] with three or more rows with consecutive id's, and the number of people is greater than or equal to 100 for each. Return the result table ordered by visit_date in ascending order.*

![stadium-table-original](https://user-images.githubusercontent.com/62628676/103490187-1f6e3600-4de8-11eb-83b8-6b1b0cf10e99.png)

Initially, I was thinking “oh, this won’t be too difficult. I’ll just implement the same solution framework as before.” That’s what I did, but I ran into a bug that no one explained could happen in the resources I read earlier.

I’ll do a step-by-step walkthrough of my initial solution (shown below) that causes the bug.

```sql
WITH status_cte AS (
    SELECT *,
           CASE WHEN people >= 100 THEN 'high_traffic'
                ELSE 'low_traffic' END AS status
      FROM Stadium
     ORDER BY id
),
dense_rank_cte AS (
    SELECT *, DENSE_RANK() OVER(PARTITION BY status ORDER BY id) AS rankings
      FROM status_cte
     ORDER BY id
),
grouping_cte AS (
    SELECT *, id - rankings AS groupings
      FROM dense_rank_cte
     ORDER BY id
),
agg_cte AS (
    SELECT groupings, SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) AS num_days_high_traffic
      FROM grouping_cte
     GROUP BY 1
    HAVING SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) >= 3
)
SELECT id, visit_date, people
  FROM grouping_cte
 WHERE groupings IN (SELECT groupings FROM agg_cte)
 ORDER BY visit_date;
```

### CTE #1
I add a column called status using a CASE statement that tells me if the record is high_traffic or low_traffic depending on if the number of people that visited that day is greater than or equal to 100.

```sql
WITH status_cte AS (
    SELECT *,
           CASE WHEN people >= 100 THEN 'high_traffic'
                ELSE 'low_traffic' END AS status
      FROM Stadium
     ORDER BY id
),
```

![stadium-table-cte-1](https://user-images.githubusercontent.com/62628676/103490480-05355780-4dea-11eb-8628-335609681f9e.png)

### CTE #2
I add a column called rankings using DENSE_RANK() OVER(PARTITION BY status ORDER BY id). These rankings help set up the groupings I’ll create in the next CTE.

```sql
dense_rank_cte AS (
    SELECT *, DENSE_RANK() OVER(PARTITION BY status ORDER BY id) AS rankings
      FROM status_cte
     ORDER BY id
),
```

![stadium-table-cte-2](https://user-images.githubusercontent.com/62628676/103492093-a75b3c80-4df6-11eb-9f1e-a31cd9542174.png)

### CTE #3
I add a column called groupings by subtracting the rankings from the id column. Because of the logic of the rankings, the ids will be in the same grouping only if they’re consecutive and belong to the same status (at least that’s what you want if there were no bug). In other words, by using PARTITION BY status, the rankings are incremented for each status; this creates the incremental offset that we need so that when ranking is subtracted from id, we get our desired groupings.

However, if you look at the table below, grouping #2 has a low_traffic record grouped together with the high_traffic records. That’s possible due to chance: if the ids and rankings for the statuses align a certain way, the low_traffic and high_traffic records could have the same grouping number. This is not desired behavior, and this is what causes the bug. I’ll come back to this point.

```sql
grouping_cte AS (
    SELECT *, id - rankings AS groupings
      FROM dense_rank_cte
     ORDER BY id
),
```
![stadium-table-cte-3](https://user-images.githubusercontent.com/62628676/103492809-8ea15580-4dfb-11eb-8236-26740abd41d8.png)

### CTE #4
I GROUP BY the groupings and count the number of days for each grouping where the record is a high_traffic record (i.e. 100 or more people visited that day), which I call the num_days_high_traffic column. I then filter the groupings for only those groupings where the num_days_high_traffic is greater than or equal to 3, which means that those are the groupings where the number of consecutive ids are greater than or equal to 3 AND the number of people who visited those days is greater than or equal to 100.

```sql
agg_cte AS (
    SELECT groupings, SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) AS num_days_high_traffic
      FROM grouping_cte
     GROUP BY 1
    HAVING SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) >= 3
)
```

![stadium-table-cte-4](https://user-images.githubusercontent.com/62628676/103493548-0ec9ba00-4e00-11eb-9712-2162d3a670cd.png)

### Main Query
Finally, I SELECT the id, visit_date, and people for the records that belong in grouping #2 (that’s the only grouping that satisfies the solution criteria) and ORDER BY visit_date. This should give us the answer if it were not for the bug in CTE #3. Because of the bug, my answer also returns the id, visit_date, and people from id 4 because it’s part of grouping #2 even though it’s a low_traffic record.

```sql
SELECT id, visit_date, people
  FROM grouping_cte
 WHERE groupings IN (SELECT groupings FROM agg_cte)
 ORDER BY visit_date;
```
![incorrect-answer](https://user-images.githubusercontent.com/62628676/103492836-b42e5f00-4dfb-11eb-91a1-54984eec0843.png)

# Fixing Gaps & Islands Bug
There are two ways I could fix the bug: a hacky way and a sustainable way. Let’s look at the hacky way first.

## Hacky Fix
The hacky fix is simple. In my main query, I add another condition to my WHERE clause to only return high_traffic records. That means that even though id 4 is a low_traffic record in grouping #2, it’s filtered out by my WHERE clause. Great, but not the best fix because if someone were to inspect the intermediate results of my code (i.e. the CTEs), they would still see that grouping #2 has high_traffic and low_traffic records mixed together and wonder what happened there.

```sql
SELECT id, visit_date, people
  FROM grouping_cte
 WHERE groupings IN (SELECT groupings FROM agg_cte) AND status = 'high_traffic'
 ORDER BY visit_date;
```

## Sustainable Fix
The sustainable fix takes a little more work than the hacky fix, but not that much more. We just add another CTE after CTE #3 to differentiate the low_traffic groupings from the high_traffic groupings. In this new CTE, I use a CASE statement to add an H (H for high_traffic) to the grouping number for high_traffic records and an L for low_traffic records. This way, if high_traffic and low_traffic records share the same grouping number, they’ll be differentiated by the letters H and L. The rest of the solution follows the same pattern except I’ll be referring to the differentiated groupings when I do my GROUP BY and in my main query. Using this fix, I don’t need to add an extra condition to the WHERE clause of my main query to filter out the low_traffic records because they belong to a different grouping entirely. Awesome!

```sql
grouping_differentiated_cte AS (
    SELECT *, CASE WHEN status = 'high_traffic' THEN CONCAT(CAST(groupings AS char), 'H')
                   ELSE CONCAT(CAST(groupings AS char), 'L') END AS groupings_diff
      FROM grouping_cte
     ORDER BY id
),
```

![stadium-table-cte-3.1](https://user-images.githubusercontent.com/62628676/103492820-9bbe4480-4dfb-11eb-9ac1-21282dc51e86.png)

![correct-answer](https://user-images.githubusercontent.com/62628676/103492259-98c15500-4df7-11eb-9004-9590a47614cc.png)

```sql
WITH status_cte AS (
    SELECT *,
           CASE WHEN people >= 100 THEN 'high_traffic'
                ELSE 'low_traffic' END AS status
      FROM Stadium
     ORDER BY id
),
dense_rank_cte AS (
    SELECT *, DENSE_RANK() OVER(PARTITION BY status ORDER BY id) AS rankings
      FROM status_cte
     ORDER BY id
),
grouping_cte AS (
    SELECT *, id - rankings AS groupings
      FROM dense_rank_cte
     ORDER BY id
),
grouping_differentiated_cte AS (
    SELECT *, CASE WHEN status = 'high_traffic' THEN CONCAT(CAST(groupings AS char), 'H')
                   ELSE CONCAT(CAST(groupings AS char), 'L') END AS groupings_diff
      FROM grouping_cte
     ORDER BY id
),
agg_cte AS (
    SELECT groupings_diff, SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) AS num_days_high_traffic
      FROM grouping_differentiated_cte
     GROUP BY 1
    HAVING SUM(CASE WHEN status = 'high_traffic' THEN 1 ELSE 0 END) >= 3
)
SELECT id, visit_date, people
  FROM grouping_differentiated_cte
 WHERE groupings_diff IN (SELECT groupings_diff FROM agg_cte)
 ORDER BY visit_date;
```
<span class="caption">This is the full solution with the sustainable fix.</span>

# Conclusion
Now whenever you encounter a gaps and islands problem, you’ll know how to solve it and won’t make the same mistake I made. I hope I was able to expand the universe of data science knowledge just a tiny bit with this post.

All code on [GitHub](https://github.com/binh748/leetcode-problems/blob/main/sql_solutions.sql).
