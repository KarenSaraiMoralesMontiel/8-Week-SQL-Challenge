# 🍕 Case Study #2: Pizza Runner

<img src='https://8weeksqlchallenge.com/images/case-study-designs/2.png' alt="Pizza Runner Image" width="500" height="520">

***

## 📖 Table of Contents
1. [Bussiness Task](#bussiness-task)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Data Cleaning](#data-cleaning)
4. [Solution](#analysis-questions)


Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-2/).

***

## Bussiness Task
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to Uberize it - and so Pizza Runner was launched!

***

## Entitity Relationship Diagram
![Pizza Runners ERD Image](image.png)

***

## 🧼 Data Cleaning

### 🛠 Customer Orders

This is the `pizza_runner.customer_orders` table. As we can see the columns `customer_orders.exclusions` and  `customer_orders.extras` present irregularations in their data wich will cause our analysis to be less accurate.

order_id |	customer_id	|pizza_id	|exclusions	|extras	|order_time|
 ------- | -------------| --------- | --------- | ------| -------- |
1 |	101|	1	|	| |	01/01/2020 18:05
2|	101	|1	||		|01/01/2020 19:00
3|	102	|1	||		|02/01/2020 23:51
3|	102	|2	|	|NULL	|02/01/2020 23:51
4|	103	|1	|4	|	|04/01/2020 13:23
4|	103	|1	|4	|	|04/01/2020 13:23
4|	103	|2	|4	|	|04/01/2020 13:23
5|	104	|1	|null|	1	|08/01/2020 21:00
6|	101	|2	|null|	null	|08/01/2020 21:03
7|	105	|2	|null	|1	|08/01/2020 21:20
8|	102	|1|	null	|null	|09/01/2020 23:54
9|	103	|1|	4|	1, 5	|10/01/2020 11:22
10|	104	|1|	null	|null	|11/01/2020 18:34
10|	104|	1	|2, 6	|1, 4	|11/01/2020 18:34

The columns are plagued with **NULL** values, empty strings '' and 'null' strings. we can also see that multiple ponts are in the exclusions and extras.
First we create a tempt table to handle the missing values before creating the new descriptive tables.

````sql
CREATE TEMP TABLE temp_customer_orders AS 
SELECT
	order_id,
	customer_id, pizza_id,
		CASE 
    -- return empty string if there were no exclusions
    WHEN exclusions IS NULL OR TRIM(exclusions) LIKE 'null' THEN NULL
    WHEN exclusions = '' THEN NULL
    ELSE TRIM(exclusions)
END AS exclusions,

	CASE
        -- return empty string if there were no extras added
		WHEN extras IS NULL or TRIM(extras) LIKE 'null' THEN  NULL
        WHEN extras = '' THEN NULL
	  	ELSE TRIM(extras)
	 END AS extras,
	order_time
FROM pizza_runner.customer_orders;
````

#### Output
|order_id|	customer_id	|pizza_id|	exclusions	|extras	|order_time|
| ------ | ------------ | ------ | ------------ | ----- | -------- | 
|1	|101|	1|NULL	|NULL	|	01/01/2020 18:05 |
|2	|101|	1|NULL	|NULL	|	01/01/2020 19:00|
|3	|102|	1|NULL	|NULL	|	02/01/2020 23:51|
|3	|102|	2|NULL	|NULL	|	02/01/2020 23:51|
|4	|103|	1|	4|NULL	|	04/01/2020 13:23|
|4	|103|	1|	4|NULL	|	04/01/2020 13:23|
|4	|103|	2|	4|NULL	|	04/01/2020 13:23|
|5	|104|	1|NULL	 |	1	|08/01/2020 21:00|
|6	|101|	2|	 NULL|	NULL	|08/01/2020 21:03|
|7	|105|	2|	NULL |	1	|08/01/2020 21:20|
|8	|102|	1|	NULL |	NULL	|09/01/2020 23:54|
|9	|103|	1|	4|	1, 5|	10/01/2020 11:22 |
|10	|104|	1|NULL	 |	NULL	|11/01/2020 18:34| 
|10	|104|	1|	2, 6|	1, 4	|11/01/2020 18:34|

***

### 🛠 Runner Orders

This is the `pizza_runner.runner_orders` table. As we can see the columns `runner_orders.pickup_time`,  `runner_orders.distance` , `runner_orders.duration` and `runner_orders.cancellation` present irregularations in their data wich will cause our analysis to be less accurate.

|order_id|	runner_id|	pickup_time	|distance|	duration|	cancellation|
| ------| ---------- | ------------ | ------ | -------- | ------------- |
|1|	1|	01/01/2020 18:15 |	20km|	32 minutes	| |
|2|	1|	01/01/2020 19:10 |	20km|	27 minutes	|
|3|	1|	03/01/2020 00:12 |	13.4km|	20 mins	|NULL|
|4|	2|	04/01/2020 13:53  |	23.4|	40|	NULL|
|5|	3|	08/01/2020 21:10 |	10|	15	|NULL|
|6|	3|	null|	null |	null|	Restaurant Cancellation|
|7|	2|	08/01/2020 21:30	|25km|	25mins	|null|
|8|	2|	10/01/2020 00:15	|23.4 km|	15 minute|	null |
|9|	2|	null|	null|	null|	Customer Cancellation |
|10|	1|	11/01/2020 18:50|	10km|	10minutes|	null|

We create a temporary table and clean the used data.

````sql
CREATE TEMP TABLE temp_runner_orders AS 
SELECT 
	order_id,
	runner_id,
	CASE
        -- Impute with null values
		WHEN pickup_time is NULL THEN NULL
		WHEN pickup_time = 'null' THEN NULL
		ELSE TRIM(pickup_time)
	END AS pickup_time,
	CASE
        -- Impute with null values
		WHEN distance is NULL THEN NULL
		WHEN distance = 'null' THEN NULL
		WHEN distance LIKE '%km' THEN TRIM('km' from distance)
		ELSE TRIM(distance)
	END AS distance,
	CASE 
        -- Impute with null values
		WHEN duration IS NULL THEN NULL
		WHEN duration = 'null' THEN NULL
		WHEN duration LIKE '%min%' THEN SPLIT_PART(duration, 'min', 1)
        ELSE TRIM(duration)
	END AS duration,
	CASE 
        -- Impute with null values
		WHEN cancellation LIKE '' THEN NULL
		WHEN cancellation IS NULL THEN NULL
		WHEN cancellation = 'null' THEN NULL
		ELSE TRIM(cancellation)
	END AS cancellation
FROM pizza_runner.runner_orders;
````

Then alter the table type.

````sql 
ALTER TABLE temp_runner_orders
ALTER COLUMN pickup_time TYPE DATE USING pickup_time::date,
ALTER COLUMN distance TYPE FLOAT USING distance::float,
ALTER COLUMN duration TYPE INT USING duration::int;

````
#### Output
|order_id|	runner_id	|pickup_time	|distance | duration |cancellation|
| ------ | ------------ | ------------- | --------| ------- | ---------- |
|1	|1	|01/01/2020|	20|	32|	NULL|
|2	|1	|01/01/2020|	20|	27|	NULL|
|3	|1	|03/01/2020|	13.4|	20|	NULL|
|4	|2	|04/01/2020|	23.4|	40|	NULL|
|5	|3	|08/01/2020|	10	|15	|NULL|
|6	|3	|NULL|	NULL|	NULL	|Restaurant Cancellation|
|7	|2	|08/01/2020|	25|	25	   |NULL|
|8	|2	|10/01/2020|	23.4|	15	|NULL|
|9	|2	|NULL	|NULL|	NULL	|Customer Cancellation|
|10|	1|	11/01/2020	|10|	10|	NULL|

Now we are all set to use this temporary tables to answer the solutions from this case study!

***

## Solution

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

***


### 8.How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT COUNT(pizza_id) exclusions_and_extra_orders_count
FROM temp_customer_orders
WHERE exclusions IS NOT NULL
	AND extras IS NOT NULL;
````

**Answer:**
| exclusions_and_extra_orders_count |
| --------------------------------- |
| 2                                 |

***


### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql

````

**Answer:**

***


### 10.What was the volume of orders for each day of the week?

````sql

````

**Answer:**

***


## B. Runner and Customer Experience

### 1. 

````sql

````

**Answer:**

***


