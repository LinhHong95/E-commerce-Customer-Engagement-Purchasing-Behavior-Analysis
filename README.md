# E-commerce-Customer-Engagement-Purchasing-Behavior-Analysis
## Table of Contents
- [Introduction](#introduction)
- [Business Question](#business-question)
- [Tools and Techniques](#tools-and-techniques)
- [Data Description](#data-description)
- [Workflow](#workflow)
- [Summary](#summary)
## Introduction
In this project, I will analyse customer engagement and purchasing behaviour using SQL to gain insights into how users interact with the website, navigate the sales funnel, and what factors drive purchases. These actionable insights will help the business refine targeted campaigns and develop more effective marketing strategies.
## Business Question
The Marketing Manager would like to evaluate how customers responded to the recently implemented marketing campaigns. This analysis should help the Marketing Manager assess the effectiveness of the marketing campaigns, refine targeting strategies, and optimise future marketing efforts based on customer behaviour.
## Tools and Techniques
- Tool: SQL (Google BigQuery)
- Techniques
    - Common Table Expressions (CTEs) - Used to break down complex queries for readability and reusability.
    - Data Aggregation & Filtering - Applied to calculate key performance metrics
    - Date Handling
## Data Description
- The dataset is part of **Google Analytics Sample Data**, publicly available on **BigQuery**. It contains **session-level user interactions** from the **Google Merchandise Store**, an actual e-commerce website that sells Google-branded products.
- Source: https://bigquery.cloud.google.com/table/bigquery-public-data:google_analytics_sample.ga_sessions_20170801
## Workflow
### 1. Defining Key Questions & Identifying Relevant Data
#### 1.1. Customer Engagement Questions
- How many visits, page views and transactions do customers make in Q1 2017?
- What percentage of users are first-time visitors versus returning customers?
- Which traffic sources drive the most engaged customers in terms of time spent on site?
- Which traffic sources generate the most customers, and how do their bounce rates compare?
- Do customers who make purchases browse more pages than those who don’t?
#### 1.2. Purchasing Behavior Questions
- How many transactions does the average customer make?
- How much does a customer typically spend per session?
- How do users progress from product view to add-to-cart to purchase?
- Which products are the most popular among customers?
- Which traffic sources generate the most revenue?
#### 1.3. Relevant Data
<img width="726" alt="Image" src="https://github.com/user-attachments/assets/c65c2baf-ea29-4966-844e-85926b54ac4c" />

### 2. Querying & Data Processing
#### 2.1. How many visits, page views and transactions do customers make in Q1 2017?
~~~~sql
SELECT
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
 , COUNT(totals.visits) AS total_visits
 , SUM(totals.pageviews) AS total_pageviews
 , COUNT(totals.transactions) AS total_transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 1;
~~~~
<img width="574" alt="Image" src="https://github.com/user-attachments/assets/11115ceb-1c58-4f9e-b466-d32d212dfdb7" />

#### 2.2. What percentage of users are first-time visitors versus returning customers?
~~~~sql
SELECT
  FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
  , COUNT(totals.newVisits) AS new_visitors
  , ROUND(COUNT(totals.newVisits)/COUNT(totals.visits)*100.00, 2) AS new_visitor_percentage
  , (COUNT(totals.visits) - COUNT(totals.newVisits)) AS returning_visitors
  , ROUND((COUNT(totals.visits) - COUNT(totals.newVisits))/COUNT(totals.visits)*100.00, 2) AS returning_visitor_percentage
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 1;
~~~~
<img width="697" alt="Image" src="https://github.com/user-attachments/assets/1e71aaaa-456b-4337-a8a4-bbfb81bfba0b" />

#### 2.3. Which traffic sources drive the most engaged customers in terms of time spent on site?
~~~~sql
SELECT 
  trafficSource.source AS traffic_source
  , AVG(totals.timeOnSite) AS avg_time_on_site
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
~~~~
<img width="386" alt="Image" src="https://github.com/user-attachments/assets/d49b534d-13ea-41d1-bd50-6a464570c8cf" />

#### 2.4. Which traffic sources generate the most customers, and how do their bounce rates compare?
~~~~sql
SELECT 
  trafficSource.source AS traffic_source
  , COUNT(totals.visits) AS total_visits
  , COUNT(totals.bounces) AS total_bounces
  , ROUND(100.00 * COUNT(totals.bounces) / COUNT(totals.visits), 2) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _table_suffix BETWEEN '0101' AND '0331'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
~~~~
<img width="634" alt="Image" src="https://github.com/user-attachments/assets/7f10096c-52e0-4dcf-aa96-5df7595b458f" />

#### 2.5. Do customers who make purchases browse more pages than those who don’t?
~~~~sql
WITH
purchasers AS(
 SELECT 
   FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
   , FLOOR(SUM(totals.pageviews)/ COUNT(DISTINCT fullVisitorId)) AS avg_pageviews_purchased
 FROM
   `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
   , UNNEST(hits) AS hits
   , UNNEST(hits.product) AS products
 WHERE _table_suffix BETWEEN '0101' AND '0331'
   AND productRevenue IS NOT NULL
   AND totals.transactions >= 1
 GROUP BY 1
),
-- Calculate the average number of page views by non-purchasers
non_purchasers AS(
 SELECT 
   FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
   , FLOOR(SUM(totals.pageviews)/ COUNT(DISTINCT fullVisitorId)) AS avg_pageviews_non_purchased
 FROM
   `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
   , UNNEST(hits) AS hits
   , UNNEST(hits.product) AS products
 WHERE _table_suffix BETWEEN '0101' AND '0331'
   AND productRevenue IS NULL
   AND totals.transactions IS NULL
 GROUP BY 1
)
-- Join 2 tables
SELECT
 purchasers.month
 , avg_pageviews_purchased
 , avg_pageviews_non_purchased
FROM purchasers
FULL JOIN non_purchasers
 USING(month)
ORDER BY 1;
~~~~
<img width="585" alt="Image" src="https://github.com/user-attachments/assets/2f7f0bf6-e86a-4372-91ee-ea642c4a8caf" />

#### 2.6. How many transactions does the average customer make?
~~~~sql
SELECT 
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
 , ROUND(SUM(totals.transactions)/COUNT(DISTINCT fullVisitorId), 2) AS avg_transactions_per_user
FROM
 `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 , UNNEST(hits) AS hits
 , UNNEST(hits.product) AS products
WHERE _table_suffix BETWEEN '0101' AND '0331'
  AND productRevenue IS NOT NULL
  AND totals.transactions >= 1
GROUP BY 1
ORDER BY 1;
~~~~
<img width="327" alt="Image" src="https://github.com/user-attachments/assets/633e0e06-91d6-437f-b118-c469f51ae071" />

#### 2.7. How much does a customer typically spend per session?
~~~~sql
SELECT 
 FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
 , CAST(SUM(productRevenue) / COUNT(totals.visits) AS INT64) AS avg_revenue_per_visit
FROM
 `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 , UNNEST(hits) AS hits
 , UNNEST(hits.product) AS products
WHERE _table_suffix BETWEEN '0101' AND '0331'
  AND productRevenue IS NOT NULL
  AND totals.transactions >= 1
GROUP BY 1
ORDER BY 1;
~~~~
<img width="326" alt="Image" src="https://github.com/user-attachments/assets/12c45d89-4c3e-4d67-9233-c607f6afbb29" />

#### 2.8. How do users progress from product view to add-to-cart to purchase?
~~~~sql
WITH product_data AS(
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month
    , COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN v2ProductName END) AS num_product_view
    , COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN v2ProductName END) AS num_add_to_cart
    , COUNT(CASE WHEN eCommerceAction.action_type = '6' AND productRevenue IS NOT NULL THEN v2ProductName END) AS num_purchase
  FROM
   `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
   , UNNEST(hits) AS hits
   , UNNEST(hits.product) AS products
  WHERE _table_suffix BETWEEN '0101' AND '0331'
    AND eCommerceAction.action_type IN ('2', '3', '6')
  GROUP BY 1
  ORDER BY 1
)
SELECT 
  month
  , num_product_view
  , num_add_to_cart
  , num_purchase
  , ROUND(num_add_to_cart/num_product_view *100, 2) AS add_to_cart_rate
  , ROUND(num_purchase/num_product_view *100, 2) AS purchase_rate
FROM product_data; 
~~~~
<img width="822" alt="Image" src="https://github.com/user-attachments/assets/07510f5f-b4b3-407a-be10-02af6b5443e2" />

#### 2.9. Which products are the most popular among customers?
~~~~sql
SELECT
  v2ProductName AS product
  , SUM(productQuantity) AS unit_sold
  , SUM(productRevenue) AS product_revenue
FROM
 `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 , UNNEST(hits) AS hits
 , UNNEST(hits.product) AS products
WHERE _table_suffix BETWEEN '0101' AND '0331'
  AND productRevenue IS NOT NULL
  AND totals.transactions >= 1
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
~~~~
<img width="509" alt="Image" src="https://github.com/user-attachments/assets/42ab2a83-3161-407b-8834-3b6b35c4fdbc" />

#### 2.10. Which traffic sources generate the most revenue?
~~~~sql
SELECT 
  trafficSource.source AS traffic_source
  , SUM(productRevenue) AS revenue_per_traffic_source
FROM
 `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
 , UNNEST(hits) AS hits
 , UNNEST(hits.product) AS products
WHERE _table_suffix BETWEEN '0101' AND '0331'
  AND productRevenue IS NOT NULL
  AND totals.transactions >= 1
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;
~~~~
<img width="385" alt="Image" src="https://github.com/user-attachments/assets/3d3563db-d38c-40e8-a0c6-9b6cddda1645" />

### 3. Insights & Recommendations
#### 3.1. Insights
- Visitor Trends & Traffic Sources:
  - New Visitors Growth increased 11.58% from January to March 2017 and accounted for ~75% of total visitors, indicating strong acquisition but potential retention gaps.
  - Direct & Google drive the highest number of customers and revenue with low bounce rate, respectively 44.31% and 49.79%. YouTube ranks 3rd in customer acquisition, but with a high bounce rate of 67.87%, suggesting inefficiency.
- Purchasing Behavior
  - Non-purchasers browse 3x more pages than purchasers, indicating interest but a lack of a clear purchasing trigger.
  - Purchase rate increased from 8.31% → 12.64% from January to March. Add-to-cart rate improved from 28.47% → 37.29%, suggesting better engagement.
  - Customers made more transactions in March, indicating seasonal demand or improved marketing efforts.
  - Average spending per session: 50M, highlighting significant revenue potential per engaged visitor.
  - Maze Pen, Water Bottle, and Sunglasses were the most popular items.
#### 3.2. Recommendations
- Execute a deeper YouTube Performance Review by analysing ad formats, audience engagement, and bounce rate solutions.
- Perform A/B Testing to test Trigger Conversions for Non-Purchasers by offering limited-time promotions or personalized product recommendations and improving call-to-action strategies on product pages. Investigating reasons for the March transaction spike (seasonality, campaigns, pricing changes) would help.
- Implement email marketing, retargeting ads to convert new visitors into customers.
- Conduct a deeper analysis of Customer Segmentation to identify high-value customers and personalise marketing efforts.
## Summary
This project analyses customer spending and behavior by writing SQL queries to track session engagement, traffic sources, and conversion rates. It compares purchasers vs. non-purchasers in terms of page views and transactions. It also examine which products are the most popular and identify the traffic sources that generate the most revenue. These insights guide retention strategies, conversion optimisation, and ad campaign improvements. Recommendations include A/B testing YouTube ads, personalized promotions, and leveraging March buying trends. Further analysis on customer segmentation and bounce tracking will enhance future strategies.
