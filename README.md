# First-Touch/Last-Touch Attribution Analysis
## Table of Contents
- [Project Overview](#project-overview)
  
- [Data Sources](#data-sources)
  
- [Tools](#tools)
  
- [Exploratory Data Analysis](#exploratory-data-analysis)
  
- [Data Analysis](#data-analysis)
  
- [Results and Recommendations](#results-and-recommendations)

## Project Overview
### This project analyzes first and last touch attribution on a T-Shirt website. Namely, we analyze which marketing campaign is responsible for customers' successful  purchases. This dataset will help us to better understand our online marketing campaigns and make good decisions about future campaigns for our imaginary company Cool T-Shirts. First-touch attribution refers to the source that leads a user to a website page for the first time. For example, an ad on Facebook leads a user to discover the Cool T-Shirts homepage. Last-touch attribution is the last click that led the user to make a purchase. For example, a follow-up email campaign leads to user to the "Shop" page where they make the final purchase. 

## Data Sources
Attribution Analysis Data: The primary dataset used for this analysis is the "attribution_data.csv" file, containing detailed information about first and last touch attribution data per customer.

## Tools
- SQL Server - Data Analysis

## Exploratory Data Analysis
EDA involved exploring the attribution data to answer key questions, such as:

1. What do we know about the company's current website and campaigns?
  - How many campaigns and sources does CoolTShirts use and how are they related? 
   - What pages are on their website?
  
2. What is the user journey?
   - How many first touches is each campaign responsible for?
   - How many last touches is each campaign responsible for?
   - How many visitors make a purchase?
   - How many last touches on the purchase page is each campaign responsible for?
   - What is the typical user journey?
  
3. How should the business optimize campaign budget?
   - CoolTShirts can re-invest in 5 campaigns. Which should they pick and why?

### Data Analysis

1. Start by understanding the columns in the table and their relationship to one another. 

```sql
FROM page_visits
GROUP BY user_id;
SELECT DISTINCT page_name
FROM page_visits;
```

2. Understand the user journey from first touch to last touch attribution. How many first touches is each campaign responsible for?

```sql
WITH first_touch AS (
    SELECT user_id,
        MIN(timestamp) as first_touch_at
    FROM page_visits
    GROUP BY user_id),
ft_attr AS(
SELECT ft.user_id,
    ft.first_touch_at,
    pv.utm_source,
    pv.utm_campaign
FROM first_touch ft
JOIN page_visits pv
    ON ft.user_id = pv.user_id
    AND ft.first_touch_at = pv.timestamp
)
SELECT ft_attr.utm_source, ft_attr.utm_campaign, COUNT (*)
FROM ft_attr
GROUP BY 1,2
ORDER BY 3 DESC;
```

3. How many last touches is each campaign responsible for?
   
```sql
WITH last_touch AS (
    SELECT user_id,
        MAX(timestamp) as last_touch_at
    FROM page_visits
    GROUP BY user_id),
lt_attr AS(
SELECT lt.user_id,
    lt.last_touch_at,
    pv.utm_source,
    pv.utm_campaign
FROM last_touch lt
JOIN page_visits pv
    ON lt.user_id = pv.user_id
    AND lt.last_touch_at = pv.timestamp
)
SELECT lt_attr.utm_source, lt_attr.utm_campaign, COUNT (*)
FROM lt_attr
GROUP BY 1,2
ORDER BY 3 DESC;
```

4. How many visitors make a purchase?
```sql
SELECT COUNT(DISTINCT(user_id)) AS visitor_purchases
FROM page_visits
WHERE page_name LIKE '4 - purchase';
```

5. How many last touches on the purchase page is each campaign responsible for?

```sql
WITH last_touch AS (
    SELECT user_id,
        MAX(timestamp) as last_touch_at
    FROM page_visits
    WHERE page_name LIKE '4 - purchase'
    GROUP BY user_id),
lt_attr AS(
SELECT lt.user_id,
    lt.last_touch_at,
    pv.utm_source,
    pv.utm_campaign
FROM last_touch lt
JOIN page_visits pv
    ON lt.user_id = pv.user_id
    AND lt.last_touch_at = pv.timestamp
)
SELECT lt_attr.utm_source, lt_attr.utm_campaign, COUNT (*) AS purchases_from_lt
FROM lt_attr
GROUP BY 1,2
ORDER BY 3 DESC;
```

## Results and Recommendations
### Given this analysis, we can make some business recommendations as to the top five campaigns CTS should reinvest in for future marketing.
### Business Recommendations
CTS should invest in the following five campaigns, in this order of importance:

1.  weekly-newsletter
2. retargetting-ad
3. retargetting-campaign
4. paid-search
5. ten-crazy-cool-tshirts-facts


   





