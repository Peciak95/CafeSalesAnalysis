# Exploratory Data Analysis

To get an overview of the dataset, we retrieve the first 10 rows from source table by following query:

**Select top 10 rows**
```sql
SELECT TOP 10 * 
FROM raw.CafeSalesSrc;
```

**Result:**

| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|----------|----------|----------------|-------------|----------------|------------|------------------|
| TXN_1000555    | Tea      | 1        | 1.5            | 1.5         | Credit Card    | In-store   | 2023-10-19       |
| TXN_1001832    | Salad    | 2        | 5.0            | 10.0        | Cash           | Takeaway   | UNKNOWN          |
| TXN_1002457    | Cookie   | 5        | 1.0            | 5.0         | Digital Wallet | Takeaway   | 2023-09-29       |
| TXN_1003246    | Juice    | 2        | 3.0            | 6.0         | NULL           | NULL       | 2023-02-15       |
| TXN_1004184    | Smoothie | 1        | 4.0            | 4.0         | Credit Card    | In-store   | 2023-05-18       |
| TXN_1004563    | Tea      | 5        | 1.5            | 7.5         | Credit Card    | In-store   | 2023-10-28       |
| TXN_1005331    | Coffee   | 1        | 2.0            | 2.0         | Digital Wallet | Takeaway   | 2023-11-04       |
| TXN_1005377    | Cake     | 5        | UNKNOWN        | 15.0        | Digital Wallet | Takeaway   | 2023-06-03       |
| TXN_1005472    | Coffee   | 4        | 2.0            | 8.0         | Credit Card    | NULL       | 2023-04-21       |
| TXN_1006942    | Salad    | 1        | 5.0            | 5.0         | Credit Card    | In-store   | 2023-11-30       |

### Observations:
- The dataset contains missing **NULL** and incorrect **UNKNOWN** values.
- Some numeric columns (e.g., `Price_Per_Unit`) have non-numeric values **(UNKNOWN)**.
- `Transaction_Date` has an invalid value **(UNKNOWN)**.
- **Payment_Method** and **Location** contain **NULL** values.


##  **Check data types**
To understand how the columns are stored, we check their data types:
```sql
SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS;
```
**Result:**
| COLUMN_NAME      | DATA_TYPE |
|------------------|-----------|
| Transaction_ID   | nvarchar  |
| Item             | nvarchar  |
| Quantity         | nvarchar  |
| Price_Per_Unit   | nvarchar  |
| Total_Spent      | nvarchar  |
| Payment_Method   | nvarchar  |
| Location         | nvarchar  |
| Transaction_Date | nvarchar  |

After review in MS Excel (using functions like **UNIQUE**,**MAX**,**LEN**) to determine the optimal length for each field we can consider following data types:

| Column            | Current Type   | Recommended Type | Explanation                                                                                   |
|-------------------|----------------|------------------|-------------------------------------------------------------------------------------------------|
| Transaction_ID    | NVARCHAR(50)   | INT (after splitting) and CHAR(3) | Due to better performance and good practice this column should be splitted. Text part is not longer than 3 characters and numeric part is integer.|
| Item              | NVARCHAR(50)   | VARCHAR(20)      | The longest item in the dataset has 8 characters, so we can use VARCHAR(20) to allow flexibility while optimizing storage. |
| Quantity          | NVARCHAR(50)   | SMALLINT              | Quantity represents the number of items sold, which will always be a whole number (no fractional values). |
| Price_Per_Unit    | NVARCHAR(50)   | DECIMAL(9,2)     | Prices have at most two decimal places and typically fit within 6 characters, but DECIMAL(9,2) provides the same storage efficiency while allowing flexibility. |
| Total_Spent       | NVARCHAR(50)   | DECIMAL(9,2)     | Similar to Price_Per_Unit for ensure consistency and prevent from rounding errors.                 |
| Payment_Method    | NVARCHAR(50)   | VARCHAR(20)      | The longest payment method in the dataset is 14 characters, so VARCHAR(20) provides some buffer |
| Location          | NVARCHAR(50)   | VARCHAR(20)      | The longest location name is 8 characters, so VARCHAR(20) ensure balance between efficiency and flexibility.  |
| Transaction_Date  | NVARCHAR(50)   | DATE             | Since we only store the transaction date without timestamps, DATE is the optimal choice.        |

