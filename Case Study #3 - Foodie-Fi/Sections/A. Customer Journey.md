# A. Customer Journey

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