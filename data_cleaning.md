# Data Cleaning Process
This document walks through the full data cleaning process for the Idaho Elk Harvest Analysis project, from raw CSV files to a clean, analysis-ready table in MySQL.

--- 

## Table of Contents 
- [Step 1: Google Sheets - Compiling and Preparing Data](#step-1-google-sheets---compiling-and-preparing-data)
- [Step 2: Adding a Primary Key](#step-2-adding-a-primary-key)
- [Step 3: Replacing Blank Entries with NULL](#step-3-replacing-blank-entries-with-null)
- [Step 4: Checking for Remaining Nulls](#step-4-checking-for-remaining-nulls)
- [Step 5: Modifying Column Data Types](#step-5-modifying-column-data-types)
- [Step 6: Fixing the Import — Loading Data via Terminal](#step-6-fixing-the-import-loading-data-via-terminal)
- [Step 7: Verifying the Import](#step-7-verifying-the-import)
- [Step 8: Final Cleaning and Validation](#step-8-final-cleaning-and-validation)

---

## Step 1: Google Sheets - Compiling and Preparing Data 
The raw data from the Idaho Department of Fish and Game was supplied as individual CSV files, one per year. Before bringing the data into SQL, I handled some foundational prep work in Google Sheets:

- Combined all CSVs into one master file so the entire dataset (2001–2024) could be analyzed together
- Renamed columns to be SQL-friendly: all lowercase, no spaces, and no special characters (e.g., % symbols were replaced with _rate to indicate they represented rate values)
- Added a year column to each row for clarity, since the year was previously only implied by which file the data came from

---

## Step 2: Adding a Primary Key
With the data loaded into MySQL, the first thing I did was add an id column to give every row a unique identifier. This is good practice and makes troubleshooting much easier down the line.
```sql
ALTER TABLE elk_harvest
ADD COLUMN id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY FIRST;
```

---

## Step 3: Replacing Blank Entries with NULL
The dataset had blank entries in several columns. Instead of making these values zero, or empty strings, I wanted them to represent a null value.
```sql
UPDATE elk_harvest
SET six_point_plus_rate = NULL WHERE six_point_plus_rate = '';

UPDATE elk_harvest
SET spike_rate = NULL WHERE spike_rate = '';

UPDATE elk_harvest
SET antlerless = NULL WHERE antlerless = '';

UPDATE elk_harvest
SET antlered = NULL WHERE antlered = '';

UPDATE elk_harvest
SET hunters = NULL WHERE hunters = '';

UPDATE elk_harvest
SET harvest = NULL WHERE harvest = '';

UPDATE elk_harvest
SET success_rate = NULL WHERE success_rate = '';
```

---

## Step 4: Checking for Remaining Nulls
After updating the blank values, I ran a check to confirm everything that should be Null was listed as such. I also wanted to see if there was any missing data across the key columns.
```sql
SELECT id, harvest, hunters, antlered, antlerless, six_point_plus_rate, success_rate, spike_rate
FROM elk_harvest
WHERE harvest IS NULL
   OR hunters IS NULL
   OR antlered IS NULL
   OR antlerless IS NULL
   OR six_point_plus_rate IS NULL
   OR success_rate IS NULL
   OR spike_rate IS NULL;
```

---

## Step 5: Modifying Column Data Types
The columns needed proper data types assigned to ensure accurate calculations and storage. Rate columns were set to Decimal for precision and count columns were set to INT.
```sql
ALTER TABLE elk_harvest
MODIFY take_method VARCHAR(50) NOT NULL,
MODIFY region_unit VARCHAR(10) NOT NULL,
MODIFY year INT NOT NULL,
MODIFY harvest INT NULL,
MODIFY hunters INT NULL,
MODIFY antlered INT NULL,
MODIFY antlerless INT NULL,
MODIFY six_point_plus_rate DECIMAL(5,2) NULL,
MODIFY success_rate DECIMAL(5,2) NULL,
MODIFY spike_rate DECIMAL(5,2) NULL;
```
I then confirmed the data types looked correct: 
```sql
DESCRIBE elk_harvest;
```
And did a quick spot check on the data: 
```sql
SELECT * FROM elk_harvest LIMIT 20;
```

---

## Step 6: Fixing the Import (Loading Data via Terminal)
This was one of the trickier parts of the process. MySQL Workbench was silently dropping rows that contained blank values during the initial import, which meant I was losing data without realizing it. Since those blanks needed to be NULL (not zero), I had to re-import the data directly through the terminal using LOAD DATA LOCAL INFILE, which gave me more control over how blank values were handled on the way in.
```sql
LOAD DATA LOCAL INFILE "/Users/shannonjoyce/Desktop/elk_harvest - tableExport.csv"
INTO TABLE elk_harvest
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS
(year, take_method, region_unit, harvest, hunters, success_rate,
 days_hunted, antlered, antlerless, spike_rate, six_point_plus_rate)
SET
  harvest = NULLIF(harvest, ''),
  hunters = NULLIF(hunters, ''),
  success_rate = NULLIF(success_rate, ''),
  days_hunted = NULLIF(days_hunted, ''),
  antlered = NULLIF(antlered, ''),
  antlerless = NULLIF(antlerless, ''),
  spike_rate = NULLIF(spike_rate, ''),
  six_point_plus_rate = NULLIF(six_point_plus_rate, '');
```
The NULLIF(column, '') function tells MySQL to treat any empty string as NULL at the point of import, preventing rows from being dropped.

---

## Step 7: Verifying the Import 
Once the re-import was complete, I ran a series of checks to confirm everything looked right.
#### Check total row count: 
```sql
SELECT COUNT(*) FROM elk_harvest;
```
#### Confirm all hunting units were imported correctly:
```sql
SELECT DISTINCT region_unit FROM elk_harvest ORDER BY region_unit;
```
#### Spot check that NULL values were applied correctly:
```sql
SELECT * FROM elk_harvest WHERE antlerless IS NULL LIMIT 5;
```

---

## Step 8: Final Cleaning and Validation
With the data fully imported, I completed the remaining clean up steps.
#### Re-added the id column 
```sql
ALTER TABLE harvest_data
ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY;
```
#### Reassigned column data types to finalize the schema:
```sql
ALTER TABLE `idaho_elk_harvest`.`elk_harvest`
CHANGE COLUMN `year` `year` INT NOT NULL,
CHANGE COLUMN `take_method` `take_method` VARCHAR(50) NOT NULL,
CHANGE COLUMN `region_unit` `region_unit` VARCHAR(10) NOT NULL,
CHANGE COLUMN `success_rate` `success_rate` DECIMAL(5,2) NULL DEFAULT NULL,
CHANGE COLUMN `antlered` `antlered` INT NULL DEFAULT NULL,
CHANGE COLUMN `spike_rate` `spike_rate` DECIMAL(5,2) NULL DEFAULT NULL,
CHANGE COLUMN `six_point_plus_rate` `six_point_plus_rate` DECIMAL(5,2) NULL DEFAULT NULL;
```
#### Checked for duplicate rows zero duplicates returned, all good:
```sql
SELECT year, region_unit, take_method, hunters, harvest, days_hunted,
       success_rate, antlered, antlerless, spike_rate, six_point_plus_rate,
       COUNT(*) AS duplicate_count
FROM elk_harvest
GROUP BY year, region_unit, take_method, hunters, harvest, days_hunted,
         success_rate, antlered, antlerless, spike_rate, six_point_plus_rate
HAVING COUNT(*) > 1;
```
#### Checked for unexpected NULLs in key columns
One row returned with a count of 0, all good:
```sql
SELECT COUNT(*) FROM elk_harvest
WHERE year IS NULL OR take_method IS NULL OR region_unit IS NULL;
```
#### Confirmed the year range was correct 
```sql
SELECT DISTINCT year FROM elk_harvest ORDER BY year;
```
At this point the data was clean, fully imported, and ready for analysis. 
See [sql_analysis.md](sql_analysis.md) for the queries used to answer the project's key questions.

