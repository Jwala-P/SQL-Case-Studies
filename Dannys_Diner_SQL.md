SQL - Danny's Diner

1. What is the total amount each customer spent at the restaurant?

Code:

SELECT s.customer_id, SUM(m.price) AS total_amount 
FROM sales s 
INNER JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_amount DESC;

Output:

Query #1 
customer_id	total_amount
A	76
B	74
C	36

2. How many days has each customer visited the restaurant?

Code:

SELECT s.customer_id, COUNT(DISTINCT s.order_date) AS total_days
FROM sales s
GROUP BY s.customer_id;

Output:
Query #2
customer_id	total_days
A	4
B	6
C	2

3. What was the first item from the menu purchased by each customer?

Code:

WITH ranked_orders AS (
    SELECT 
        s.customer_id, 
        m.product_name, 
        RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS order_no
    FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
)
SELECT customer_id, 
       product_name
FROM ranked_orders
WHERE order_no = 1
ORDER BY customer_id;

Output:

Query #3
customer_id	product_name
A	curry
A	sushi
B	curry
C	ramen
C	ramen

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

Code:

WITH orders AS (
	SELECT m.product_name,COUNT(s.product_id) AS total_orders
  	FROM sales s
  	INNER JOIN menu m ON s.product_id = m.product_id
  	GROUP BY m.product_name
 	ORDER BY total_orders DESC
)
SELECT *
FROM orders
LIMIT 1;

Output:

Query #4
product_name	total_orders
ramen	8

5. Which item was the most popular for each customer?

Code:

WITH orders AS (
	SELECT s.customer_id,m.product_name,COUNT(s.product_id) AS total_orders, RANK() OVER (PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS rank
  	FROM sales s
  	INNER JOIN menu m ON s.product_id = m.product_id
  	GROUP BY m.product_name,s.customer_id
 	ORDER BY total_orders DESC
)
SELECT customer_id, product_name, total_orders
FROM orders
WHERE rank = 1;

Output:

Query #5
customer_id	product_name	total_orders
C	ramen	3
A	ramen	3
B	sushi	2
B	ramen	2
B	curry	2

6. Which item was purchased first by the customer after they became a member?

Code:
WITH member_orders AS (
    SELECT 
        s.customer_id, 
        m.product_name,
  		s.order_date,
        RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS order_no
    FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
  	
  	INNER JOIN members me ON s.customer_id = me.customer_id
  
  	WHERE s.order_date>=me.join_date
  
)

SELECT customer_id, product_name, order_date 
FROM member_orders 
WHERE order_no=1;

Output:

Query #6
customer_id	product_name	order_date
A	curry	2021-01-07
B	sushi	2021-01-11

7. Which item was purchased just before the customer became a member?

Code:

WITH member_orders AS (
    SELECT 
        s.customer_id, 
        m.product_name,
  		s.order_date,
        RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS order_no
    FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
  	
  	INNER JOIN members me ON s.customer_id = me.customer_id 
  	WHERE s.order_date < me.join_date
)

SELECT customer_id, product_name, order_date 
FROM member_orders
WHERE order_no=1;

Output:

Query #7
customer_id	product_name	order_date
A	sushi	2021-01-01
A	curry	2021-01-01
B	sushi	2021-01-04

8. What is the total items and amount spent for each member before they became a member?

Code:

WITH member_orders AS (
    SELECT 
        s.customer_id, 
  		COUNT(s.product_id),
  		SUM(m.price)
    FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
  	
  	LEFT JOIN members me ON s.customer_id = me.customer_id 
  	WHERE 
    me.join_date IS NULL OR s.order_date < me.join_date
  
  	GROUP BY s.customer_id
)

SELECT mo.customer_id, mo.sum
FROM member_orders mo
ORDER BY mo.customer_id;

Output:

Query #8
customer_id	sum
A	25
B	40
C	36

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Code:

WITH member_orders AS (
    SELECT 
        s.customer_id, 
  		s.product_id,
  		m.price*CASE 
  					WHEN s.product_id=1 THEN 20 
  					ELSE 10
  					END AS points
    FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
)

SELECT mo.customer_id, SUM(mo.points) AS total_points
FROM member_orders mo
GROUP BY mo.customer_id
ORDER BY mo.customer_id;

Output:

Query #9
customer_id	total_points
A	860
B	940
C	360

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

Code:

WITH member_orders AS (
    SELECT 
        s.customer_id, 
  		s.product_id,
  		m.price*CASE 
  					WHEN s.product_id=1 THEN 20
  					WHEN s.order_date BETWEEN me.join_date AND me.join_date + INTERVAL '6 DAYS' THEN 20
  					ELSE 10
  					END AS points
  	FROM 
        sales s
    INNER JOIN 
        menu m ON s.product_id = m.product_id
  	INNER JOIN 	
  		members me ON me.customer_id=s.customer_id
  	WHERE s.order_date<='2021-01-31'
)

SELECT mo.customer_id, SUM(mo.points) AS total_points
FROM member_orders mo
GROUP BY mo.customer_id
ORDER BY mo.customer_id;

Output:

Query #10 
customer_id	total_points
A	1370
B	820