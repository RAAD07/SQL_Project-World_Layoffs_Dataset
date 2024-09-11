# SQL_Project_World-Layoffs-Dataset
This SQL project comprises Data Cleaning in the first part and Exploratory Data Analysis (EDA) in the second part.

## Part 1: Data Cleaning
In the Data Cleaning part, we will be focussing on 4 things.
1. Remove Duplicates
2. Standardize the data
3. Handle Null values or blank values
4. Remove Rows and Columns if necessary or if they are for no use

### checking the raw data
``` sql
SELECT * FROM layoffs; #layoffs hold all the raw data
```
### creating a duplicate table to work and protect the raw data
``` sql
CREATE TABLE layoffs_worksheet 
LIKE layoffs;

SELECT * FROM layoffs_worksheet;
```

### inserting all the rows of layoff table into new layoffs_worksheet table
``` sql
INSERT layoffs_worksheet
SELECT * FROM layoffs;
```

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
/* inserting values into this new table and assigning row_num by creating a partition on each column so that 
each unique row is assigned as 1 and if there is any duplicates then it will be assigned as 2 in the row_num */
INSERT INTO layoffs_worksheet2
SELECT *,
ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, 
stage, country, funds_raised_millions) as row_num
FROM layoffs_worksheet;

SELECT * FROM layoffs_worksheet2;


-- 1. Removing the duplicates

#checking the duplicates
SELECT * FROM layoffs_worksheet2
WHERE row_num > 1;

#deleting the duplicates
DELETE
FROM layoffs_worksheet2
WHERE row_num > 1;

#rechecking whether there are still any duplicates or not
SELECT * FROM layoffs_worksheet2
WHERE row_num>1;


-- 2. Standardazining Column

#checking whether the trim function working right or not
SELECT company, TRIM(company) as company_name
FROM layoffs_worksheet2;

#updating the company name in the database
UPDATE layoffs_worksheet2
SET company = TRIM(company);

UPDATE layoffs_worksheet2
SET location = TRIM(location);

UPDATE layoffs_worksheet2
SET country = TRIM(country);

UPDATE layoffs_worksheet2
SET industry = TRIM(industry);

#checking the new update
SELECT DISTINCT (company)
FROM layoffs_worksheet2;

#checking the industry
SELECT DISTINCT (industry)
FROM layoffs_worksheet2
ORDER BY industry;

#checking how many are related to crypto as there are 3 different types of industry found related to crypto
SELECT * FROM layoffs_worksheet2
WHERE industry LIKE '%crypto%';

#updating all the idustry related to crypto as crypto
UPDATE layoffs_worksheet2
SET industry = 'Crypto'
WHERE industry LIKE '%crypto%';

#checking the industries after update
SELECT DISTINCT (industry)
FROM layoffs_worksheet2
ORDER BY industry;

#checking the location whether same type of location was input differently by mistake or not
SELECT DISTINCT (location)
FROM layoffs_worksheet2;

#checking the country whether same country was input differently by mistake or not
SELECT DISTINCT (country)
FROM layoffs_worksheet2
ORDER BY country;

#checking whether all the country named United States are selected or not
SELECT DISTINCT (country) FROM layoffs_worksheet2
WHERE country LIKE '%United States%';

#fixing United States
UPDATE layoffs_worksheet2
SET country= 'United States'
WHERE country LIKE '%United States%';

#rechecking the countries after updating
SELECT DISTINCT (country)
FROM layoffs_worksheet2
ORDER BY country;

#checking the queries to format the date column
SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_worksheet2;

#updating the date column
UPDATE layoffs_worksheet2
SET `date`= STR_TO_DATE(`date`, '%m/%d/%Y');

SELECT *
FROM layoffs_worksheet2
LIMIT 100;

#the date column was still in text type and we are going to change the data type from text to date
ALTER TABLE layoffs_worksheet2
MODIFY `date` DATE;


-- 3. Handling the null values or missing values

#checking the null or missing values in some columns
SELECT *
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
OR
total_laid_off IS NULL OR total_laid_off =''
OR
percentage_laid_off IS NULL OR percentage_laid_off =''
OR
funds_raised_millions IS NULL OR funds_raised_millions ='';

#checking the industry column with missing values
SELECT *
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry ='';

#checking whether these missing companies have multiple rows from where we can pull the missing values
SELECT *
FROM layoffs_worksheet2
WHERE company IN
(
SELECT company
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
);

/* checking the companies have missing values for industry whether they have values for the industry column
for some other rows or not so that we can populate the missing values with that industry type */
SELECT * FROM layoffs_worksheet2 t1
JOIN layoffs_worksheet2 t2
ON t1.company=t2.company
WHERE (t1.industry is NULL or t1.industry='') AND (t2.industry IS NOT NULL AND t2.industry!='');

/* populating the missing values of the industry columns with the relevant industry values
pulled from other rows of the same company */
UPDATE layoffs_worksheet2 t1
JOIN layoffs_worksheet2 t2
ON t1.company=t2.company
SET t1.industry=t2.industry
WHERE (t1.industry IS NULL OR t1.industry='') AND (t2.industry IS NOT NULL AND t2.industry!='');

/* checking whether the missing values are updated or not after executing the earlier queries
and Airbnb and Carvana were one of those companies which had missing values in the industry column */
SELECT * FROM layoffs_worksheet2
WHERE company IN ('Airbnb','Carvana');

#rechecking whether there are still any missing values or not in the industry column
SELECT *
FROM layoffs_worksheet2
WHERE company IN
(
SELECT company
FROM layoffs_worksheet2
WHERE industry IS NULL OR industry =''
); 
/* seems all the rows are updated except 1 which did not have any other rows to pull the industry type from
and we will leave it like this because we do not have anything to do for this row
and putting wrong info is worse than having no info */


