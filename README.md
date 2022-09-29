# DannyMa_SQL_Challenge

```
-- 1. What is the total amount each customer spent at the restaurant?


 SELECT M.[customer_id],SUM(P.[price])
 FROM [DannyMa].[sales] M
 LEFT JOIN [DannyMa].[menu] P
 ON M.product_id =P.product_id
 GROUP BY M.customer_id;

 -- 2. How many days has each customer visited the restaurant?
 
 
SELECT [customer_id],COUNT ([order_date]) No_of_days
FROM [DannyMa].[sales]
GROUP BY customer_id;

-- 3. What was the first item from the menu purchased by each customer?


SELECT  S.[customer_id],M.[product_name],S.[order_date]
INTO #table
FROM [DannyMa].[sales] S
LEFT JOIN [DannyMa].[menu] M
ON S.[product_id]=M.[product_id]


SELECT *
FROM #table;

WITH item AS (SELECT customer_id, 
product_name,
rank() over(partition by  customer_id order by order_date) first_item
FROM #table)
SELECT customer_id, product_name 
FROM item
WHERE first_item = 1;

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?


SELECT TOP 1 a.[product_name], COUNT(*) Frequency
FROM [DannyMa].[menu] a
LEFT JOIN [DannyMa].[sales] b
ON a.[product_id] =b.[product_id]
GROUP BY a.product_name
ORDER BY COUNT(*) DESC

-- 5. Which item was the most popular for each customer?


WITH product AS (SELECT  c.customer_id, c.product_name,c.Count,
RANK() OVER(PARTITION BY c.customer_id ORDER BY C.Count DESC) product_rank
FROM (
SELECT COUNT(*) Count,S.[customer_id], M.product_name 
FROM [DannyMa].[sales] S
LEFT JOIN [DannyMa].[menu] M
ON S.product_id= M.product_id
GROUP BY S.[customer_id], M.product_name) c
GROUP BY C.customer_id,C.product_name, C.Count)
SELECT customer_id, product_name,Count
FROM product
WHERE product_rank= 1

-- 6. Which item was purchased first by the customer after they became a member?


SELECT m.customer_id, n.product_name
FROM (SELECT  O.customer_id, order_date,
O.product_id, 
ROW_NUMBER() OVER(PARTITION BY O.customer_id ORDER BY order_date) rank
FROM(
SELECT a.[customer_id], a.[product_id],b.joined_date, a.order_date
FROM [DannyMa].[sales] a
INNER JOIN [DannyMa].[members] b
ON a.customer_id =b.customer_id
and order_date> joined_date) O) m
LEFT JOIN [DannyMa].[menu] n
ON m.product_id = n.product_id
WHERE m.rank = 1;

-- 7. Which item was purchased just before the customer became a member?


SELECT m.customer_id, n.product_name
FROM (SELECT  O.customer_id, order_date,
O.product_id, 
RANK() OVER(PARTITION BY O.customer_id ORDER BY order_date ) rank
FROM(
SELECT a.[customer_id], a.[product_id],b.joined_date, a.order_date
FROM [DannyMa].[sales] a
INNER JOIN [DannyMa].[members] b
ON a.customer_id =b.customer_id
and order_date< joined_date) O) m
LEFT JOIN [DannyMa].[menu] n
ON m.product_id = n.product_id
WHERE m.rank = 1;

-- 8. What is the total items and amount spent for each member before they became a member?


SELECT m.customer_id, SUM([price]) amount
FROM (SELECT  O.customer_id, order_date,
O.product_id, 
RANK() OVER(PARTITION BY O.customer_id ORDER BY order_date DESC) rank
FROM(
SELECT a.[customer_id], a.[product_id],b.joined_date, a.order_date
FROM [DannyMa].[sales] a
INNER JOIN [DannyMa].[members] b
ON a.customer_id =b.customer_id
and order_date< joined_date) O) m
LEFT JOIN [DannyMa].[menu] n
ON m.product_id = n.product_id
WHERE m.rank = 1
GROUP BY  m.customer_id;


-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?


SELECT mul.customer_id, SUM(mul.Points)
FROM 
(SELECT S.[customer_id],
CASE WHEN M.[product_name] = 'curry' THEN [price]* 10
     WHEN M.[product_name] = 'sushi' THEN [price]* 10
     WHEN M.[product_name] = 'ramen' THEN [price]* 20 end Points
FROM[DannyMa].[sales] S
LEFT JOIN [DannyMa].[menu] M
ON S.product_id= M.product_id) mul
GROUP BY customer_id;


-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

SELECT customer_id, product_id,first_week
INTO #first_week
FROM(
SELECT a.[customer_id], a.[product_id],b.joined_date,
a.order_date,DATEADD(day,7,joined_date) first_week
FROM [DannyMa].[sales] a
INNER JOIN [DannyMa].[members] b
ON a.customer_id =b.customer_id
and order_date> joined_date) a
WHERE order_date <= first_week;

SELECT *
FROM #first_week

SELECT  f.customer_id,SUM(m.price)*20
FROM #first_week f
LEFT JOIN [DannyMa].[menu] m
ON f.product_id=m.product_id
GROUP BY f.customer_id

```
