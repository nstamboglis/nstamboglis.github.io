---
layout: post
title: "8 Weeks SQL Challenge: Week 1"
date: 2022-03-13
---

**TLDR**: Knowing SQL is a must for data professionals. No point learning advanced techniques without knowing how to extract data. No one is going to extract data for you (or, at least, it is rare to avoid bottlenecks). SQL is a great tool to learn how to *think* in terms of data structures. I personally love subqueries, the functions PARTITION BY, CASE and indexes. Order matters.

**SQL Motivation**

Ok, after flirting with the idea of deep diving into SQL, I've finally made the decision to get started with Danny MA's <a href="https://8weeksqlchallenge.com/">8  Weeks SQL Challenge</a>. The decision was easy (SQL is a key skill), the implementation a bit harder (my 3-months old toddler keeps me busy). 

Committing to this challenge was my turning point. Working in small sprints made it happen.

I already know a bit about SQL as I use it professionally at work. So why starting this challenge? Why not jumping straight into deep learning? Two reasons: first, I wanted to move beyond the context-specific problems I'm already solving at work, second, I wanted to boost my confidence and finally state that I know advanced SQL.

So, here's how the first week went.

**Welcome to Dannys'**

Danny is a gourmant (he loves good food). In a few clicks one can enter and explore what his customers' order from the menu. Luckily Danny is also skilled in SQL, which means that information on such tastes is well organised and accessibles. In 7 words: one can learn SQL through real-life examples.

All exercises are both relistic and a good approximation of stakeholders' questions. Also, one can test-drive them in real time in a dedicated environment, which is helpful for effective learning (although <a href="https://dbeaver.io/">DBeaver</a> the error messages are bit clearer).

**Anatomy of a query**
Thinking in SQL terms is both fun and effective. The required mindset to use this language effectively is quite different from the data-analysis one. I am trained as an economist, hence when I think about data issues I have a regression-like perspective: 

<ul   style="font-size:20px;">
  <li>What explains what?</li>
  <li>Where does partial correlation lies?</li>
  <li>How robust the identified partial-correlation is likely to be?</li>
</ul>

When it comes to SQL I find the approach to be rather different. When I SQL (is it a verb?) I think in terms of nested cubes, or matryoshka dolls. I see the problem at hand and I try to address it in consecutive steps: 

<ul  style="font-size:20px;">
  <li>What data structure would make the problem clearer?</li>
  <li>What features / calculated fields do I need to add to get my answer?</li>
  <li>Which aggregations are needed to summarize my answer effectively?</li>
</ul>

This mindset is great to really understand databases, a skill for life. On the plus side developing this point of view greatly helps in communicating with my fellow data engineers, reducing the chances of getting lost in translation.

**Week 1: no jokes**

In the lines below I share what I've found more useful during the learning process and what I think might be helpful for future use. Danny's week one case study can be found <a href="https://8weeksqlchallenge.com/case-study-1/">here</a>. You can find my complete anwers <a href="https://github.com/nstamboglis/8WeekSQLChallenge/blob/main/DM8WSC_W1.sql">here</a> if you're curious (let me know if you find bugs and/or better ways of solving the use cases!). 

*Sub-queries:*

Sub-queries queries are essentially steps to break down the problem at hand. Via a sub-queries the analyst can first extract the information he/she wants (in the internal query), to then elaborate it further in a sub-sequent query (the external one), as in the example below:

<body>
<pre>
<code  style="color:grey;font-size:18px;">
-- What is the total amount each customer spent at the restaurant?
SELECT
  A.customer_id,
  sum(A.expenditure) AS tot_expenditure
FROM(
  SELECT 
    sales.customer_id,
    sales.product_id,
    menu.price,
    (sales.product_id * menu.price) AS expenditure
  FROM dannys_diner.sales 
  LEFT JOIN dannys_diner.menu 
  ON sales.product_id = menu.product_id 
) A
GROUP BY A.customer_id
ORDER BY A.customer_id;
</code>
</pre>
</body>

