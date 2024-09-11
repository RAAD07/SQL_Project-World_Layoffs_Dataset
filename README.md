# SQL_Project_World-Layoffs-Dataset
This SQL project comprises Data Cleaning in the first part and Exploratory Data Analysis (EDA) in the second part.

## Part 1: Data Cleaning
In the Data Cleaning part, we will be focussing on 4 things.
1. Remove Duplicates
2. Standardize the data
3. Handle Null values or blank values
4. Remove Rows and Columns if necessary or if they are for no use

### Checking the raw data
``` sql
SELECT * FROM layoffs; #layoffs hold all the raw data
```
### Creating a duplicate table to work and protect the raw data
``` sql
CREATE TABLE layoffs_worksheet 
LIKE layoffs;

SELECT * FROM layoffs_worksheet;
```

### Inserting all the rows of the layoff table into the new layoffs_worksheet table
``` sql
INSERT layoffs_worksheet
SELECT * FROM layoffs;
```

### Creating another table to manipulate data and create extra columns

/*creating a new table named layoffs_worksheet2 by copying clipboard>create statement
and add one extra column named row_num*/

``` sql
CREATE TABLE `layoffs_worksheet2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT * FROM layoffs_worksheet2;
```
### Inserting values to this new table and adding an extra column

/* inserting values into this new table and assigning row_num by creating a partition on each column so that 
each unique row is assigned as 1 and if there are any duplicates then it will be assigned as 2 in the row_num */

``` sql
INSERT INTO layoffs_worksheet2
SELECT *,
ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) as row_num
FROM layoffs_worksheet;

SELECT * FROM layoffs_worksheet2;
```

## 1. Removing the duplicates

### Checking the duplicates
``` sql
SELECT * FROM layoffs_worksheet2
WHERE row_num > 1;
```

### Deleting the duplicates
``` sql
DELETE
FROM layoffs_worksheet2
WHERE row_num > 1;
```

### Rechecking whether there are still any duplicates or not
``` sql
SELECT * FROM layoffs_worksheet2
WHERE row_num>1;
```

## 2. Standardizing Columns

### Checking and Trimming extra spaces from company names
``` sql
SELECT company, TRIM(company) as company_name
FROM layoffs_worksheet2;
```

### Updating the company name in the database
``` sql
UPDATE layoffs_worksheet2
SET company = TRIM(company);
```

### Updating the location in the database
``` sql
UPDATE layoffs_worksheet2
SET location = TRIM(location);
```

### Updating the country name in the database
``` sql
UPDATE layoffs_worksheet2
SET country = TRIM(country);
```

### Updating the industry name in the database
``` sql
UPDATE layoffs_worksheet2
SET industry = TRIM(industry);
```
### Checking the new update
``` sql
SELECT DISTINCT (company)
FROM layoffs_worksheet2;
```
### Checking the industry
``` sql
SELECT DISTINCT (industry)
FROM layoffs_worksheet2
ORDER BY industry;
```
### Checking how many are related to crypto as there are 3 different types of industry found related to crypto
``` sql
SELECT * FROM layoffs_worksheet2
WHERE industry LIKE '%crypto%';
```
### Updating all the industry related to crypto as crypto
``` sql
UPDATE layoffs_worksheet2
SET industry = 'Crypto'
WHERE industry LIKE '%crypto%';
```

### Checking the industries after the update
``` sql
SELECT DISTINCT (industry)
FROM layoffs_worksheet2
ORDER BY industry;
```

### Checking the location whether the same type of location was input differently by mistake or not
``` sql
SELECT DISTINCT (location)
FROM layoffs_worksheet2;
```

### Checking the country whether the same country was input differently by mistake or not
``` sql
SELECT DISTINCT (country)
FROM layoffs_worksheet2
ORDER BY country;
```
### Checking whether all the countries named the United States are selected or not
``` sql
SELECT DISTINCT (country) FROM layoffs_worksheet2
WHERE country LIKE '%United States%';
```

### Fixing United States
``` sql
UPDATE layoffs_worksheet2
SET country= 'United States'
WHERE country LIKE '%United States%';
```

### Rechecking the countries after updating
``` sql
SELECT DISTINCT (country)
FROM layoffs_worksheet2
ORDER BY country;
```

### Checking the queries to format the date column
``` sql
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_worksheet2;
```

### Updating the date column
``` sql
UPDATE layoffs_worksheet2
SET `date`= STR_TO_DATE(`date`, '%m/%d/%Y');

