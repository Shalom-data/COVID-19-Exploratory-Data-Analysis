# Exploring the COVID Dataset: SQL Analysis (January 2020 - April 2021)
This project explores and analyzes COVID-19 data from January 2020 to April 2021 using SQL. The goal is to extract insights on COVID-19 infection rates, mortality, and vaccination progress across various countries and continents.

## Table of Contents
- [Project Overview](#project-overview)
- [Dataset Information](#dataset-information)
- [Queries and Insights](#queries-and-insights)
   - [Data Selection and Filtering](#data-selection-and-filtering)
   - [Country-Specific COVID Insights](#country-specific-covid-insights)
   - [Global Statistics](#global-statistics)
   - [Vaccination Analysis](#vaccination-analysis)
- [Technical Optimizations](#technical-optimizations)
- [Conclusion](#conclusion)
- - [Technologies Used](#technologies-used)
- [License](#license)
  

## Project Overview
This project explores the COVID-19 pandemic's impact across the world using publicly available data on cases, deaths, and vaccinations. Through various SQL queries, the project analyzes infection and death rates by location, highlights periods of high mortality, and assesses vaccination progress. The analysis covers trends in specific countries like Nigeria and the United States and insights on countries with high infection or vaccination rates.

## Dataset Information
Two tables are used for this analysis:

- **CovidDeaths** - Contains information on total cases, new cases, total deaths, and population data.
- **CovidVaccinations** - Contains data on vaccinations, including new vaccinations per day.
- The datasets include global data from `January 2020` to `April 2021`.


## Queries and Insights

### Data Selection and Filtering

- **Basic Data Overview:** Selects all records where continent is specified, ordering by `country` and `date`.


```sql
SELECT *
FROM covidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;
```


- **Selecting Necessary Columns:** Filters essential columns, including location, date, total_cases, new_cases, total_deaths, and population, ordered for clarity.


```sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM CovidDeaths
ORDER BY 1, 2, 5;
```


- **Select Distinct Locations:** Calculates the number of distinct locations

   - There are **219** `Distinct Locations` present in the dataset.


```sql
SELECT COUNT(DISTINCT location)
FROM CovidDeaths;
```

### Country-Specific COVID Insights

- **Nigeria’s COVID Death Rate:** Analyzes the `total_cases` vs `total_deaths` in Nigeria.
   - the first death in Nigeria was recorded on 23rd March 2020.
   - In April 30, 2021, Nigeria recorded a death rate of approximately **1.25%**(2,061 deaths).


```sql
SELECT location, date, total_cases, total_deaths,
       ROUND((CAST(total_deaths AS float) / CAST(total_cases AS float) * 100), 2) AS DeathPercentage
FROM CovidDeaths
WHERE location LIKE '%Nigeria' AND continent IS NOT NULL
ORDER BY 1, 2;
```


- **United States COVID Death Rate:** Examines `total_cases` vs `total_deaths` in United States.
   - The first 7 cases were recorded on the `26/01/2020` and by `01/03/2020` there were 73 total cases and 1 death recorded
   - The Unite States recorded a 1.8%(571,084) death rate as of April 30, 2021.


```sql
SELECT location, date, total_cases, total_deaths,
       ROUND((CAST(total_deaths AS float) / CAST(total_cases AS float) * 100), 2) AS DeathPercentage
FROM covid_deaths
WHERE location LIKE '%states' AND continent IS NOT NULL
ORDER BY 1, 2;
```


- **Date Range Validation:** Ensures the dataset covers `January 2020` to `April 2021` for all locations.


```sql
SELECT location, MIN(date) AS min_date, MAX(date) AS max_date
FROM CovidDeaths
GROUP BY location;
```


- **Infection Rate by Population:** Compares infection rates in Nigeria and the US, showing that **0.08%** of Nigeria’s population and **9.77%** of the US population were infected by the end of April 2021.


```sql
SELECT location, date, population, total_cases,
       ROUND((CAST(total_cases AS float) / CAST(population AS float) * 100), 2) AS PercentageInfected
FROM CovidDeaths
WHERE location IN ('Nigeria', 'United States') AND continent IS NOT NULL
ORDER BY 1, 2;
```

### Global Statistics
- **Highest Infection Rates:** Identifies countries with the highest infection rates relative to their populations
   - Andorra had the highest infection rate of 17.13% with respect to population, followed by Montenegro(15.51%) and Czechia(15.23%).


```sql
SELECT location, population, MAX(total_cases) AS Infection_count,
       MAX(ROUND((CAST(total_cases AS float) / CAST(population AS float) * 100), 2)) AS Infection_rate
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location, population 
ORDER BY Infection_rate DESC;
```


- **Highest Death Counts by Country:** Finds the US had the highest COVID death count (576,232).

```sql
SELECT location, MAX(CAST(total_deaths AS INT)) AS Highest_death_count
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY Highest_death_count DESC;

```


- **Continent Analysis:** Highlights North America as the `continent` with the most deaths.

```sql
SELECT continent, MAX(CAST(total_deaths AS INT)) AS Highest_death_count
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY Highest_death_count DESC;
```



- **Periods of High Death Rates:** Analyzes peak death rates globally
   - death rate was very high between february and april 2020 with `February 2020` showing the highest rate (**29.52%**).

```sql
SELECT date, SUM(new_cases) AS Total_cases, SUM(CAST(new_deaths AS INT)) AS Total_deaths,
       ROUND(SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100, 2) AS Global_Death_Percentage
FROM CovidDeaths
WHERE new_cases <> 0 AND new_deaths <> 0 AND continent IS NOT NULL
GROUP BY date
ORDER BY 4 DESC;
```
|Date       |Total_cases|Total_deaths|Global_Death_Percentage|
|-----------|------------|-----------|-----------|
|2020-02-24 |	542	|160	|29.52
|2020-02-19 |	410	|115	|28.05
|2020-02-20 |	531	|123	|23.16
|2020-02-22 |	1503	|207	|13.77
|2020-04-07 |	68726	|8258	|12.02
|2020-04-15 |	75374	|8468	|11.23


- **Global Death Percentage:** Calculates an overall death rate of **2.13%** with **10,227,068** deaths.

```sql
SELECT SUM(new_cases) AS Total_cases, SUM(CAST(new_deaths AS INT)) AS Total_deaths,
       ROUND(SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100, 2) AS Global_Death_Percentage
FROM CovidDeaths
WHERE new_cases <> 0 AND new_deaths <> 0 AND continent IS NOT NULL;
```


### Vaccination Analysis

- **Data Join:** Joins `CovidDeaths` and `CovidVaccinations` tables on location and date to analyze vaccinations against cases.

```sql
SELECT *
FROM CovidDeaths d
JOIN CovidVaccinations v ON d.location = v.location AND d.date = v.date;
```

- **Rolling Vaccination Totals:** Calculates cumulative vaccinations by location to assess trends.

```sql
CREATE INDEX idx_deaths ON CovidDeaths(location, date);
CREATE INDEX idx_vacs ON CovidVaccinations(location, date);  --INDEXING TO OPTIMIZE QUERY PERFORMANCE

SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations,
       SUM(CAST(v.new_vaccinations AS BIGINT)) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS RollupPeopleVaccinated
FROM CovidDeaths AS d
JOIN CovidVaccinations AS v ON d.location = v.location AND d.date = v.date
WHERE d.continent IS NOT NULL AND d.location IN ('Gibraltar','Israel','United Arab Emirates')
ORDER BY 2, 3;
```


- **Max Vaccination Percentage:** Determines the highest vaccination rates by location.
   -  Gibraltar had the highest vaccination rate of **182%** followed by Israel with **121%**.
   - the resulting values are above 100% which suggests anomalies in the dataset


```sql
--we can't use a derived column on the same query from which it was derived, so we need to create a TEMP TABLE.

DROP TABLE IF EXISTS #Percentage_vaccinated;
CREATE TABLE #Percentage_vaccinated (
    continent NVARCHAR(255),
    location NVARCHAR(255),
    date DATETIME,
    population NUMERIC,
    new_vaccination NUMERIC,
    cumulation_people_vaccinated NUMERIC
);

INSERT INTO #Percentage_vaccinated
SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations,
       SUM(CAST(v.new_vaccinations AS BIGINT)) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS RollupPeopleVaccinated
FROM CovidDeaths AS d
JOIN CovidVaccinations AS v ON d.location = v.location AND d.date = v.date
WHERE d.continent IS NOT NULL;

SELECT location, MAX(cumulation_people_vaccinated / population) * 100 AS max_vac
FROM #Percentage_vaccinated
GROUP BY location
ORDER BY MAX(cumulation_people_vaccinated / population) * 100 DESC;
```


- **Exploring Anomalies:** Identifies why Gibraltar and Israel have vaccination rates over 100%.
- The running total of people vaccinted was way more than the population, this could be attributed to input erors/political factors.

  
```sql
SELECT *
FROM #Percentage_vaccinated
WHERE location IN ('Gibraltar', 'Israel');
```


## Technical Optimizations
- **Indexing:** Indexes on location and date columns to optimize query performance.
- **Temporary Tables:** Uses a temporary table for cumulative vaccination calculations to enable efficient subquery management.

  
## Conclusion
This analysis provides insight into global and regional COVID-19 impacts. The queries reveal significant disparities in infection and death rates, highlighting countries and continents most affected by the pandemic. Additionally, the vaccination data shows varied progress and identifies anomalies in data reporting.

## Technologies Used
- SQL Server Management Studio (SSMS)
- SQL Window Functions
- Temporary Tables and Indexing for performance optimization
  
## License
This project is licensed under the MIT License. See [LICENSE](https://opensource.org/licenses/MIT) for details.
