Fetch Exercise:

Project Overview
This repository contains the solution for the Fetch Exercise, which involves querying data from a set of related tables to extract meaningful information. The project focuses on querying brand data from a set of receipts, ranking brands by the number of receipts, and comparing monthly statistics. The key objective is to write optimized  queries to solve specific business requirements related to brand performance analysis.

Key Requirements
Ranking the Top Brands by Receipts Scanned for the Recent Month

Find the top 5 brands by the number of receipts scanned in the most recent month.
Comparing Top Brands Between Current and Previous Month

Rank the top brands by receipts scanned for the most recent month and compare their rankings to the previous month.
Most Spend by Users Created in the Past 6 Months

Identify which brand has the highest spend among users who were created in the past 6 months.
Monthly Comparison of Brand Rankings

Compare the ranking of top brands by receipts scanned for the most recent month with the previous month.
Database Schema
The project uses the following schema:

Users: Contains information about users.
_id (PK)
role
createdDate
lastLogin
active

Receipts: Contains information about receipts scanned by users.
_id (PK)
userId (FK)
bonusPointsEarned
totalSpent
purchaseDate
dateScanned
RewardsReceiptItems: Links receipts to specific brands.
receiptId (FK)
barcode
partnerItemId
brandId (FK)
finalPrice
quantityPurchased
Brands: Contains information about brands.
_id (PK)
name
category

Queries Implemented
1. Ranking of Top 5 Brands by Receipts Scanned for the Most Recent Month



WITH ranked_brands AS (
    SELECT b.name,
           COUNT(r._id) AS recp_count,
           RANK() OVER (ORDER BY COUNT(r._id) DESC) AS rnk,
           EXTRACT(MONTH FROM TO_TIMESTAMP(r.purchaseDate / 1000)) AS purchase_month,
           EXTRACT(YEAR FROM TO_TIMESTAMP(r.purchaseDate / 1000)) AS purchase_year
    FROM brands b
    LEFT OUTER JOIN rewardsreceiptitems ri ON ri.brandId = b._id
    LEFT OUTER JOIN receipts r ON r._id = ri.receiptId
    WHERE r.purchaseDate >= DATEADD(MONTH, -1, CURRENT_DATE()) -- Filter for the last month
    GROUP BY b.name
)
SELECT rb.name,
       rb.recp_count,
       rb.rnk,
       CASE 
           WHEN rb.purchase_month = EXTRACT(MONTH FROM CURRENT_DATE()) 
                AND rb.purchase_year = EXTRACT(YEAR FROM CURRENT_DATE()) THEN 'This Month'
           WHEN rb.purchase_month = EXTRACT(MONTH FROM DATEADD(MONTH, -1, CURRENT_DATE())) 
                AND rb.purchase_year = EXTRACT(YEAR FROM DATEADD(MONTH, -1, CURRENT_DATE())) THEN 'Previous Month'
           ELSE 'Other'
       END AS month_comparison
FROM ranked_brands rb
WHERE rb.rnk <= 5
ORDER BY rb.rnk;

2. Brand with the Most Spend Among Users Created in the Past 6 Months
SELECT b.name,
       SUM(r.totalSpent) AS total_spent
FROM brands b
LEFT OUTER JOIN rewardsreceiptitems ri ON ri.brandId = b._id
LEFT OUTER JOIN receipts r ON r._id = ri.receiptId
LEFT OUTER JOIN users u ON u._id = r.userId
WHERE u.createdDate >= DATEADD(MONTH, -6, CURRENT_DATE())
GROUP BY b.name
ORDER BY total_spent DESC
LIMIT 1;

3. Monthly Comparison of Top Brands by Receipts Scanned?

WITH top_brand_rank AS (
    SELECT b.name,
           COUNT(r._id) AS recp_count,
           RANK() OVER (ORDER BY COUNT(r._id) DESC) AS rnk,
           EXTRACT(MONTH FROM TO_TIMESTAMP(r.purchaseDate / 1000)) AS purchase_month,
           EXTRACT(YEAR FROM TO_TIMESTAMP(r.purchaseDate / 1000)) AS purchase_year
    FROM brands b
    LEFT OUTER JOIN rewardsreceiptitems ri ON ri.brandId = b._id
    LEFT OUTER JOIN receipts r ON r._id = ri.receiptId
    GROUP BY b.name
)
SELECT tb.name,
       tb.recp_count,
       tb.rnk,
       CASE 
           WHEN tb.purchase_month = EXTRACT(MONTH FROM CURRENT_DATE()) 
                AND tb.purchase_year = EXTRACT(YEAR FROM CURRENT_DATE()) THEN 'Current_Month'
           WHEN tb.purchase_month = EXTRACT(MONTH FROM DATEADD(MONTH, -1, CURRENT_DATE())) 
                AND tb.purchase_year = EXTRACT(YEAR FROM DATEADD(MONTH, -1, CURRENT_DATE())) THEN 'Previous_Month'
           ELSE 'Other_Month'
       END AS comparisons
FROM top_brand_rank tb
WHERE tb.rnk <= 5
ORDER BY tb.rnk;


Setup
Clone the repository:


git clone https://github.com/BhavaniMakkena/Fetch-Exercise.git
Navigate to the project directory:
cd Fetch-Exercise

Run the  queries against the appropriate database to retrieve the required results.

Technologies Used
SQL for data querying and analysis
Snowflake for data processing and query execution
GitHub for version control and project management