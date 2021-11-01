# 8 WEEKS SQL Challange - Week 1

 <a href="https://github.com/orkunaran/danny_ma_8weeksSQL/issues">
  <img alt="GitHub issues" src="https://img.shields.io/github/issues/orkunaran/danny_ma_8weeksSQL_week1">
 </a>
 
 <img src = 'https://badges.pufler.dev/visits/orkunaran/danny_ma_8weeksSQL_week1'>
<p>

<img src = 'https://8weeksqlchallenge.com/images/case-study-designs/1.png' >
<p>



## WEEK 1 - Danny's Dinner
	
This challenge needs us to determine customer movements with 10 different questions. You may find my approaches to these challenges below. But first let's summarize the challenge description. And you may find the source here in this [link](https://8weeksqlchallenge.com/case-study-1/).
	
	
## Challenge Description
	
We have 3 tables from a restaurant database: sales, menu and members. Each table includes specific information (see diagram below). Our mission is to assist the restaurant about customers visiting patterns, how much they have spent and their favourite items.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

	
<img src = 'https://miro.medium.com/max/595/1*fEmZXjnIof5BHL_sLGDVUg.png' >
<p>

	
You may find the .sql file that creates the tables in the repository (MySQL).

## Challanges and Solutions

You may find my solutions in MySQL and my intuition on them with some comments.  


Question 1 : What is the total amount each customer spent at the restaurant?

Solution: That was a starter challenge. I joined two tables; sales and menu. And returned customer_id and SUM of prices (aliased as money_spent) and grouped them by customer_id. 
	
``` sql
SELECT customer_id, SUM(price) AS money_spent FROM sales 
	JOIN menu ON menu.product_id = sales.product_id
	GROUP BY customer_id
```


Question 2. How many days has each customer visited the restaurant?
	
Solution : I used DISTINCT() and COUNT() function here. Reasons are obvious; I needed to count the dates that the customers visited, but in case of avoiding duplicates in orders (visitors might had ordered 2 items in one day) I used DISTINCT() function. 
	
	
``` sql
SELECT customer_id, COUNT(DISTINCT(order_date)) FROM sales
	GROUP BY customer_id
```

Question 3. What was the first item from the menu purchased by each customer?
	
Solution: I used SELECT within SELECT operation here. The first SELECT is to choose unique customer_id, and product names from sales and menu tables. After that I added a condition WHERE order_date is equal to MINIMUM order_dates. There was a downside of that challenge: One customer ordered 2 items in the first day he/she visited; thus we cannot say which one of the two items is the first one cause we had no additional information such as time. But on the other hand, we might not need that also. 
	
```sql
SELECT DISTINCT(customer_id), product_name FROM sales s
	JOIN menu m ON m.product_id = s.product_id
	WHERE s.order_date = ANY (SELECT MIN(order_date) FROM sales GROUP BY customer_id)
```


Question 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
	
Solution: Another JOIN challenge; I joined two tables (sales and menu) on product_id. Grouped the table by product_name, sorted DESCending and LIMITed the output by 1 row for each product_name. 
	
```sql
SELECT  COUNT(product_name) AS count, product_name FROM sales s 
	JOIN menu m ON s.product_id = m.product_id
	GROUP BY product_name ORDER BY count DESC LIMIT 1
```

Question 5. Which item was the most popular for each customer?
	
Solution: I learnt DENSE_RANK and RANK functions here. So what I did was created a table named 'r' in which the customers ranked by their COUNT of bought products. After that, rest was easy; choose customer_id, productname  and count from 'r' where the ranks are equal to 1. 

```sql
WITH r AS 
	(SELECT s.customer_id,
		m.product_name,
		COUNT(s.product_id) as count,
        DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS r
	FROM menu m 
	JOIN sales s 
	ON s.product_id = m.product_id
	GROUP BY s.customer_id, s.product_id, m.product_name) 
SELECT customer_id, product_name, count
FROM r
WHERE r = 1
```

Question 6. Which item was purchased first by the customer after they became a member?
	
Solution: More practice on DENSE_RANK(); Created a ranks table in which customers ranked by their order date. After ordering, choosed order dates which are bigger or equal to membership date, and same as above choosed Ranks equal to 1. 

```sql
 WITH ranks AS
(
SELECT s.customer_id,
       m.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS ranks
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members AS mem
ON mem.customer_id = s.customer_id
WHERE s.order_date >= mem.join_date)
SELECT * FROM ranks
WHERE ranks = 1
```


7. Which item was purchased just before the customer became a member? 
	
Solution: Same as question 6, just changes order date lower than membershipdate. 
````sql
WITH ranks AS
(
SELECT s.customer_id,
		s.order_date,
       m.product_name,
	DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS ranks, mem.join_date
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members AS mem
ON mem.customer_id = s.customer_id
WHERE s.order_date < mem.join_date)
SELECT * FROM ranks
WHERE ranks = 1
````

8. What is the total items and amount spent for each member before they became a member?

Solution: Joined 3 tables together and choosed order_date lower than joining date, and group by the customer_id.

````sql

SELECT s.customer_id, 
	count(s.product_id) AS total_items, 
        SUM(price) AS money_spent
FROM sales AS s
JOIN menu AS m ON m.product_id = s.product_id
JOIN members AS mem ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY s.customer_id
````
	
	
Question 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
	
Solution: A CASE WHEN challenge. I was aware of that function thanks to HackerRank. CASE WHEN function creates a table named 'points'. Points are calculated as mentioned as in the question. After that, points and sales tables were JOINed together, and selected SUM(points) which grouped by customer_id.


	
	
````sql
WITH points AS 
(
SELECT *,
    CASE 
    WHEN m.product_name = 'sushi' THEN price * 20
    WHEN m.product_name != 'sushi' THEN price * 10
    END AS points
FROM menu m
    )
SELECT customer_id, SUM(points) AS points
FROM sales s
JOIN points p ON p.product_id = s.product_id
GROUP BY s.customer_id
````

OR 

````sql
SELECT customer_id, SUM(points) AS points
FROM
(
SELECT m.product_id,
	s.customer_id,
    CASE 
    WHEN m.product_name = 'sushi' THEN price * 20
    ELSE price * 10
    END AS points
FROM menu m 
JOIN sales s ON m.product_id = s.product_id
) AS t
GROUP BY customer_id
````

Question 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Solution: A bit more complex than the ones before; first created 'points' table with column that gives that the order made how many days after becoming a member (first_week). After that wrote a CASE WHEN function calculates total_points according to description. Finally, get the 1st month with EXTRACT function and get month equal to 1. 

````sql
SELECT customer_id, SUM(total_points)
FROM 
(WITH points AS
(
SELECT s.customer_id, 
	(s.order_date - mem.join_date) AS first_week,
        m.price,
        m.product_name,
        s.order_date
    FROM sales s
	JOIN menu m ON s.product_id = m.product_id
	JOIN members AS mem
	ON mem.customer_id = s.customer_id
    WHERE s.order_date > mem.join_date)
SELECT customer_id,
	CASE 
	WHEN first_week <= 7 THEN price * 20
        WHEN first_week > 7 AND product_name = 'sushi' THEN price * 20
	WHEN first_week > 7 AND product_name != 'sushi' THEN price * 10
        END AS total_points,
        order_date
FROM points 
WHERE EXTRACT(MONTH FROM order_date) = 1) as t
GROUP BY customer_id

````