-- 4. Remove Rows and Columns if necessary or if they are for no use

/* As I will work with the total_laid_off, percentage_laid_off and funds_raised_millions data in the PART 2 
of this project so if I have rows where both of these columns are missing then that row is for no use for us
*/

#checking for missing values in all of those columns
SELECT *
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='')
AND (funds_raised_millions IS NULL or funds_raised_millions='');

#deleting all the rows that have null or missing values in those 3 columns
DELETE
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='')
AND (funds_raised_millions IS NULL or funds_raised_millions='');

/* I wanted to delete the rows where total_laid_off and percentage_laid_off data are missing
but there are 316 rows which have missing values in those 2 columns and as I am not sure whether
deleting so many rows will be a good choise or not so keeping them for now in the datasets.
I will delete them later on if needed. 
*/
SELECT *
FROM layoffs_worksheet2
WHERE (total_laid_off IS NULL or total_laid_off='')
AND (percentage_laid_off IS NULL or percentage_laid_off='');

#checking the final datasets and whether I need to delete any other rows or unnecessary columns
SELECT * FROM layoffs_worksheet2;


#deleting row_num column as the purpose of creating that column is served and it is for no use now
ALTER TABLE layoffs_worksheet2
DROP COLUMN row_num;

#checking and counting the rows of the final datasets and checking the difference between the raw dataset
SELECT * FROM layoffs_worksheet2;
SELECT COUNT(*) FROM layoffs_worksheet2;
SELECT * FROM layoffs;
SELECT COUNT(*) FROM layoffs;


-- PART 2 : Exploratory Data Analysis (EDA)

SELECT * FROM layoffs_worksheet2;

#checking the maximum total_laid_off and maximum percentage_laid_off
SELECT MAX(total_laid_off) AS total_layoff, MAX(percentage_laid_off) as total_layoff_percentage
FROM layoffs_worksheet2;

#checking which companies have laid off all their employees and counting their numbers
SELECT *
FROM layoffs_worksheet2
WHERE percentage_laid_off=1
ORDER BY total_laid_off DESC;

SELECT COUNT(*) AS number_of_companies_with_full_layoff
FROM layoffs_worksheet2
WHERE percentage_laid_off=1;

#funds raise by the companies that laid off all their employees
SELECT *
FROM layoffs_worksheet2
WHERE percentage_laid_off=1
ORDER BY funds_raised_millions DESC;

#companies that laid of the most
SELECT company, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY company
ORDER BY total_number_of_laid_off_employees DESC;

#which industries that laid of the most
SELECT industry, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY industry
ORDER BY total_number_of_laid_off_employees DESC;

#checking the time period of the layoff
SELECT MIN(`date`) AS the_starting_date,
MAX(`date`) AS the_end_date,
DATEDIFF(MAX(`date`), MIN(`date`)) AS time_period_in_days
FROM layoffs_worksheet2;

#which countries that laid of the most
SELECT country, SUM(total_laid_off) AS total_number_of_laid_off_employees
FROM layoffs_worksheet2
GROUP BY country
ORDER BY total_number_of_laid_off_employees DESC;

#checking whether Bangladesh is on the list or not
SELECT *
FROM layoffs_worksheet2
WHERE country LIKE '%Bangladesh%';

/* checking in which stage of the company most of the layoff happened
series a means early stage than series b and post IPO means post initial public offering which includes amazon, google */
SELECT stage, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY stage
ORDER BY total_laid_off_in_a_year DESC;
/* seems like post ipo, and mid level (series c,d,e) had the most number of layoff 
along with companies who got acquired */

#checking in which year most of the layoff happened
SELECT YEAR(`date`) AS layoff_year, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY layoff_year
ORDER BY total_laid_off_in_a_year DESC;

#checking in which month most of the layoff happened
SELECT MONTH(`date`) AS layoff_month, SUM(total_laid_off) as total_laid_off_in_a_month
FROM layoffs_worksheet2
GROUP BY layoff_month
ORDER BY total_laid_off_in_a_month DESC;
#seems like january had the most amount of layoff but we can only see the month and cannot see the year

#checking the month and the year where most of the layoff happened
SELECT SUBSTR(`date`, 1, 7) as layoff_year_and_month, SUM(total_laid_off) as total_laid_of_in_a_month_of_an_year
FROM layoffs_worksheet2
WHERE SUBSTR(`date`, 1, 7) IS NOT NULL OR SUBSTR(`date`, 1, 7)!=''
GROUP BY layoff_year_and_month
ORDER BY layoff_year_and_month;

#checking the month and year of layoff with the total till that month
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

#checking which companies laid of most of their employees in which months 
SELECT company, YEAR(`date`) AS layoff_year, SUM(total_laid_off) as total_laid_off_in_a_year
FROM layoffs_worksheet2
GROUP BY company, layoff_year
ORDER BY total_laid_off_in_a_year DESC, company, layoff_year DESC;

#top 5 companies in each year that laid off their employees
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

#top 5 industries in each year that laid off their employees
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

#top 5 countries in each year that laid off their employees
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

#top companies that raised funds
SELECT company, SUM(funds_raised_millions) as total_funds_in_millions
FROM layoffs_worksheet2
GROUP BY company
ORDER BY total_funds_in_millions DESC;

#top 5 company in each year that had most amount of funds
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

#top 5 industries in each year that had most amount of funds
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

#top 5 countries (company belongs to that country) in each year that had most amount of funds
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
#suprisingly Lithuania was in 4th rank in 2022 that had most amount of funds

#therefore checking the companies from Lithuania
SELECT * FROM layoffs_worksheet2
WHERE country LIKE '%Lithuania%';
#UBER is from Lithuania and that makes sense now.

## Part 2: Exploratory Data Analysis (EDA)