## 1. Cleaning the `Transaction_ID` Column

### 1.1 Checking Uniqueness
We verify if all transaction IDs are unique:


```sql
SELECT COUNT(Transaction_ID) AS total_rows, 
       COUNT(DISTINCT(Transaction_ID)) AS distinct_rows
FROM raw.CafeSalesSrc;
```
**Result:**
| total_row      | distinct_rows |
|------------------|-------------|
| 10000   | 10000 |

All transaction IDs are unique

### 1.2 Extracting the Numeric Part

Since Transaction_ID follows the pattern "TXN_1000555", we extract only the numeric portion:

```sql
SELECT TOP 10 
       SUBSTRING(Transaction_ID, 5, LEN(Transaction_ID)) AS Transaction_ID
FROM raw.CafeSalesSrc;
```
**Result:**
| Transaction_ID |
|---------------|
| 1000555       |
| 1001832       |
| 1002457       |
| 1003246       |
| 1004184       |
| 1004563       |
| 1005331       |
| 1005377       |
| 1005472       |
| 1006942       |

Now we check if removing the prefix still results in unique values:

```sql
SELECT COUNT(DISTINCT(SUBSTRING(Transaction_ID,5,LEN(Transaction_ID)))) AS unique_ids
FROM raw.CafeSalesSrc;
```
**Result:**
| unique_ids |
|------------------|
| 10000   |

### 1.3 Observations

The numeric part remains unique, so we can safely split the column into:
- `Transaction_ID_Prefix` (e.g., TXN)
- `Transaction_ID` (numeric part)

## 2. `Item` Column

### 2.1 Unique Values
Let's take a look at unique items in `Item` column by the following query:
```sql
SELECT DISTINCT(Item)
FROM raw.CafeSalesSrc
```
**Result:**
| Item      |
|-----------|
| Coffee    |
| Juice     |
| Cookie    |
| Sandwich  |
| UNKNOWN   |
| ERROR     |
| NULL      |
| Tea       |
| Salad     |
| Smoothie  |
| Cake      |

We may noticed that column contains inconsistent values like **UNKNOWN** or **ERROR** and missing values **NULL**. This incorrect values need to be standardized as **"UNKNOWN"** for better understanding of dataset.
### 2.2 Inspect Incorrect Values

```sql
SELECT TOP 10 *
FROM raw.CafeSalesSrc
WHERE Item IN ('UNKNOWN','ERROR') OR Item IS NULL
```
**Result:**
| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| TXN_1012349    | UNKNOWN  | 4        | 3.0            | 12.0        | Cash           | In-store  | 2023-07-08       |
| TXN_1037464    | UNKNOWN  | 5        | 4.0            | 20.0        | Digital Wallet | NULL      | 2023-02-14       |
| TXN_1054915    | NULL     | 5        | 1.0            | 5.0         | NULL           | In-store  | 2023-01-22       |
| TXN_1057209    | UNKNOWN  | 5        | 1.0            | 5.0         | Credit Card    | In-store  | 2023-09-17       |
| TXN_1059293    | UNKNOWN  | 5        | 4.0            | 20.0        | UNKNOWN        | NULL      | 2023-12-04       |
| TXN_1082717    | ERROR    | UNKNOWN  | NULL           | 9.0         | Digital Wallet | In-store  | 2023-12-13       |
| TXN_1104849    | UNKNOWN  | 4        | 4.0            | 16.0        | Digital Wallet | Takeaway  | 2023-07-05       |
| TXN_1118799    | ERROR    | 4        | 4.0            | NULL        | Credit Card    | Takeaway  | 2023-09-19       |
| TXN_1122699    | UNKNOWN  | 5        | 5.0            | 25.0        | Digital Wallet | In-store  | 2023-10-14       |
| TXN_1124796    | ERROR    | 5        | 1.5            | 7.5         | Credit Card    | NULL      | 2023-10-28       |