First we extract information on expenditures (internal query), then we compute the amount each customer spent in total (external query). Love it.

*Case When:*

The case-when function is increadibly useful to use when dealing with special cases, such as customers loyalty schemes weighting products differently, as in the example below: 

<body>
<pre>
<code  style="color:grey;font-size:18px;">
-- If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
SELECT
  SUM(table2.price * table2.score_weights) as total_score,
  table2.customer_id
FROM(
  SELECT
    table1.customer_id,
    table1.product_id,
    table1.order_date,
    table1.join_date,
    (table1.order_date - table1.join_date) AS days_FROM_join,
    menu.product_name,
    menu.price,
    CASE
      WHEN menu.product_name = 'sushi' then 2
      ELSE 1
    END AS score_weights
  FROM(
    SELECT
      sales.customer_id,
      sales.product_id,
      sales.order_date,
      members.join_date
    FROM dannys_diner.sales
    LEFT JOIN dannys_diner.members ON sales.customer_id = members.customer_id
    ORDER BY sales.order_date DESC
  ) AS table1
  LEFT JOIN dannys_diner.menu ON table1.product_id = menu.product_id
  WHERE (table1.order_date - table1.join_date) >= 0
  ORDER BY
  table1.customer_id DESC,
  table1.order_date asc) AS table2
GROUP BY table2.customer_id
ORDER BY total_score desc;
</code>
</pre>
</body>

*Partition by:*

This function is extremely useful to divide the result sets in partitions and to perform computations in each subset of partitioned data. In other words, we take a subset of the data (such as customer's entries) and we perform computations in that specific group of entries (e.g. sales by customer)

<body>
<pre>
<code  style="color:grey;font-size:18px;">
-- What was the first item FROM the menu purchased by each customer?
SELECT customer_id, product_id
FROM(
  SELECT 
    row_number() over(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date asc
      RANGE BETWEEN 
      UNBOUNDED PRECEDING AND
      UNBOUNDED FOLLOWING) 
    row_number_ind,
    sales.customer_id, 
    sales.product_id
FROM dannys_diner.sales) tab1
WHERE row_number_ind = 1;
</code>
</pre>
</body>

In the example above it is possible to create a row counter for each row associated with a given customer id. This will be quite useful as a trick for the point below.

*Indexes:*

We can use the partition by trick above to use it as a sort-of-index for our table. This allows us to select the first row associated to a given customer (such as its first order ever). There might be more efficient ways of doing so, but I find indexes quite fun.

<body>
<pre>
<code style="color:grey;font-size:18px;">
-- What was the first item FROM the menu purchased by each customer?
SELECT customer_id, product_id
FROM(
  SELECT 
    row_number() over(
      PARTITION BY sales.customer_id
      ORDER BY sales.order_date asc
      RANGE BETWEEN 
      UNBOUNDED PRECEDING AND
      UNBOUNDED FOLLOWING)
    row_number_ind, 
    sales.customer_id, 
    sales.product_id
FROM dannys_diner.sales) tab1
WHERE row_number_ind = 1;
</code>
</pre>
</body>

*Bonus point: style & order matter!*

At the end of the day style and order make a huge difference in the quality of a query, but in terms of readibility and in terms of reliability of results. A slight change in the order of the *order by* option in one of the queries above whould have led to completely different results when indexes were involved. Also, I like a *tidy* query where its single elements are clearly identifiable. It will make going back to the exercise much easier, as well as sharing the result with a colleague or a friend. Also, it's a sign of *craftmanship*.

**Next steps**

Completing the first week was a motivation booster for me. Now I plan to learn the resources and explore other peoples' solutions. I'd like to make sure I'm approaching the tasks correctly rather than progressing with faulty ways. Once the challenge will be completed I plan to purchase Dannys' solutions to make sure I got them right (price is reasonable).
