# Cafe Sales Analysis

## Project Overview
*Project presents process of cleaning and analyzing data gathered from whole year of cafe sales. 
The final results are presented on an interactive dashboard developed in Power BI.*

## Project Objectives
- **Data Cleaning:** Process of idetifying and fixing missing and error values 
- **Data Transformation:** Process of converting each column to proper data format
- **Data Analysis:** Analyse, conslucions and recommendations for the future
- **Data Visualization:** Creating visualisations in the Power Bi to bettre understanding insights.

## Project Workflow

### 1. Database Creation
Create new database called "CafeSales" in Microsoft SQL Server Management Studio

### 2. Schema Creation
Create the following schemas:
- **raw:** Contains the raw data imported directly from the flat file (.csv).
- **staging:** Used for each step of data cleaning process.
- **analyzing:** Contains the cleaned and transformed data for analysis.

```sql
CREATE SCHEMA raw;
CREATE SCHEMA staging;
CREATE SCHEMA analyzing;
```
### 3. Data Import
Import the flat file into the **raw** schema.

- Data Type Selection: Pre-analysis in MS Excel (using functions like **UNIQUE**,**MAX**,**LEN**) to establish optimal length for each column.
- Since several columns contain values such as **"ERROR"** (e.g., in the `Quantity` column) or **"UNKNOWN"** (e.g., in `Price_Per_Unit`), we import the data as NVARCHAR(50). Data types will be converted after the cleaning process.

That is how raw table looks like:
```sql
SELECT TOP 10 * 
FROM raw.CafeSalesSrc
```
| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method  | Location  | Transaction_Date |
|---------------|---------|----------|---------------|------------|----------------|-----------|----------------|
| TXN_1000555   | Tea     | 1        | 1.5           | 1.5        | Credit Card    | In-store  | 2023-10-19     |
| TXN_1001832   | Salad   | 2        | 5.0           | 10.0       | Cash           | Takeaway  | UNKNOWN        |
| TXN_1002457   | Cookie  | 5        | 1.0           | 5.0        | Digital Wallet | Takeaway  | 2023-09-29     |
| TXN_1003246   | Juice   | 2        | 3.0           | 6.0        | NULL           | NULL      | 2023-02-15     |
| TXN_1004184   | Smoothie| 1        | 4.0           | 4.0        | Credit Card    | In-store  | 2023-05-18     |
| TXN_1004563   | Tea     | 5        | 1.5           | 7.5        | Credit Card    | In-store  | 2023-10-28     |
| TXN_1005331   | Coffee  | 1        | 2.0           | 2.0        | Digital Wallet | Takeaway  | 2023-11-04     |
| TXN_1005377   | Cake    | 5        | UNKNOWN       | 15.0       | Digital Wallet | Takeaway  | 2023-06-03     |
| TXN_1005472   | Coffee  | 4        | 2.0           | 8.0        | Credit Card    | NULL      | 2023-04-21     |
| TXN_1006942   | Salad   | 1        | 5.0           | 5.0        | Credit Card    | In-store  | 2023-11-30     |


### 4. Data Cleaning Process

### 4.1 Remove Irrelevant Data

- Each column is necessary however we may consider removing the column **Total_Spent** because it's multiplied `Quantity` and `Price_Per_Unit` and we can do it at the data analysis stage, but at the same time this column will be helpful in process of refill missing values in `Quantity` OR `Price_Per_Unit` columns that's why i decided to keep this column.

### 4.2 Indentification of Duplicates

```sql
WITH duplicate_cte AS
(
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY 
		Transaction_ID,
		Item,
		Quantity,
		Price_Per_Unit,
		Total_Spent,
		Payment_Method,
		'Location',
		Transaction_Date 
		Order By Transaction_ID) as row_num
		from raw.CafeSalesSrc
)
SELECT * 
From duplicate_cte
WHERE row_num >1
```
The result of the query is an empty table, which confirms that there are no duplicates.

### 4.3 Fix Structural Errors

- [Exploratory Data](EDA.md) Analysis (EDA) to identify potential problems with structure and suggest a solution.

- [Data cleaning](DataCleaning.md) According to [Summary](EDA.md#summary) in [EDA File](EDA.md)

### 5. [Data Analysis](/DataAnalysis.md)


## 6. Data Visualization

The data was visualised in PowerBi dashboard:

### Dashboard overview:
The dashboard contains 3 pages:

### 1. Overview Page
Overview page includes:
- Total transactions
- Total revenue
- Average Transaction Value
- Total Sold Products
- Monthly revenue
### 2. Products Page 
Product page includes:
- Best selling products
- Total revenue by product
- Revenue share
- Revenue by products over time
### 3. Transactions Page 
Transactions page includes:
- Monthly revenue by payment method
- Monthly revenue by location
- Amount of transactions splitted by different payment method
- Amount of transactions splitted by different locations

### Screenshots
#### Overview Page
![image](/Screenshots/OverviewPage.PNG)
#### Products Page
![image](/Screenshots/ProductsPage.PNG)
#### Transactions Page
![image](/Screenshots/TransactionsPage.PNG)

### Key insights and recommendations

**Total amount of sold products**
- Calculated total amount of sold products can be used to set realistic sales targets in the future or compare this indicator with past years.
**Total revenue**
- The total revenue metric helps assess overall financial performance and set revenue targets. Comparing this indicator with previous or future periods can reveal growth trends and sales effectiveness.
**Average transaction value**
- For the purpose of expanding future analysis, the data collection process could be expanded to include customer data to better analyze orders.
**Monthly Revenue**
- Worst month: February
- Best month: June 
- Sales results from both months suggest that the cafe is susceptible to seasonality caused by the season (lowest sales in winter and highest in summer). It might be worthwhile to introduce seasonal winter teas to increase sales in winter.
**Best selling product (by quantity)**
- The most popular product is coffee, it means strong customers preference
- The **"UNKNOWN"** category counts 1471 transactions.
- We may consider promotional efforts for top products such as Coffee, Salad, and Tea.
- Because of the significant amount of UNKNOWN products, the ETL process should be inspect, then changes should be implemented to improve data quality.
**Revenue by Product**
- Salad, sandwich and smoothie generates the most revenue
- The data collection process should be expanded of information about order, it will help determine which products are sold together to better menage price list and discounts to increase revenue.
- Also the data collection process should be improved to eliminate **"UNKNOWN"** products from dataset.
**Revenue share**
- The most profitable product is Salad (21.43 %)
- The least profitable product is Coockie (4.04%)
**Transacions analysis**
- The Data Gathering process should be improved because of significat amount of **"UNKNOWN"** values  in `payment_method` column.
- Among known methods od payment there's no clear preferences.
- Similarly to payment methods there's no clear preference in locations of orders chosen bo customers.

# Dataset
This dataset is is sourced from [Kaggle](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training)
# License
CC BY-SA 4.0