### 2.3 **Cleaning strategy**

**Issues:**

- **NULL** values represent missing product names.
- **"ERROR"** values are probably data entry or data collection process mistakes.
- **"UNKNOWN"** is already being used for missing data, so we will standardize all missing or incorrect values as "UNKNOWN".

**Solution:**

Replace **NULL** and **"ERROR"** with **"UNKNOWN"** to maintain consistency:

## 3. `Quantity` column

### 3.1 Unique Values

To identify inconsistence, let's check the unique values in the `Quantity `column:

```sql
SELECT DISTINCT(Quantity)
FROM raw.CafeSalesSrc;
```
**Result:**
| Quantity |
|----------|
| 2        |
| 3        |
| UNKNOWN  |
| ERROR    |
| NULL     |
| 4        |
| 5        |
| 1        |

### 3.2 Inspect Incorrect Values

Let's inspect rows with **"UNKNOWN"**, **"ERROR"**, or **NULL** values , by following query:

```sql
SELECT TOP 10 *
FROM raw.CafeSalesSrc
WHERE Quantity IN ('UNKNOWN','ERROR') OR Quantity IS NULL;
```

**Result:**

| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| TXN_1010950    | Cookie   | ERROR    | 1.0            | 1.0         | Digital Wallet | Takeaway  | 2023-01-07       |
| TXN_1022126    | Tea      | ERROR    | 1.5            | 4.5         | NULL           | UNKNOWN   | UNKNOWN          |
| TXN_1054915    | NULL     | 5        | 1.0            | 5.0         | NULL           | In-store  | 2023-01-22       |
| TXN_1056277    | Cookie   | UNKNOWN  | 1.0            | 5.0         | NULL           | NULL      | 2023-02-06       |
| TXN_1062822    | Juice    | UNKNOWN  | 3.0            | 6.0         | Credit Card    | NULL      | 2023-06-28       |
| TXN_1082717    | ERROR    | UNKNOWN  | NULL           | 9.0         | Digital Wallet | In-store  | 2023-12-13       |
| TXN_1090854    | Cookie   | UNKNOWN  | 1.0            | 3.0         | Credit Card    | In-store  | 2023-01-07       |
| TXN_1101527    | Smoothie | UNKNOWN  | 4.0            | 20.0        | Digital Wallet | NULL      | 2023-04-20       |
| TXN_1124900    | NULL     | 4        | 4.0            | 16.0        | Credit Card    | In-store  | 2023-09-08       |
| TXN_1165762    | NULL     | 3        | 2.0            | 6.0         | Credit Card    | NULL      | 2023-10-22       |

### 3.3 Observations

- The column contains valid integer values (1-5).

- **"UNKNOWN"** and **"ERROR"** - these are not numerical values and should be handled.

- Inconsistent values in the `Quantity` column could be replaced by dividing `Total_Spent` by `Price_Per_Unit`, if  both columns have valid values. This approach will be implemented in the data cleaning phase.

### 3.4 **Cleaning strategy**

- Replace **"UNKNOWN"** and **"ERROR"** with **NULL** to allow mathematical operations like **SUM** oraz **AVERAGE**.
- Calculate the `Quantity` column based on `Total_Spent` and `Price_Per_Unit` if both exists.

## 4. `Price_Per_Unit` and `Total_Spent` column

The process of cleaning the `Price_Per_Unit` `Total_Spent` column is similar to that of the `Quantity` column, as it also contains invalid values such as **"UNKNOWN"**, **"ERROR"**, and **NULL**. To clean the data, we will identify the rows with these invalid values and replace them with appropriate calculation or **NULL**.

