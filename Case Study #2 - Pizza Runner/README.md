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
CREATE TEMP TABLE temp_runner_orders AS 
SELECT
	order_id,
	customer_id, pizza_id,
		CASE 
    -- return empty string if there were no exclusions
    WHEN exclusions IS NULL OR TRIM(exclusions) LIKE 'null' THEN ''
    ELSE TRIM(exclusions)
END AS exclusions,

	CASE
        -- return empty string if there were no extras added
		WHEN extras IS NULL or TRIM(extras) LIKE 'null' THEN  ''
	  	ELSE TRIM(extras)
	 END AS extras,
	order_time
FROM pizza_runner.customer_orders;
````

#### Output
|order_id|	customer_id	|pizza_id|	exclusions	|extras	|order_time|
| ------ | ------------ | ------ | ------------ | ----- | -------- | 
|1	|101|	1|	|	|	01/01/2020 18:05 |
|2	|101|	1|	|	|	01/01/2020 19:00|
|3	|102|	1|	|	|	02/01/2020 23:51|
|3	|102|	2|	|	|	02/01/2020 23:51|
|4	|103|	1|	4|	|	04/01/2020 13:23|
|4	|103|	1|	4|	|	04/01/2020 13:23|
|4	|103|	2|	4|	|	04/01/2020 13:23|
|5	|104|	1|	 |	1	|08/01/2020 21:00|
|6	|101|	2|	 |		|08/01/2020 21:03|
|7	|105|	2|	 |	1	|08/01/2020 21:20|
|8	|102|	1|	 |		|09/01/2020 23:54|
|9	|103|	1|	4|	1, 5|	10/01/2020 11:22 |
|10	|104|	1|	 |		|11/01/2020 18:34| 
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
		WHEN pickup_time is NULL THEN NULL
		WHEN pickup_time = 'null' THEN NULL
		ELSE TRIM(pickup_time)
	END AS pickup_time,
	CASE
		WHEN distance is NULL THEN NULL
		WHEN distance = 'null' THEN NULL
		WHEN distance LIKE '%km' THEN TRIM('km' from distance)
		ELSE TRIM(distance)
	END AS distance,
	CASE 
		WHEN duration IS NULL THEN NULL
		WHEN duration = 'null' THEN NULL
		WHEN duration LIKE '%min%' THEN SPLIT_PART(duration, 'min', 1)
        ELSE TRIM(duration)
	END AS duration,
	CASE 
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