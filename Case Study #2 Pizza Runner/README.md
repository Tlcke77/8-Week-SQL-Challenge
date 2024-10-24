# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Questions and Solutions](#questions-and-solutions)

All the information regarding this case study can be from the source using the following link: [here](https://8weeksqlchallenge.com/case-study-2/). 

I chose to use MySQL to solve this case study instead of the provided embedded DB Fiddle, which defaults to PostgreSQL 13.

:warning: It's important to note that there is a discrepancy between the data shown in the "customer_orders" (2021) table on the [case study site](https://8weeksqlchallenge.com/case-study-2/) and the year provided in the INSERT command used to create and populate the database for the case study (2020). :warning:
***

## Entity Relationship Diagram

![image]()
***

## 🧼 Data Cleaning

### 1. Updating the `customer_orders` Table

The following SQL command is used to update the `customer_orders` table:

```sql
UPDATE customer_orders
SET exclusions = IFNULL(exclusions, ''),
    extras = IFNULL(extras, '');
```
**Outcome**: This ensures that these fields are never NULL, which can help prevent errors in data processing or reporting.

## 2. Updating the `runner_orders` Table

### a. Updating NULL Values to Empty Strings

```sql
UPDATE runner_orders
SET 
    pickup_time = IFNULL(pickup_time, ' '),
    distance = IFNULL(distance, ' '),
    duration = IFNULL(duration, ' '),
    cancellation = IFNULL(cancellation, ' ');
```
**Outcome**: Once again, this ensures that these fields are never NULL, which can help prevent errors in data processing or reporting.

### b. Cleaning Up the Distance Column

```sql
UPDATE runner_orders
SET distance = TRIM(REPLACE(distance, 'km', ''));
```
**Outcome**: This results in a distance value that is purely numerical, which can facilitate numerical calculations and comparisons.

### c. Cleaning Up the Duration Column

```sql
UPDATE runner_orders
SET duration = TRIM(REPLACE(REPLACE(duration, 'minutes', ''), 'minute', ''));
```
**Outcome**: This ensures that duration values are purely numerical, facilitating better calculations and comparisons.

***

## Solution

## A. Pizza Metrics

### 1. How many pizzas were ordered?

```sql
SELECT COUNT(pizza_id) AS pizza_orders
FROM customer_orders
;
```
#### Steps:
- Use **COUNT** to find all the total pizzas ordered.

#### Answer:
| pizza_orders | 
| ----------- | 
| 14           | 

- 14 pizzas were ordered.

***

**2. How many unique customer orders were made?**

```sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM customer_orders
;
```
#### Steps:
- Use **COUNT** with **DISTINCT** to find the unique orders.

#### Answer:
| unique_orders | 
| ----------- | 
| 10           | 

- There were 10 unique pizza orders.

***

**3. How many successful orders were delivered by each runner?**

```sql
SELECT runner_id, COUNT(order_id) AS successful_orders
FROM runner_orders
WHERE distance != 0
GROUP BY runner_id
;
```
#### Steps:
- Use **COUNT** to find all successful orders.
- Usa a **WHERE** to eliminate orders that were not deilvered.

#### Answer:
| runner_id | successful_orders |
| ----------- | ----------- |
| 1           | 4          |
| 2           | 3          |
| 3           | 1          |

- Runner 1 had 4 successful delivered orders.
- Runner 2 had 3 successful delivered orders.
- Runner 3 had 1 successful delivered orders.

***

**4. How many of each type of pizza was delivered?**

```sql
SELECT pizza_name, COUNT(c.pizza_id) AS delivered
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
JOIN pizza_names AS p
	ON c.pizza_id = p.pizza_id
WHERE distance != 0
GROUP BY pizza_name
;
```
#### Steps:
- Use **COUNT** to find all delivered orders.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- **JOIN** the `customer_orders` and the `pizza_names` tables using the pizza_id.
- Usa a **WHERE** to eliminate orders that were not deilvered.

#### Answer:
| pizza_name | delivered |
| ----------- | ----------- |
| Meatlovers  | 9          |
| Vegetarian  | 3          |

- There were 9 meatlovers pizzas delivered.
- There were 3 vegetarian pizzas delivered.

***

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT customer_id, pizza_name, COUNT(c.pizza_id) AS ordered
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
JOIN pizza_names AS p
	ON c.pizza_id = p.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id
;
```
#### Steps:
- Use **COUNT** to find all pizza ordered.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- **JOIN** the `customer_orders` and the `pizza_names` tables using the pizza_id.

#### Answer:
| customer_id | pizza_name | ordered |
| ----------- | ---------- |------------  |
| 101           | Meatlovers        |  2   |
| 101           | Vegetarian        |  1   |
| 102          | Meatlovers        |  2   |
| 102           | Vegetarian        |  1   |
| 103           | Meatlovers        |  3   |
| 103          | Vegetarian        |  1   |
| 104           | Meatlovers        |  3   |
| 105           | Vegetarian        |  1   |

- Customer 101 ordered meatlovers 2 times.
- Customer 101 ordered vegetarian 1 times.
- Customer 102 ordered meatlovers 2 times.
- Customer 102 ordered vegetarian 1 times.
- Customer 103 ordered meatlovers 3 times.
- Customer 103 ordered vegetarian 1 times.
- Customer 104 ordered meatlovers 3 times.
- Customer 105 ordered vegetarian 1 times.
 
###6. What was the maximum number of pizzas delivered in a single order?

```sql
WITH pizza_cte AS 
(
SELECT c.order_id, COUNT(c.pizza_id) AS total_pizza_ordered
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
WHERE distance != 0
GROUP BY c.order_id
)
SELECT MAX(total_pizza_ordered) AS pizza_delivered
FROM pizza_cte
;
```
#### Steps:
- Create a Common Table Expression (CTE) named `pizza_cte`. Within the cte use a **COUNT** to find the total of pizza ordered.
  - The `pizza_cte` finds the total pizzas in each order.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- Usa a **WHERE** to eliminate orders that were not deilvered.
- Use **MAX** to find the max number of pizzas.

#### Answer:
| pizza_delivered | 
| ----------- | 
| 3           | 

- The max number of  pizzas delivered in a single order is 3.

***

###7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT c.customer_id,
	SUM(CASE WHEN exclusions != '' OR extras != '' THEN 1 ELSE 0 END) AS modified,
	SUM(CASE WHEN exclusions = '' AND extras = '' THEN 1 ELSE 0 END) AS unmodified
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
WHERE distance != 0
GROUP BY c.customer_id
;
```
#### Steps:
- Use **SUM** with a **CASE** statement to count:
  - **Modified Orders**: When either `exclusions` or `extras` is not empty.
  - **Unmodified Orders**: When both `exclusions` and `extras` are empty.
- **JOIN** the `customer_orders` table with the `runner_orders` table using `order_id` to combine relevant data.
- Use a **WHERE** clause to filter out orders with a `distance` equal to 0.

#### Answer:
| customer_id | modified | unmodified |
| ----------- | ---------- |------------  |
| 101           | 0        |  2   |
| 102           | 0        |  3   |
| 103          | 3        |  0   |
| 104           | 2        |  1   |
| 105           | 1        |  0   |

- Customer 101 modified their pizza 0 times and unmodified it 2 times.
- Customer 102 modified their pizza 0 times and unmodified it 3 times.
- Customer 103 modified their pizza 3 times and unmodified it 0 times.
- Customer 104 modified their pizza 2 times and unmodified it 1 times.
- Customer 105 modified their pizza 1 times and unmodified it 0 times.

***

###8. How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT
	SUM(CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1 ELSE 0 END) AS modified
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
WHERE distance != 0
	AND exclusions != '' 
	AND extras != '';
```
#### Steps:
- Use **SUM** with a **CASE** statement to count:
    - **Condition**: Check if both `exclusions` and `extras` are **NOT NULL**.
    - **True**: If both conditions are met, it returns `1`.
    - **False**: If either condition is not met, it returns `0`.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- Usa a **WHERE** to:
  - filter out orders with a `distance` equal to 0.
  - filter out exclusions` field is  an empty string.
  - filter out extras field is  an empty string.

#### Answer:
| modified | 
| ----------- | 
| 1           | 

- 1 pizza was delivered that had both exclusions and extras.

***

**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT HOUR(order_time) AS hour_ordered, COUNT(order_id) AS total_ordered
FROM customer_orders
GROUP BY hour_ordered
ORDER BY hour_ordered ASC
;
```
#### Steps:
- Use **HOUR** to find the hour the pizza was ordered.
- Use **COUNT** to find all delivered orders.

#### Answer:
|  hour_ordered | total_ordered |
| ----------- | ----------- |
| 11  | 1          |
| 13  | 3          |
| 18  | 3          |
| 19  | 1          |
| 21  | 3          |
| 23  | 3          |

- At hour 11 there was 1 pizza ordered.
- At hour 13 there was 3 pizzas ordered.
- At hour 18 there was 3 pizzas ordered.
- At hour 19 there was 1 pizza ordered.
- At hour 21 there was 3 pizzas ordered.
- At hour 23 there was 3 pizzas ordered.

***

###10. What was the volume of orders for each day of the week?

```sql
SELECT DATE_FORMAT(order_time, '%W') AS weekday, COUNT(order_id) AS total_ordered
FROM customer_orders
GROUP BY weekday_orderded
;
```
#### Steps:
- Use **DATE_FORMAT** to:
  - This function formats the `order_time` column to return the day of the week (e.g., "Monday", "Tuesday").
  - The result is labeled as `weekday`.
- Use **COUNT** to find the total orders.

#### Answer:
|  weekday | total_ordered |
| ----------- | ----------- |
| Wednesday  | 5          |
| Thursday  | 3          |
| Saturday  | 5         |
| Friday  | 1          |

- On wednesday 5 pizzas were ordered.
- On Thursday 2 pizzas were ordered.
- On Saturday 5 pizzas were ordered.
- On Friday 1 pizzas were ordered.

***

## B. Runner and Customer Experience

###1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```sql
SELECT WEEK(registration_date, 1) AS registration_week, COUNT(runner_id) AS runner_signup
FROM runners
GROUP BY WEEK(registration_date, 1)
;
```
#### Steps:
- Use **WEEK** to group the registration dates by week, starting on Monday
  - The 1 indicates that the week starts on monday.
- Use **COUNT** to find number of runners who signed up in each week.

#### Answer:
|  registration_week | runner_signup |
| ----------- | ----------- |
| 0  | 2          |
| 1  | 1          |
| 2  | 1         |

- On week 0 there were 2 runners that signed up.
- On week 1 there were 1 runner that signed up.
- On week 2 there were 1 runner that signed up.

***

###2. at was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT runner_id, AVG(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)) AS avg_pickup_time
FROM customer_orders AS c
JOIN runner_orders AS r
    ON c.order_id = r.order_id
WHERE pickup_time IS NOT NULL  
GROUP BY runner_id
;
```
#### Steps:
- Use **AVG** to find the average time in minutes it took for each runner to arrive at the Pizza Runner HQ.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- USE **WHERE** to filter out pickup_times that are NULL.

#### Answer:
|  runner_id | avg_pickup_time |
| ----------- | ----------- |
| 1  | 15.3333          |
| 2  | 23.4000          |
| 3  | 10.0000         |

- Runner 1 had an average time of 15.3333.
- Runner 2 had an average time of 23.4000.
- Runner 3 had an average time of 10.0000.

***

###3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH cte AS
(
SELECT c.order_id, COUNT(c.pizza_id) AS total_pizza_ordered, AVG(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)) AS avg_pickup_time
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
WHERE distance != 0
GROUP BY c.order_id
)
SELECT cte.total_pizza_ordered, AVG(cte.avg_pickup_time) AS avg_make_time
FROM cte
GROUP BY cte.total_pizza_ordered
;
```
#### Steps:
- reate a Common Table Expression (CTE) to count the total pizzas ordered and calculate the average pickup time for each order.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.

#### Answer:
|  total_pizza_ordered | avg_make_time |
| ----------- | ----------- |
| 1  | 12.000         |
| 2  | 18.000         |
| 3  | 29.000         |

- When 1 pizza is ordered the average make time is 12 minutes.
- When 2 pizzas are ordered the average make time is 18 minutes.
- When 3 pizzas are ordered the average make time is 29 minutes.

***

###4. What was the average distance travelled for each customer?
```sql
SELECT customer_id, AVG(distance) AS avg_distance
FROM customer_orders AS c
JOIN runner_orders AS r
	ON c.order_id = r.order_id