SELECT *
FROM layoffs_worksheet2
LIMIT 100;
```

### Changing the date column data type from text to date
``` sql
ALTER TABLE layoffs_worksheet2
MODIFY `date` DATE;
```

## 3. Handling the null values and missing values

### Checking the null or missing values in some columns
```sql
SELECT *
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
OR
total_laid_off IS NULL OR total_laid_off =''
OR
percentage_laid_off IS NULL OR percentage_laid_off =''
OR
funds_raised_millions IS NULL OR funds_raised_millions ='';
```

### Checking the industry column with missing values
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry ='';
```

### Checking whether these missing companies have multiple rows from where we can pull the missing values
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE company IN
(
SELECT company
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
);
```

### Checking missing values for the Industry column

/* checking the companies have missing values for industry and whether they have values for the industry column
for some other rows or not so that we can populate the missing values with that industry type */
``` sql
SELECT * FROM layoffs_worksheet2 t1
JOIN layoffs_worksheet2 t2
ON t1.company=t2.company
WHERE (t1.industry is NULL or t1.industry='') AND (t2.industry IS NOT NULL AND t2.industry!='');
```

### Populating the missing values of the industry column

/* populating the missing values of the industry column with the relevant industry values
pulled from other rows of the same company */
```sql
UPDATE layoffs_worksheet2 t1
JOIN layoffs_worksheet2 t2
ON t1.company=t2.company
SET t1.industry=t2.industry
WHERE (t1.industry IS NULL OR t1.industry='') AND (t2.industry IS NOT NULL AND t2.industry!='');
```
### Checking the updated columns which had missing values previously

/* checking whether the missing values are updated or not after executing the earlier queries
and Airbnb and Carvana were some of those companies that had missing values in the industry column */
``` sql
SELECT * FROM layoffs_worksheet2
WHERE company IN ('Airbnb','Carvana');
```

### Rechecking whether there are still any missing values or not in the industry column
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE company IN
(
SELECT company
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
);
```
/* It seems all the rows are updated except 1 which did not have any other rows to pull the industry type from
and we will leave it like this because we do not have anything to do for this row
and putting wrong info is worse than having no info */


 ## 4. Remove Rows and Columns if necessary or if they are for no use

/* As I will work with the total_laid_off, percentage_laid_off and funds_raised_millions data in the PART 2 
of this project so if I have rows where both of these columns are missing then that row is for no use to us
*/

### Checking for missing values in all of those columns
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='')
AND (funds_raised_millions IS NULL or funds_raised_millions='');
```

### Deleting all the rows that have null or missing values in those 3 columns
``` sql
DELETE
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='')
AND (funds_raised_millions IS NULL or funds_raised_millions='');
```
/* I wanted to delete the rows where total_laid_off and percentage_laid_off data are missing
but 316 rows have missing values in those 2 columns and I am not sure whether
deleting so many rows will be a good choice or not so keeping them for now in the datasets.
I will delete them later on if needed. 
*/

``` sql
SELECT *
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='');
```

### Checking the final datasets and whether I need to delete any other rows or unnecessary columns
``` sql
SELECT * FROM layoffs_worksheet2;
```

### Deleting the row_num column as the purpose of creating that column is served and it is for no use now
``` sql
ALTER TABLE layoffs_worksheet2
DROP COLUMN row_num;
```

### Checking and counting the rows of the final datasets and checking the difference between the raw dataset
``` sql
SELECT * FROM layoffs_worksheet2;
SELECT COUNT(*) FROM layoffs_worksheet2;
SELECT * FROM layoffs;
SELECT COUNT(*) FROM layoffs;
```

## PART 2: Exploratory Data Analysis (EDA)

### Checking the clean dataset
``` sql
SELECT * FROM layoffs_worksheet2;
```

### Checking the maximum total_laid_off and maximum percentage_laid_off
``` sql
SELECT MAX(total_laid_off) AS total_layoff, MAX(percentage_laid_off) as total_layoff_percentage
FROM layoffs_worksheet2;
```

### Checking which companies have laid off all their employees and counting their numbers
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE percentage_laid_off=1
ORDER BY total_laid_off DESC;
```
``` sql
SELECT COUNT(*) AS number_of_companies_with_full_layoff
FROM layoffs_worksheet2
WHERE percentage_laid_off=1;
```

