# Cloud-Hosted Housing Market Analytics (GCP & Power BI)

### Dashboard Link : https://app.powerbi.com/groups/82a62ebe-b171-4c92-830f-97b126645c22/list?experience=power-bi

## Problem Statement

This project focuses on cloud data staging and advanced time-intelligence financial reporting. The architecture involved staging a large-scale housing dataset in Google Cloud Platform (BigQuery) for initial structural corrections, connecting it directly to Power BI, and executing statistical data imputation alongside complex DAX modeling to analyze real estate market trends, inflation impacts, and year-over-year growth.

A significant challenge in enterprise reporting is ensuring data quality and managing complex temporal calculations without degrading downstream performance. By utilizing Google BigQuery as the primary cloud data warehouse, this pipeline pushes heavy structural transformations back to the source server. Inside Power BI, the model leverages optimized DAX variables and iterator functions to dynamically calculate outlier-resistant metrics (such as median price changes) and rolling financial periods, providing stakeholders with a highly accurate, AI-enhanced view of regional real estate valuation drivers.

### Repository Assets

- `Dataset/`: `Housing Data.csv` (Raw housing market dataset).
- `Cloud_SQL/`: `BigQuery_Staging.sql` (Cloud data staging and conditional UPDATE statements).
- `PowerQuery/`: `Data_Imputation_Steps.m` (M-code detailing null handling and statistical mode imputation).
- `Report/`: `Housing_Market_Analytics.pbix` (The final interactive BI dashboard).

### Steps followed

- Step 1 : **Cloud Data Staging (Google BigQuery):** Loaded the raw housing dataset into Google BigQuery to establish a scalable cloud data warehouse[cite: 9]. To protect production data integrity, a duplicate staging table (`poetic-abacus-510912-i4.1.Test`) was created[cite: 9].
- Step 2 : Executed conditional structural updates directly within the BigQuery SQL interface. To resolve dimensional anomalies before they reached the visualization layer, the square meter column was standardized using the following `UPDATE` statement[cite: 9]:
    
    ```sql
    update `poetic-abacus-510912-i4.1.Test`
    set sqm = 100
    where no_rooms = 3;
    ```
    

<img width="1753" height="832" alt="SS1" src="https://github.com/user-attachments/assets/c31cd3fc-8830-42c9-8b23-a250d23dec07" />


- Step 3 : **Statistical Data Imputation (Power Query):** Connected Power BI to the BigQuery environment and utilized Power Query Editor for localized data quality management[cite: 9]. Null values in the categorical `City` column were replaced with an "Unknown" default string to prevent blank slicer categories[cite: 9].
- Step 4 : Addressed missing financial data (<1% nulls in `dk_ann_infl_rate%` and `yield_on_mortgage_credit_bonds%`) using statistical Mode imputation[cite: 9]. By replacing blanks with the most frequently occurring value, the statistical distribution of the dataset was preserved without dropping valuable transaction records[cite: 9].
- Step 5 : Engineered foundational calculated columns for deeper analysis. Reconstructed the historical `Offer Price` using the percentage change delta, and derived property `Age` using the absolute difference between transaction year and construction year[cite: 9]:
    
    ```
    Age = ABS(YEAR('Housing'[date].[Date]) - YEAR('Housing'[year_build]))
    ```
    
- Step 6 : **Optimized Time-Intelligence DAX:** Engineered robust financial metrics using advanced DAX variables (`VAR` / `RETURN`) and error-handling (`IF` / `BLANK`)[cite: 9].

The following measure calculates Year-Over-Year Sales Growth dynamically based on the maximum date context:

```
dax
YOY_Sales_Growth =
var CurrentYearSales = CALCULATE(SUM('Housing'[purchase_price]),YEAR('Housing'[date]) = YEAR(MAX('Housing'[date])))
var PreviousYearSales = CALCULATE(SUM('Housing'[purchase_price]),YEAR('Housing'[date]) = YEAR(MAX('Housing'[date]))-1)
RETURN
IF(PreviousYearSales<>0, (CurrentYearSales - PreviousYearSales)/PreviousYearSales,BLANK())
```


- Step 7 : Deployed iterator functions to calculate accurate pricing shifts. Because real estate data contains extreme luxury outliers, `MEDIANX` was utilized to determine the true `Median Sales Price Change` across rolling periods rather than a skewed simple average[cite: 9].
- Step 8 : Utilized time-intelligence functions such as `DATESINPERIOD` to calculate a rolling `Last 12 Month Sales` metric, stepping back `12` months from the `MAX` date[cite: 9]. `TOTALYTD` was also deployed to track cumulative capital flow[cite: 9].
- Step 9 : Applied advanced context modifiers to protect baseline visual integrity. The `ALLEXCEPT` function was used to calculate `Sales by Region`, ensuring regional totals remained anchored even when other categorical slicers were engaged[cite: 9]:
    
    ```
    Sales by Region = CALCULATE(SUM('Housing'[purchase_price]),ALLEXCEPT('Housing','Housing'[region]))
    ```
    
- Step 10 : **AI-Driven Analytics:** Deployed Power BI's built-in Machine Learning visual, 'Key Influencers', to automatically identify the hidden factors driving property values upward[cite: 9].

<img width="410" height="252" alt="SS2" src="https://github.com/user-attachments/assets/e146ced4-33a3-4929-9724-ec1331c4d207" />


- Step 11 : Structured the final semantic model into a highly interactive three-page UI (*House Market Overview*, *Sales Performance*, and *House Type Analysis*)[cite: 9] incorporating universal slicers for Area, City, and Sales Type[cite: 9].

# Dashboard Snapshots

### Page 1: House Market Overview

<img width="1286" height="716" alt="Page1" src="https://github.com/user-attachments/assets/d9198d41-b2b0-4e78-b0e3-1ed4baac3947" />

### Page 2: Sales Performance & Key Influencers

<img width="1291" height="716" alt="Page2" src="https://github.com/user-attachments/assets/aac817fb-d1d2-410c-8304-041d935e7af2" />

### Page 3: House Type Analysis

<img width="1287" height="715" alt="Page3" src="https://github.com/user-attachments/assets/43906f9b-9abc-49dc-8661-7a0b72966d4f" />

# Insights

A comprehensive, multi-page analytical application was created in Power BI and connected to a GCP BigQuery backend.

Following inferences can be drawn from the dashboard:

### [1] Outlier Tracking (Offer vs. Purchase)

```
The high-density Scatter Plot highlights extreme market deviations where intense bidding wars drove final purchase prices significantly above the engineered asking prices, establishing clear geographical hotspots for high demand[cite: 9].
```

### [2] Automated Root-Cause Value Drivers

```
By feeding the calculated `Age` column into the Key Influencers AI model, the system automatically identified historical and heritage properties (specifically those exceeding 113 years of age) as a primary statistical driver for purchase price increases[cite: 9].
```

### [3] Macroeconomic Vulnerability by Sector

```
The House Type Analysis matrix reveals exactly how macroeconomic headwinds intersect with specific real estate brackets. Stakeholders can immediately track how yield shifts on mortgage credit bonds and the Danish annual inflation rate (dk_ann_infl_rate%) disproportionately influence the final SQM valuation of Farms versus Urban Apartments[cite: 9].


```

### [4] Sales Trajectory by Transaction Type
```
The Year-Over-Year trajectory confirms distinct behavioral growth curves across different real estate liquidity channels (Auction, Regular Sale, Family Sale, and Other)[cite: 9]. Family sales demonstrate extreme variance compared to standard regular sales, informing analysts to segment these transactions when forecasting future baseline market liquidity[cite: 9].
```
