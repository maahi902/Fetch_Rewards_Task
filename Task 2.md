# Task II: SQL Queries

## 1. What are the top 5 brands by receipts scanned for the most recent month?

```
SELECT b.name AS brand_name, COUNT(*) AS receipts_scanned
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
WHERE EXTRACT(MONTH FROM r.dateScanned) = EXTRACT(MONTH FROM CURRENT_DATE)
GROUP BY b.name
ORDER BY receipts_scanned DESC
LIMIT 5;
```

## 2. How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?

- STEP 1: Retrieve Data from the Recent Month
```
SELECT b.name AS brand_name, COUNT(*) AS receipts_scanned
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
WHERE EXTRACT(MONTH FROM r.dateScanned) = EXTRACT(MONTH FROM CURRENT_DATE)
GROUP BY b.name
ORDER BY receipts_scanned DESC
LIMIT 5;
```
- STEP 2: Retrieve Data for the Previous Month
```
SELECT b.name AS brand_name, COUNT(*) AS receipts_scanned
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
WHERE EXTRACT(MONTH FROM r.dateScanned) = EXTRACT(MONTH FROM CURRENT_DATE - INTERVAL '1' MONTH)
GROUP BY b.name
ORDER BY receipts_scanned DESC
LIMIT 5;
```
- STEP 3: Compare Rankings
```
import pandas as pd
# Execute the SQL queries and store results in dataframes
recent_month_df = pd.read_sql_query(recent_month_query, your_database_connection)
previous_month_df = pd.read_sql_query(previous_month_query, your_database_connection)

# Merge dataframes on brand_name to compare rankings
merged_df = recent_month_df.merge(previous_month_df, on='brand_name', how='outer', suffixes=('_recent', '_previous'))

# Compare rankings and identify changes
for index, row in merged_df.iterrows():
    brand_name = row['brand_name']
    rank_recent = row['rank_recent']
    rank_previous = row['rank_previous']
    
    if pd.notna(rank_recent) and pd.notna(rank_previous):
        if rank_recent < rank_previous:
            print(f"{brand_name} improved its ranking from {rank_previous} to {rank_recent}.")
        elif rank_recent > rank_previous:
            print(f"{brand_name} dropped its ranking from {rank_previous} to {rank_recent}.")
        else:
            print(f"{brand_name} maintained its ranking at {rank_recent}.")
    elif pd.notna(rank_recent):
        print(f"{brand_name} entered the top 5 in the recent month with a ranking of {rank_recent}.")
    else:
        print(f"{brand_name} dropped out of the top 5 in the recent month.")
```

## 3. When considering average spend from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?
   
- STEP 1: Calculate Average Spend for 'Accepted' Receipts

```
SELECT AVG(totalSpent) AS avg_spend_accepted
FROM Receipts
WHERE rewardsReceiptStatus = 'Accepted';
```

- STEP 2: Calculate Average Spend for 'Rejected' Receipts
```
SELECT AVG(totalSpent) AS avg_spend_rejected
FROM Receipts
WHERE rewardsReceiptStatus = 'Rejected';
```

- STEP 3: Compare Average Spend
```
SELECT
    CASE
        WHEN AVG(CASE WHEN rewardsReceiptStatus = 'Accepted' THEN totalSpent ELSE 0 END) >
             AVG(CASE WHEN rewardsReceiptStatus = 'Rejected' THEN totalSpent ELSE 0 END)
        THEN 'Average spend from ''Accepted'' receipts is greater.'
        WHEN AVG(CASE WHEN rewardsReceiptStatus = 'Accepted' THEN totalSpent ELSE 0 END) <
             AVG(CASE WHEN rewardsReceiptStatus = 'Rejected' THEN totalSpent ELSE 0 END)
        THEN 'Average spend from ''Rejected'' receipts is greater.'
        ELSE 'Average spend from ''Accepted'' and ''Rejected'' receipts is equal.'
    END AS comparison_result
FROM Receipts
WHERE rewardsReceiptStatus IN ('Accepted', 'Rejected');
```
## 4. When considering total number of items purchased from receipts with 'rewardsReceiptStatus’ of ‘Accepted’ or ‘Rejected’, which is greater?

- STEP 1: Calculate Total Number of Items for 'Accepted' Receipts
```
SELECT SUM(purchasedItemCount) AS total_items_accepted
FROM Receipts
WHERE rewardsReceiptStatus = 'Accepted';
```

- STEP 2: Calculate Total Number of Items for 'Rejected' Receipts
```
SELECT SUM(purchasedItemCount) AS total_items_rejected
FROM Receipts
WHERE rewardsReceiptStatus = 'Rejected';
```

- STEP 3: Compare Total Number of Items
```
SELECT
    CASE
        WHEN SUM(CASE WHEN rewardsReceiptStatus = 'Accepted' THEN purchasedItemCount ELSE 0 END) >
             SUM(CASE WHEN rewardsReceiptStatus = 'Rejected' THEN purchasedItemCount ELSE 0 END)
        THEN 'Total number of items from ''Accepted'' receipts is greater.'
        WHEN SUM(CASE WHEN rewardsReceiptStatus = 'Accepted' THEN purchasedItemCount ELSE 0 END) <
             SUM(CASE WHEN rewardsReceiptStatus = 'Rejected' THEN purchasedItemCount ELSE 0 END)
        THEN 'Total number of items from ''Rejected'' receipts is greater.'
        ELSE 'Total number of items from ''Accepted'' and ''Rejected'' receipts is equal.'
    END AS comparison_result
FROM Receipts
WHERE rewardsReceiptStatus IN ('Accepted', 'Rejected');
```

## 5. Which brand has the most spend among users who were created within the past 6 months?

- STEP 1: Identify Users Created Within the Past 6 Months
```
SELECT DISTINCT userId
FROM Users
WHERE createdDate >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH);
```
- STEP 2: Calculate Total Spend for Each Brand
```
SELECT
    b.name AS brand_name,
    SUM(r.totalSpent) AS total_spend
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
JOIN Users u ON r.userId = u._id
WHERE u.createdDate >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
GROUP BY b.name
ORDER BY total_spend DESC
LIMIT 1;
```

###6. Which brand has the most transactions among users who were created within the past 6 months?

-STEP 1: Identify Users Created Within the Past 6 Months

```
SELECT DISTINCT userId
FROM Users
WHERE createdDate >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH);
```

-STEP 2: Calculate Total Transactions for Each Brand

```
SELECT
    b.name AS brand_name,
    COUNT(DISTINCT r._id) AS total_transactions
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
JOIN Users u ON r.userId = u._id
WHERE u.createdDate >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
GROUP BY b.name
ORDER BY total_transactions DESC
LIMIT 1;
```
