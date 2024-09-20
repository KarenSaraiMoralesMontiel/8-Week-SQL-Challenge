# C. Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?

````sql
WITH meatlovers_toppings_cte AS (
	SELECT  TRIM(unnest(string_to_array(toppings, ',')))::numeric AS topping
	FROM pizza_runner.pizza_recipes
	WHERE pizza_id = 1),

 vegetarian_toppings_cte AS (
	SELECT  TRIM(unnest(string_to_array(toppings, ',')))::numeric AS topping
	FROM pizza_runner.pizza_recipes
	WHERE pizza_id = 2)

SELECT meatlovers_toppings_cte.topping 	standard_topping_id, 
	   pizza_toppings.topping_name standard_topping_name
FROM meatlovers_toppings_cte 
INNER JOIN vegetarian_toppings_cte
	ON meatlovers_toppings_cte.topping = vegetarian_toppings_cte.topping
LEFT JOIN pizza_runner.pizza_toppings pizza_toppings
	ON vegetarian_toppings_cte.topping = pizza_toppings.topping_id;
````

**Answer:**
| standard_topping_id | standard_topping_name |
| ------------------- | --------------------- |
| 2                   | Cheese                |
| 4                   | Mushrooms             |

- Both pizzas have cheese and mushrooms.

***

### 2. What was the most commonly added extra?

````sql
WITH toping_list_cte AS (
	SELECT TRIM(unnest(string_to_array(extras, ',')))::numeric AS topping_id
	FROM temp_customer_orders)
SELECT toping_list_cte.topping_id topping_id, pizza_toppings.topping_name topping_name,
		count(toping_list_cte.topping_id) max_topping_count
FROM toping_list_cte
LEFT JOIN pizza_runner.pizza_toppings pizza_toppings
		ON toping_list_cte.topping_id = pizza_toppings.topping_id
GROUP BY toping_list_cte.topping_id, topping_name
ORDER BY max_topping_count DESC
LIMIT 1;
````

**Answer:**
| topping_id | topping_name | max_topping_count |
| ---------- | ------------ | ----------------- |
| 1          | Bacon        | 4                 |

- Most commonly extra as the bacon. Delicious!

***


### 3. What was the most common exclusion?

````sql
WITH toping_list_cte AS (
	SELECT TRIM(unnest(string_to_array(exclusions, ',')))::numeric AS topping_id
	FROM temp_customer_orders)
SELECT toping_list_cte.topping_id topping_id, pizza_toppings.topping_name topping_name,
		count(toping_list_cte.topping_id) max_topping_count
FROM toping_list_cte
LEFT JOIN pizza_runner.pizza_toppings pizza_toppings
		ON toping_list_cte.topping_id = pizza_toppings.topping_id
GROUP BY toping_list_cte.topping_id, topping_name
ORDER BY max_topping_count DESC
LIMIT 1;
````

**Answer:**
| topping_id | topping_name | max_topping_count |
| ---------- | ------------ | ----------------- |
| 4          | Cheese        | 4                |

- The most excluded topping is cheese.

***

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
 - `Meat Lovers`
 - `Meat Lovers - Exclude Beef`
 - `Meat Lovers - Extra Bacon`
 - `Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers`

````sql
-- IT only shows orders with exclusions or extras, 
--it doesn't show normal orders
WITH names_list_cte AS (
    SELECT 
        orders_cte.order_id,
        pizza_names.pizza_name,
        orders_cte.order_time,
        -- Use unnesting only if there are exclusions or extras
        TRIM(unnest(string_to_array(orders_cte.extras, ',')))::int AS extra_topping_id,
        TRIM(unnest(string_to_array(orders_cte.exclusions, ',')))::int AS exclusion_topping_id
    FROM 
        temp_customer_orders orders_cte
	LEFT JOIN 
        pizza_runner.pizza_names pizza_names
        ON pizza_names.pizza_id = orders_cte.pizza_id
),
aggregate_lists_cte AS (
    SELECT 
        names_list_cte.order_id,
        names_list_cte.pizza_name, 
        names_list_cte.order_time,
        STRING_AGG(pizza_toppings.topping_name::text, ', ') AS extra_toppings_names,
        STRING_AGG(pizza_toppings2.topping_name::text, ', ') AS exclusion_toppings_names
    FROM 
        names_list_cte
    LEFT JOIN 
        pizza_runner.pizza_toppings pizza_toppings
        ON names_list_cte.extra_topping_id = pizza_toppings.topping_id
    LEFT JOIN 
        pizza_runner.pizza_toppings pizza_toppings2
        ON names_list_cte.exclusion_topping_id = pizza_toppings2.topping_id
    GROUP BY 
        names_list_cte.order_id,
        names_list_cte.pizza_name, 
        names_list_cte.order_time
)
SELECT order_id,
	   order_time,
	   pizza_name ||
	CASE 
		WHEN extra_toppings_names IS NOT NULL THEN ' - Include ' || extra_toppings_names
		ELSE ''
	END ||
	CASE 
		WHEN exclusion_toppings_names IS NOT NULL THEN ' - Exclude ' || exclusion_toppings_names
		ELSE ''
	END AS order_item
