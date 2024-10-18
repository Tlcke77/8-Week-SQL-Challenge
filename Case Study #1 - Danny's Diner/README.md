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

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |


- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

**

