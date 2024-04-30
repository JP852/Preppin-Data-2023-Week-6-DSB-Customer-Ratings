# Preppin-Data-2023-Week-6-DSB-Customer-Ranking

![image](https://github.com/JP852/Preppin-Data-2023-Week-6-DSB-Customer-Ratings/assets/142391590/088f02cb-2951-4294-95d1-6c739b952733)


## Link to Challenge

https://preppindata.blogspot.com/2023/02/2023-week-6-dsb-customer-ratings.html

## SQL Techniques Used

- CTEs
- Aggregation
- Group By
- Where Clause
- Case Statements
- In Function
- Unpivot

## Questions

### Output to Product


Reshape the data so we have 5 rows for each customer, with responses for the Mobile App and Online Interface being in separate fields on the same row.
Clean the question categories so they don't have the platform in from of them.
e.g. Mobile App - Ease of Use should be simply Ease of Use.
Exclude the Overall Ratings, these were incorrectly calculated by the system.
Calculate the Average Ratings for each platform for each customer.
Calculate the difference in Average Rating between Mobile App and Online Interface for each customer.
Catergorise customers as being:
Mobile App Superfans if the difference is greater than or equal to 2 in the Mobile App's favour.
Mobile App Fans if difference >= 1.
Online Interface Fan.
Online Interface Superfan.
Neutral if difference is between 0 and 1.
Calculate the Percent of Total customers in each category, rounded to 1 decimal place.


```
WITH PRE_PIVOT AS(

SELECT 
    CUSTOMER_ID
    ,SPLIT_PART(PIVOT_COLUMN,'___',1) AS DEVICE
    ,SPLIT_PART(PIVOT_COLUMN,'___',2) AS FACTOR 
    ,VALUE    
        FROM(
SELECT 
    * 
    FROM PD2023_WK06_DSB_CUSTOMER_SURVEY) as source
        UNPIVOT(VALUE FOR pivot_column 
        IN (MOBILE_APP___EASE_OF_USE
        ,MOBILE_APP___EASE_OF_ACCESS
        ,MOBILE_APP___NAVIGATION
        ,MOBILE_APP___LIKELIHOOD_TO_RECOMMEND
        ,MOBILE_APP___OVERALL_RATING
        ,ONLINE_INTERFACE___EASE_OF_USE
        ,ONLINE_INTERFACE___EASE_OF_ACCESS
        ,ONLINE_INTERFACE___NAVIGATION
        ,ONLINE_INTERFACE___LIKELIHOOD_TO_RECOMMEND
        ,ONLINE_INTERFACE___OVERALL_RATING))
            AS PIVOT)
    ,FORMATED_DATA AS(
SELECT 
    * 
    FROM PRE_PIVOT PIVOT (SUM(VALUE) FOR DEVICE 
    IN ('MOBILE_APP','ONLINE_INTERFACE')) AS P
        WHERE FACTOR != 'OVERALL_RATING')
    ,CATEGORIES AS (
SELECT 
    CUSTOMER_ID
    ,AVG("'MOBILE_APP'") AS AVG_MOBILE
    ,AVG("'ONLINE_INTERFACE'") AS AVG_ONLINE
    ,AVG("'MOBILE_APP'") - AVG("'ONLINE_INTERFACE'") AS DIFF_IN_RATINGS
    ,CASE 
    WHEN AVG("'MOBILE_APP'") - AVG("'ONLINE_INTERFACE'") >= 2 THEN 'Mobile App Superfans'
    WHEN AVG("'MOBILE_APP'") - AVG("'ONLINE_INTERFACE'") >= 1 THEN 'Mobile App Fans'
    WHEN AVG("'MOBILE_APP'") - AVG("'ONLINE_INTERFACE'") <= -2 THEN 'Online Interface Superfans'
    WHEN AVG("'MOBILE_APP'") - AVG("'ONLINE_INTERFACE'") <= -1 THEN 'Online Interface Fans'
    ELSE 'Neutral'
        END as fan_category
            FROM FORMATED_DATA
               GROUP BY CUSTOMER_ID)
SELECT 
    FAN_CATEGORY
    ,ROUND((COUNT(CUSTOMER_ID) / (SELECT COUNT(CUSTOMER_ID) 
        FROM CATEGORIES))*100,1) AS PERCENT_OF_CUSTOMERS
            FROM CATEGORIES
               GROUP BY FAN_CATEGORY

```
