# Data Cleaning

According to conclusions from [EDA](EDA.md#summary) we are going through data cleaning process column by column:

## 1. Create staging table

```sql
SELECT *
INTO staging.CafeSalesStg
FROM raw.CafeSalesSrc
```

## 2. `Transaction_ID` Column

### 2.1 Create columns `Transaction_ID_Prefix` and `Transaction_ID_Num`

```sql
ALTER TABLE staging.CafeSalesStg
ADD Transaction_ID_Prefix CHAR(3),
    Transaction_ID_Num INT;
```

**Results**

| Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date | Transaction_ID_Prefix | Transaction_ID_Num |
|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|-----------------------|--------------------|
| TXN_1000555    | Tea      | 1        | 1.5            | 1.5         | Credit Card    | In-store  | 2023-10-19       | NULL                  | NULL               |
| TXN_1001832    | Salad    | 2        | 5.0            | 10.0        | Cash           | Takeaway  | UNKNOWN          | NULL                  | NULL               |
| TXN_1002457    | Cookie   | 5        | 1.0            | 5.0         | Digital Wallet | Takeaway  | 2023-09-29       | NULL                  | NULL               |
| TXN_1003246    | Juice    | 2        | 3.0            | 6.0         | NULL           | NULL      | 2023-02-15       | NULL                  | NULL               |
| TXN_1004184    | Smoothie | 1        | 4.0            | 4.0         | Credit Card    | In-store  | 2023-05-18       | NULL                  | NULL               |
| TXN_1004563    | Tea      | 5        | 1.5            | 7.5         | Credit Card    | In-store  | 2023-10-28       | NULL                  | NULL               |
| TXN_1005331    | Coffee   | 1        | 2.0            | 2.0         | Digital Wallet | Takeaway  | 2023-11-04       | NULL                  | NULL               |
| TXN_1005377    | Cake     | 5        | UNKNOWN        | 15.0        | Digital Wallet | Takeaway  | 2023-06-03       | NULL                  | NULL               |
| TXN_1005472    | Coffee   | 4        | 2.0            | 8.0         | Credit Card    | NULL      | 2023-04-21       | NULL                  | NULL               |
| TXN_1006942    | Salad    | 1        | 5.0            | 5.0         | Credit Card    | In-store  | 2023-11-30       | NULL                  | NULL               |

### 2.2 Populating new columns

**Transaction_ID_Prefix**
To populate this column we have to extract all charakters before "_" sign by following query:

```sql
SELECT SUBSTRING(Transaction_ID, 1, CHARINDEX('_', Transaction_ID) - 1) As Prefix
FROM staging.CafeSalesStg
```
And then fill the column with values by following query:

```sql
UPDATE staging.CafeSalesStg
SET Transaction_ID_Prefix = SUBSTRING(Transaction_ID, 1, CHARINDEX('_', Transaction_ID) - 1)
```

**Transaction_ID_Num** <BR>
To populate this column we have to extract all charakters after "_" sign by following query:

```sql
SELECT SUBSTRING(Transaction_ID, CHARINDEX('_', Transaction_ID) + 1, LEN(Transaction_ID)) as id_numbers
FROM staging.CafeSalesStg
```
And then fill the column with values by following query:

```sql
UPDATE staging.CafeSalesStg
SET Transaction_ID_Num = SUBSTRING(Transaction_ID, CHARINDEX('_', Transaction_ID) + 1, LEN(Transaction_ID));
```

### 2.3 Delete Original column

After populate new columns we can delete the original column from table by following query:

```sql
ALTER TABLE staging.CafeSalesStg
DROP COLUMN Transaction_ID
```
And change name of `Transaction_ID_Num` column for Original Name 

```sql
EXEC sp_rename 'staging.CafeSalesStg.Transaction_ID_Num', 'Transaction_ID';
```
**Result**

| Transaction_ID_Prefix | Transaction_ID | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|-----------------------|----------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| TXN                   | 1000555        | Tea      | 1        | 1.5            | 1.5         | Credit Card    | In-store  | 2023-10-19       |
| TXN                   | 1001832        | Salad    | 2        | 5.0            | 10.0        | Cash           | Takeaway  | UNKNOWN          |
| TXN                   | 1002457        | Cookie   | 5        | 1.0            | 5.0         | Digital Wallet | Takeaway  | 2023-09-29       |
| TXN                   | 1003246        | Juice    | 2        | 3.0            | 6.0         | NULL           | NULL      | 2023-02-15       |
| TXN                   | 1004184        | Smoothie | 1        | 4.0            | 4.0         | Credit Card    | In-store  | 2023-05-18       |
| TXN                   | 1004563        | Tea      | 5        | 1.5            | 7.5         | Credit Card    | In-store  | 2023-10-28       |
| TXN                   | 1005331        | Coffee   | 1        | 2.0            | 2.0         | Digital Wallet | Takeaway  | 2023-11-04       |
| TXN                   | 1005377        | Cake     | 5        | UNKNOWN        | 15.0        | Digital Wallet | Takeaway  | 2023-06-03       |
| TXN                   | 1005472        | Coffee   | 4        | 2.0            | 8.0         | Credit Card    | NULL      | 2023-04-21       |
| TXN                   | 1006942        | Salad    | 1        | 5.0            | 5.0         | Credit Card    | In-store  | 2023-11-30       |

## 3. Text columns

Before replace values let's create second stage table by following query:

```sql
SELECT *
INTO staging.CafeSalesStg2
FROM staging.CafeSalesStg
```

### 3.1 Replace "ERROR" and NULL with "UNKNOWN" in `Item` column

```sql
UPDATE staging.CafeSalesStg2
SET Item = 'UNKNOWN'
WHERE Item = 'ERROR' OR Item IS NULL
```
### 3.2 Replace "ERROR" and NULL with "UNKNOWN" in `Payment_method` column

```sql
UPDATE staging.CafeSalesStg2
SET Payment_Method = 'UNKNOWN'
WHERE Payment_Method = 'ERROR' OR Payment_Method IS NULL
```
### 3.3 Replace "ERROR" and NULL with "UNKNOWN" in Location column

```sql
UPDATE staging.CafeSalesStg2
SET [Location] = 'UNKNOWN'
WHERE [Location] = 'ERROR' OR [Location] IS NULL
```
## 4. Numeric columns

As it was mentioned in [EDA](EDA.md#summary) numeric columns should be converted "ERROR" and "UNKNOWN" to NULL and recalculate values where possible.

### 4.1 Quantity column

Before replace values let's create second stage table by following query:

```sql
SELECT *
INTO staging.CafeSalesStg3
FROM staging.CafeSalesStg2
```

First we check rows with values to replace by following query:

```sql
SELECT TOP 10 * 
FROM staging.CafeSalesStg3
WHERE Quantity IN ('ERROR','UNKNOWN') OR Quantity IS NULL
```
**Result**
| Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date | Transaction_ID_Prefix | Transaction_ID |
|----------|----------|----------------|-------------|----------------|-----------|------------------|-----------------------|----------------|
| Cookie   | ERROR    | 1.0            | 1.0         | Digital Wallet | Takeaway  | 2023-01-07       | TXN                   | 1010950        |
| Tea      | ERROR    | 1.5            | 4.5         | UNKNOWN        | UNKNOWN   | UNKNOWN          | TXN                   | 1022126        |
| Cookie   | UNKNOWN  | 1.0            | 5.0         | UNKNOWN        | UNKNOWN   | 2023-02-06       | TXN                   | 1056277        |
| Juice    | UNKNOWN  | 3.0            | 6.0         | Credit Card    | UNKNOWN   | 2023-06-28       | TXN                   | 1062822        |
| UNKNOWN  | UNKNOWN  | NULL           | 9.0         | Digital Wallet | In-store  | 2023-12-13       | TXN                   | 1082717        |
| Cookie   | UNKNOWN  | 1.0            | 3.0         | Credit Card    | In-store  | 2023-01-07       | TXN                   | 1090854        |
| Smoothie | UNKNOWN  | 4.0            | 20.0        | Digital Wallet | UNKNOWN   | 2023-04-20       | TXN                   | 1101527        |
| UNKNOWN  | UNKNOWN  | 3.0            | 15.0        | Cash           | UNKNOWN   | NULL             | TXN                   | 1166001        |
| Sandwich | NULL     | 4.0            | 12.0        | Credit Card    | UNKNOWN   | 2023-01-16       | TXN                   | 1204426        |
| UNKNOWN  | ERROR    | NULL           | 20.0        | Credit Card    | UNKNOWN   | 2023-08-19       | TXN                   | 1208561        |

As we can see some values in `Quantity` column can be calculated if `Price_Per_Unit` column and `Total_Spent` column are valid.
We can count how much of them we can calculate by following query:

- **Total rows to replace**
```sql
SELECT COUNT(*) as rows_to_replace
FROM staging.CafeSalesStg3 
WHERE Quantity IN ('ERROR','UNKNOWN') OR Quantity IS NULL
```
**Result**
| rows_to_replace |
|-----------------|
| 479             |

- **Total rows which can be calculated**
```sql
SELECT COUNT(*) as rows_to_replace
FROM staging.CafeSalesStg3
WHERE 
	(Quantity IN ('ERROR','UNKNOWN') OR Quantity IS NULL)
	AND
	(Price_Per_Unit NOT IN ('ERROR','UNKNOWN') OR Price_Per_Unit IS NOT NULL)
	AND
	(Total_Spent  NOT IN ('ERROR','UNKNOWN') OR Total_Spent IS NOT NULL)
```
**Result**
| rows_to_calculate |
|-----------------|
| 441          |

The most of missing values can be calculated.

**Validate Calculation**

By following query we can calculate missing values in `Quantity` column by dividing `Total_Spent` by `Price_Per_Unit`
```sql
SELECT
    Quantity,
    CAST (CAST(Total_Spent AS DECIMAL(9,2)) / CAST(Price_Per_Unit AS DECIMAL(9,2)) AS SMALLINT) AS CalculatedQuantity,
    Price_Per_Unit,
    Total_Spent
FROM staging.CafeSalesStg3
WHERE 
	(Quantity IN ('ERROR','UNKNOWN') OR Quantity IS NULL)
	AND
	(Price_Per_Unit NOT IN ('ERROR','UNKNOWN') AND Price_Per_Unit IS NOT NULL)
	AND
	(Total_Spent  NOT IN ('ERROR','UNKNOWN') AND Total_Spent IS NOT NULL)
```
**Result**

| Quantity | CalculatedQuantity | Price_Per_Unit | Total_Spent |
|----------|--------------------|----------------|-------------|
| ERROR    | 1                  | 1.0            | 1.0         |
| ERROR    | 3                  | 1.5            | 4.5         |
| UNKNOWN  | 5                  | 1.0            | 5.0         |
| UNKNOWN  | 2                  | 3.0            | 6.0         |
| UNKNOWN  | 3                  | 1.0            | 3.0         |
| UNKNOWN  | 5                  | 4.0            | 20.0        |
| UNKNOWN  | 5                  | 3.0            | 15.0        |
| NULL     | 3                  | 4.0            | 12.0        |
| UNKNOWN  | 3                  | 4.0            | 12.0        |
| NULL     | 5                  | 1.0            | 5.0         |

`Quantity` values are calculated properly, now we can update values by following query:

```sql
UPDATE staging.CafeSalesStg3
SET Quantity = CAST (CAST(Total_Spent AS DECIMAL(9,2)) / CAST(Price_Per_Unit AS DECIMAL(9,2)) AS SMALLINT)
WHERE 
	(Quantity IN ('ERROR','UNKNOWN') OR Quantity IS NULL)
	AND
	(Price_Per_Unit NOT IN ('ERROR','UNKNOWN') AND Price_Per_Unit IS NOT NULL)
	AND
	(Total_Spent  NOT IN ('ERROR','UNKNOWN') AND Total_Spent IS NOT NULL)
```
Now, we have left 38 rows which can't be calculated.

To be consistent and allow calculations we need to replace the **UNKNOWN** and **ERROR** values with **NULL** by following query:

```sql
UPDATE staging.CafeSalesStg3
SET Quantity = NULL
WHERE
	Quantity IN ('ERROR', 'UNKNOWN') OR Quantity IS NULL
```

### 4.2 `Price_Per_Unit` column

Before replace values let's create second stage table by following query:

```sql
SELECT *
INTO staging.CafeSalesStg4
FROM staging.CafeSalesStg3
```
Now let's count how many rows should be replaced
```sql
SELECT COUNT(*) as rows_to_replace
FROM staging.CafeSalesStg4 
WHERE Price_Per_Unit IN ('ERROR','UNKNOWN') OR Price_Per_Unit IS NULL
```
**Result**
| rows_to_replace |
|-----------------|
| 533             |

We also determine how many of them can be calculated by dividing `Total_Spent` by `Quantity`:
```sql
SELECT COUNT(*) as rows_to_calculate
FROM staging.CafeSalesStg4
WHERE 
    (Price_Per_Unit IN ('ERROR','UNKNOWN') OR Price_Per_Unit IS NULL)
    AND (Quantity NOT IN ('ERROR','UNKNOWN') AND Quantity IS NOT NULL)
    AND (Total_Spent NOT IN ('ERROR','UNKNOWN') AND Total_Spent IS NOT NULL)
```
**Result**
| rows_to_calculate |
|-----------------|
| 495             |

**Validate Calculation** <BR>
We should preview the calculated values before applying updates:
```sql
SELECT
    Price_Per_Unit,
    CAST (CAST(Total_Spent AS DECIMAL(9,2)) / CAST(Quantity AS SMALLINT) AS DECIMAL(9,2)) AS Calculated_Price_Per_Unit,
    Quantity,
    Total_Spent
FROM staging.CafeSalesStg4
WHERE 
    (Price_Per_Unit IN ('ERROR','UNKNOWN') OR Price_Per_Unit IS NULL)
    AND (Quantity NOT IN ('ERROR','UNKNOWN') AND Quantity IS NOT NULL)
    AND (Total_Spent NOT IN ('ERROR','UNKNOWN') AND Total_Spent IS NOT NULL)
```

**Result**
| Price_Per_Unit | Calculated_Price_Per_Unit | Quantity | Total_Spent |
|----------------|---------------------------|----------|-------------|
| NULL           | 4.00                      | 4        | 16.0        |
| UNKNOWN        | 2.00                      | 2        | 4.0         |
| UNKNOWN        | 3.00                      | 3        | 9.0         |
| UNKNOWN        | 4.00                      | 5        | 20.0        |
| ERROR          | 4.00                      | 1        | 4.0         |
| UNKNOWN        | 3.00                      | 5        | 15.0        |
| NULL           | 4.00                      | 3        | 12.0        |
| UNKNOWN        | 1.50                      | 3        | 4.5         |
| NULL           | 4.00                      | 1        | 4.0         |
| ERROR          | 3.00                      | 1        | 3.0         |

**Update `Price_Per_Unit` Values**

```sql
UPDATE staging.CafeSalesStg4
SET Price_Per_Unit = CAST (CAST(Total_Spent AS DECIMAL(9,2)) / CAST(Quantity AS DECIMAL(9,2)) AS DECIMAL(9,2))
WHERE 
    (Price_Per_Unit IN ('ERROR','UNKNOWN') OR Price_Per_Unit IS NULL)
    AND (Quantity NOT IN ('ERROR','UNKNOWN') AND Quantity IS NOT NULL)
    AND (Total_Spent NOT IN ('ERROR','UNKNOWN') AND Total_Spent IS NOT NULL)
```
Finally, let's set the remaining invalid values to **NULL**:
```sql
UPDATE staging.CafeSalesStg4
SET Price_Per_Unit = NULL
WHERE Price_Per_Unit IN ('ERROR', 'UNKNOWN') OR Price_Per_Unit IS NULL
```

### 4.3 `Total_Spent` Column

Before replace values let's create stage table by following query:

```sql
SELECT *
INTO staging.CafeSalesStg5
FROM staging.CafeSalesStg4
```
Now let's count how many rows should be replaced
```sql
SELECT COUNT(*) as rows_to_replace
FROM staging.CafeSalesStg5
WHERE Total_Spent IN ('ERROR','UNKNOWN') OR Total_Spent IS NULL
```
**Result**
| rows_to_replace |
|-----------------|
| 502             |

We also determine how many of them can be calculated by multiplying `Price_Per_Unit` by `Quantity`:
```sql
SELECT COUNT(*) AS rows_to_calculate
FROM staging.CafeSalesStg5
WHERE
    (Total_Spent IN ('ERROR', 'UNKNOWN') OR Total_Spent IS NULL)
    AND (Quantity IS NOT NULL AND Quantity NOT IN ('ERROR', 'UNKNOWN'))
    AND (Price_Per_Unit IS NOT NULL AND Price_Per_Unit NOT IN ('ERROR', 'UNKNOWN'));
```
**Result**
| rows_to_calculate |
|-----------------|
| 462            |

**Validate Calculation**
We can preview the calculated values before applying updates:
```sql
SELECT
    Total_Spent,
    CAST(Quantity AS SMALLINT) * CAST(Price_Per_Unit AS DECIMAL(9,2)) AS Calculated_Total_Spent,
    Quantity,
    Price_Per_Unit
FROM staging.CafeSalesStg5
WHERE
    (Total_Spent IN ('ERROR', 'UNKNOWN') OR Total_Spent IS NULL)
    AND (Quantity IS NOT NULL AND Quantity NOT IN ('ERROR', 'UNKNOWN'))
    AND (Price_Per_Unit IS NOT NULL AND Price_Per_Unit NOT IN ('ERROR', 'UNKNOWN'));
```

**Result**
| Total_Spent | Calculated_Total_Spent | Quantity | Price_Per_Unit |
|-------------|------------------------|----------|----------------|
| NULL        | 3.00                   | 3        | 1.0            |
| UNKNOWN     | 3.00                   | 1        | 3.0            |
| NULL        | 9.00                   | 3        | 3.0            |
| NULL        | 10.00                  | 2        | 5.0            |
| NULL        | 5.00                   | 5        | 1.0            |
| UNKNOWN     | 4.00                   | 2        | 2.0            |
| UNKNOWN     | 4.00                   | 1        | 4.0            |
| NULL        | 7.50                   | 5        | 1.5            |
| NULL        | 12.00                  | 3        | 4.0            |
| NULL        | 1.50                   | 1        | 1.5            |

**Update `Total_Spent` Values**

```sql
UPDATE staging.CafeSalesStg5
SET Total_Spent = CAST(Quantity AS SMALLINT) * CAST(Price_Per_Unit AS DECIMAL(9,2))
WHERE
    (Total_Spent IN ('ERROR', 'UNKNOWN') OR Total_Spent IS NULL)
    AND (Quantity IS NOT NULL AND Quantity NOT IN ('ERROR', 'UNKNOWN'))
    AND (Price_Per_Unit IS NOT NULL AND Price_Per_Unit NOT IN ('ERROR', 'UNKNOWN'));
```
Finally, set the remaining invalid values to NULL:
```sql
UPDATE staging.CafeSalesStg5
SET Total_Spent = NULL
WHERE
    Total_Spent IN ('ERROR', 'UNKNOWN') OR Total_Spent IS NULL;
```

## 5. `Transaction_Date` Column

Before replace values let's create stage table by following query:

```sql
SELECT *
INTO staging.CafeSalesStg6
FROM staging.CafeSalesStg5
```
First, we check rows where `Transaction_Date` has invalid values:

```sql
SELECT COUNT(*) as rows_to_replace
FROM staging.CafeSalesStg6 
WHERE Transaction_Date IN ('ERROR','UNKNOWN') OR Transaction_Date IS NULL
```
**Result**
| rows_to_replace |
|-----------------|
| 460             |

To replace **ERROR** and **UNKNOWN** with **NULL** we run the following query:

```sql
UPDATE staging.CafeSalesStg6
SET Transaction_Date = NULL
WHERE Transaction_Date IN ('ERROR', 'UNKNOWN')
```

Now we ensure all valid dates are properly formatted as YYYY-MM-DD by following query

```sql
UPDATE staging.CafeSalesStg6
SET Transaction_Date = CONVERT(DATE,Transaction_Date,23)
WHERE Transaction_Date IS NOT NULL
```

## 5.Conversion of Data types
After all we have prepared table to convert all the data types by following query;

```sql
CREATE TABLE staging.CafeSalesCleaned (
	Transaction_ID INT,
	Transaction_ID_Prefix CHAR(3),
	Item VARCHAR(20),
	Quantity SMALLINT,
	Price_Per_Unit DECIMAL(9,2),
	Total_Spent DECIMAL (9,2),
	Payment_Method VARCHAR(20),
	[Location] VARCHAR(20),
	Transaction_Date DATE
	)
```
Now we can populate whole table by following query:
```sql
INSERT INTO staging.CafeSalesCleaned (
    Transaction_ID,
    Transaction_ID_Prefix,
    Item,
    Quantity,
    Price_Per_Unit,
    Total_Spent,
    Payment_Method,
    [Location],
    Transaction_Date
)
SELECT
    CAST(Transaction_ID AS INT) AS Transaction_ID,
    CAST(Transaction_ID_Prefix AS CHAR(3)) AS Transaction_ID_Prefix,
    CAST(Item AS VARCHAR(20)) AS Item,
    CAST(Quantity AS SMALLINT) AS Quantity,
    CAST(Price_Per_Unit AS DECIMAL(9,2)) AS Price_Per_Unit,
    CAST(Total_Spent AS DECIMAL(9,2)) AS Total_Spent,
    CAST(Payment_Method AS VARCHAR(20)) AS Payment_Method,
    CAST([Location] AS VARCHAR(20)) AS [Location],
    CAST(Transaction_Date AS DATE) AS Transaction_Date
FROM staging.CafeSalesStg6;
```
**Result:**

| Transaction_ID | Transaction_ID_Prefix | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|-----------------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| 9853220        | TXN                   | Cookie   | 5        | 1.00           | 5.00        | Credit Card    | In-store  | 2023-04-02       |
| 9853577        | TXN                   | Sandwich | 2        | 4.00           | 8.00        | Credit Card    | In-store  | 2023-04-30       |
| 9855567        | TXN                   | Cookie   | 3        | 1.00           | 3.00        | Credit Card    | UNKNOWN   | 2023-05-25       |
| 9857674        | TXN                   | Tea      | 5        | 1.50           | 7.50        | UNKNOWN        | Takeaway  | 2023-06-12       |
| 9857832        | TXN                   | Tea      | 3        | 1.50           | 4.50        | Cash           | In-store  | 2023-03-31       |
| 9858152        | TXN                   | Smoothie | 2        | 4.00           | 8.00        | Digital Wallet | In-store  | 2023-01-28       |
| 9858627        | TXN                   | Juice    | 4        | 3.00           | 12.00       | UNKNOWN        | UNKNOWN   | 2023-01-22       |
| 9859715        | TXN                   | Smoothie | 5        | 4.00           | 20.00       | UNKNOWN        | UNKNOWN   | 2023-07-11       |
| 9860308        | TXN                   | Smoothie | 2        | 4.00           | 8.00        | Digital Wallet | UNKNOWN   | 2023-02-20       |
| 9861083        | TXN                   | Sandwich | 5        | 4.00           | 20.00       | Credit Card    | In-store  | 2023-01-21       |
| 9861432        | TXN                   | Cookie   | 2        | 1.00           | 2.00        | UNKNOWN        | Takeaway  | 2023-12-09       |
| 9861782        | TXN                   | Sandwich | 4        | 4.00           | 16.00       | Credit Card    | In-store  | 2023-07-15       |
| 9863114        | TXN                   | Juice    | 5        | 3.00           | 15.00       | UNKNOWN        | Takeaway  | 2023-02-01       |
| 9864212        | TXN                   | Smoothie | 5        | 4.00           | 20.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 9864890        | TXN                   | Sandwich | 3        | 4.00           | 12.00       | UNKNOWN        | In-store  | 2023-12-06       |
| 9865855        | TXN                   | Coffee   | 3        | 2.00           | 6.00        | Digital Wallet | Takeaway  | 2023-04-18       |
| 9866728        | TXN                   | Juice    | 1        | 3.00           | 3.00        | Cash           | In-store  | 2023-10-26       |
| 9867586        | TXN                   | Smoothie | 4        | 4.00           | 16.00       | UNKNOWN        | In-store  | 2023-06-02       |
| 9867948        | TXN                   | Smoothie | 2        | 4.00           | 8.00        | Digital Wallet | UNKNOWN   | 2023-10-07       |
| 9868053        | TXN                   | Juice    | 3        | 3.00           | 9.00        | Digital Wallet | In-store  | NULL             |

## 6. Can we populate some more missing data?

Lest's take a look at the table from following query:

```sql
SELECT TOP 20* 
from staging.CafeSalesCleaned
WHERE 
		Item = 'UKNKNOWN' OR
		Quantity IS NULL OR
		Price_Per_Unit IS NULL OR
		Total_Spent IS NULL OR
		Payment_Method = 'UKNKNOWN' OR
		[Location] = 'UKNKNOWN' OR
		Transaction_Date IS NULL
```
| Transaction_ID | Transaction_ID_Prefix | Item     | Quantity | Price_Per_Unit | Total_Spent | Payment_Method | Location  | Transaction_Date |
|----------------|-----------------------|----------|----------|----------------|-------------|----------------|-----------|------------------|
| 9864212        | TXN                   | UNKNOWN  | 5        | 4.00           | 20.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 9868053        | TXN                   | Juice    | 3        | 3.00           | 9.00        | Digital Wallet | In-store  | NULL             |
| 9868954        | TXN                   | UNKNOWN  | 4        | 4.00           | 16.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 9877722        | TXN                   | Sandwich | 5        | 4.00           | 20.00       | Digital Wallet | In-store  | NULL             |
| 9882485        | TXN                   | UNKNOWN  | 5        | 5.00           | 25.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 9894270        | TXN                   | Cookie   | 4        | 1.00           | 4.00        | Credit Card    | In-store  | NULL             |
| 9924421        | TXN                   | UNKNOWN  | 5        | 3.00           | 15.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 9924732        | TXN                   | Sandwich | NULL     | 4.00           | NULL        | Credit Card    | In-store  | 2023-01-18       |
| 9930357        | TXN                   | UNKNOWN  | 1        | 3.00           | 3.00        | UNKNOWN        | UNKNOWN   | NULL             |
| 9944500        | TXN                   | Smoothie | NULL     | 4.00           | NULL        | Cash           | In-store  | 2023-01-03       |
| 9958548        | TXN                   | UNKNOWN  | 4        | 1.50           | 6.00        | UNKNOWN        | UNKNOWN   | NULL             |
| 9999124        | TXN                   | Juice    | 2        | 3.00           | 6.00        | Digital Wallet | Takeaway  | NULL             |
| 1001832        | TXN                   | Salad    | 2        | 5.00           | 10.00       | Cash           | Takeaway  | NULL             |
| 1022126        | TXN                   | UNKNOWN  | 3        | 1.50           | 4.50        | UNKNOWN        | UNKNOWN   | NULL             |
| 1026799        | TXN                   | UNKNOWN  | 5        | 3.00           | 15.00       | UNKNOWN        | UNKNOWN   | NULL             |
| 1034956        | TXN                   | Sandwich | 4        | 4.00           | 16.00       | Cash           | In-store  | NULL             |
| 1037486        | TXN                   | Salad    | 5        | 5.00           | 25.00       | Digital Wallet | In-store  | NULL             |
| 1082717        | TXN                   | UNKNOWN  | NULL     | NULL           | 9.00        | UNKNOWN        | UNKNOWN   | 2023-12-13       |
| 1093800        | TXN                   | Sandwich | 3        | 4.00           | 12.00       | Cash           | Takeaway  | NULL             |
| 1102876        | TXN                   | UNKNOWN  | 1        | 1.00           | 1.00        | UNKNOWN        | UNKNOWN   | NULL             |

**Missing Prices** <BR>
As you can see some products like the sandwich do not have a price however we know the prices of these products. We can get the price list from the table using the following query.

```sql
SELECT DISTINCT
		Item,
		Price_Per_Unit
FROM staging.CafeSalesCleaned
WHERE 
	Item <> ('UNKNOWN') AND
	Item IS NOT NULL AND
	Price_Per_Unit IS NOT NULL
Order By Price_Per_Unit ASC
```
**Result - Product List Price**
| Item     | Price_Per_Unit |
|----------|----------------|
| Cookie   | 1.00           |
| Tea      | 1.50           |
| Coffee   | 2.00           |
| Juice    | 3.00           |
| Cake     | 3.00           |
| Smoothie | 4.00           |
| Sandwich | 4.00           |
| Salad    | 5.00           |

Now, we know the price of the products but not every missing values can be determined by this product list price because we can notice that some products has the same price so we don't know which of these products are missing. <br>
List of products that can be filled:

| Item     | Price_Per_Unit |
|----------|----------------|
| Cookie   | 1.00           |
| Tea      | 1.50           |
| Coffee   | 2.00           |
| Salad    | 5.00           |

So now we can update `Price_Per_Unit` by following query:

```sql
UPDATE staging.CafeSalesCleaned
SET Price_Per_Unit = (
    SELECT Price_Per_Unit
    FROM (VALUES
        ('Cookie', 1.00),
        ('Tea', 1.50),
        ('Coffee', 2.00),
		('Juice', 3.00),
		('Cake', 3.00),
		('Smoothie', 4.00),
		('Sandwich', 4.00),
        ('Salad', 5.00)
    ) AS PriceList(Item, Price_Per_Unit)
    WHERE PriceList.Item = staging.CafeSalesCleaned.Item
)
WHERE 
    Price_Per_Unit IS NULL;
```
Let's check again missing values in Numeric columns by following query:

```sql
SELECT TOP 10
	Item,
	Price_Per_Unit,
	Quantity,
	Total_Spent
FROM staging.CafeSalesCleaned
WHERE 
	Item <> 'UNKNOWN' AND
	(Quantity IS NULL OR
	Price_Per_Unit IS NULL OR
	Total_Spent IS NULL
	)
ORDER BY Price_Per_Unit ASC
```
**Result**
| Item   | Price_Per_Unit | Quantity | Total_Spent |
|--------|----------------|----------|-------------|
| Cookie | 1.00           | 5        | NULL        |
| Cookie | 1.00           | NULL     | NULL        |
| Cookie | 1.00           | 2        | NULL        |
| Cookie | 1.00           | NULL     | 1.00        |
| Cookie | 1.00           | NULL     | NULL        |
| Cookie | 1.00           | NULL     | 1.00        |
| Cookie | 1.00           | 2        | NULL        |
| Cookie | 1.00           | NULL     | 2.00        |
| Tea    | 1.50           | 3        | NULL        |
| Tea    | 1.50           | NULL     | NULL        |

We have some more informations about prices so we can recalculate:
- `Total_Spent` based on `Price_Per_Unit` and `Quantity` if both exists
- `Quantity` based on `Price_Per_Unit` and `Total_Spent` if both exists

**Total_Spent**
```sql
UPDATE staging.CafeSalesCleaned
SET Total_Spent = Quantity * Price_Per_Unit
WHERE 
    Total_Spent IS NULL AND
    (Quantity IS NOT NULL AND
	Price_Per_Unit IS NOT NULL
	)
```
**Quantity**

```sql
UPDATE staging.CafeSalesCleaned
SET Quantity = Total_Spent / Price_Per_Unit
WHERE 
    Quantity IS NULL AND
    (Total_Spent IS NOT NULL AND
	Price_Per_Unit IS NOT NULL);
```

**Missing Items** <BR>

Additionaly we can refill missing items based on the product price list by following query:

```sql
UPDATE staging.CafeSalesCleaned
SET Item = (
    SELECT Item
    FROM (VALUES
        ('Cookie', 1.00),
        ('Tea', 1.50),
        ('Coffee', 2.00),
        ('Salad', 5.00)
    ) AS PriceList(Item, Price_Per_Unit)
    WHERE PriceList.Price_Per_Unit = staging.CafeSalesCleaned.Price_Per_Unit
)
WHERE 
	Item = 'UNKNOWN' AND Price_Per_Unit IN (1.0,1.5,2.0,5.0)
```

## 7. Summary

### Item column 
Let's compare the amount of rows that were fixed in by following query:

```sql
WITH ItemCounts AS (
    SELECT 
        COUNT(*) AS Total_Before_Clean
    FROM raw.CafeSalesSrc 
    WHERE Item IN ('UNKNOWN', 'ERROR') OR Item IS NULL
),
CleanedItemCounts AS (
    SELECT 
        COUNT(*) AS Total_After_Clean
    FROM staging.CafeSalesCleaned 
    WHERE Item IN ('UNKNOWN', 'ERROR') OR Item IS NULL
)
SELECT 
    i.Total_Before_Clean,
    c.Total_After_Clean,
    i.Total_Before_Clean - c.Total_After_Clean AS fixed_rows
FROM ItemCounts i, CleanedItemCounts c;
```
**Result**

| Total_Before_Clean | Total_After_Clean | fixed_rows |
|--------------------|-------------------|------------|
| 969                | 480               | 489        |

### Quantity column

```sql
WITH QuantityCounts AS (
    SELECT 
        COUNT(*) AS Total_Before_Clean
    FROM raw.CafeSalesSrc 
    WHERE Quantity IN ('UNKNOWN', 'ERROR') OR Quantity IS NULL
),
CleanedQuantityCounts AS (
    SELECT 
        COUNT(*) AS Total_After_Clean
    FROM staging.CafeSalesCleaned 
    WHERE Quantity IS NULL
)
SELECT 
    q.Total_Before_Clean,
    c.Total_After_Clean,
    q.Total_Before_Clean - c.Total_After_Clean AS fixed_rows
FROM QuantityCounts q, CleanedQuantityCounts c;

```
**Result**

| Total_Before_Clean | Total_After_Clean | fixed_rows |
|--------------------|-------------------|------------|
| 479                | 23                | 456        |


### Price_Per_Unit column

```sql
WITH PricePerUnitCounts AS (
    SELECT 
        COUNT(*) AS Total_Before_Clean
    FROM raw.CafeSalesSrc 
    WHERE Price_Per_Unit IN ('UNKNOWN', 'ERROR') OR Price_Per_Unit IS NULL
),
CleanedPricePerUnitCounts AS (
    SELECT 
        COUNT(*) AS Total_After_Clean
    FROM staging.CafeSalesCleaned 
    WHERE Price_Per_Unit IS NULL
)
SELECT 
    p.Total_Before_Clean,
    c.Total_After_Clean,
    p.Total_Before_Clean - c.Total_After_Clean AS fixed_rows
FROM PricePerUnitCounts p, CleanedPricePerUnitCounts c;

```
**Result**

| Total_Before_Clean | Total_After_Clean | fixed_rows |
|--------------------|-------------------|------------|
| 533                | 6                 | 527        |

### Total_Spent Column

```sql
WITH TotalSpentCounts AS (
    SELECT 
        COUNT(*) AS Total_Before_Clean
    FROM raw.CafeSalesSrc 
    WHERE Total_Spent IN ('UNKNOWN', 'ERROR') OR Total_Spent IS NULL
),
CleanedTotalSpentCounts AS (
    SELECT 
        COUNT(*) AS Total_After_Clean
    FROM staging.CafeSalesCleaned 
    WHERE Total_Spent IS NULL
)
SELECT 
    p.Total_Before_Clean,
    c.Total_After_Clean,
    p.Total_Before_Clean - c.Total_After_Clean AS fixed_rows
FROM TotalSpentCounts p, CleanedTotalSpentCounts c;
```

**Result**

| Total_Before_Clean | Total_After_Clean | fixed_rows |
|--------------------|-------------------|------------|
| 502                | 23                | 479        |


# [Data Analysis](/DataAnalysis.md)
