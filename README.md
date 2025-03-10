# Online_Food_Delivery_App_Data_Analysis_Using_MySQL
This repository contains an extensive analysis of an online food delivery app dataset.

### 1. What is the total amount each customer spent on the app?
```
SELECT
          user_id,
          SUM(price) AS Total_amt_spent
FROM
          Sales S INNER JOIN Product P
ON
          S.product_id = P.product_id
GROUP BY
          user_id;
```
### 2. How many days has each customer visited the app?
```
SELECT
         user_id,
         COUNT(DISTINCT created_date) AS no_of_days
FROM
         sales
GROUP BY
         user_id;
```
### 3. What was the first product purchased by each customer?
```
WITH temp AS (
SELECT
       user_id,
       product_name,
       RANK() OVER (PARTITION BY user_id ORDER BY created_date) AS ranks
FROM
       Sales S INNER JOIN Product P
ON
       S.product_id = P.product_id
)
SELECT
       user_id,
       product_name
FROM
       temp
WHERE
       ranks = 1;
```

### 4. What is the most purchased item on the menu and how many times it was purchased by all customers?
```
SELECT
       user_id,
       COUNT(*) AS purchase_count
FROM
       sales
WHERE
       product_id =
       (SELECT
                  product_id
        FROM
                  sales
        GROUP BY
                  product_id
        ORDER BY
                  COUNT(product_id) DESC
        LIMIT
                  1)
GROUP BY
         user_id;

```
### 5. Which item was the most popular for each customer?
```
WITH temp AS (
SELECT
         user_id,
         product_id,
         COUNT(product_id) AS product_count,
         RANK() OVER (PARTITION BY user_id ORDER BY COUNT(product_id) DESC) AS ranks
FROM
         sales
GROUP BY
         user_id,
         product_id
)

SELECT
       user_id,
       product_id
FROM
       temp
WHERE
       ranks = 1;
```
### 6. Which item was purchased first by the customer after they became a gold member?
```
WITH temp AS (
SELECT
       S.user_id,
       product_id,
       RANK() OVER (PARTITION BY S.user_id ORDER BY created_date) AS Ranks
FROM
       sales S JOIN Gold_subscription G
ON
       S.user_id = G.user_id
WHERE
       created_date >= gold_signup_date
)
SELECT
       user_id,
       product_id
FROM
       temp
WHERE
       Ranks = 1;
```
### 7. Which item was purchased just before the customer became a gold member?
```
WITH temp AS (
SELECT
       S.user_id,
       product_id,
       RANK() OVER (PARTITION BY S.user_id ORDER BY created_date DESC) AS Ranks
FROM
       sales S JOIN Gold_subscription G
ON
       S.user_id = G.user_id
WHERE
       created_date < gold_signup_date
)
SELECT
       user_id,
       product_id
FROM
       temp
WHERE
       Ranks = 1;
```
### 8. What is the total orders and amount spent for each customer before they became a gold member?
```
SELECT
         S.user_id,
         COUNT(S.product_id) AS order_count,
         SUM(price) AS Total_amt
FROM
         Sales S JOIN Product P JOIN Gold_subscription G
ON
         S.product_id = P.product_id AND S.user_id = G.user_id
WHERE
         created_date < gold_signup_date
GROUP BY
         S.user_id
ORDER BY
         S.user_id;
```
### 9. If buying each product generates points, for e.g Rs.5 = 2 points and each product has different purchasing points, for e.g for Biriyani, Rs.5 = 1 Point, for Cheesecake, Rs.10 = 5 Points and for Kebab, Rs.5 = 1 Point. 
### Now calculate points,collected by each customer and for which product, most points have been given till now?
```
WITH temp AS (
SELECT
         S.user_id,
         S.product_id,
         SUM(price) AS Total_price,
         CASE
             WHEN S.product_id = 1 THEN 5
             WHEN S.product_id = 2 THEN 2
             WHEN S.product_id = 3 THEN 5
             ELSE 0
         END AS Unit_amount
FROM
         Sales S JOIN Product P
ON
         S.product_id = P.product_id
GROUP BY
         S.user_id,
         S.product_id
ORDER BY
         S.user_id,
         S.product_id
)
SELECT
         user_id,
         SUM(ROUND(Total_price/Unit_amount)) AS Total_Points
FROM
         temp
GROUP BY
         user_id;
```
```
WITH temp AS (
SELECT
         S.user_id,
         S.product_id,
         SUM(price) AS Total_price,
         CASE
             WHEN S.product_id = 1 THEN 5
             WHEN S.product_id = 2 THEN 2
             WHEN S.product_id = 3 THEN 5
             ELSE 0
         END AS Unit_amount
FROM
         Sales S JOIN Product P
ON
         S.product_id = P.product_id
GROUP BY
         S.user_id,
         S.product_id
ORDER BY
         S.user_id,
         S.product_id
)
SELECT
          product_id,
          SUM(ROUND(Total_price/Unit_amount)) AS Total_Points
FROM
          temp
GROUP BY
          product_id
ORDER BY
          Total_Points DESC
LIMIT
          1;
```
### 10. In the first one year, after a customer joins the gold program (including their join date) irrespective of what the customer has purchased they earn 5 points for every Rs.10 spent.
### Who earned more 1 or 3 and what was their point earnings in their first year?
### 11. Rank all transactions of the customers.
### 12. Rank all transactions for each member whenever they are a gold member for every non gold member transaction, mark as 'NA'.