WHERE distance != 0
GROUP BY customer_id
;
```
#### Steps:
- Use **AVG** to calculate the average distance traveled for each customer.
- Use **WHERE** Filter out orders with a distance of 0.

#### Answer:
|  customer_id | avg_distance |
| ----------- | ----------- |
| 101  | 20        |
| 102  | 16.733333333333334         |
| 103  | 23.399999999999995         |
| 104  | 10         |
| 105  | 15         |

- Customer 101's average distance was 20 km.
- Customer 102's average distance was 16.733333333333334 km.
- Customer 103's average distance was 23.399999999999995 km.
- Customer 104's average distance was 10 km.
- Customer 105's average distance was 15 km.

***

###5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT MAX(duration) - MIN(duration) AS difference
FROM runner_orders
WHERE distance != 0
;
```
#### Steps:
- Use **MAX** to find the longest delivery time and **MIN** for the shortest.
  - Calculate the difference between these two values.
- Use **WHERE** Filter out orders with a distance of 0.

#### Answer:
|  difference | 
| ----------- |
| 30  | 

- The difference was 30 km.

***

###6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT r.runner_id, c.order_id, r.distance,
  ROUND((r.distance/r.duration * 60), 2) AS speed
FROM runner_orders AS r
JOIN customer_orders AS c
  ON r.order_id = c.order_id
