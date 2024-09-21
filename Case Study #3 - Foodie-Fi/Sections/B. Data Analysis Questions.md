# B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

````sql
SELECT COUNT(DISTINCT customer_id) total_customers
FROM foodie_fi.subscriptions;
````

**Answer:**
| total_customers |
| --------------- |
| 1000            |

***

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value?

````sql
SELECT
  TO_CHAR(start_date, 'Month') AS trial_month,
  COUNT(DISTINCT customer_id) AS trial_total_customers
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY TO_CHAR(start_date, 'Month'), EXTRACT(MONTH FROM start_date)
ORDER BY EXTRACT(MONTH FROM start_date);
````

**Answer:**
|trial_month | trial_total_customers |
| ---------- | --------------------- | 
|January  	|88|
|February 	|68|
|March    	|94|
|April    	|81|
|May      	|88|
|June     	|79|
|July     	|89|
|August   	|88|
|September |	87|
|October  |	79|
|November |	75|
|December |	84|


***

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

````sql
SELECT
  plan_name,
  COUNT(DISTINCT customer_id) total_count_after_2020
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING (plan_id)
WHERE EXTRACT(YEAR FROM start_date) > 2020
GROUP BY plan_name;
````

**Answer:** 
|plan_name | total_count_after_2020 |
|basic monthly|	8 |
|churn|	71 |
|pro annual|	63 |
|pro monthly|	60 |

***

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

````sql
SELECT
  plan_name,
  COUNT(DISTINCT customer_id) churn_count,
  ROUND(100*COUNT(DISTINCT customer_id)::numeric/(
  SELECT COUNT(DISTINCT customer_id)
  FROM foodie_fi.subscriptions), 1) percentage
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING (plan_id)
WHERE plan_name = 'churn'
GROUP BY plan_name;
````

**Answer:**
| plan_name | churn_count | percentage |
| --------- | ----------- | ---------- |
| churn     | 307         | 30.7       |

***

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

````sql
WITH customer_plans_cte AS (
  SELECT 
    plan_name,
    customer_id,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS movements_num
  FROM foodie_fi.subscriptions subscriptions
  LEFT JOIN foodie_fi.plans
    USING (plan_id)
)
SELECT 
    COUNT(CASE 
    WHEN movements_num = 2 AND plan_name = 'churn' THEN 1 
    ELSE 0 END) AS churned_customers,
    ROUND(100.0 * COUNT(
    CASE 
      WHEN movements_num = 2 AND plan_name = 'churn' THEN 1 
      ELSE 0 END) 
      / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  ) AS churn_percentage FROM customer_plans_cte
WHERE plan_name = 'churn'
AND movements_num = 2;
````

**Answer:**
| churned_customers | churn_percentage |
| ----------------- | ---------------- |
| 92                | 9                |


***