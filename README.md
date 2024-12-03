# README: Exploratory Data Analysis (EDA) on Layoff Data

## Overview

This README provides an outline of an Exploratory Data Analysis (EDA) process conducted on the `layoffs_staging2` table from the `world_layoffs` database. The goal of this analysis is to identify trends, patterns, and anomalies in the data related to global layoffs over the 2020-2023 range.

### Objectives
- Examine the data for outliers, trends, and key statistics.
- Analyze layoffs by percentage, location, industry, and time.
- Identify companies with significant layoffs, both single-day events and cumulative.
- Explore rolling totals and temporal patterns in layoffs.

---

## Observations
- **Startups:** Many smaller companies completely shut down during this period, often after failing to sustain operations despite significant funding.
- **Industries:** Technology and manufacturing sectors experienced some of the largest layoffs.
- **Trends:** Layoffs peaked during specific economic events, visible in yearly and monthly aggregations.
- **Outliers:** Companies like Quibi, which raised massive funds yet failed, stand out as notable case studies.
  
---


## Steps and Insights

### **1. Initial Exploration**
- Query to retrieve all data:
  ```sql
  SELECT * 
  FROM world_layoffs.layoffs_staging2;
  ```
- Understand key metrics like total layoffs, percentage laid off, and company-specific insights.

---

### **2. Easier Queries**
- **Maximum layoffs on a single day:**
  ```sql
  SELECT MAX(total_laid_off)
  FROM world_layoffs.layoffs_staging2;
  ```

- **Analyzing percentage of layoffs:**
  ```sql
  SELECT MAX(percentage_laid_off), MIN(percentage_laid_off)
  FROM world_layoffs.layoffs_staging2
  WHERE percentage_laid_off IS NOT NULL;
  ```

- **Companies with 100% layoffs:**
  ```sql
  SELECT *
  FROM world_layoffs.layoffs_staging2
  WHERE percentage_laid_off = 1;
  ```

- Observations:
  - Startups dominate the 100% layoffs category, often indicating closure.
  - Examples include BritishVolt and Quibi.

---

### **3. Grouped and Summarized Insights**
- **Companies with the largest layoffs:**
  ```sql
  SELECT company, SUM(total_laid_off)
  FROM world_layoffs.layoffs_staging2
  GROUP BY company
  ORDER BY 2 DESC
  LIMIT 10;
  ```

- **Layoffs by location:**
  ```sql
  SELECT location, SUM(total_laid_off)
  FROM world_layoffs.layoffs_staging2
  GROUP BY location
  ORDER BY 2 DESC
  LIMIT 10;
  ```

- **Layoffs by year:**
  ```sql
  SELECT YEAR(date), SUM(total_laid_off)
  FROM world_layoffs.layoffs_staging2
  GROUP BY YEAR(date)
  ORDER BY 1 ASC;
  ```

- Insights include:
  - High concentrations of layoffs in specific industries and locations.
  - Temporal patterns showing peaks during economic downturns.

---

### **4. Advanced Analysis**
- **Yearly Top Companies for Layoffs:**
  ```sql
  WITH Company_Year AS 
  (
    SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY company, YEAR(date)
  )
  , Company_Year_Rank AS (
    SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
    FROM Company_Year
  )
  SELECT company, years, total_laid_off, ranking
  FROM Company_Year_Rank
  WHERE ranking <= 3
  AND years IS NOT NULL
  ORDER BY years ASC, total_laid_off DESC;
  ```

- **Rolling total of layoffs per month:**
  ```sql
  WITH DATE_CTE AS 
  (
    SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY dates
    ORDER BY dates ASC
  )
  SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
  FROM DATE_CTE
  ORDER BY dates ASC;
  ```



---

## Usage Notes
- All queries are designed to run in a SQL environment supporting the `world_layoffs` database schema.
- Adjust filtering conditions (e.g., date ranges, percentage thresholds) as needed to focus on specific areas of interest.

### Future Directions
- Integrate visualization tools (e.g., Power BI, Tableau) for trend representation.
- Analyze employee demographics to understand impacts across groups.

--- 

**Prepared by:** Joseph Maldonado  
**Date:** December 3, 2024
