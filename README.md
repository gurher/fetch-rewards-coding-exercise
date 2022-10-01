# Fetch Rewards Coding Exercise - Data Analyst

## First: Review Existing Unstructured Data and Diagram a New Structured Relational Data Model

![image](https://user-images.githubusercontent.com/78076900/193401344-536aa13d-c0fa-4d33-8224-00e888eba7d0.png)

Every different uuid of receipts contained multiple items contained in the list in the rewardsReceiptItemList column. I created an extra table itemsList that contains values from rewardsReceiptItemList in order to create an efficient relational database. Id column in itemsList is a foreign key from receipts table and formed a 1:m relationship. ItemsList table should also contain brand_id foregin key from brands table. This will help verify what unique items of brand were purchased and recorded in the receipts. Last but not least, users table should contain unique informations of the users and could be linkedin with user_id in receipts table to find which users made the purchase.

## Second: Write a query that directly answers a predetermined question from a business stakeholder
Write a SQL query against your new structured relational data model that answers one of the following bullet points below of your choosing. Commit it to the git repository along with the rest of the exercise.

- What are the top 5 brands by receipts scanned for most recent month?

```
SELECT 
        itemlist.description,
        COUNT(*) AS "Number of Brands"
FROM itemlist
WHERE itemlist._id IN 
                    (                    
                    SELECT DISTINCT _id AS "receipts_id"
                    FROM receipts
                    WHERE strftime("%m", dateScanned) =  (SELECT STRFTIME("%m", MAX(dateScanned))
                                                            FROM receipts )
                        AND strftime("%Y", dateScanned) = (SELECT STRFTIME("%Y",MAX(dateScanned))
                                                            FROM receipts)                                                                 
                    )
    AND itemlist.description IS NOT NULL
GROUP BY itemlist.description
ORDER BY 2  
LIMIT 5
```
- How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?
```
SELECT 
        IFNULL(itemlist.brandCode,'NULL'),
        COUNT(*) AS 'Number of Brands'
FROM itemlist
WHERE itemlist._id IN 
                    (                    
                    SELECT DISTINCT _id AS "receipts_id"
                            
                    FROM receipts
                    WHERE strftime("%m", dateScanned) =  (SELECT STRFTIME("%m", DATE(MAX(dateScanned),'-1 months'))
                                                            FROM receipts )
                        AND strftime("%Y", dateScanned) = (SELECT STRFTIME("%Y",DATE(MAX(dateScanned),'-1 months'))
                                                            FROM receipts)                                                                
                    )
    
GROUP BY itemlist.brandCode
ORDER BY 2 DESC    
LIMIT 5    
```
- When considering average spend from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?
```
SELECT 
    receipts.rewardsReceiptStatus,
    AVG(receipts.totalSpent) AS 'Average Spent'    
    
FROM receipts
WHERE receipts.totalSpent IS NOT NULL
GROUP BY receipts.rewardsReceiptStatus
```
- When considering total number of items purchased from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?
```
SELECT 
    receipts.rewardsReceiptStatus,
    SUM(receipts.purchasedItemCount) AS 'Total Items Count'  
    
FROM receipts
WHERE receipts.purchasedItemCount IS NOT NULL
GROUP BY receipts.rewardsReceiptStatus
```
- Which brand has the most spend among users who were created within the past 6 months?
```
WITH users_6months_id AS (
SELECT DISTINCT _id    
FROM users
WHERE createdDate >= ( SELECT DATE(MAX(createdDate),'start of month','-6 months') 
                      FROM users )
)

SELECT 
    itemlist.brandCode,
    SUM(receipts.totalSpent) AS 'totalspent'
    
FROM receipts
INNER JOIN users_6months_id
    ON receipts.userId = users_6months_id._id
INNER JOIN itemlist
    ON receipts._id = itemlist._id 
WHERE itemlist.brandCode IS NOT NULL
GROUP BY itemlist.brandCode
ORDER BY 2 DESC
LIMIT 1 
```
- Which brand has the most transactions among users who were created within the past 6 months?
```
WITH users_6months_id AS (
SELECT DISTINCT _id    
FROM users
WHERE createdDate >= ( SELECT DATE(MAX(createdDate),'start of month','-6 months') 
                      FROM users )
)

SELECT 
    itemlist.brandCode,
    COUNT(*) AS 'Transactions Count'
    
FROM receipts
INNER JOIN users_6months_id
    ON receipts.userId = users_6months_id._id
INNER JOIN itemlist
    ON receipts._id = itemlist._id 
WHERE itemlist.brandCode IS NOT NULL
GROUP BY itemlist.brandCode
ORDER BY 2 DESC
LIMIT 1 
```

## Third: Evaluate Data Quality Issues in the Data Provided
Using the programming language of your choice (SQL, Python, R, Bash, etc...) identify at least one data quality issue. We are not expecting a full blown review of all the data provided, but instead want to know how you explore and evaluate data of questionable provenance.



## Fourth: Communicate with Stakeholders
Hi,

As I was reviewing different sources of data for analytic purposes, it came to my attention that there are a very small number of brand types that could be used to find business insights.

My objective was to find the top brands purchased by users. There are two sources of data tables which contain tables of brands. One is from users' purchased receipts data table and another is from a data table which only contains brand information. 

However, there were too many test values in a brand table and I had to look at the receipts data table to find valid brand names. However, the receipts table also had an issue with a large number of NULL values which indicates empty brand name values. Therefore, it was hard to see what brands were most frequently or least purchased from users' receipts data.  

In order to fully understand and resolve this issue, I would like to know the method implemented by the company to obtain the data. From my experience, there are usually two ways to acquire this kind of data. One is a scraping method which refers to the process of using bots to extract data, while the API method provides direct access to the data. My assumption is that we are currently using scraping methods as there are way too many empty values in the data, but I would like to make sure that I am on the right page with the method used. If I am correct, we will have to improve our scraping skills to upgrade the quality of our data. Another possible suggestion is to manually create every possible brand and items of each brand to improve the quality of the data item. By doing so, it will be easier to identify top brands and items purchased by analyzing the receipts. I would really appreciate your clearance on this matter and hope this could result in improvement of the data quality.

Best,
<br>
Edward Her
