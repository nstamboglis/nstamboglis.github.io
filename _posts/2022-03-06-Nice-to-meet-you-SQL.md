---
layout: post
title: "Nice to meet you SQL!"
date: 2022-03-06
---

**TLDR**: Knowing SQL is a must for data professionals. No point learning advanced techniques without knowing how to extract data. No one is going to extract data for you (or, at least, it is rare to avoid bottlenecks). SQL is a great tool to learn how to *think* in terms of data structures. I personally love subqueries, the functions PARTITION BY, CASE and indexes. Order matters.

**SQL Motivation**

Ok, after flirting with the idea of deep diving into SQL, I've finally made the decision to get started with Danny MA's <a href="https://8weeksqlchallenge.com/">8  Weeks SQL Challenge</a>. The decision was easy (SQL is a key skill), the implementation a bit harder (I just had a 3-months old toddler). 

Committing to this challenge was my turning point. Working in small sprints made it happen.

I already know a bit about SQL as I use it professionally at work. So why starting this challenge? Why not jumping straight into deep learning? Two reasons: first, I wanted to move beyond the context-specific problems I'm already solving at work, second, I wanted to boost my confidence and finally state that I know advanced SQL (F-You Impostor Syndrome!).

So, here's how the first week went.

**Welcome to Dannys'**

Danny is a gourmant (he loves good food). In a few clicks one can enter and explore what his customers' order from the menu. Luckily Danny is also skilled in SQL, which means that information on such tastes is well organised and accessibles. In 7 words: one can learn SQL through real-life examples.

I love the realism of the exercises, good approximation of stakeholders' questions. Also, one can test-drive them in real time in a dedicated environment, which is helpful for effective learning (although <a href="https://dbeaver.io/">DBeaver</a> the error messages are bit clearer).

**Anatomy of a query**

I love thinking in SQL terms. The required mindset to use this language effectively is quite different from the data-analysis one. I am trained as an economist, hence when I think about data issues I have a regression-mindset: 

<ul>
  <li>What explains what?</li>
  <li>Where does partial correlation lies?</li>
  <li>How robust the identified partial-correlation is likely to be?</li>
</ul>

When it comes to SQL I find the approach to be rather different. When I SQL (is it a verb?) I think in terms of nested cubes, or matryoshka dolls. I see the problem at hand and I try to address it in consecutive steps: 

<ul>
  <li>What data structure would make the problem clearer?</li>
  <li>What features / calculated fields do I need to add to get my answer?</li>
  <li>Which aggregations are needed to summarize my answer effectively?</li>
</ul>

This mindset is great to really understand databases, a skill for life. On the plus side getting into this mindset greatly helps in communicating with my fellow data engineers, reducing the chances of getting lost in translation.

**Week 1: no jokes**

Here I share what I loved the most about the learning process and what I think might be helpful for future use. You can find my complete anwers <a href="https://github.com/nstamboglis/8WeekSQLChallenge/blob/main/DM8WSC_W1.sql">here</a> if you're curious (let me know if you find bugs and/or better ways of solving the use cases!). 

*Sub-queries:*

I love the SQL concept of sub-queries. Essentially via a sub-query the analyst can first extract the information he/she wants (in the internal query), to then elaborate it further in a sub-sequent query (the external one), as in the example below:


<code>
<p>-- What is the total amount each customer spent at the restaurant?</p>
<p>SELECT </p>
	<p style="margin-left:5%; margin-right:5%;"> A.customer_id,</p> 
	<p style="margin-left:5%; margin-right:5%;"> sum(A.expenditure) AS tot_expenditure </p>
<p>FROM(</p>
	<p style="margin-left:5%; margin-right:5%;"> SELECT</p> 
		<p style="margin-left:10%; margin-right:5%;"> sales.customer_id, </p>
		<p style="margin-left:10%; margin-right:5%;"> sales.product_id, </p>
		<p style="margin-left:10%; margin-right:5%;"> menu.price, </p>
		<p style="margin-left:10%; margin-right:5%;"> (sales.product_id * menu.price) AS expenditure</p>
	<p style="margin-left:5%; margin-right:5%;"> FROM dannys_diner.sales</p> 
	<p style="margin-left:5%; margin-right:5%;"> LEFT JOIN dannys_diner.menu </p> 
  <p style="margin-left:5%; margin-right:5%;"> ON sales.product_id = menu.product_id</p> 
<p> ) A</p>
<p>GROUP BY A.customer_id</p>
<p>ORDER BY A.customer_id;</p>
</code>

First we extract information on expenditures (internal query), then we compute the amount each customer spent in total (external query). Love it.

*Case When:*

The case-when function is increadibly useful to use when dealing with special cases, such as customers loyalty schemes weighting products differently, as in the example below: 