### Funds raised by the companies that laid off all their employees
``` sql
SELECT *
FROM layoffs_worksheet2
WHERE percentage_laid_off=1
ORDER BY funds_raised_millions DESC;
```

### Companies that laid off the most
``` sql
SELECT company, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY company
ORDER BY total_number_of_laid_off_employees DESC;
```

### Which industries laid off the most
``` sql
SELECT industry, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY industry
ORDER BY total_number_of_laid_off_employees DESC;
```

### Checking the time period or tenure of the layoff
``` sql
SELECT MIN(`date`) AS the_starting_date,
MAX(`date`) AS the_end_date,
DATEDIFF(MAX(`date`), MIN(`date`)) AS time_period_in_days
FROM layoffs_worksheet2;
```

### Which countries laid of the most
``` sql
SELECT country, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY country
ORDER BY total_number_of_laid_off_employees DESC;
```

### Checking whether Bangladesh is on the list or not
```sql
SELECT *
FROM layoffs_worksheet2
WHERE country LIKE '%Bangladesh%';
```
### Checking at which stage of the company most of the layoffs happened
/* Series A means early stage than Series B and post IPO means post-initial public offering which includes Amazon and Google */
``` sql
SELECT stage, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY stage
ORDER BY total_laid_off_in_a_year DESC;
```
/* It seems like post IPO, and mid-level (series c,d,e) had the most number of layoffs 
along with companies who got acquired */

### Checking in which year most of the layoffs happened
``` sql
SELECT YEAR(`date`) AS layoff_year, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY layoff_year
ORDER BY total_laid_off_in_a_year DESC;
```

### Checking in which month most of the layoffs happened
``` sql
SELECT MONTH(`date`) AS layoff_month, SUM(total_laid_off) as total_laid_off_in_a_month
FROM layoffs_worksheet2
GROUP BY layoff_month
ORDER BY total_laid_off_in_a_month DESC;
#seems like january had the most amount of layoff but we can only see the month and cannot see the year
```

### Checking the month and the year where most of the layoff happened
``` sql
SELECT SUBSTR(`date`, 1, 7) as layoff_year_and_month, SUM(total_laid_off) as total_laid_of_in_a_month_of_an_year
FROM layoffs_worksheet2
WHERE SUBSTR(`date`, 1, 7) IS NOT NULL OR SUBSTR(`date`, 1, 7)!=''
GROUP BY layoff_year_and_month
ORDER BY layoff_year_and_month;
```

### Checking the month and year of layoff with the total till that month
``` sql
WITH roling_total AS
(
SELECT SUBSTR(`date`, 1, 7) as layoff_year_and_month, SUM(total_laid_off) as total_laid_of_in_a_month_of_an_year
FROM layoffs_worksheet2
WHERE SUBSTR(`date`, 1, 7) IS NOT NULL OR SUBSTR(`date`, 1, 7)!=''
GROUP BY layoff_year_and_month
ORDER BY layoff_year_and_month
)
SELECT *,
SUM(total_laid_of_in_a_month_of_an_year) OVER(ORDER BY layoff_year_and_month) as total_layoff_till_this_month
FROM roling_total;
```

