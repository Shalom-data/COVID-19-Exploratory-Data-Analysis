# Exploring the COVID Dataset: SQL Analysis (January 2020 - April 2021)
This project explores and analyzes COVID-19 data from January 2020 to April 2021 using SQL. The goal is to extract insights on COVID-19 infection rates, mortality, and vaccination progress across various countries and continents.

## Table of Contents
- Project Overview
- Dataset Information
- Queries and Insights
   - Data Selection and Filtering
   - Country-Specific COVID Insights
   - Global Statistics
   - Vaccination Analysis
- Technical Optimizations
- Conclusion

## Project Overview
This project explores the COVID-19 pandemic's impact across the world using publicly available data on cases, deaths, and vaccinations. Through various SQL queries, the project analyzes infection and death rates by location, highlights periods of high mortality, and assesses vaccination progress. The analysis covers trends in specific countries like Nigeria and the United States and insights on countries with high infection or vaccination rates.

## Dataset Information
Two tables are used for this analysis:

- **CovidDeaths** - Contains information on total cases, new cases, total deaths, and population data.
- **CovidVaccinations** - Contains data on vaccinations, including new vaccinations per day.
- The datasets include global data from `January 2020` to `April 2021`.


## Queries and Insights
### Data Selection and Filtering
- Basic Data Overview: Selects all records where continent is specified, ordering by country and date.

```sql
SELECT *
FROM covidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;
```

- Selecting Necessary Columns: Filters essential columns, including location, date, total_cases, new_cases, total_deaths, and population, ordered for clarity.

```sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM CovidDeaths
ORDER BY 1, 2, 5;
```

- Select Distinct Locations: Calculates the number of distinct locations (219) present in the dataset.

```sql
SELECT COUNT(DISTINCT location)
FROM CovidDeaths;
```

### Country-Specific COVID Insights
- Nigeria’s COVID Death Rate: Analyzes Nigeria’s cases and deaths, showing that as of April 30, 2021, Nigeria had a death rate of approximately 1.25%.

```sql
SELECT location, date, total_cases, total_deaths,
       ROUND((CAST(total_deaths AS float) / CAST(total_cases AS float) * 100), 2) AS DeathPercentage
FROM CovidDeaths
WHERE location LIKE '%Nigeria' AND continent IS NOT NULL
ORDER BY 1, 2;
```

- United States COVID Death Rate: Examines cases and deaths, revealing a 1.8% death rate as of April 30, 2021.

```sql
SELECT location, date, total_cases, total_deaths,
       ROUND((CAST(total_deaths AS float) / CAST(total_cases AS float) * 100), 2) AS DeathPercentage
FROM covid_deaths
WHERE location LIKE '%states' AND continent IS NOT NULL
ORDER BY 1, 2;
```

- Date Range Validation: Ensures the dataset covers `January 2020` to `April 2021` for all locations.

```sql
SELECT location, MIN(date) AS min_date, MAX(date) AS max_date
FROM CovidDeaths
GROUP BY location;
```

- Infection Rate by Population: Compares infection rates in Nigeria and the US, showing that 0.08% of Nigeria’s population and 9.77% of the US population were infected by the end of April 2021.

```sql
SELECT location, date, population, total_cases,
       ROUND((CAST(total_cases AS float) / CAST(population AS float) * 100), 2) AS PercentageInfected
FROM CovidDeaths
WHERE location IN ('Nigeria', 'United States') AND continent IS NOT NULL
ORDER BY 1, 2;
```

### Global Statistics
- Highest Infection Rates: Identifies countries with the highest infection rates relative to their populations (e.g., Andorra at 17.13%).

```sql
SELECT location, population, MAX(total_cases) AS Infection_count,
       MAX(ROUND((CAST(total_cases AS float) / CAST(population AS float) * 100), 2)) AS Infection_rate
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location, population 
ORDER BY Infection_rate DESC;
```

- Highest Death Counts by Country: Finds the US had the highest COVID death count (576,232).

```sql
SELECT location, MAX(CAST(total_deaths AS INT)) AS Highest_death_count
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY Highest_death_count DESC;

```
- Continent Analysis: Highlights North America as the continent with the most deaths.

```sql
SELECT continent, MAX(CAST(total_deaths AS INT)) AS Highest_death_count
FROM CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY Highest_death_count DESC;
```

- Periods of High Death Rates: Analyzes peak death rates globally, with February 2020 showing the highest rate (29.52%).

```sql
SELECT date, SUM(new_cases) AS Total_cases, SUM(CAST(new_deaths AS INT)) AS Total_deaths,
       ROUND(SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100, 2) AS Global_Death_Percentage
FROM CovidDeaths
WHERE new_cases <> 0 AND new_deaths <> 0 AND continent IS NOT NULL
GROUP BY date
ORDER BY 4 DESC;
```

- Global Death Percentage: Calculates an overall death rate of 2.13% with 10,227,068 deaths.

```sql
SELECT SUM(new_cases) AS Total_cases, SUM(CAST(new_deaths AS INT)) AS Total_deaths,
       ROUND(SUM(CAST(new_deaths AS INT)) / SUM(new_cases) * 100, 2) AS Global_Death_Percentage
FROM CovidDeaths
WHERE new_cases <> 0 AND new_deaths <> 0 AND continent IS NOT NULL;
```


### Vaccination Analysis

- Data Join: Joins CovidDeaths and CovidVaccinations tables on location and date to analyze vaccinations against cases.

```sql
SELECT *
FROM CovidDeaths d
JOIN CovidVaccinations v ON d.location = v.location AND d.date = v.date;
```

- Rolling Vaccination Totals: Calculates cumulative vaccinations by location to assess trends.

```sql
CREATE INDEX idx_deaths ON CovidDeaths(location, date);
CREATE INDEX idx_vacs ON CovidVaccinations(location, date);

SELECT d.continent, d.location, d.date, d.population, v.new_vaccinations,
       SUM(CAST(v.new_vaccinations AS BIGINT)) OVER (PARTITION BY d.location ORDER BY d.location, d.date) AS RollupPeopleVaccinated
FROM CovidDeaths AS d
JOIN CovidVaccinations AS v ON d.location = v.location AND d.date = v.date
WHERE d.continent IS NOT NULL
ORDER BY 2, 3;
```

- Max Vaccination Percentage: Determines the highest vaccination rates by location, with Gibraltar achieving the highest (182%).

```sql
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

- Exploring Anomalies: Identifies why Gibraltar and Israel have vaccination rates over 100%, attributing this to data input inconsistencies.

```sql
SELECT *
FROM #Percentage_vaccinated
WHERE location IN ('Gibraltar', 'Israel');
```

## Technical Optimizations
- Indexing: Indexes on location and date columns to optimize query performance.
- Temporary Tables: Uses a temporary table for cumulative vaccination calculations to enable efficient subquery management.

  
## Conclusion
This analysis provides insight into global and regional COVID-19 impacts. The queries reveal significant disparities in infection and death rates, highlighting countries and continents most affected by the pandemic. Additionally, the vaccination data shows varied progress and identifies anomalies in data reporting.
