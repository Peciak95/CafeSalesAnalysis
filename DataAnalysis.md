# Data Analysis

After cleansing the data in SQL, the data analysis process will be carried out in Power Bi due to the fact that the table is relatively small. 
This approach ensures fast processing time and not overloading Power Bi, allowing interactive data exploration and visualization which is easier to understand for non technical stakeholders.

## 1.Overall Sales Metrics

### 1.1 Total Quantity Sold

### Objectives
The objective of this analysis is to evaluate the overall sales performance by calculating the total number of products sold. This metric serves as a baseline indicator of business volume and can be used for further trend analysis or performance comparisons.

### Methodology
Data from the table is aggregated by summing the `Quantity` column. This provides the total count of individual items sold over the period, offering insights into overall customer demand and sales activity.

### SQL Query
```sql
SELECT SUM(Quantity) as Total_Quantity
FROM analyzing.CafeSalesFinal
```
### Result

Total_Quantity |
|--------------|
| 30180        |

Since some transactions lack `quantity` values, ignoring them would underestimate total sales. We estimate the missing values using the average `quantity` per transaction.

### Estimating missing transactions
**Transactions with missing Quantity:**

```sql
SELECT COUNT(*) as Transactions_Withot_Quantity
FROM analyzing.CafeSalesFinal
WHERE Quantity IS NULL
```
**Result:**

Transactions_Withot_Quantity |
|--------------|
| 23           |

**Average quantity per transaction**
```sql
SELECT CAST(AVG(CAST(Quantity AS DECIMAL(10,2))) as DECIMAL(10,2)) as Average_Quantity_Per_Transaction
FROM analyzing.CafeSalesFinal;
```
**Result:**

Average_Quantity_Per_Transaction |
|--------------|
| 3.02         |

### Final Adjusted Sales

- Estimated missing quantity: 23 * 3.02 &asymp; 69
- Total estimated sales: 30180 + 69 &asymp; **30264**
- Precentage increase: &asymp; 0.23% 

The estimated missing quantities increase total sales by approximately 0.23%.

### Conclusions and Recommendations

- Calculated total amount of sold products can be used to set realistic sales targets in the future.
- Calculated total amount of sold products can be used to compare this indicator with past years.

### 1.2 Total Revenue

### Objectives
The goal of this analysis is to determine the total revenue generated from all transactions. This metric provides insight into overall business performance and can be used to compare revenue trends over different periods.

### Methodology

The total revenue is calculated by summing the `Total_Spent` column, which represents the amount spent in each transaction.

### SQL Query

```sql
SELECT CAST(SUM(Total_Spent) as DECIMAL(9,0)) AS Total_Revenue
FROM analyzing.CafeSalesFinal;
```
### Result
|Total_Revenue |
|--------------|
| 89096        |

Since some transactions lack `total_spent` values, ignoring them would underestimate total revenue. We estimate the missing values using the average revenue per transaction.

### Estimating missing values
**Transactions with missing `total_spent` values:**

```sql
SELECT COUNT(*) as rows_without_value
From analyzing.CafeSalesFinal
WHERE Total_Spent IS NULL
```
**Result:**

rows_without_value |
|--------------|
| 23           |

**Average `total_spent` per transaction**
```sql
SELECT CAST(AVG(Total_Spent) as DECIMAL(9,2)) AS Average_Revenue_Per_Transaction
FROM analyzing.CafeSalesFinal;
```
**Result:**

Average_Revenue_Per_Transaction |
|--------------|
| 8.93         |

### Final Adjusted REvenue

- Estimated missing revenue: 23 * 8.93 &asymp; 205
- Total estimated sales: 89096 + 205 &asymp; **89301**
- Precentage increase: &asymp; 0.23% 

The estimated missing transactions increase total sales by approximately 0.23%.

### Conclusions and Recommendations

- The total revenue metric helps assess overall financial performance and set revenue targets.
- Comparing this indicator with previous periods can reveal growth trends and sales effectiveness.

### 1.3 Average transaction value

### Objectives
The purpose of the indicator is to gain insight into the average value of orders. 

### SQL Query

```sql
SELECT CAST(AVG(Total_Spent) as DECIMAL(9,2)) AS Average_Revenue_Per_Transaction
FROM analyzing.CafeSalesFinal;
```
**Result:**
| Average_Revenue_Per_Transaction |
|---------------------------------|
| 8.93                            |
### Conclusions and Recommendations

- For the purpose of expanding future analysis, the data collection process could be expanded to include customer data to better analyze orders.

### 1.4 Monthly revenue

### Objectives
The purpose of the query is to inspect total revenue by month over date since January to December

### Methodology
`Total_Spent` column is aggregiated by month, this means that all transactions made in a given month are summed.

### SQL Query

```sql
SELECT DATENAME(mm, MIN(Transaction_Date)) as Month_Name,
       SUM(Total_Spent) as Total_Revenue
FROM analyzing.CafeSalesFinal
WHERE Transaction_Date IS NOT NULL
GROUP BY DATEPART(mm, Transaction_Date)
ORDER BY DATEPART(mm, Transaction_Date) ASC
```
**Result:**
| Month_Name | Total_Revenue |
|------------|---------------|
| January    | 7254.00       |
| February   | 6644.00       |
| March      | 7216.00       |
| April      | 7179.00       |
| May        | 6957.50       |
| June       | 7353.00       |
| July       | 6877.50       |
| August     | 7112.50       |
| September  | 6871.00       |
| October    | 7314.00       |
| November   | 6967.00       |
| December   | 7177.00       |
### Conclusions and Recommendations

- Worst month: February
- Best month: June 
- Sales results from both months suggest that the cafe is susceptible to seasonality caused by the season (lowest sales in winter and highest in summer). It might be worthwhile to introduce seasonal winter teas to increase sales in winter.