### Checking which companies laid off most of their employees in which months 
``` sql
SELECT company, YEAR(`date`) AS layoff_year, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY company, layoff_year
ORDER BY total_laid_off_in_a_year DESC, company, layoff_year DESC;
```

### Top 5 companies in each year that laid off their employees
``` sql
WITH top_companies AS
(
SELECT company, YEAR(`date`) as year_of_layoff, SUM(total_laid_off) AS total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY company, year_of_layoff
),
company_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_layoff ORDER BY total_laid_off_in_a_year DESC) AS ranking
FROM top_companies
WHERE year_of_layoff IS NOT NULL OR year_of_layoff!=''
)
SELECT * FROM company_ranking
WHERE ranking<=5;
```

### Top 5 industries in each year that laid off their employees
``` sql
WITH top_industries AS
(
SELECT industry, YEAR(`date`) as year_of_layoff, SUM(total_laid_off) AS total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY industry, year_of_layoff
),
industry_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_layoff ORDER BY total_laid_off_in_a_year DESC) AS ranking
FROM top_industries
WHERE year_of_layoff IS NOT NULL OR year_of_layoff!=''
)
SELECT * FROM industry_ranking
WHERE ranking<=5;
```

### Top 5 countries in each year that laid off their employees
``` sql
WITH top_countries AS
(
SELECT country, YEAR(`date`) as year_of_layoff, SUM(total_laid_off) AS total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY country, year_of_layoff
),
country_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_layoff ORDER BY total_laid_off_in_a_year DESC) AS ranking
FROM top_countries
WHERE year_of_layoff IS NOT NULL OR year_of_layoff!=''
)
SELECT * FROM country_ranking
WHERE ranking<=5;
```

### Top companies that raised funds
``` sql
SELECT company, SUM(funds_raised_millions) as total_funds_in_millions
FROM layoffs_worksheet2
GROUP BY company
ORDER BY total_funds_in_millions DESC;
```

### Top 5 companies in each year that had the most amount of funds
``` sql
WITH top_companies AS
(
SELECT company, YEAR(`date`) as year_of_funding, SUM(funds_raised_millions) AS total_funds_collected_in_a_year_in_millions
FROM layoffs_worksheet2
GROUP BY company, year_of_funding
),
company_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_funding ORDER BY total_funds_collected_in_a_year_in_millions DESC) AS ranking
FROM top_companies
WHERE year_of_funding IS NOT NULL OR year_of_funding!=''
)
SELECT * FROM company_ranking
WHERE ranking<=5;
```

### Top 5 industries in each year that had the most amount of funds
``` sql
WITH top_industries AS
(
SELECT industry, YEAR(`date`) as year_of_funding, SUM(funds_raised_millions) AS total_funds_collected_in_a_year_in_millions
FROM layoffs_worksheet2
GROUP BY industry, year_of_funding
),
industry_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_funding ORDER BY total_funds_collected_in_a_year_in_millions DESC) AS ranking
FROM top_industries
WHERE year_of_funding IS NOT NULL OR year_of_funding!=''
)
SELECT * FROM industry_ranking
WHERE ranking<=5;
```

### Top 5 countries (company belongs to that country) in each year that had the most amount of funds
``` sql
WITH top_countries AS
(
SELECT country, YEAR(`date`) as year_of_funding, SUM(funds_raised_millions) AS total_funds_collected_in_a_year_in_millions
FROM layoffs_worksheet2
GROUP BY country, year_of_funding
),
country_ranking AS
(
SELECT *,
DENSE_RANK() OVER(PARTITION BY year_of_funding ORDER BY total_funds_collected_in_a_year_in_millions DESC) AS ranking
FROM top_countries
WHERE year_of_funding IS NOT NULL OR year_of_funding!=''
)
SELECT * FROM country_ranking
WHERE ranking<=5;
```
Surprisingly, Lithuania was in 4th rank in 2022 with the most funds. Therefore, checking the companies from Lithuania
``` sql
SELECT * FROM layoffs_worksheet2
WHERE country LIKE '%Lithuania%';

#UBER is from Lithuania and that makes sense now.
```
