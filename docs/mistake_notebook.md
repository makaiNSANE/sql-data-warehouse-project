## 🚫 Mistake Notebook

This page serves to act as a personal log for mistakes I made along the creation process of the data warehouse, while asking three questions that begin the process of rectifying said mistake within my work.
---

## ❓ First Mistake

1. My first mistake within this project happened as I was creating the **Silver Layer** of the overall pipeline, cleaning the old data gathered within the **Bronze Layer**. Within this line of code:
---
 ```
SELECT
cst_id,
cst_key,
TRIM(cst_firstname) AS cst_firstname,
TRIM(cst_lastname) AS cst_lastname,
cst_marital_status,
cst_gndr,
cst_create_date,
*
FROM (
SELECT
*,
ROW_NUMBER() OVER (PARTITION BY cst_id ORDER BY cst_create_date DESC) as flag_last
FROM bronze.crm_cust_info
) as t
WHERE flag_last = 1 AND cst_id = 29466

```
---

The mistake I made came from leaving both ',' after 'cst_create_date', as well as '*' on the line after 'cst_create_date' within my code before I executed, '*' within SQL, pulls in every single column from the subquery, including the untrimmed columns as well.
So within these lines of code, I essentially called both the untrimmed 'cst_firstname & cst_lastname' and the trimmed versions at the same time, resulting in an error.
---
## Understanding why the First Mistake Happened

1. This mistake happened due to simply me misremembering the proper syntax rules that SQL follows when trimming columns or calling queries for certain columns.
---

## Rectifying the First Mistake

1. Simply understanding the SQL syntax rules better and double checking or running a tool to potentially triple check my code is necessary to avoiding simple syntax errors.
---

## ❓ The Second Mistake

2. The second mistake I made was when I attempted to create the stored processes for truncating and loading the silver tables in order to prepare them for the **Gold Layer** within the pipeline.
   After attempting to run my code, I ran into this error:
---
"why is this message popping up: 


```
Msg 217, Level 16, State 1, Procedure silver.load_silver, Line 3 [Batch Start Line 2]
Maximum stored procedure, function, trigger, or view nesting level exceeded (limit 32).
```
---
The mistake I made came from leaving the 'EXEC. silver.load_silver' line within this set of code:
---
	-- Stored Processes (Silver Tables)
	CREATE OR ALTER PROCEDURE silver.load_silver AS
	EXEC silver.load_silver
	BEGIN
	PRINT '>> Truncating Table: silver.crm_cust_info';
	TRUNCATE TABLE silver.crm_cust_info
	PRINT '>> Inserting Data Into: silver.crm_cust_info';
	INSERT INTO silver.crm_cust_info (
---
My code was essentially calling itself over and over, until it hit the hard limit of 32 stored procedures, resulting in the error above. This mistake does not need its own rectification section, as the solution is to simply either call that function highlighted entirely on its own,
or delete it entirely when trying to create the stored processes for truncating and loading tables within a data pipeline. It's a recursive function within the overall body of code.
---

## ❓ The Third Mistake

3. When attempting to call the corrected tables in order to create the **Gold Layer** facts & dimensions for the pipeline, SQL would refuse to call the previously created 'silver.load_silver' stored process
   in order to begin the creation of the next layer. This mistake was simply due to being in the 'master' database, instead of the 'DataWarehouse' database, entirely just me being unaware of which database I was on.
   But, this mistake further led me to discovering that there was duplicate data when calling for certain tables within my concatenation of the necessary information to create the gold layer.
   I found duplicate customer ids within my ERP Tables, designated as 'cid'. I noticed this after I tried to concatenate certain columns from the bronze and silver tables, as well as the stored process for the silver tables. using 'LEFT JOIN', which was the cleanest method of transforming multiple data sets into one cleaned, efficient dataset.
   ---

## Rectifying the Third Mistake

3. When rectifying my mistake, I ended up recognizing that I simply need to check whether the entire table has been duplicated by comparing an overall and distinct count against both the bronze and silver ERP tables and check for 'cst_ids' that were not NULL.
   I used these two lines of code to run said check in order to find out whether the previous info from both the bronze and silver table were actually truncated from the stored process of the silver tables and had the cleaned, proper data inserted after:
   ---
   -- How many times has load_silver actually been run / did TRUNCATE actually fire?
SELECT COUNT(*) FROM silver.crm_cust_info;

SELECT COUNT(DISTINCT cst_id) FROM bronze.crm_cust_info WHERE cst_id IS NOT NULL;
---
When I executed these lines, I found out that the stored process essentially did not complete, and therefore the tables were not actually truncated. I ran the process twice before figuing out the solution, so there is the reason for this mistake occuring in the first place.
To rectify this, I simply opened a new query, copy/pasted my old lines of code for 'silver.load_silver', and made explicit note of seeing that 'TRUNCATE TABLE silver.crm_cust_info' actually ran and completed.
I then ran this check in order to verify that the truncate function was successful, and it absolutely was:
---
SELECT COUNT(*) FROM silver.crm_cust_info;
SELECT COUNT(DISTINCT cst_id) FROM bronze.crm_cust_info WHERE cst_id IS NOT NULL;
---
This is the end of this version of my mistake notebook for the data warehouse project. Again, this is simply to serve as a learning/logging tool for my mistakes so that I and anyone else who is looking to create this project will avoid these mistakes in the future, further improving our skills, because learning is not done when everything goes right, learning happens when you make the mistakes and make a focused effort to learn from them.

Thank You.





