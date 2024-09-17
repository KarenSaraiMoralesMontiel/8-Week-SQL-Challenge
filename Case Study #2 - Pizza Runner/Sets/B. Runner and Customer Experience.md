# B. Runner and Customer Experience

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
SELECT 
		DATE_TRUNC(
			'day', 
			'2021-01-01'::date + (FLOOR((registration_date - '2021-01-01'::date) / 7) * 7) * INTERVAL '1 day'
				  ) AS week_start,
		COUNT(runner_id) AS runners_signed_up
FROM pizza_runner.runners
GROUP BY week_start
ORDER BY week_start; 
````

**Answer:**
|    week_start    | runners_signed_up |
| ---------------- | ----------------- |
| 01/01/2021 00:00 |	2              |
| 08/01/2021 00:00 |	1              |
| 15/01/2021 00:00 |	1              |

- First week starting 01/01/2021 2 runners signed to be pizza runners.
- Second week starting 08/01/2021 1 runner signed to be a pizza runner.
- Third week starting 08/01/2021 1 runner signed to be a pizza runner.

***

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
WITH pickup_time_cte AS (
	SELECT customer_orders.order_id,
	   	   runner_orders.runner_id,
	       EXTRACT(EPOCH FROM (runner_orders.pickup_time - customer_orders.order_time)) / 60 AS pickup_minutes
	FROM temp_customer_orders AS customer_orders
	INNER JOIN temp_runner_orders runner_orders
	ON customer_orders.order_id = runner_orders.order_id
)
SELECT runner_id,
		ROUND(AVG(pickup_minutes), 2) as average_run_minutes
FROM pickup_time_cte
GROUP BY runner_id
ORDER BY runner_id;
````

**Answer:**
| runner_id | average_run_minutes |
| --------- | ------------------- |
| 1         | 15.68               |
| 2         | 23.72               |
| 3         | 10.47               |

- Runner 1 has an average of 15.68 minutes.
- Runner 2 has an average of 23.72 minutes.
- Runner 3 has an average of 10.47 minutes.

***

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
WITH preparing_time_cte AS (
	SELECT customer_orders.order_id,
	   COUNT(customer_orders.pizza_id) as num_pizzas,
	   --customer_orders.order_time,
	   --runner_orders.pickup_time,
	   runner_orders.pickup_time - customer_orders.order_time as pickup_minutes
	FROM temp_customer_orders customer_orders
	INNER JOIN temp_runner_orders runner_orders
	ON customer_orders.order_id = runner_orders.order_id
	GROUP BY customer_orders.order_id, 
	   customer_orders.order_time,
	   runner_orders.pickup_time
	ORDER BY customer_orders.order_id
	)
	
	SELECT num_pizzas,
		   AVG(pickup_minutes) avg_preparing_time
FROM preparing_time_cte
GROUP BY num_pizzas;
````


**Answer:**
| num_pizzas | avg_preparing_time |
| ---------- | ------------------ |
| 3          | 0:29:17            |
| 2          | 00:18:22.5         |
| 1          | 00:12:21.4         |

- It takes the longest time to prepare 3 pizzas.

***

### 4. What was the average distance travelled for each customer?

````sql
SELECT customer_orders.customer_id, 
		ROUND(AVG(runner_orders.distance::numeric), 2) avg_distance_km
FROM temp_customer_orders customer_orders
JOIN temp_runner_orders runner_orders
ON customer_orders.order_id = runner_orders.order_id
GROUP BY customer_orders.customer_id
ORDER BY avg_distance_km DESC;
````

**Answer:**
| customer_id | avg_distance_km |
| ----------- | --------------- |
| 105         | 25.00           |
| 103         | 23.40           |
| 101         | 20.00           |
| 102         | 16.73           |
| 104         | 10.00           |

- Customer 105 has the higuest average distance, which implies this customer is the one the most far away from the pizzeria.
- Customer 104 has the lowest average distance, which implies this customer is the closest to the pizzeria.

***

### 5. What was the difference between the longest and shortest delivery times for all orders?

````sql
SELECT MAX(duration) - MIN(duration) delivery_time_difference_min
FROM temp_runner_orders;
````

**Answer:**
| delivery_time_difference_min |
| ---------------------------- |
| 30                           |

- The range of duration is 30 minutes of all orders.

***

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
SELECT order_id,
		runner_id,
		duration,
		distance,
		ROUND((distance / duration*60)::numeric,2) avg_kim_per_sec
FROM temp_runner_orders
WHERE (distance / duration * 60) IS NOT NULL
ORDER BY runner_id,avg_kim_per_sec;
````

**Answer:**
|order_id|	runner_id|	duration|	distance|	avg_kim_per_sec|
| -----  | --------- | -------- | --------- | ---------------- |
|1|	1|	32|	20|	37.5|
|3|	1|	20|	13.4|	40.2|
|2|	1|	27|	20|	44.44|
|10|1|	10|	10|	60|
|4|	2|	40|	23.4|	35.1|
|7|	2|	25|	25|	60|
|8|	2|	15|	23.4|	93.6|
|5|	3|	15|	10|	40|


- Runner 1’s average speed runs from 37.5km/h to 60km/h.
- Runner 2’s average speed runs from 35.1km/h to 93.6km/h.
- Runner 3’s average speed is 40km/h

***

### 7. What is the successful delivery percentage for each runner?

````sql
WITH orders_cte_by_runner_cte AS (
	SELECT RUNNER_ID,
		count(order_id) total_orders,
		count(CASE WHEN cancellation IS NULL THEN 1 END) AS sucessfull_deliveries
	FROM temp_runner_orders
	group by RUNNER_ID)
	
SELECT runner_id,
	ROUND((sucessfull_deliveries / total_orders::numeric) * 100, 2) sucessfull_rate
FROM orders_cte_by_runner_cte;
````

**Answer:**
| runner_id | sucessfull_rate |
| --------- | --------------- |
| 1         | 100.00          |
| 2         | 75.00           |
| 3         | 50.00           |

- Runner 1 has the higuest sucessfull rate at 100 %.
- Runner 2 has a 75 % sucessfull rate.
- Runner 3 has a 50 % sucessfull rate.

Sucessfull rates however are not a reliable metric to score the runners, it could be multiple reasons why an order could not be delivered from bad seasonal weather to a cancellation from either the restaurant or the customer! 