# Ecommerce-Business-Exploration-SQL-


<img width="3536" height="2185" alt="What_is_E_commerce_and_What_are_its_Applications_2_d2eb0d4402" src="https://github.com/user-attachments/assets/f01c9c79-4372-4ff9-8238-f919cd214dab" />

## :rocket: Project Overview 
- This SQL project aims to query and filter data from a large e-commerce company in 2017 to understand business performance and customer behavior.
- Finding significantly insightful ideas by extracting revenue, total visits, average transaction, etc., to enhance the company's market position.
- Deep investigation into customers' behavior to figure out some features that influence purchase decisions.
- Finding some relation between product categories that the user purchases together
- Implement cohort analysis to see how things go down from pageview to payment.

## :fax: Data Sources 
- Dataset of Google Analytics sessions from 01/08/2016 - 01/08/2017, which recorded every information about each session and stored it in Google Cloud.
- Check out the full dataset here:
  - [Click Here](https://console.cloud.google.com/bigquery?project=bigquery-public-data&p=bigquery-public-data&d=google_analytics_sample&t=ga_sessions_20170801&page=table)
- The dataset is kinda large, but here is an abstract of its substance:

| Field Name                              | Data Type | Description |
|-----------------------------------------|-----------|-------------|
| fullVisitorId                           | STRING    | The unique visitor ID. |
| date                                    | STRING    | The date of the session in YYYYMMDD format. |
| totals                                  | RECORD    | This section contains aggregate values across the session. |
| totals.bounces                          | INTEGER   | Total bounces (for convenience). For a bounced session, the value is 1, otherwise, it is null. |
| totals.hits                             | INTEGER   | Total number of hits within the session. |
| totals.pageviews                        | INTEGER   | Total number of pageviews within the session. |
| totals.visits                           | INTEGER   | The number of sessions (for convenience). This value is 1 for sessions withevents. The value is null if there are no interaction events in the session. |
| totals.transactions                     | INTEGER   | Total number of ecommerce transactions within the session. |
| trafficSource.source                    | STRING    | The source of the traffic source. Could be the name of the search engine, the referring hostname, or a value of the utm_source URL parameter. |
| hits                                    | RECORD    | This row and nested fields are populated for any and all types of hits. |
| hits.eCommerceAction                    | RECORD    | This section contains all of the ecommerce hits that occurred during the session. This is a repeated field and has an entry for each hit that was collected. |
| hits.eCommerceAction.action_type        | STRING    | The action type. Usually applies to all products in a hit, except when `hits.product.isImpression = TRUE` (product impression during the action). |
| hits.product                            | RECORD    | This row and nested fields will be populated for each hit that contains Enhanced Ecommerce PRODUCT data. |
| hits.product.productQuantity            | INTEGER   | The quantity of the product purchased. |
| hits.product.productRevenue             | INTEGER   | The revenue of the product, expressed as the value passed to Analytics multiplied by 10^6 (e.g., 2.40 would be given as 2400000). |
| hits.product.productSKU                 | STRING    | Product SKU. |
| hits.product.v2ProductName              | STRING    | Product Name. |

## :hammer_and_pick: Tools:
- SQL - Google Big Query

## :open_book: Main Process:
***1. Calculate total visits, pageviews, transactions for Jan, Feb, and March 2017 (order by month)***
- Query: 
```sql
WITH m1 AS(SELECT  DISTINCT FORMAT_DATE('%Y%m',(PARSE_DATE('%Y%m%d',date))) month,
                  SUM(totals.visits)  OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) visits,
                  SUM(totals.pageviews) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) pageviews,
                  SUM(totals.transactions) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) transactions
          FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
          WHERE _table_suffix BETWEEN '0101' AND '0131')
          ,
    m2 AS(SELECT  DISTINCT FORMAT_DATE('%Y%m',(PARSE_DATE('%Y%m%d',date))) month,
                  SUM(totals.visits)  OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) visits,
                  SUM(totals.pageviews) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) pageviews,
                  SUM(totals.transactions) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) transactions
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
            WHERE _table_suffix BETWEEN '0201' AND '0231')
,
   m3 AS(SELECT  DISTINCT FORMAT_DATE('%Y%m',(PARSE_DATE('%Y%m%d',date))) month,
                SUM(totals.visits)  OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) visits,
                SUM(totals.pageviews) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) pageviews,
                SUM(totals.transactions) OVER(PARTITION BY EXTRACT(MONTH FROM (PARSE_DATE('%Y%m%d',date)))) transactions
        FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
        WHERE _table_suffix BETWEEN '0301' AND '0331') 

SELECT *
FROM m1
UNION ALL 
SELECT *
FROM m2
UNION ALL 
SELECT *
FROM m3
```
- Output:

| Row | Month   | Visits  | Pageviews | Transactions |
|-----|---------|---------|-----------|--------------|
| 1   | 201703  | 69,931  | 259,522   | 993          |
| 2   | 201702  | 62,192  | 233,373   | 733          |
| 3   | 201701  | 64,694  | 257,708   | 713          |

- Both Visits, Pageviews, and Transactions in March/2017 skyrocketed.
- Transactions in 01/2017 was too low, even though Visits and Pageviews were higher than 02/2017

***2. Bounce rate per traffic source in July 2017 (Bounce_rate = num_bounce/total_visit) (order by total_visit DESC)***
- Query:
```sql
SELECT DISTINCT trafficSource.source, 
       SUM(totals.visits) total_visits,
       SUM(totals.bounces) total_no_of_bounces,
       ROUND(SUM(totals.bounces)*100/SUM(totals.visits),3) AS bounce_rate 

FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
WHERE _table_suffix BETWEEN '0701' AND '0731'
GROUP BY trafficSource.source
ORDER BY total_visits DESC
```
- Output:

| Row | Source                  | Total Visits | Total Bounces | Bounce Rate (%) |
|-----|-------------------------|--------------|---------------|-----------------|
| 1   | google                  | 38,400       | 19,798        | 51.557          |
| 2   | (direct)                | 19,891       | 8,606         | 43.266          |
| 3   | youtube.com             | 6,351        | 4,238         | 66.730          |
| 4   | analytics.google.com    | 1,972        | 1,064         | 53.955          |
| 5   | Partners                | 1,788        | 936           | 52.349          |

- For these top 5 most visited pages, YouTube got the highest bounce rate.

***3. Revenue by traffic source by week, by month in June 2017***
- Query:
```sql
WITH wek AS( SELECT DISTINCT FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d',date)) time,
              trafficSource.source,
              ROUND(SUM(product.productRevenue/1000000) OVER(PARTITION BY trafficSource.source,FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d',date))),2) revenue
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
            UNNEST (hits) hits,
            UNNEST (hits.product) product
            WHERE product.productRevenue IS NOT NULL
),
     mon AS( SELECT DISTINCT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) time,
                   trafficSource.source,
                   ROUND(SUM(product.productRevenue/1000000) OVER(PARTITION BY trafficSource.source,FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))),2) revenue
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
            UNNEST (hits) hits,
            UNNEST (hits.product) product
            WHERE product.productRevenue IS NOT NULL
)

SELECT 'Week'AS time_type, *
FROM wek
UNION ALL 
SELECT 'Month'AS time_type, *
FROM mon
ORDER BY revenue DESC 
```
- Output:

| Row | Time Type | Time   | Source   | Revenue     |
|-----|-----------|--------|----------|-------------|
| 1   | Month     | 201706 | (direct) | 97,333.62   |
| 2   | Week      | 201724 | (direct) | 30,908.91   |
| 3   | Week      | 201725 | (direct) | 27,295.32   |
| 4   | Month     | 201706 | google   | 18,757.18   |
| 5   | Week      | 201723 | (direct) | 17,325.68   |

***4. Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.***
- Query:
```sql
WITH jun AS(
    SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) month,
          (SELECT SUM(totals.pageviews)/
                  COUNT(DISTINCT fullVisitorId)
           FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
           UNNEST (hits) hits,
           UNNEST (hits.product) product
           WHERE _table_suffix BETWEEN '0601' AND '0630'
           AND product.Productrevenue IS NOT NULL 
           AND totals.transactions >= 1
                               ) AS avg_pageviews_purchase,
          (SELECT SUM(totals.pageviews) / 
                  COUNT(DISTINCT fullVisitorId)
           FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
           UNNEST (hits) hits,
           UNNEST (hits.product) product
           WHERE _table_suffix BETWEEN '0601' AND '0630'
           AND product.Productrevenue IS NULL
           AND totals.transactions IS NULL
                              ) AS avg_pageviews_non_purchase                            
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
      WHERE _table_suffix BETWEEN '0601' AND '0630'
      GROUP BY FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))
  ),
    july AS(
    SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date)) month,
          (SELECT SUM(totals.pageviews)/
                  COUNT(DISTINCT fullVisitorId)
           FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
           UNNEST (hits) hits,
           UNNEST (hits.product) product
           WHERE _table_suffix BETWEEN '0701' AND '0731'
           AND product.Productrevenue IS NOT NULL 
           AND totals.transactions >= 1
                               ) AS avg_pageviews_purchase,
          (SELECT SUM(totals.pageviews) / 
                  COUNT(DISTINCT fullVisitorId)
           FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
           UNNEST (hits) hits,
           UNNEST (hits.product) product
           WHERE _table_suffix BETWEEN '0701' AND '0731'
           AND product.Productrevenue IS NULL
           AND totals.transactions IS NULL
                              ) AS avg_pageviews_non_purchase          
            FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*` 
            WHERE _table_suffix BETWEEN '0701' AND '0731'
            GROUP BY FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d',date))
  )
SELECT *
FROM jun
UNION ALL
SELECT *
FROM july;  
```
- Output:

| month   | avg_pageviews_purchase | avg_pageviews_non_purchase |
|---------|------------------------|----------------------------|
| 201706  | 94.02                  | 316.87                     |
| 201707  | 124.24                 | 334.06                     |

- The average pageviews of non-purchases is x3 that of the purchase, it might because some users just want to check for the product, price or compare price to other stores.
- Sugest investigate more on a specific page or products
***5. Average number of transactions per user that made a purchase in July 2017***