<code>
<p>-- If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?</p>
<p>SELECT</p>
<p style="margin-left:5%; margin-right:5%;"> 	sum(table2.price * table2.score_weights) as total_score,</p>
<p style="margin-left:5%; margin-right:5%;">     table2.customer_id</p>
 <p>FROM(</p>
<p style="margin-left:5%; margin-right:5%;"> 	SELECT </p>
<p style="margin-left:10%; margin-right:5%;">        table1.customer_id,</p>
<p style="margin-left:10%; margin-right:5%;">        table1.product_id,</p>
<p style="margin-left:10%; margin-right:5%;">        table1.order_date,</p>
<p style="margin-left:10%; margin-right:5%;">        table1.join_date,</p>
<p style="margin-left:10%; margin-right:5%;">        (table1.order_date - table1.join_date) AS days_FROM_join,</p>
<p style="margin-left:10%; margin-right:5%;">        menu.product_name,</p>
<p style="margin-left:10%; margin-right:5%;">        menu.price,</p>
<p style="margin-left:10%; margin-right:5%;">        CASE </p>
<p style="margin-left:15%; margin-right:5%;">        	WHEN menu.product_name = 'sushi' then 2</p>
<p style="margin-left:15%; margin-right:5%;">            ELSE 1 </p>
<p style="margin-left:10%; margin-right:5%;">        END AS score_weights</p>
<p style="margin-left:5%; margin-right:5%;">     FROM(</p>
<p style="margin-left:10%; margin-right:5%;">        SELECT</p>
<p style="margin-left:15%; margin-right:5%;">            sales.customer_id,</p>
<p style="margin-left:15%; margin-right:5%;">            sales.product_id,</p>
<p style="margin-left:15%; margin-right:5%;">            sales.order_date,</p>
<p style="margin-left:15%; margin-right:5%;">            members.join_date</p>
<p style="margin-left:10%; margin-right:5%;">        FROM dannys_diner.sales</p>
<p style="margin-left:10%; margin-right:5%;">        LEFT JOIN dannys_diner.members ON sales.customer_id = members.customer_id</p>
 <p style="margin-left:10%; margin-right:5%;">     ORDER BY sales.order_date DESC</p>
 <p style="margin-left:5%; margin-right:5%;">    ) AS table1</p>
 <p style="margin-left:5%; margin-right:5%;">    LEFT JOIN dannys_diner.</p>menu ON table1.product_id = menu.product_id
<p>	WHERE (table1.order_date - table1.join_date) >= 0</p>
<p style="margin-left:5%; margin-right:5%;">     ORDER BY </p>
<p style="margin-left:10%; margin-right:5%;">        table1.customer_id DESC,</p>
<p style="margin-left:10%; margin-right:5%;">        table1.order_date asc) AS table2</p>
 <p> GROUP BY table2.customer_id</p>
 <p> ORDER BY total_score desc;</p>
</code>

*Partition by:*

This function is extremely useful to divide the result sets in partitions and to perform computations in each subset of partitioned data. In other words, we take a subset of the data (such as customer's entries) and we perform computations in that specific group of entries (e.g. sales by customer)

<code>
<p>-- What was the first item FROM the menu purchased by each customer?</p>
<p>SELECT customer_id, product_id</p>
<p>FROM(</p>
	<p style="margin-left:5%; margin-right:5%;">SELECT row_number() over(</p>
 	<p style="margin-left:5%; margin-right:5%;">PARTITION BY sales.customer_id</p>
  <p style="margin-left:5%; margin-right:5%;">	ORDER BY sales.order_date asc</p>
  <p style="margin-left:5%; margin-right:5%;">	RANGE BETWEEN </p>
      <p style="margin-left:5%; margin-right:5%;">	UNBOUNDED PRECEDING AND</p> 
      <p style="margin-left:5%; margin-right:5%;">	UNBOUNDED FOLLOWING) <p style="margin-left:5%; margin-right:5%;">row_number_ind, </p>
	<p style="margin-left:5%; margin-right:5%;">sales.customer_id, </p>
	<p style="margin-left:5%; margin-right:5%;">sales.product_id</p>
<p>FROM dannys_diner.sales) tab1</p>
<p>WHERE row_number_ind = 1;</p>
</code>

In the example above it is possible to create a row counter for each row associated with a given customer id. This will be quite useful as a trick for the point below.

*Indexes:*

We can use the partition by trick above to use it as a sort-of-index for our table. This allows us to select the first row associated to a given customer (such as its first order ever). There might be more efficient ways of doing so, but I find indexes quite fun.

<code>
<p>-- What was the first item FROM the menu purchased by each customer?</p>
<p>SELECT customer_id, product_id</p>
<p>FROM(</p>
	<p style="margin-left:5%; margin-right:5%;">SELECT row_number() over(</p>
 	<p style="margin-left:5%; margin-right:5%;">PARTITION BY sales.customer_id</p>
  <p style="margin-left:5%; margin-right:5%;">	ORDER BY sales.order_date asc</p>
  <p style="margin-left:5%; margin-right:5%;">	RANGE BETWEEN </p>
      <p style="margin-left:5%; margin-right:5%;">	UNBOUNDED PRECEDING AND</p> 
      <p style="margin-left:5%; margin-right:5%;">	UNBOUNDED FOLLOWING) <p style="margin-left:5%; margin-right:5%;">row_number_ind, </p>
	<p style="margin-left:5%; margin-right:5%;">sales.customer_id, </p>
	<p style="margin-left:5%; margin-right:5%;">sales.product_id</p>
<p>FROM dannys_diner.sales) tab1</p>
<p>WHERE row_number_ind = 1;</p>
</code>

*Bonus point: style & order matter!*

At the end of the day style and order make a huge difference in the quality of a query, but in terms of readibility and in terms of reliability of results. A slight change in the order of the *order by* option in one of the queries above whould have led to completely different results when indexes were involved. Also, I like a *tidy* query where its single elements are clearly identifiable. It will make going back to the exercise much easier, as well as sharing the result with a colleague or a friend. Also, it's a sign of *craftmanship*.

**Next steps**

Completing the first week was a motivation booster for me. Now I plan to learn the resources and explore other peoples' solutions. I'd like to make sure I'm approaching the tasks correctly rather than progressing with faulty ways. Once the challenge will be completed I plan to purchase Dannys' solutions to make sure I got them right (price is reasonable).
