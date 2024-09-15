## A. Pizza Metrics

### 1. How many pizzas were ordered?

````sql
SELECT 
    COUNT(order_id) total_orders
FROM temp_customer_orders;
````

**Answer:**
| total_orders |
| ------------ |
| 14 |

- 14 pizzas have delivered.

***

### 2. How many unique customer orders were made?

````sql
SELECT 
    COUNT(DISTINCT order_id) AS unique_order_count
FROM temp_customer_orders;
````

**Answer:**

| unique_order_count |
| ------------------ |
| 10                 |

- Customers have ordered 10 unique orders.

***

### 3. How many successful orders were delivered by each runner?

````sql
SELECT runner_id, 
	   COUNT(order_id) sucessful_deliveries
FROM temp_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;

````

**Answer:**
|runner_id | sucessful_deliveries |
| ---------| -------------------- |
| 1        |  4                   |
| 2        |  3                   |
| 3        |  1                   |

- Runner 1 has delivered 4 orders sucessfully.
- Runner 2 has delivered 3 orders sucessfully.
- Runner 3 has delivered 1 order sucessfully.

***

### 4. How many of each type of pizza was delivered?

````sql
SELECT pizza_names.pizza_name, 
       COUNT(t_customer_orders.order_id) AS order_count
FROM temp_customer_orders AS t_customer_orders
INNER JOIN pizza_runner.pizza_names AS pizza_names
    ON t_customer_orders.pizza_id = pizza_names.pizza_id
INNER JOIN temp_runner_orders AS t_runner_orders
    ON t_customer_orders.order_id = t_runner_orders.order_id
WHERE t_runner_orders.cancellation IS NULL
GROUP BY pizza_names.pizza_name;
````

**Answer:**
| pizza_name  | order_count |
| ----------- | ----------- |
| Meatllovers | 9           |
| Vegetarian  | 3           |

- Runners delivered 9 Meatlovers.
- Runners delivered 3 Meatlovers

***

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

````sql
SELECT t_customer_orders.customer_id,
		SUM(CASE WHEN pizza_names.pizza_name = 'Meatlovers' THEN 1 ELSE 0 END) AS Meatlovers,
		SUM(CASE WHEN pizza_names.pizza_name = 'Vegetarian' THEN 1 ELSE 0 END) AS Vegetarian
FROM temp_customer_orders AS t_customer_orders
LEFT JOIN pizza_runner.pizza_names AS pizza_names
    ON t_customer_orders.pizza_id = pizza_names.pizza_id
LEFT JOIN temp_runner_orders AS t_runner_orders
    ON t_customer_orders.order_id = t_runner_orders.order_id
GROUP BY t_customer_orders.customer_id
ORDER BY t_customer_orders.customer_id;
````

**Answer:**
| customer_id | Meatlovers | Vegetarian |
| ----------- | ---------- | ---------- |
| 101         | 2          | 1          |
| 102         | 2          | 1          |
| 103         | 3          | 1          |
| 104         | 3          | 0          |
| 105         | 0          | 1          |

- Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 102 ordered 2 Meatlovers pizzas and 2 Vegetarian pizzas.
- Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 1 Meatlovers pizza.
- Customer 105 ordered 1 Vegetarian pizza.

***

### 6. What was the maximum number of pizzas delivered in a single order?

````sql
WITH MaxPizzas AS (
    SELECT COUNT(pizza_id) AS maximum_pizzas
    FROM temp_customer_orders
    GROUP BY order_id
)

SELECT order_id, COUNT(pizza_id) AS maximum_pizzas
FROM temp_customer_orders
GROUP BY order_id
HAVING COUNT(pizza_id) = (SELECT MAX(maximum_pizzas) FROM MaxPizzas)
````

**Answer:**
| order_id | maximum_pizzas |
| -------- | -------------- |
| 4        | 3              |

- Order 4 asked for 3 pizzas.

***

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
WITH changed_pizzas AS (
    SELECT 
		co.customer_id,
        co.order_id,
        CASE 
            WHEN co.exclusions IS NOT NULL OR co.extras IS NOT NULL THEN 'Yes'
            ELSE 'No'
        END AS changed_pizza
    FROM temp_customer_orders co
    LEFT JOIN temp_runner_orders ro
        ON co.order_id = ro.order_id
    WHERE ro.cancellation IS NULL
)
SELECT 
	customer_id, 
    SUM(CASE WHEN changed_pizza = 'Yes' THEN 1 ELSE 0 END) AS at_least_one_change,
    SUM(CASE WHEN changed_pizza = 'No' THEN 1 ELSE 0 END) AS no_change
FROM changed_pizzas
GROUP BY customer_id
ORDER BY customer_id;
````

**Answer:**
| customer_id | at_least_one_change | no_change |
| ----------- | ------------------- | --------- |
| 101         | 0                   | 2         |
| 102         | 0                   | 3         |
| 103         | 3                   | 0         |
| 104         | 2                   | 1         |
| 105         | 1                   | 0         |


- Customer 101 and 102 likes his/her pizzas per the original recipe.
- Customer 103, 104 and 105 have their own preference for pizza topping and requested at least 1 change (extra or exclusion topping) on their pizza.

***

### 8.How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT 
	COUNT(customer_orders.pizza_id) exclusions_and_extra_orders_count
FROM temp_customer_orders customer_orders
INNER JOIN temp_runner_orders runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE customer_orders.exclusions IS NOT NULL
	AND customer_orders.extras IS NOT NULL 
	AND runner_orders.cancellation IS NOT NULL;
````

**Answer:**
| exclusions_and_extra_orders_count |
| --------------------------------- |
| 1                                 |

- Only 1 pizza delivered that had both extra and exclusion topping.

***

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT 
    EXTRACT(hour FROM order_time) AS order_hour,
    COUNT(DISTINCT order_id) AS order_count
FROM temp_customer_orders
WHERE order_time IS NOT NULL
GROUP BY order_hour
ORDER BY order_hour;
````

**Answer:**
| order_hour | order_count |
| ---------- | ----------- |
| 11         | 1           |
| 13         | 1           |
| 18         | 2           |
| 19         | 1           |
| 21         | 3           |
| 23         | 2           |


- Highest volume of pizza ordered is at 13 (1:00 pm), 18 (6:00 pm) and 21 (9:00 pm).
- Lowest volume of pizza ordered is at 11 (11:00 am), 19 (7:00 pm) and 23 (11:00 pm).


***


### 10.What was the volume of orders for each day of the week?

````sql

SELECT 
  TRIM(TO_CHAR(order_time, 'Day')) AS order_day,
  COUNT(order_id) AS order_count
FROM temp_customer_orders
GROUP BY TO_CHAR(order_time, 'Day')
ORDER BY order_count DESC;
````

**Answer:**
| order_day | order_count |
| --------- | ----------- |
| Saturday  | 5           |
| Wednesday | 5           |
| Thursday  | 3           |
| Friday    | 1           |


- On Saturdays and Wednesday more orders are placed.
- Friday is the day with the least orders.