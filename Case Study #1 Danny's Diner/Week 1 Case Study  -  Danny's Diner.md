![](../src/Pasted%20image%2020250110153610.png)


---
## Table of Contents
[Introduction](#Introduction) 
[Problem Statement](#Problem%20Statement)
[Case Study Questions and Solutions](#Case%20Study%20Questions%20and%20Solutions) 
[Bonus Questions](#Bonus%20Questions) 

Source: https://8weeksqlchallenge.com/case-study-1/

---

## Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

---

## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

Danny has shared with you 3 key datasets for this case study:

- `sales`
- `menu`
- `members`

You can inspect the entity relationship diagram and example data below.

## Entity Relationship Diagram

![](../src/Pasted%20image%2020250110153734.png)



---

# Case Study Questions and Solutions

1.  **What is the total amount each customer spent at the restaurant?**

```sql
SELECT customer_id
	,sum(price) total
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY customer_id
ORDER BY total DESC
```

![](../src/Pasted%20image%2020250110154936.png)



2. **What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT s.customer_id
	,count(DISTINCT s.order_date) visit_count
FROM sales s
GROUP BY s.customer_id
```

![](../src/Pasted%20image%2020250110160038.png)





3. **What was the first item from the menu purchased by each customer?**

```sql
WITH ranked_sales
AS (
	SELECT s.customer_id
		,s.order_date
		,m.product_name
		,dense_rank() OVER (
			PARTITION BY s.customer_id ORDER BY s.order_date
			) rank
	FROM sales s
	JOIN menu m using (product_id)
	)
SELECT customer_id
	,product_name
FROM ranked_sales
WHERE rank = 1
GROUP BY customer_id
	,product_name
```

![](../src/Pasted%20image%2020250110161133.png)

4.  **What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT m.product_name
	,count(*) num_purchases
FROM sales s
JOIN menu m using (product_id)
GROUP BY m.product_name
ORDER BY num_purchases DESC limit 1


SELECT customer_id
	,COUNT(*) AS total_purchases
FROM sales s
JOIN menu m ON s.product_id = m.product_id
WHERE product_name = 'ramen'
GROUP BY customer_id
ORDER BY total_purchases DESC


```

![](../src/Pasted%20image%2020250110161909.png)

![](../src/Pasted%20image%2020250110161940.png)

6.  **Which item was the most popular for each customer?** 

```sql
-- show join date

with purchased_first as (
select s.customer_id,order_date,join_date,product_name,
	ROW_NUMBER() over (partition by s.customer_id order by order_date) rank
from sales s
join menu m
on s.product_id = m.product_id
join members ms
on ms.customer_id = s.customer_id
where order_date >= join_date)
select  *
from purchased_first
where rank = 1

```


![](../src/Pasted%20image%2020250110162238.png)

![](../src/Pasted%20image%2020250110162243.png)

 7. **Which item was purchased just before the customer became a member?**

```sql
WITH purchased_first
AS (
	SELECT s.customer_id
		,order_date
		,join_date
		,product_name
		,dense_rank() OVER (
			PARTITION BY s.customer_id ORDER BY order_date DESC
			) rank
	FROM sales s
	JOIN menu m ON s.product_id = m.product_id
	JOIN members ms ON ms.customer_id = s.customer_id
	WHERE order_date < join_date
	)
SELECT *
FROM purchased_first
WHERE rank = 1
```
![](../src/Pasted%20image%2020250110162704.png)


8. **What is the total items and amount spent for each member before they became a member?**

```sql
select s.customer_id,sum(price) total
from sales s
join menu m
on s.product_id = m.product_id
join members ms
on s.customer_id = ms.customer_id
where join_date > order_date
group by s.customer_id


```


![](../src/Pasted%20image%2020250110162756.png)

9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
WITH total_points
AS (
	SELECT s.customer_id
		,product_name
		,price
		,CASE 
			WHEN product_name = 'sushi'
				THEN price * 20
			ELSE price * 10
			END AS points
	FROM sales s
	LEFT JOIN menu m ON s.product_id = m.product_id
	LEFT JOIN members ms ON s.customer_id = ms.customer_id
	)
SELECT customer_id
	,sum(points) AS total_points
FROM total_points
GROUP BY customer_id
ORDER BY total_points DESC
```

![](../src/Pasted%20image%2020250110162950.png)

10. **In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

```sql
SELECT s.customer_id,
       SUM(CASE
               WHEN product_name = 'sushi' THEN price * 20
               WHEN order_date BETWEEN join_date AND DATEADD(day, 7, join_date) THEN price * 20
               ELSE price * 10
           END) AS points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members  ON s.customer_id = members.customer_id
WHERE order_date < '2021-02-01'
GROUP BY s.customer_id;

```

![](../src/Pasted%20image%2020250110163046.png)
## Bonus Questions

### Join All The Things

The following questions are related creating basic data tables that Danny and his team can use to quickly derive insights without needing to join the underlying tables using SQL.

![](src/Pasted%20image%2020230607225742.png)


```sql
select s.customer_id,order_date,product_name,price,
		CASE 
			when order_date >= join_date then 'Y'
            else 'N'
 		END as member
from sales s
left join menu m
 on s.product_id  = m.product_id
left join members ms
on s.customer_id = ms.customer_id
order by customer_id,order_date
```

![](../src/Pasted%20image%2020250110163245.png)


### Rank All The Things

Danny also requires further information about the `ranking` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null `ranking` values for the records when customers are not yet part of the loyalty program.

![](../src/Pasted%20image%2020250110163412.png)
```sql
with data as (
select s.customer_id,order_date,product_name,price,
		CASE 
			when order_date >= join_date then 'Y'
            else 'N'
 		END as member
from sales s
left join menu m
 on s.product_id  = m.product_id
left join members ms
on s.customer_id = ms.customer_id
)
select *,
		CASE
        	WHEN member = 'N' then NULL
            else 
            rank() over (partition by customer_id,member order by order_date)
        end as ranking
FROM data

```

![](../src/Pasted%20image%2020250110163443.png)