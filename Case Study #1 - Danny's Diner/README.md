# Case Study #1 - Danny's Diner

Link to this case study can be found here: https://8weeksqlchallenge.com/case-study-1/

## Database

I decided to work in MySQL instead of the embedded DB Fiddle so I created a database and loaded the tables as below.

```sql
CREATE DATABASE 8WeekSQLChallenge_case_study_1;

USE 8WeekSQLChallenge_case_study_1;

CREATE TABLE sales (
  customer_id VARCHAR(1),
  order_date DATE,
  product_id INT
);

INSERT INTO sales
  (customer_id, order_date, product_id)
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');

CREATE TABLE menu (
  product_id INT,
  product_name VARCHAR(5),
  price int
);

INSERT INTO menu
  (product_id, product_name, price)
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
    
CREATE TABLE members (
  customer_id VARCHAR(1),
  join_date date
);

INSERT INTO members
  (customer_id, join_date)
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
```

## Case Study Questions and My Solutions

#### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
  customer_id,
  SUM(price) AS total_amt
FROM sales
JOIN menu
  ON menu.product_id = sales.product_id
GROUP BY customer_id;
```

#### Result:

| customer_id | total_amt |
| --- | ---|
| A | 76 |
| B | 74 |
| C | 36 |

![{54F249D3-0ABF-4572-872D-887D16E7EF0B}](https://github.com/user-attachments/assets/d9ae1d91-a91f-4f7c-affe-d2d00def1465)

***

#### 2. How many days has each customer visited the restaurant?

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
```

#### Result:

| customer_id | days_visited |
| ---| ---|
| A | 4 |
| B | 6 |
| C | 2 |

![{0700F1F7-8DDC-499E-B5FF-402343586F8F}](https://github.com/user-attachments/assets/4bb776b7-257d-4cc2-b2cf-36a4be27f825)

***

#### 3. What was the first item from the menu purchased by each customer?

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

#### Result:

| customer_id | first_item |
| ---| ---|
| A | sushi |
| B | curry |
| C | ramen |

![{F72A750D-9916-4475-88E7-D13F43FA286F}](https://github.com/user-attachments/assets/82de26ad-f426-49c5-89ba-6414e46ce979)

***

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

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

#### Result:

| product_name | times_purchased |
| ---| ---|
| ramen | 8 |

![{16278C1C-994B-4C67-A797-021BDD4BEDBB}](https://github.com/user-attachments/assets/13d0a91b-0708-44af-9391-6bbe4cfe35a4)

***

#### 5. Which item was the most popular for each customer?

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

#### Result:

| customer_id | product_name | times_purchased |
| ---| ---| --- |
| A | ramen | 3 |
| B | curry | 2 |
| B | sushi | 2 |
| B | ramen | 2 |
| C | ramen | 3 |

![{D635A8F5-C855-42B1-87EF-F42D4A4033C2}](https://github.com/user-attachments/assets/c3fad67d-5920-444b-937d-9aff3352f0ad)

***

#### 6. Which item was purchased first by the customer after they became a member?

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

#### Result:

| customer_id | product_name |
| ---| ---|
| A | ramen |
| B | sushi |

![{327877DD-3189-4E5D-BA07-905AD3671B6F}](https://github.com/user-attachments/assets/74276118-b5a4-4b90-9746-ed80f1f44801)

***

#### 7. Which item was purchased just before the customer became a member?

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

#### Result:

| customer_id | product_name |
| ---| ---|
| A | sushi |
| B | sushi |

![{7B7533D3-5207-434F-A76F-2D53DB7D2767}](https://github.com/user-attachments/assets/f9432a7e-cbe4-492c-adc2-a9fa035f3b71)

***

#### 8. What is the total items and amount spent for each member before they became a member?

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

#### Result:

| customer_id | total_items | total_spent |
| --- | --- | --- |
| A | 2 | 25 |
| B | 3 | 40 |

![{49BEC91A-0E24-4BD6-98BF-98AAB67A168D}](https://github.com/user-attachments/assets/d45e5938-033e-46dc-ad6f-c999c8503164)

***

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

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

#### Result:

| customer_id | total_points |
| --- | --- |
| A | 860 |
| B | 940 |
| C | 360 |

![{FD71A471-085E-4F9B-8134-3F064B8F60BB}](https://github.com/user-attachments/assets/259f6d2a-1cf4-45c5-9135-93793e51eaf8)

***

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

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

#### Result:

| customer_id | total_points |
| --- | --- |
| A | 1020 |
| B | 320 |

![{59F1162D-3E47-46C9-A15E-3B6DB1887EE4}](https://github.com/user-attachments/assets/6b33d6d2-daed-468d-b60c-5f42c3371e55)

***

## Bonus Questions

#### Join All The Things

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

#### Result:

| customer_id | order_date | product_name | price | member |
| --- | --- | --- | --- | --- |
| A	| 2021-01-01	| curry	| 15	| N |
| A	| 2021-01-01	| sushi	| 10	| N |
| A	| 2021-01-07	| curry	| 15	| Y |
| A	| 2021-01-10	| ramen	| 12	| Y |
| A	| 2021-01-11	| ramen	| 12	| Y |
| A	| 2021-01-11	| ramen	| 12	| Y |
| B	| 2021-01-01	| curry	| 15	| N |
| B	| 2021-01-02	| curry	| 15	| N |
| B	| 2021-01-04	| sushi	| 10	| N |
| B	| 2021-01-11	| sushi	| 10	| Y |
| B	| 2021-01-16	| ramen	| 12	| Y |
| B	| 2021-02-01	| ramen	| 12	| Y |
| C	| 2021-01-01	| ramen	| 12	| N |
| C	| 2021-01-01	| ramen	| 12	| N |
| C	| 2021-01-07	| ramen	| 12	| N |

***

#### Rank All The Things

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

#### Result:

| customer_id | order_date | product_name | price | member | ranking |
| --- | --- | --- | --- | --- | --- |
| A	| 2021-01-01	| curry	| 15	| N	| null |
| A	| 2021-01-01	| sushi	| 10	| N	| null |
| A	| 2021-01-07	| curry	| 15	| Y	| 1 |
| A	| 2021-01-10	| ramen	| 12	| Y	| 2 |
| A	| 2021-01-11	| ramen	| 12	| Y	| 3 |
| A	| 2021-01-11	| ramen	| 12	| Y	| 3 |
| B	| 2021-01-01	| curry	| 15	| N	| null |
| B	| 2021-01-02	| curry	| 15	| N	| null |
| B	| 2021-01-04	| sushi	| 10	| N	| null |
| B	| 2021-01-11	| sushi	| 10	| Y	| 1 |
| B	| 2021-01-16	| ramen	| 12	| Y	| 2 |
| B	| 2021-02-01	| ramen	| 12	| Y	| 3 |
| C	| 2021-01-01	| ramen	| 12	| N	| null |
| C	| 2021-01-01	| ramen	| 12	| N	| null |
| C	| 2021-01-07	| ramen	| 12	| N	| null |

***
