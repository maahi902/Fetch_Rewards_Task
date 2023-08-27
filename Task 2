## Task II: SQL Queries

1. What are the top 5 brands by receipts scanned for the most recent month?

```
SELECT b.name AS brand_name, COUNT(*) AS receipts_scanned
FROM Brands b
JOIN Receipts r ON b._id = r.rewardsReceiptItemList.brandId
WHERE EXTRACT(MONTH FROM r.dateScanned) = EXTRACT(MONTH FROM CURRENT_DATE)
GROUP BY b.name
ORDER BY receipts_scanned DESC
LIMIT 5;
```

2. How does the ranking of the top 5 brands by receipts scanned for the recent month compare to the ranking for the previous month?

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