## 2. Product Analysis
### 2.1 Best Selling Product (by quantity)
### Objectives
The goal of this analysis is to determine which products generate the highest sales volume. This insight will help optimize product offerings and inform promotional strategies. Note that some transactions were categorized as "UNKNOWN" due to data quality issues during cleaning.
### Methodology
Data from table was grouped by the `Item` column. The total quantity for each product was calculated by summing the `Quantity` column. <BR>
The **"UNKNOWN"** category represents transactions with ambiguous product identification.
### SQL Query
```sql
SELECT	Item, 
		SUM(Quantity) AS Total_Quantity
FROM analyzing.CafeSalesFinal
GROUP BY Item
ORDER BY Total_Quantity DESC;
```
### Result
| Item     | Total_Quantity |
|----------|----------------|
| Coffee   | 3904           |
| Salad    | 3819           |
| Tea      | 3650           |
| Cookie   | 3598           |
| Juice    | 3505           |
| Cake     | 3468           |
| Sandwich | 3429           |
| Smoothie | 3336           |
| UNKNOWN  | 1471           |

### Conclusions and Recommendations

- The most popular product is coffee, it means strong customers preference
- The **"UNKNOWN"** category counts 1471 transactions.
- We may consider promotional efforts for top products such as Coffee, Salad, and Tea.
- Because of the significant amount of UNKNOWN products, the ETL process should be inspect, then changes should be implemented to improve data quality.

### 2.2 Revenue by Product

### Objectives

The goal of the analysis is to determine how each product contributes to total revenue for optimize pricing and marketing strategies.

### Methodology

`Total_Spent` column is summed and grouped by each `Item`.

### SQL Query

```sql
SELECT	Item, 
		CAST(SUM(Total_Spent) as DECIMAL(9,0)) AS Total_Revenue
FROM analyzing.CafeSalesFinal
GROUP BY Item
ORDER BY Total_Revenue DESC
```
**Result:**
| Item     | Total_Revenue |
|----------|---------------|
| Salad    | 19095         |
| Sandwich | 13716         |
| Smoothie | 13344         |
| Juice    | 10515         |
| Cake     | 10404         |
| Coffee   | 7808          |
| Tea      | 5475          |
| UNKNOWN  | 5141          |
| Cookie   | 3598          |

### Conclusions and Recommendations

- Salad, sandwich and smoothie generates the most revenue
- The data collection process should be expanded of information about order, it will help determine which products are sold together to better menage price list and discounts to increase revenue.
- Also the data collection process should be improved to eliminate **"UNKNOWN"** products from dataset.

### 2.3 Revenue by products over time

### Objectives
This analyze was made to inspect how revenue changes over time partitioned by each product

### Methodology

This analyse due to unreadability of output table was made in PowerBi as a dynamic chart.

### 2.4 Revenue share

### Objectives
To analyze how each product affects overall revenue to estimate the products with the largest share.
### Methodology
`Total_Spent` columns was summed and aggreagiated based on product, next each product revenue was divided by total revenue to calculate percentage of share.
### SQL Query
```sql
SELECT 
    Item, 
    CAST(SUM(Total_Spent) AS DECIMAL(9, 0)) AS Total_Revenue,
    CAST((SUM(Total_Spent) * 100.0) / SUM(SUM(Total_Spent)) OVER() AS DECIMAL(5,2)) AS Revenue_Percentage
FROM analyzing.CafeSalesFinal
GROUP BY Item
ORDER BY Total_Revenue DESC;
```
**Result:**
| Item     | Total_Revenue | Revenue_Percentage |
|----------|---------------|--------------------|
| Salad    | 19095         | 21.43              |
| Sandwich | 13716         | 15.39              |
| Smoothie | 13344         | 14.98              |
| Juice    | 10515         | 11.80              |
| Cake     | 10404         | 11.68              |
| Coffee   | 7808          | 8.76               |
| Tea      | 5475          | 6.15               |
| UNKNOWN  | 5141          | 5.77               |
| Cookie   | 3598          | 4.04               |


### Conclusions and Recommendations

- The most profitable product is Salad (21.43 %)
- The least profitable product is Coockie (4.04%)

## 3. Transactions analysis
### Objectives
The goal of this step is to answer question: <BR>
*"Which method of payment is more popular at the cafe and whether customers prefer takeaway or on-site purchases"*

### Methodology

To calculate the total amount of transactions in each payment channel and orders location preffered we need to sum all rows in each category.

### SQL Query - `Payment method`
```sql
SELECT	Payment_Method,
		COUNT(*) as total_transactions
FROM analyzing.CafeSalesFinal
GROUP BY Payment_Method
ORDER BY COUNT(*) DESC
```
**Result:**
| Payment_Method | total_transactions |
|----------------|--------------------|
| UNKNOWN        | 3178               |
| Digital Wallet | 2291               |
| Credit Card    | 2273               |
| Cash           | 2258               |

### SQL Query - `Location`
```sql
SELECT	[Location],
		COUNT(*) as total_transactions
FROM analyzing.CafeSalesFinal
GROUP BY [Location]
ORDER BY COUNT(*) DESC
```
**Result:**
| Location  | total_transactions |
|-----------|--------------------|
| UNKNOWN   | 3961               |
| Takeaway  | 3022               |
| In-store  | 3017               |

### Conclusions and Recommendations

- The Data Gathering process should be improved because of significat amount of **"UNKNOWN"** values  in `payment_method` column.
- Among known methods od payment there's no clear preferences.
- Similarly to payment methods there's no clear preference in locations of orders chosen bo customers.

# [Data Visualization](/README.md#6-data-visualization)

