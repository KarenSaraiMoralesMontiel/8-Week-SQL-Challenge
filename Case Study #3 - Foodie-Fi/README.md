# 🥑 Case Study #3: Foodie-Fi
[![View Main Folder](https://img.shields.io/badge/View-Main_Folder-971901?)](https://github.com/KarenSaraiMoralesMontiel/8-Week-SQL-Challenge/tree/main)
[![View My Portfolio](https://img.shields.io/badge/View-My_Profile-green?logo=GitHub)](https://github.com/KarenSaraiMoralesMontiel/Portfolio)

<img src='https://8weeksqlchallenge.com/images/case-study-designs/3.png' alt="Foodie-Fi Image" width="500" height="520">

***

## 📖 Table of Contents
1. [Bussiness Task](#bussiness-task)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Solution](#solutions)
    - [A. Customer Journey](#a-customer-journey)
    - [B. Data Analysis Questions](#b-data-analysis-questions)
    - [C. Challenge Payment Question](#c-challenge-payment-question)
    - [D. Outside The Box Questions ](#d-outside-the-box-questions)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-3/).

## Bussiness Task
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny finds a few smart friends to launch his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

## Entity Relationship Diagram
![Foodie-Fi ERD](image.png)

## Solution

## A. Customer Journey
[![Go back to Tabe of Contents](https://img.shields.io/badge/View-Main_Folder-971901?)](#-table-of-contents)

<details>

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

````sql
SELECT 
	  subscriptions.customer_id,
	  plans.plan_name,
	  plans.price,
	  subscriptions.start_date
FROM foodie_fi.subscriptions subscriptions
LEFT JOIN  foodie_fi.plans plans
USING (plan_id)
WHERE subscriptions.customer_id IN (1,2,11,13,15,16,18,19);
````

**Answer:**
| customer_id | plan_name | price | start_date |
| ----------- | --------- | ----- | ---------- |
|1	|trial	|0	|01/08/2020|
|1	|basic monthly	|9.9	|08/08/2020|
|2	|trial	|0	|20/09/2020|
|2	|pro annual	|199	|27/09/2020|
|11	|trial	|0	|19/11/2020|
|11	|churn	|NULL	|26/11/2020|
|13	|trial	|0	|15/12/2020|
|13	|basic monthly	|9.9	|22/12/2020|
|13	|pro monthly	|19.9	|29/03/2021|
|15	|trial	|0	|17/03/2020|
|15	|pro monthly	|19.9	|24/03/2020|
|15	|churn	|NULL	|29/04/2020|
|16	|trial	|0	|31/05/2020|
|16	|basic monthly	|9.9	|07/06/2020|
|16	|pro annual	|199	|21/10/2020|
|18	|trial	|0	|06/07/2020|
|18	|pro monthly	|19.9	|13/07/2020|
|19	|trial	|0	|22/06/2020|
|19	|pro monthly	|19.9	|29/06/2020|
|19	|pro annual	|199	|29/08/2020|


Let's start with customer 1! They started a trial plan on August 1, 2020 updated automatically to basic monthly plan a week after and has been on that plan ever since.

| customer_id | plan_name | price | start_date |
| ----------- | --------- | ----- | ---------- |
|1	|trial	|0	|01/08/2020|
|1	|basic monthly	|9.9	|08/08/2020|

Customer 15 started with a trial on March 31, 2020 and updated to pro monthy a week after but cancelled a month and five days later.

| customer_id | plan_name | price | start_date |
| ----------- | --------- | ----- | ---------- |
|15	|trial	|0	|17/03/2020|
|15	|pro monthly	|19.9	|24/03/2020|
|15	|churn	|NULL	|29/04/2020|

Customer 19 started with a trialplan on June 22, 2020,  upgraded it to a pro monthly plan a week after and two months later updated to a pro annual plan.

| customer_id | plan_name | price | start_date |
| ----------- | --------- | ----- | ---------- |
|19	|trial	|0	|22/06/2020|
|19	|pro monthly	|19.9	|29/06/2020|
|19	|pro annual	|199	|29/08/2020|

</details>

***

## B. Data Analysis Questions
[![Go back to Tabe of Contents](https://img.shields.io/badge/View-Main_Folder-971901?)](#-table-of-contents)

<details>

### 1. How many customers has Foodie-Fi ever had?

````sql
SELECT COUNT(DISTINCT customer_id) total_customers
FROM foodie_fi.subscriptions;
````

**Answer:**
| total_customers |
| --------------- |
| 1000            |

- Foodie-Fi has had 1000 customers in total.

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

- The higuest amount of trial customers occurs in March, followed by July and then January or August.

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
| -------- | ---------------------- |
|basic monthly|	8 |
|churn|	71 |
|pro annual|	63 |
|pro monthly|	60 |

- A lot of our customer desactivated their accounts.

***

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

````sql
SELECT
  plan_name,
  COUNT(DISTINCT customer_id) churn_count,
  ROUND(100*COUNT(DISTINCT customer_id)::numeric/(
  SELECT COUNT(DISTINCT customer_id)
  FROM foodie_fi.subscriptions), 2) percentage
FROM foodie_fi.subscriptions
LEFT JOIN foodie_fi.plans
USING (plan_id)
WHERE plan_name = 'churn'
GROUP BY plan_name;
````

**Answer:**
| plan_name | churn_count | percentage |
| --------- | ----------- | ---------- |
| churn     | 307         | 30.70      |

- 30% of all customer have desactivated their account.

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
	(100.0 * COUNT(
    CASE 
      WHEN movements_num = 2 AND plan_name = 'churn' THEN 1 
      ELSE 0 END) 
	  / (SELECT COUNT(DISTINCT customer_id) 
      FROM foodie_fi.subscriptions)
  )::NUMERIC AS churn_percentage FROM customer_plans_cte
WHERE plan_name = 'churn'
AND movements_num = 2;
````

**Answer:**
| churned_customers | churn_percentage |
| ----------------- | ---------------- |
| 92                | 9                |

 - Only 9% of customers bother to desactivate their accounts after the frial trial.

***

### 6. What is the number and percentage of customer plans after their initial free trial?

I first created a cte called `movements_cte` using `ROW_NUMBER()` over the customer_id ordering them by the start_date in ascending order joining the tables `plans` and `subscriptions`.

In the outer querry I did a self join to get the first movements and second_movements and select the ones whose first movement is `trial`.

Laslty, I performed calculations to get the count_per_plan and the percentage of plans.

````sql
WITH movements_cte AS (
	SELECT customer_id,
		plans.plan_id,
		plans.plan_name,
		ROW_NUMBER() OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date) movements
FROM foodie_fi.plans plans
RIGHT JOIN foodie_fi.subscriptions subscriptions
ON plans.plan_id = subscriptions.plan_id)
SELECT second_movement.plan_id,second_movement.plan_name, 
	   COUNT(DISTINCT first_trial.customer_id) customer_count,
	   ROUND(100 * COUNT(DISTINCT first_trial.customer_id) / (
	   		SELECT COUNT(DISTINCT customer_id)
		   		FROM foodie_fi.subscriptions
	   )::numeric,2) percentage
FROM movements_cte first_trial
JOIN movements_cte second_movement
  ON first_trial.customer_id = second_movement.customer_id
  AND first_trial.movements = 1
  AND second_movement.movements = 2
WHERE first_trial.plan_name = 'trial'
GROUP BY 1,2;
````

Another way to solve it is by using `LEAD` on plan_id in the cte window over a `PARTITION BY` customer_id before ordering them by start_date.

In the outer querry select all the rows with previous plan as 0.

Finally we calculate the count per plan and the percentage of each plan.

````sql
WITH movements_cte AS (
	SELECT customer_id,
		subscriptions.plan_id previous_plan,
		LEAD(subscriptions.plan_id) OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date) next_plan
FROM foodie_fi.subscriptions subscriptions
)
SELECT movements_cte.next_plan plan_id,
		plans.plan_name,
	   COUNT(DISTINCT movements_cte.customer_id),
	   ROUND((100 * COUNT(DISTINCT movements_cte.customer_id)::numeric / 
	   (
	   		SELECT COUNT(DISTINCT customer_id)
	   		FROM movements_cte)), 2)
FROM movements_cte
JOIN foodie_fi.plans plans  
	ON movements_cte.next_plan = plans.plan_id
WHERE previous_plan = 0
GROUP BY 1,2;
````

**Answer:**
|plan_id|plan_name	|customer_count	|percentage|
| ----  | --------- | ------------- | -------- |
|1 |basic monthly|	546|	54.60|
|2|pro monthly|	325|	32.50|
|3|pro annual|	37|	3.70|
|4| churn |	92|	9.20|

- Almost 80% of all customers upgrade to a monthly subscriptions while only 3.7% of customers upgrade to pro annual and only 9% disactivate their accounts

***

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

We first create a cte called `movements_desc_cte` using `ROW_NUMBER()` over the customer_id ordering them by the start_date in descending order.

In the outer querry we select all the movements where `movements_desc` is equal to one to get the last movement.

Lastly, I performed calculations to get the count_per_plan and the percentage of plans.

````sql
WITH movements_desc_cte AS (
	SELECT plans.plan_id,
		plans.plan_name,
	  ROW_NUMBER() OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date DESC) movements_desc
FROM foodie_fi.plans plans
JOIN foodie_fi.subscriptions subscriptions
	ON plans.plan_id = subscriptions.plan_id
WHERE start_date::date <= '2020-12-31')
SELECT plan_id,
	   plan_name,
	   COUNT(plan_name)::numeric count_per_plan,
	   ROUND(100 * COUNT(plan_name) / (
	   		SELECT COUNT(plan_name)
		    FROM movements_desc_cte
		   WHERE movements_desc = 1
	   )::numeric, 2) percentage_before_date
FROM movements_desc_cte
WHERE movements_desc = 1
GROUP BY 1,2
ORDER BY plan_id;
````

Another way to solve it is by using `LEAD` on start_date in the cte window over a `PARTITION BY` customer_id before `2020-12-31`.

In the outer querry select all the rows with `next_date` as null to select the last movement.

Finally we calculate the count per plan and the percentage of each plan.

````sql
WITH movements_cte AS (
	SELECT
		subscriptions.customer_id,
		plans.plan_id,
		plans.plan_name,
		LEAD(subscriptions.start_date) OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date) next_date
FROM foodie_fi.plans plans
JOIN foodie_fi.subscriptions subscriptions
on plans.plan_id = subscriptions.plan_id
	WHERE start_date <= '2020-12-31'
)
SELECT
	plan_id,
	plan_name, 
	COUNT(DISTINCT customer_id)::numeric AS customers,
  ROUND(100.0 * 
    COUNT(DISTINCT customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM movements_cte)
  ,2) AS percentage
FROM movements_cte 
WHERE next_date IS NULL
GROUP BY 1,2
ORDER BY 1;
````

**Answer:**
|plan_id|	plan_name|	count_per_plan| percentage_before_date |
| ----- | -------- | -------------- | ---------------------- |
|0|	trial|	19| 1.90 |
|1|	basic monthly|	224| 22.40 |
|2|	pro monthly|	326| 32.60  |
|3|	pro annual|	195| 19.50     |
|4|	churn|	236| 23.60       |

- Almost 50% of customers before 2021 were on the monthly subscriptions.

***

### 8. How many customers have upgraded to an annual plan in 2020?

We can create a movements_cte using `ROW_NUMBER` partitioned by `customer_id` ordering them by `start_date` where the start_year is `2020`.

In the outer querry count the plan_name where the `plan_name` is pro_annual and the `start_date_year` is `2020`.

Finally we count the records.

````sql
WITH movements_cte AS (
	SELECT plans.plan_id,
		plans.plan_name,
	  ROW_NUMBER() OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date) movements
FROM foodie_fi.plans plans
JOIN foodie_fi.subscriptions subscriptions
	ON plans.plan_id = subscriptions.plan_id
WHERE EXTRACT(YEAR FROM subscriptions.start_date ) = '2020')
SELECT  
	COUNT(plan_name) total_count_pro_annual
FROM movements_cte
WHERE movements > 1
AND plan_name = 'pro annual';
````
Another way is We can create a movements_cte using `LEAD` on `plan_id` and to the `start_date` partitioned by `customer_id` and another one getting the year from the start_date

In the outer querry count the plan_name where the `plan_name` is `3` and the `start_year` is `2020`.

Finally we count the records.

````sql
WITH movements_cte AS (
	SELECT customer_id,
		plan_id previous_plan,
		LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) next_plan,
		LEAD(EXTRACT(YEAR FROM start_date)) OVER (PARTITION BY customer_id ORDER BY start_date) start_year
FROM  foodie_fi.subscriptions
)
SELECT count(next_plan) total_count_pro_annual
FROM movements_cte
WHERE next_plan = 3
and start_year = '2020';
````

**Answer:**
|total_count_pro_annual|
|  ----- |
|  195   |

- Only 195 customers have upgraded to pro annual. 

***

### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

````sql
WITH start_plan_cte AS
  (SELECT plan_id,
   		  start_date final_date,
          FIRST_VALUE(start_date) OVER (PARTITION BY customer_id
                                       ORDER BY start_date) plan_start_date
   FROM foodie_fi.subscriptions subscriptions)
SELECT round(avg( final_date - plan_start_date), 2) AS avg_conversion_to_pro_annual_days
FROM start_plan_cte
WHERE plan_id = 3;
````

**Answer:**
| avg_conversion_to_pro_annual_days |
| --------------------------------- |
| 104.62                            |

- It takes a little more than three months for customers to choose to upgrade to pro annual.

***

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

We use `WIDTH_BUCKET`. 

````sql
WITH start_plan_cte AS (
    SELECT plan_id,
           start_date AS final_date,
           FIRST_VALUE(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS plan_start_date
    FROM foodie_fi.subscriptions
),
counts AS (
    SELECT (WIDTH_BUCKET(final_date - plan_start_date, 0, 365, 12)) AS bucket,
           COUNT(plan_id) AS pro_annual_count
    FROM start_plan_cte
    WHERE plan_id = 3
    GROUP BY bucket
    ORDER BY bucket
)
SELECT CONCAT(
           (bucket - 1) * 30 + 1, ' - ', bucket * 30, ' days'
       ) AS bucket_range,
       pro_annual_count
FROM counts;
````

**Answer:**
|bucket_range |	pro_annual_count |
| ----------- | ---------------- |
|1 - 30 days|	49 |
|31 - 60 days|	24 |
|61 - 90 days|	35 |
|91 - 120 days|	35 |
|121 - 150 days|	43|
|151 - 180 days|	37|
|181 - 210 days|	24|
|211 - 240 days|	4|
|241 - 270 days|	4|
|271 - 300 days|	1|
|301 - 330 days|	1|
|331 - 360 days|	1|

- We see the higuest upgrade to pro annual was its higuest the first thirty days. As days passed only one one person bothered to upgrade - customers have not seen appeal to the pro annual account.

***

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

````sql
WITH movements_cte AS (
	SELECT customer_id,
		subscriptions.plan_id previous_plan,
		LEAD(subscriptions.plan_id) OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date) next_plan,
		LEAD(subscriptions.start_date) OVER (PARTITION BY subscriptions.customer_id ORDER BY subscriptions.start_date DESC) next_date
FROM foodie_fi.subscriptions subscriptions
	
)
SELECT COUNT(DISTINCT customer_id) downgrade_pro_monthly_basic
FROM movements_cte
WHERE previous_plan = 2
	AND next_plan = 1
AND EXTRACT(YEAR FROM next_date) = 2020;
````

**Answer:**
| downgrade_pro_monthly_basic | 
| --------------------------- |
| 0 |

- No customer has downgraded from pro monthly to basic monthly, which is good! No customer has been dissastified with the upgrade!

</details>

***

## C. Challenge Payment Question
[![Go back to Tabe of Contents](https://img.shields.io/badge/View-Main_Folder-971901?)](#-table-of-contents)

The Foodie-Fi team wants you to create a new `payments` table for the year 2020 that includes amounts paid by each customer in the `subscriptions` table with the following requirements:

- monthly payments always occur on the same day of month as the original `start_date` of any monthly paid plan.
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately.
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period.
- once a customer churns they will no longer make payments.

<details>

````sql

````

**Answer:**


***


</details>

## D. Outside The Box Questions
[![Go back to Tabe of Contents](https://img.shields.io/badge/View-Main_Folder-971901?)](#-table-of-contents)

The following are open ended questions which might be asked during a technical interview for this case study - there are no right or wrong answers, but answers that make sense from both a technical and a business perspective make an amazing impression!

<details>

### 1. How would you calculate the rate of growth for Foodie-Fi?

````sql

````

**Answer:**


### 2. What key metrics would you recommend Foodie-Fi management to track over time to assess performance of their overall business?

````sql

````

**Answer:**


### 3. What are some key customer journeys or experiences that you would analyse further to improve customer retention?

````sql

````

**Answer:**


### 4. If the Foodie-Fi team were to create an exit survey shown to customers who wish to cancel their subscription, what questions would you include in the survey?

````sql

````

**Answer:**


### 5. What business levers could the Foodie-Fi team use to reduce the customer churn rate? How would you validate the effectiveness of your ideas?

````sql

````

**Answer:**


</details>

