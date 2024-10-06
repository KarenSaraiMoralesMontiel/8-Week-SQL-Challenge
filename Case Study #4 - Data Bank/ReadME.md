# 💰 Case Study #4: Data Bank
[![View Main Folder](https://img.shields.io/badge/View-Main_Folder-971901?)](https://github.com/KarenSaraiMoralesMontiel/8-Week-SQL-Challenge/tree/main)
[![View My Portfolio](https://img.shields.io/badge/View-My_Profile-green?logo=GitHub)](https://github.com/KarenSaraiMoralesMontiel/Portfolio)

<img src='https://8weeksqlchallenge.com/images/case-study-designs/4.png' alt="Data Mart Image" width="500" height="520">

***

## 📖 Table of Contents
1. [Bussiness Task](#bussiness-task)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Solutions](#solutions)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-4/).

## Bussiness Task
anny thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

## Entity Relationship Diagram
![Data-Bank ERD](image.png)

## Solutions

## A. Customer Nodes Exploration

### 1. How many unique nodes are there on the Data Bank system?

````sql

````

**Answer:**

### 2. What is the number of nodes per region?

````sql

````

**Answer:**


### 3. How many customers are allocated to each region?

````sql

````

**Answer:**


### 4. How many days on average are customers reallocated to a different node?

````sql

````

**Answer:**


### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

````sql

````

**Answer:**

***

## B. Customer Transactions

### 1. What is the unique count and total amount for each transaction type?

````sql

````

**Answer:**


### 2. What is the average total historical deposit counts and amounts for all customers?

````sql

````

**Answer:**


### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

````sql

````

**Answer:**


### 4. What is the closing balance for each customer at the end of the month?
````sql

````

**Answer:**


### 5. What is the percentage of customers who increase their closing balance by more than 5%?

````sql

````

**Answer:**



***

## C. Data Allocation Challenge

To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

- Option 1: data is allocated based off the amount of money at the end of the previous month
- Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
- Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction
- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer

Using all of the data available - how much data would have been required for each option on a monthly basis?

### D. Extra Challenge

Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

    Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!
