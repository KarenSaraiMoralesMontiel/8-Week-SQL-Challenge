# D. Pricing and Ratings

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?

````sql
SELECT 
CONCAT('$',SUM(
	CASE 
		WHEN pizza_id = 1 THEN 12
		WHEN pizza_id = 2 THEN 10
		ELSE 0
	END)) total_revenue
FROM temp_customer_orders customer_orders
LEFT JOIN temp_runner_orders runner_orders
	ON customer_orders.order_id = runner_orders.order_id 
WHERE cancellation IS NULL;
````

**Answer:**
| total_revenue |
| ------------- |
| $138          |

- There has been a total revenue of $138.

***

### 2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra

````sql
SELECT CONCAT('$', total_revenue) AS total_revenue
FROM (
    SELECT SUM(CASE
                  WHEN pizza_id = 1 THEN 12
                  ELSE 10
              END) AS pizza_revenue,
           SUM(topping_count) AS topping_revenue,
           SUM(CASE
                  WHEN pizza_id = 1 THEN 12
                  ELSE 10
              END) + SUM(topping_count) AS total_revenue
    FROM (
        SELECT *,
               LENGTH(extras) - LENGTH(REPLACE(extras, ',', '')) + 1 AS topping_count
        FROM temp_customer_orders
        INNER JOIN pizza_runner.pizza_names  pizza_names USING (pizza_id)
        INNER JOIN temp_runner_orders USING (order_id)
        WHERE cancellation IS NULL
    ) table_1
) table_2;
````

**Answer:**
| total_revenue |
| ------------- |
| $142          |


***

### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

````sql
-- Set the schema path
SET search_path = pizza_runner;

-- Drop the table if it already exists
DROP TABLE IF EXISTS runner_rating;