FROM aggregate_lists_cte;
````

**Answer:**
|order_id	|order_time|	order_item|
| --------- | -------- | ------------ |
|7	|08/01/2020 21:20	|Vegetarian - Include Bacon
|4	|04/01/2020 13:23	|Vegetarian - Exclude Cheese
|9	|10/01/2020 11:22	|Meatlovers - Include Bacon, Chicken - Exclude Cheese
|10	|11/01/2020 18:34	|Meatlovers - Include Bacon, Cheese - Exclude BBQ Sauce, Mushrooms
|4	|04/01/2020 13:23	|Meatlovers - Exclude Cheese, Cheese
|5	|08/01/2020 21:00	|Meatlovers - Include Bacon


***

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
 -For example: `"Meat Lovers: 2xBacon, Beef, ... , Salami"`


````sql

````

**Answer:**

***


### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

````sql
WITH order_pizza_info_cte AS (
    SELECT customer_orders.order_id, 
           customer_orders.pizza_id,
           -- Concatenate toppings with extras
           pizza_recipes.toppings ||
           CASE 
               WHEN customer_orders.extras IS NOT NULL THEN ', '|| customer_orders.extras 
               ELSE ''
           END AS standard_toppings_and_extras, 
           customer_orders.exclusions
    FROM temp_customer_orders customer_orders
	LEFT JOIN temp_runner_orders runner_orders
	ON customer_orders.order_id = runner_orders.order_id
    LEFT JOIN pizza_runner.pizza_recipes pizza_recipes
    ON customer_orders.pizza_id = pizza_recipes.pizza_id
	WHERE runner_orders.cancellation IS NULL
),
unnested_toppings AS (
    -- Unnest standard toppings and extras
    SELECT order_id, 
           pizza_id,
           unnest(string_to_array(standard_toppings_and_extras, ','))::int AS topping
    FROM order_pizza_info_cte
),
unnested_exclusions AS (
    -- Unnest exclusions
    SELECT order_id, 
           unnest(string_to_array(exclusions, ','))::int AS exclusion
    FROM order_pizza_info_cte
),
filtered_toppings AS (
    -- Remove exclusions from the list of toppings
    SELECT t.order_id,
           t.pizza_id,
           t.topping
    FROM unnested_toppings t
    LEFT JOIN unnested_exclusions e
    ON t.order_id = e.order_id 
       AND t.topping = e.exclusion
    WHERE e.exclusion IS NULL
)
-- Count occurrences of each ingredient
SELECT pizza_toppings.topping_name,
       COUNT(pizza_toppings.topping_name) AS count
FROM filtered_toppings
LEFT JOIN pizza_runner.pizza_toppings pizza_toppings
ON filtered_toppings.topping = pizza_toppings.topping_id
GROUP BY pizza_toppings.topping_name
ORDER BY count DESC;
````

**Answer:**
|topping_name|	count|
| ---------- | ----- |
|Bacon|	12|
|Mushrooms|	10|
|Chicken|	10|
|Cheese|	 9|
|Pepperoni|	 9|
|Salami|	9|
|Beef|	9|
|BBQ Sauce|	7|
|Tomato Sauce|	3|
|Onions|	3|
|Tomatoes|	3|
|Peppers|	3|