# Case Study #1 - Danny's Diner

## Question and Solution

1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
	customer_id,
    SUM(price) AS total_amt
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
GROUP BY customer_id;
```

2. How many days has each customer visited the restaurant?

```sql
SELECT
	customer_id,
    COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
```
 
3. What was the first item from the menu purchased by each customer?

```sql
CREATE VIEW view_firstitem AS
SELECT
	customer_id,
    order_date,
    FIRST_VALUE(product_name) OVER(PARTITION BY customer_id ORDER BY order_date ASC) AS first_item
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id;
    
SELECT
	customer_id,
    first_item
FROM view_firstitem
GROUP BY customer_id, first_item;
```

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
    product_name,
    COUNT(product_name) AS times_purchased
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY 2 DESC
LIMIT 1;
```

5. Which item was the most popular for each customer?

```sql
CREATE VIEW view_timespurchased AS
SELECT
	customer_id,
    product_name,
    COUNT(product_name) AS times_purchased,
    RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(customer_id) DESC) AS ranking
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
GROUP BY customer_id, product_name;

SELECT
	customer_id,
    product_name,
    times_purchased
FROM view_timespurchased
WHERE ranking = 1;
```

6. Which item was purchased first by the customer after they became a member?

```sql
CREATE VIEW view_memberfirstitem AS
SELECT
	sales.customer_id,
    order_date,
    product_name,
    join_date,
    ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) AS row_numbr
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
LEFT JOIN members
	ON members.customer_id = sales.customer_id
WHERE order_date > join_date
ORDER BY customer_id, order_date;

SELECT
	customer_id,
	product_name
FROM view_memberfirstitem
WHERE row_numbr = 1;
```

7. Which item was purchased just before the customer became a member?

```sql
CREATE VIEW view_before_member_last_item AS
SELECT
	sales.customer_id,
    order_date,
    product_name,
    join_date,
    ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS row_numbr
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
LEFT JOIN members
	ON members.customer_id = sales.customer_id
WHERE order_date < join_date
ORDER BY customer_id, order_date DESC;

SELECT
	customer_id,
    product_name
FROM view_before_member_last_item
WHERE row_numbr = 1;
```
 
8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
	sales.customer_id,
    COUNT(sales.product_id) AS total_items,
    SUM(price) AS total_spent
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
JOIN members
	ON members.customer_id = sales.customer_id
WHERE join_date > order_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT
	customer_id,
    SUM(CASE
		WHEN product_name = 'sushi' THEN price * 10 * 2
        ELSE price * 10
	END) AS total_points
FROM menu
JOIN sales
	ON sales.product_id = menu.product_id
GROUP BY customer_id;
```

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT
	sales.customer_id,
    SUM(CASE
		WHEN order_date BETWEEN join_date AND join_date + INTERVAL 6 DAY THEN price * 10 * 2
        WHEN product_name = 'sushi' THEN price * 10 * 2
        ELSE price * 10
	END) AS total_points
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
JOIN members
	ON members.customer_id = sales.customer_id
WHERE order_date >= join_date AND order_date <= '2021-01-31'
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

## Bonus Questions

Join All The Things

Recreate table with customer_id, order_date, product_name, price, member (Y or N)

```sql
SELECT
	sales.customer_id,
    order_date,
	product_name,
    price,
    CASE
		WHEN join_date <= order_date THEN 'Y'
        ELSE 'N'
	END AS member
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
LEFT JOIN members
	ON members.customer_id = sales.customer_id
ORDER BY customer_id, order_date, price DESC;
```

Rank All The Things

Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
CREATE VIEW view_customerdata AS
SELECT
	sales.customer_id,
    order_date,
	product_name,
    price,
    CASE
		WHEN join_date <= order_date THEN 'Y'
        ELSE 'N'
	END AS member
FROM sales
JOIN menu
	ON menu.product_id = sales.product_id
LEFT JOIN members
	ON members.customer_id = sales.customer_id
ORDER BY customer_id, order_date, price DESC;

SELECT
	*,
    CASE
		WHEN member = 'Y' THEN RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
        ELSE 'null'
	END AS ranking
FROM view_customerdata;
```