-- Create the new runner_rating table
CREATE TABLE runner_rating (
    rating_id SERIAL PRIMARY KEY,
    order_id INTEGER NOT NULL,
    customer_id INTEGER NOT NULL,
    runner_id INTEGER NOT NULL,
    rating DECIMAL(2,1) NOT NULL CHECK (rating >= 1 AND rating <= 5),
    rating_text TEXT,
    rating_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data into the runner_rating table
INSERT INTO runner_rating (
    order_id,
    customer_id,
    runner_id,
    rating,
    rating_text,
    rating_time
)
VALUES 
  (1, 101, 1, 5, 'Excellent service, very fast!', '2020-01-01 19:34:51'),
  (2, 101, 1, 5, 'On time and professional!', '2020-01-01 20:23:03'),
  (3, 102, 1, 4.5, 'Quick but forgot extra napkins', '2020-01-03 10:12:58'),
  (4, 103, 2, 3, 'Average service, could be faster', '2020-01-04 16:47:06'),
  (5, 104, 3, 5, 'Great runner, very friendly', '2020-01-08 23:09:27'),
  (7, 105, 2, 3, 'Got delayed, but okay service', '2020-01-08 23:50:12'),
  (8, 102, 2, 3, 'A bit slow, but polite', '2020-01-10 12:30:45'),
  (10, 104, 1, 2, 'Late and didn’t communicate', '2020-01-11 20:05:35');

````

**Answer:**
|rating_id|	order_id|	customer_id|	runner_id|	rating|	rating_text|	rating_time|
| ------- | ------- | ------------ | ----------- | ------ | ---------- | ------------- |
|1	|1	|101	|1	|5	|Excellent service, very fast!	|01/01/2020 19:34|
|2	|2	|101	|1	|5	|On time and professional!	|01/01/2020 20:23|
|3	|3	|102	|1	|4.5	|Quick but forgot extra napkins	|03/01/2020 10:12|
|4	|4	|103	|2	|3	|Average service, could be faster	|04/01/2020 16:47|
|5	|5	|104	|3	|5	|Great runner, very friendly	|08/01/2020 23:09|
|6	|7	|105	|2	|3	|Got delayed, but okay service	|08/01/2020 23:50|
|7	|8	|102	|2	|3	|A bit slow, but polite	|10/01/2020 12:30|
|8	|10	|104	|1	|2	|Late and didnâ€™t communicate	|11/01/2020 20:05|

***

### 4. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
	- `customer_id`
	- `order_id`
	- `runner_id`
	- `rating`
	- `order_time`
	- `pickup_time`
	- Time between order and pickup
	- Delivery duration
	- Average speed
	- Total number of pizzas

````sql
WITH avg_speed_cte AS (
    SELECT runner_id, 
 AVG((distance / duration) * 60) AS avg_speed    
	FROM temp_runner_orders
    WHERE cancellation IS NULL
    GROUP BY runner_id
),
total_pizzas_cte AS (
    SELECT customer_id,
           order_id,
           COUNT(DISTINCT pizza_id) AS total_pizzas 
    FROM temp_customer_orders
    GROUP BY customer_id, order_id
)
SELECT DISTINCT 
       customer_orders.customer_id,
       customer_orders.order_id,
       runner_orders.runner_id,
       runner_rating.rating,
       customer_orders.order_time,
       runner_orders.pickup_time,
       pickup_time - order_time AS difference_order_and_pickup,
       runner_orders.distance,
       runner_orders.duration AS duration_minutes,
       avg_speed_cte.avg_speed,
       total_pizzas_cte.total_pizzas
FROM temp_customer_orders customer_orders
INNER JOIN temp_runner_orders runner_orders
    ON customer_orders.order_id = runner_orders.order_id
LEFT JOIN pizza_runner.runner_rating runner_rating
    ON customer_orders.order_id = runner_rating.order_id
LEFT JOIN avg_speed_cte
    ON runner_orders.runner_id = avg_speed_cte.runner_id
LEFT JOIN total_pizzas_cte
    ON customer_orders.order_id = total_pizzas_cte.order_id
    AND customer_orders.customer_id = total_pizzas_cte.customer_id
WHERE runner_orders.cancellation IS NULL;
````

**Answer:**
customer_id| order_id| runner_id| rating| order_time| pickup_time| difference_order_and_pickup| distance| duration_minutes| avg_speed	| total_pizzas |
 --------- | ------| -------- | ----- | ------- | -------- | ---------------------------------| -------- | --------------- | --------	| ------------ |
|103	|4	|2	|3	|04/01/2020 13:23	|04/01/2020 13:53	|00:29:17	|23.4	|40	|62.9	|2|
|101	|2	|1	|5	|01/01/2020 19:00	|01/01/2020 19:10	|00:10:02	|20	|27	|45.53611111	|1|
|102	|3	|1	|4.5	|02/01/2020 23:51	|03/01/2020 00:12	|00:21:14	|13.4	|20	|45.53611111	|2|
|104	|5	|3	|5	|08/01/2020 21:00	|08/01/2020 21:10	|00:10:28	|10	|15	|40	|1|
|101	|1	|1	|5	|01/01/2020 18:05	|01/01/2020 18:15	|00:10:32	|20	|32	|45.53611111	|1|
|105	|7	|2	|3	|08/01/2020 21:20	|08/01/2020 21:30	|00:10:16	|25	|25	|62.9	|1|
|102	|8	|2	|3	|09/01/2020 23:54	|10/01/2020 00:15	|00:20:29	|23.4	|15	|62.9	|1|
|104	|10	|1	|2	|11/01/2020 18:34	|11/01/2020 18:50	|00:15:31	|10	|10	|45.53611111	|1|

***

### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?

````sql
WITH total_revenue_cte AS (SELECT 

SUM(
	CASE 
		WHEN pizza_id = 1 THEN 12
		WHEN pizza_id = 2 THEN 10
		ELSE 0
	END) total_revenue
FROM temp_customer_orders customer_orders
LEFT JOIN temp_runner_orders runner_orders
	ON customer_orders.order_id = runner_orders.order_id 
WHERE cancellation IS NULL)
SELECT total_revenue, 
	SUM(distance) * 0.3::NUMERIC runner_paid, 
	total_revenue - SUM(distance) * 0.3 profit
FROM temp_runner_orders
CROSS JOIN total_revenue_cte
GROUP BY 1;
````

**Answer:**
|total_revenue|	runner_paid|	profit|
| ----------- | ---------- | -------- |
|138	|43.56	|94.44|

***