WHERE distance != 0
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY r.runner_id
;
```
#### Steps:
- Calculate the speed using the formula (distance/duration) multiplied by 60 to convert to minutes.
- **JOIN** the `customer_orders` and the `runner_orders` tables using the order_id.
- Use **WHERE** Filter out orders with a distance of 0.

#### Answer:
| runner_id | order_id | distance | speed | 
| ----------- | ---------- | -------------| ----- | 
| 1           | 1 | 20        | 37.5    | 
| 1           | 2 | 20        | 44.44    | 
| 1           | 3 | 13.4        | 40.2    | 
| 1           | 10 | 10        | 60    | 
| 2           | 8 | 23.4        | 93.6    | 
| 2           | 4 | 23.4        | 35.1    | 
| 2           | 7 | 25        | 60    | 
| 3           | 5 | 10        | 40    | 

- On order 1 runner 1 had an avergage speed of 37.5.
- On order 2 runner 1 had an avergage speed of 44.44.
- On order 3 runner 1 had an avergage speed of 40.2.
- On order 10 runner 1 had an avergage speed of 60.
- On order 8 runner 2 had an avergage speed of 93.6.
- On order 4 runner 2 had an avergage speed of 35.1.
- On order 7 runner 2 had an avergage speed of 60.
- On order 5 runner 3 had an avergage speed of 40.
  
#### Observation:
- Speed Variation:
  - Runner 1 has different speeds for each order. For example, the speed ranges from 37.5 to 60, indicating inconsistent performance.
  - Runner 2 also shows significant speed differences, with a peak speed of 93.6 for order 8 and a low of 35.1 for order 4.
- Distance and Speed Relationship:
  - Shorter distances don’t always result in higher speeds. For instance, Runner 1 had a high speed of 60 over a distance of 10, but a lower speed of 44.44 over a longer distance of 20.
    
***

###7. What is the successful delivery percentage for each runner?
```sql
SELECT runner_id, 
	ROUND(100 * SUM(CASE WHEN distance = 0 THEN 0 ELSE 1 END) / COUNT(*), 0) AS percentage
FROM runner_orders
GROUP BY runner_id
;
```
#### Steps:
- Use a **CASE** to count successful deliveries (where distance is not 0).
  - Calculate the percentage by dividing the successful deliveries by the total number of deliveries for each runner.

#### Answer:
|  runner_id | percentage |
| ----------- | ----------- |
| 1  | 100        |
| 2  | 75         |
| 3  | 50         |

- Runner 1 had a 100 percentage successful delivery rate.
- Runner 2 had a 75 percentage successful delivery rate.
- Runner 3 had a 50 percentage successful delivery rate.

:grey_exclamation: It’s unfair to credit successful deliveries to runners since order cancellations are beyond their control. :grey_exclamation:
  