Also we can replace missing values with the appropiate calculation :
- `Price_Per_Unit` = `Total_Spent` / `Quantity` if both columns have valid values.
- `Total_Spent` = `Price_Per_Unit` * `Quantity `if both columns have valid values.

## 5. `Payment_method` column

### 5.1 Unique Values

Like before, first we will check the unique values of the `Payment_Method` column by following query:

```sql
SELECT Distinct(Payment_Method)
FROM raw.CafeSalesSrc
```
**Result:**

| Payment_Method |
|---------------|
| UNKNOWN       |
| ERROR         |
| NULL          |
| Cash          |
| Credit Card   |
| Digital Wallet|

### 5.2 Inspect Incorrect Values

Let's take a look at the rows containint **"UNKNOWN"**, **"ERROR"**, or **NULL** values by following query:

```sql
SELECT TOP 10 *
FROM raw.CafeSalesSrc
WHERE	Payment_Method IN ('UNKNOWN','ERROR') OR Payment_Method IS NULL
```

**Result:**

| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| TXN_1003246    | Juice    | 2        | 3.0            | 6.0         | NULL           | NULL      | 2023-02-15       |
| TXN_1016246    | Coffee   | 1        | 2.0            | 2.0         | ERROR          | NULL      | 2023-01-19       |
| TXN_1018470    | Juice    | 3        | 3.0            | 9.0         | NULL           | ERROR     | 2023-01-14       |
| TXN_1021087    | Sandwich | 1        | 4.0            | 4.0         | NULL           | ERROR     | 2023-11-01       |
| TXN_1022126    | Tea      | ERROR    | 1.5            | 4.5         | NULL           | UNKNOWN   | UNKNOWN          |
| TXN_1022220    | Juice    | 3        | 3.0            | 9.0         | NULL           | Takeaway  | 2023-08-03       |
| TXN_1025152    | Cookie   | 4        | 1.0            | 4.0         | NULL           | NULL      | 2023-08-19       |
| TXN_1026050    | Cake     | 1        | 3.0            | 3.0         | ERROR          | NULL      | 2023-04-23       |
| TXN_1030539    | Cake     | 3        | 3.0            | 9.0         | NULL           | Takeaway  | 2023-11-06       |
| TXN_1031802    | Cookie   | 4        | 1.0            | 4.0         | ERROR          | UNKNOWN   | 2023-06-27       |

### 5.3 **Cleaning strategy**

**Issues:**

- **NULL** values represent missing payment method values.
- **"ERROR"** values are possibyl data entry or data collection process mistake.
- **"UNKNOWN"** is already being used for missing values, so we will standardize all missing or incorrect values as **"UNKNOWN"**.

**Solution:**

Replace **NULL** and **"ERROR"** with **"UNKNOWN"** to maintain consistency:

## 6.  **Location** column

### 6.1 Unique values

Like before, first we will check the distinct values of the **Location** column by following query

```sql
SELECT DISTINCT([Location]) 
FROM raw.CafeSalesSrc;
```
**Result:**

| Location  |
|-----------|
| Takeaway  |
| UNKNOWN   |
| ERROR     |
| NULL      |
| In-store  |

### 6.2 Inspect Incorrect Values

Let's take a look at rows with **"UNKNOWN"**, **"ERROR"**, or **NULL** values:

```sql
SELECT TOP 10 *
FROM raw.CafeSalesSrc
WHERE	[Location] IN ('UNKNOWN','ERROR') OR [Location] IS NULL
```

**Result:**

| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| TXN_1003246    | Juice    | 2        | 3.0            | 6.0         | NULL           | NULL      | 2023-02-15       |
| TXN_1005472    | Coffee   | 4        | 2.0            | 8.0         | Credit Card    | NULL      | 2023-04-21       |
| TXN_1016246    | Coffee   | 1        | 2.0            | 2.0         | ERROR          | NULL      | 2023-01-19       |
| TXN_1018470    | Juice    | 3        | 3.0            | 9.0         | NULL           | ERROR     | 2023-01-14       |
| TXN_1018880    | Smoothie | 5        | 4.0            | 20.0        | Digital Wallet | NULL      | 2023-07-15       |
| TXN_1020983    | Juice    | 1        | 3.0            | 3.0         | Digital Wallet | NULL      | 2023-03-23       |
| TXN_1021087    | Sandwich | 1        | 4.0            | 4.0         | NULL           | ERROR     | 2023-11-01       |
| TXN_1022126    | Tea      | ERROR    | 1.5            | 4.5         | NULL           | UNKNOWN   | UNKNOWN          |
| TXN_1025152    | Cookie   | 4        | 1.0            | 4.0         | NULL           | NULL      | 2023-08-19       |
| TXN_1026050    | Cake     | 1        | 3.0            | 3.0         | ERROR          | NULL      | 2023-04-23       |

### 6.3 **Cleaning strategy**

**Issues:**

- **NULL** values represent missing payment method values.
- **"ERROR"** values are possibyl data entry or data collection process mistake.
- **"UNKNOWN"** is already being used for missing values, so we will standardize all missing or incorrect values as **"UNKNOWN"**.

**Solution:**

Replace **NULL** and **"ERROR"** with **"UNKNOWN"** to maintain consistency:


## 6. **Transaction_Date** column

### 6.1 Checking data format

First we need to check if all values has correct data format by following query:

```sql
SELECT DISTINCT (ISDATE(Transaction_Date)) as is_date
FROM raw.CafeSalesSrc
```

**Result:**
| is_date   |
|-----------|
| 0  |
| 1  |

0 value means that in columns occurs incorrect values other than date.

### Checking incorrect values

```sql
SELECT DISTINCT(Transaction_Date) as distinct_inconsistence
FROM raw.CafeSalesSrc
WHERE ISDATE(Transaction_Date) = 0
```

**Result:**

| distinct_inconsistence |
|------------------------|
| NULL                   |
| ERROR                  |
| UNKNOWN                |

Because we have date datatype here we need to replace all incorrect values with **NULL** to allow for using functions.

We can check if all values is in the same format by following query:

```sql
SELECT *
FROM raw.CafeSalesSrc
WHERE TRY_CONVERT(DATE, Transaction_Date, 120) IS NULL AND Transaction_Date NOT IN ('UNKNOWN','ERROR')
```

This query tries to convert the date to "120" format which means "RRRR-MM-DD" without checking **"UNKNOWN"** and **"ERROR"** values.

Result of query is empty table which means that dates are in consistent format.

## SUMMARY

### Data Quality Issues:

Inconsistent values (NULL, "UNKNOWN", "ERROR")

### Data Type Conversions:

- `Transaction_ID`: Split into a **CHAR(3)** prefix and **INT** (numeric part)
- `Item`: Convert to **VARCHAR(20)**
- `Quantity`: Convert to **SMALLINT**
- `Price_Per_Unit`: Convert to **DECIMAL(9,2)**
- `Total_Spent`: Convert to **DECIMAL(9,2)**
- `Payment_Method`: Convert to **VARCHAR(20)**
- `Location`: Convert to **VARCHAR(20)**
- `Transaction_Date`: Convert to **DATE**

### Cleaning Strategies:

- `Transaction_ID`: Extract numeric part to maintain uniqueness and improve performance.
- Text Fields: Replace **"ERROR"** and **NULL** with **"UNKNOWN"** for consistency.
- Numeric Fields: Convert **"ERROR"** and **"UNKNOWN"** to **NULL** and recalculate values where possible.
- Dates: Replace invalid date entries (**"ERROR"**, **"UNKNOWN"**) with **NULL** before converting to **DATE** format.

# [Data cleaning](DataCleaning.md)