
# INFO 602 Final Mini-Project – Spark Analysis



## Project Title



Poverty and Overdose ED Visit Rates in Virginia Localities



## Proposition



Do Virginia localities with higher family poverty have higher drug overdose emergency department (ED) visit rates?



We test this proposition using:

- **VDH PUD Overdose ED Visits by Year and Geography** (Virginia Open Data Portal).  

- **ACS 1-year (2024) Table B17018 – Poverty Status in the Past 12 Months of Families by Household Type by Educational Attainment of Householder**.  



The analysis is **model‑free** and relies on Spark for data cleaning, joining, grouped aggregations, and simple descriptive statistics. 



## Data Sources



- **Overdose ED visits**  

  - Source: Virginia Department of Health – VDH PUD Overdose ED Visits by Year and Geography. 

  - Key fields used:

    - Year

    - Locality / Geography

    - ED visit rate per 10,000 visits



- **ACS Poverty (B17018)**  

  - Source: U.S. Census Bureau – ACS 1‑year 2024, Table B17018. [web:22][web:24]  

  - Key fields used:

    - `B17018_001E`: total families  

    - `B17018_002E`: families below poverty level  



From these we compute `poverty_percent = B17018_002E / B17018_001E * 100`.



## Environment and Tools



- **Language:** Python (PySpark)

- **Engine:** Apache Spark (SparkSession in Colab)

- **APIs used:**

  - Spark DataFrame API

  - Spark SQL (via `createOrReplaceTempView` as needed)



All analysis code is contained in the accompanying notebook (`.ipynb`).



## Analysis Steps (Spark)



1. **Overdose data loading and filtering**

   - Downloaded the overdose CSV from the Virginia Open Data Portal using Python `requests`. 

   - Loaded into Spark with `spark.read.option("header","true").option("inferSchema","true").csv(...)`.  

   - Filtered to:

     - Year = 2024

     - Locality-level geography only  

   - Selected relevant columns and renamed:

     - `Locality`

     - `year`

     - `od_rate` (overdose ED visit rate per 10,000 visits)



2. **ACS B17018 loading and poverty calculation**

   - Loaded the ACS B17018 CSV downloaded from data.census.gov. 

   - Dropped the header row where `NAME == "Geographic Area Name"`.  

   - Cast:

     - `B17018_001E` → `total_families` (double)  

     - `B17018_002E` → `poverty_families` (double)  

   - Computed:

     - `poverty_percent = poverty_families / total_families * 100`  

   - Filtered to Virginia rows only (`NAME` ending with `", Virginia"`), and created a standardized `county_clean` name.



3. **Name cleaning and join**

   - On overdose data, created a join key:

     - `county_join_key = upper(trim(Locality)) + " COUNTY"` for simple county localities.  

   - Joined overdose and ACS DataFrames on `county_join_key == county_clean` using an inner join.  

   - Result: a `joined` DataFrame containing `Locality`, `od_rate`, and `poverty_percent` for matched Virginia localities. 



4. **Poverty bucket analysis**

   - Defined poverty buckets using Spark `when` expressions:

     - `0–5%`

     - `5–10%`

     - `10–20%`

     - `20%+`  

   - Created `poverty_bucket` column on `joined`.  

   - Grouped by `poverty_bucket` and computed:

     - `avg_od_rate = avg(od_rate)`  

   - This produced a small summary table showing that localities in higher poverty buckets have higher average overdose ED visit rates. 



5. **Correlation analysis**

   - Computed correlation using:

     - `joined.stat.corr("poverty_percent", "od_rate")`  

   - This correlation is positive, consistent with the bucketed pattern (higher poverty → higher overdose ED visit rate). 



6. **Exports**

   - Exported locality-level data and bucket summary to CSV for visualization:

     - `Locality`, `od_rate`, `poverty_percent`  

     - `poverty_bucket`, `avg_od_rate`  



## Findings (Short)



- Overdose ED visit rates vary substantially across Virginia localities. [web:2]  

- Localities with higher `poverty_percent` consistently fall into higher average overdose ED visit rate buckets. 

- The correlation between poverty share and overdose ED visit rate is positive, supporting the proposition that worse socioeconomic conditions are associated with higher overdose ED visit rates. 



## Limitations



- ACS B17018 values are survey estimates and have margins of error; some counties may have missing or suppressed values. 

- Some combined county/city localities were excluded from the join due to naming mismatches.  

- The analysis is descriptive and does not establish causality or adjust for other potential confounding factors.



## AI Assistance



I used an AI assistant (Perplexity, powered by GPT‑5.1) to:

- Help clarify the assignment requirements and suggest a defensible proposition.  

- Propose a step‑by‑step Spark workflow for loading, cleaning, joining, and aggregating data.  

- Help debug environment and naming issues and organize the final analysis steps.



All final code and interpretation decisions were made by me.


