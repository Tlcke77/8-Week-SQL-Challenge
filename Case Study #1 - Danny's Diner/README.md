# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Questions and Solutions](#questions-and-solutions)

All the information regarding this case study can be from the source using the following link: [here](https://8weeksqlchallenge.com/case-study-1/). 

I chose to use MySQL to solve this case study instead of the provided embedded DB Fiddle, which defaults to PostgreSQL 13.
***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

***

## Entity Relationship Diagram

![image](https://raw.githubusercontent.com/Tlcke77/8-Week-SQL-Challenge/refs/heads/main/ERD%20pic.PNG)
***


## Questions and Solutions

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT s.customer_id, SUM(m.price) AS total_spend
FROM sales AS s
JOIN menu AS m
	ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY s.customer_id ASC
;
````
#### Steps:
- Use **SUM** to calcualte the total sales by each customer.
- Use **JOIN** to merge the sales table with the menu table using the product_id from both tables.

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |


- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT customer_id, COUNT(DISTINCT order_date) AS visit_count
FROM sales
GROUP BY customer_id
;
````
#### Steps:
- Use **COUNT** to count the unique number of visits for each customers.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH total_sales AS 
(
	SELECT  
		customer_id, 
		order_date, 
		product_id, 
		DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS ranks
	FROM sales 
)
SELECT DISTINCT customer_id, product_name
FROM total_sales AS ts
JOIN menu AS m
	ON ts.product_id = m.product_id
WHERE ranks = 1;
````

#### Steps:
- Create a Common Table Expression (CTE) named 'total_sales'. Within the CTE use DENSE_RANK() to rank each customer’s orders by order_date.
- Join with the menu table to get the product name.
- Filter for the first-ranked (rank = 1) order to find the earliest purchase.
- **Example**: If Customer B made their first purchase on '2021–01–10', the query will return the product ordered on that date, such as ramen.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | sushi         | 
| A           | curry      | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi in the same order as their first order.
- Customer B's first order was curry.
- Customer C's first order was ramen.

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
WITH cnt AS 
(
    SELECT product_id, COUNT(product_id) as total_orders
    FROM sales
    GROUP BY product_id
)
SELECT product_name, total_orders
FROM cnt
JOIN menu AS m
	ON cnt.product_id = m.product_id
GROUP BY product_name, total_orders
ORDER BY total_orders DESC
LIMIT 1
;
````

#### Steps:
- Count how many times each product_id was ordered using **COUNT(product_id)**.
- **Join** with the menu table to display the product name.
- Sort by the total orders in descending order to find the most frequently ordered item.
- **Example**: If sushi was ordered 25 times, and ramen 20 times, sushi will be identified as the most purchased item.

#### Answer:
| product_name | total_orders | 
| ----------- | ----------- |
| ramen       | 8 |

- Ramen was the most purchesed item with 8 orders.
  
***

**5. Which item was the most popular for each customer?**

````sql
WITH cte AS
(
	SELECT customer_id, product_name, COUNT(s.product_id) AS total_orders, DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY COUNT(s.product_id) DESC) AS ranks
	FROM sales AS s
	JOIN menu AS m
		ON s.product_id = m.product_id
	GROUP BY customer_id, product_name
)
SELECT customer_id, product_name
FROM cte
WHERE ranks = 1
;
````

:grey_exclamation: *A customer can have more than one favortie item (ex: customer B)* :grey_exclamation:

#### Steps:
- Create a **CTE** to organize the query results for calculating customer order counts by product.
- Apply DENSE_RANK() to rank the products for each customer based on order frequency.
	- **PARTITION BY customer_id** ensures the ranking is done per customer.
   	- **ORDER BY COUNT(s.product_id) DESC** ranks items from most to least ordered.
- Group by customer_id and product_name to count how many times each customer ordered a specific product.
- Select the first-ranked order to get the item purchased right after becoming a member.
- **Example**: If Customer B became a member on '2021–01–15' and ordered ramen on '2021–01–16', the query will return ramen as their first post-membership purchase.


#### Answer:
| customer_id | product_name | total_orders |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | curry        |  2   |
| B           | sushi        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favorite item on the menu is ramen.
- Customer B's favorite item is all of the items on the menu

***

**6. Which item was purchased first by the customer after they became a member?**

```sql

WITH mm_sales AS (
    SELECT 
        s.customer_id, 
        m.product_name, 
        s.order_date, 
        mm.join_date
    FROM sales AS s
    JOIN menu AS m ON s.product_id = m.product_id
    JOIN members AS mm ON s.customer_id = mm.customer_id
    WHERE s.order_date > mm.join_date 
),
ranked_sales AS (
    SELECT  
        ms.customer_id, 
        ms.product_name, 
        ms.order_date, 
        ms.join_date, 
        DENSE_RANK() OVER (PARTITION BY ms.customer_id ORDER BY ms.order_date) AS ranks
    FROM mm_sales AS ms
)
SELECT 
    rs.customer_id, 
    rs.product_name
FROM ranked_sales AS rs
WHERE rs.ranks = 1;
```

#### Steps:
- Create a **CTE** named 'mm_sales': Within the CTE, select customer_id, product_name, and order_date from the sales table. Also, include join_date from the members table.
- **Join** the sales table and .menu table: Use the product_id column to get the corresponding product names for each order.
- **Join** the members table: Connect it to the sales table on the customer_id column to access membership details.
- Apply a filter condition: In the CTE, include a condition to only select sales that occurred after the member's join_date (sales.order_date > members.join_date).
- Rank the sales records: Use the **DENSE_RANK()** to assign ranks to each order based on order_date. Use the **PARTITION BY** clause to divide the data by customer_id and the ORDER BY clause to order the rows within each partition by order_date.
- Filter for the first purchase: In the **WHERE** clause, specify to retrieve only the rows where the rank column equals 1, representing the first product purchased after joining the membership.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order after membership was ramen.
- Customer B's first order after membership was sushi.

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH mm_sales AS (
    SELECT 
        s.customer_id, 
        m.product_name, 
        s.order_date, 
        mm.join_date
    FROM sales AS s
    JOIN menu AS m ON s.product_id = m.product_id
    JOIN members AS mm ON s.customer_id = mm.customer_id
    WHERE s.order_date < mm.join_date 
),
ranked_sales AS (
    SELECT  
        ms.customer_id, 
        ms.product_name, 
        ms.order_date, 
        ms.join_date, 
        DENSE_RANK() OVER (PARTITION BY ms.customer_id ORDER BY ms.order_date DESC) AS ranks
    FROM mm_sales AS ms
)
SELECT 
    rs.customer_id, 
    rs.product_name
FROM ranked_sales AS rs
WHERE rs.ranks = 1;
````

#### Steps:
- Create a **CTE** named 'mm_sales': Within the CTE, select customer_id, product_name, order_date, and join_date from the sales and members tables.
- **Join** the sales and menu tables: Use the product_id column to access the product names corresponding to each order.
- **Join** the dannys_diner.members table: Connect it to the sales table on the customer_id column to access membership details.
- Apply a filter condition: In the CTE, include a condition to only select sales that occurred before the member's join_date (sales.order_date < members.join_date).
- Rank the sales records: Use the **DENSE_RANK()**  to assign ranks to each order based on order_date. Use the **PARTITION BY** clause to group the data by customer_id and the ORDER BY clause to sort the rows within each partition by order_date in descending order.
- Filter for the last purchase before membership: In the WHERE clause, specify to retrieve only the rows where the rank column equals 1, representing the last product purchased before becoming a member.

:grey_exclamation: *The only changes from the previous query (for question 6) are:           :grey_exclamation:
- Update the filter condition to select sales that occurred before the member's join_date (sales.order_date < members.join_date).
- Change the ORDER BY clause in the DENSE_RANK() function to sort order_date in descending order (ORDER BY ms.order_date DESC), allowing us to find the last purchase made before the membership.* 

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Customer A ordered both sushi and curry.
- Customer B ordered sushi.

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS total_cost
FROM sales AS s
JOIN menu AS m
	ON s.product_id = m.product_id 
JOIN members AS mm
	ON s.customer_id = mm.customer_id
    AND s.order_date < mm.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id 
;
```

#### Steps:
- Use **COUNT** to calcualte the total items for each customer.
- Use **SUM** to find the sume of the total prices of the items.
- **JOIN** the menu tale using the product_id and then joining the members table using the customer_id

#### Answer:
| customer_id | total_items | total_cost |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

- Customer A spent $25 on a total of 2 items.
- Customer B spent $40 on a total of 3 items.

Another way I found the answer was using a CTEs 
```sql
WITH cte1 AS
(
	SELECT s.customer_id, s.product_id, COUNT(s.product_id) AS item_cnt, (price * COUNT(s.product_id)) AS Spend
	FROM sales AS s
	JOIN menu AS m
		ON s.product_id = m.product_id
	JOIN members AS mm 
		ON s.customer_id = mm.customer_id
    WHERE s.order_date < mm.join_date 
	GROUP BY customer_id, s.product_id, price
),
cte2 AS 
(
	SELECT cte1.customer_id, SUM(spend) AS total_cost, SUM(cte1.item_cnt) AS total_items
    FROM cte1
    GROUP BY cte1.customer_id
)
SELECT cte2.customer_id, cte2.total_items, cte2.total_cost
FROM cte2
ORDER BY cte2.customer_id 
;
```
Direct SQL Query vs. Using CTEs:

- The direct SQL query calculates the total items and total cost for each customer in a single step by grouping directly from the sales, menu, and members tables. This approach is straightforward but can become complex and harder to read as more calculations are added.
- The CTE approach breaks down the calculations into two separate parts:
	- The first CTE (cte1) computes item counts and spending for each product per customer before they became a member.
	- The second CTE (cte2) summarizes the results from the first CTE, aggregating total items and costs per customer.
 - Using CTEs enhances readability and modularity, making it easier to manage complex queries by separating logic into distinct steps.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
WITH points_total AS 
(
SELECT product_id,
	CASE
		WHEN product_id = 1 THEN price * 20
        	ELSE price * 10
        	END as points
FROM menu
)
SELECT s.customer_id, SUM(pt.points) AS Points
FROM sales AS s
JOIN points_total AS pt
	ON s.product_id = pt.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id
;
```

#### Steps:
- Create a CTE called 'points_total'. Use a **CASE** to Multiply the price by 20 for sushi **(product_id = 1)** to apply the 2x points multiplier. Multiply the price by 10 for all other products.
- Then call the CTE and use a **JOIN** to merge sales table with the CTE.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Customer A has $860 total points.
- Customer B has $940 total points.
- Customer C has $360 total points.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?**

```sql
WITH dates AS 
(
	SELECT *,
		DATE_ADD(join_date, INTERVAL 6 DAY) AS valid_date,
        	LAST_DAY('2021-01-31') AS last_date
	FROM members
)
SELECT s.customer_id,
	SUM(CASE
			WHEN s.product_id = 1 THEN price * 20
            		WHEN s.order_date BETWEEN join_date AND valid_date THEN price * 2
			ELSE price * 10
			END 
		) AS points
FROM dates AS d
JOIN sales AS s
	ON d.customer_id = s.customer_id
JOIN menu AS m
	ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY s.customer_id 
ORDER BY s.customer_id 
;
```

#### Steps:
- Create a CTE named `dates`:
  - Select all relevant columns and calculate the **valid date** by adding 6 days to the `join_date` using `DATE_ADD(join_date, INTERVAL 6 DAY)`.
  - Determine the **last date** of interest by using `LAST_DAY('2021-01-31')`.
- **Join the sales table**:
  - Connect it with the `dates` CTE on `customer_id` to access sales data for each customer.
- **Join the menu table**: Use `product_id` to get the price for each product sold.
- Use a **CASE** statement to compute points:
    - For sushi (`product_id = 1`), multiply the price by **20**.
    - For sales made between the `join_date` and the **valid date**, multiply the price by **2**.
    - For all other sales, multiply the price by **10**.
- **Group by `customer_id`**:
  - Aggregate the total points earned for each customer based on the defined criteria.
- Retrieve `customer_id` and the calculated total points, ordering the results by `customer_id`.
- The main distinction from previous queries is the **double points** for all items in the first week after a customer joins, not just for sushi, highlighting the promotional aspect of the points system.

  #### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 452 |
| B           | 820 |

- Total points for Customer A is 1,020.
- Total points for Customer B is 320.

***

## BONUS QUESTIONS

*Join All The Things**

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

```sql
SELECT s.customer_id, s.order_date, m.product_name, m.price,
	CASE
		WHEN s.order_date < mm.join_date THEN 'N'
        WHEN s.order_date >= mm.join_date THEN 'Y'
        ELSE 'N'
	END AS member_status
FROM sales AS s
LEFT JOIN members AS mm
	ON s.customer_id = mm.customer_id
JOIN menu AS m
	ON s.product_id = m.product_id
ORDER BY s.customer_id, s.order_date
;
```
#### Answer: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***

**Rank All The Things**

**Danny also requires further information about the **ranking** of customer products, but he purposely does not need the ranking for non-member purchases so he expects null **ranking** values for the records when customers are not yet part of the loyalty program.**

```sql
WITH customer_data AS
(
SELECT s.customer_id, s.order_date, m.product_name, m.price,
	CASE
		WHEN s.order_date < mm.join_date THEN 'N'
        WHEN s.order_date >= mm.join_date THEN 'Y'
        ELSE 'N'
	END AS member_status
FROM sales AS s
LEFT JOIN members AS mm
	ON s.customer_id = mm.customer_id
JOIN menu AS m
	ON s.product_id = m.product_id
ORDER BY s.customer_id, s.order_date
)
SELECT *,
	CASE
		WHEN member_status = 'N' THEN NULL
        ELSE RANK() OVER (PARTITION BY cd.customer_id, cd.member_status ORDER BY order_date)
	END AS ranking
FROM customer_data AS cd 
;
```

#### Answer: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

***
