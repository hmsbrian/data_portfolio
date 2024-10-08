## COVID19 Analysis

The goal here is to grab a dataset and then create Excel files that I'll use for further analysis. For a more complete overview, click this [writeup link.](https://docs.google.com/document/d/1_IhxgkV62J9eSuoqerGbjIdIxq2Ph-THp1THaFS1NU4/edit?usp=sharing)
Will start by grabbing data on COVID19 deaths and vaccinations. [Data source.](https://ourworldindata.org/covid-deaths)

### Data manipulation

-extracting relevant data
```sql
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
order by 1,2
```

--total cases vs total deaths y creating a calculated column
```sql
SELECT location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
order by 1,2
```

### Question
How many people died of covid in the US as of the most recent date in this dataset?

-total deaths in US
```sql
SELECT location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
where location = 'United States'
order by 2 desc
```

### Question
What was the peak of death percentage in the US for the time period being measured?

-peak death percentage in US (as of 4/30/21)
```sql
SELECT location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
where location = 'United States'
order by 6 desc
```

-Modify query to filter out percentages above 6.5%, rounding to 3 decimal places.
- Peak death percentage in US (as of 4/30/21)
```sql
SELECT location, date, total_cases, new_cases, total_deaths, death_percentage
FROM (
  SELECT location, date, total_cases, new_cases, total_deaths, ROUND((total_deaths/total_cases)*100, 3) AS death_percentage
  FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
  WHERE location = 'United States'
) AS subquery
WHERE death_percentage < 6.5
ORDER BY death_percentage DESC;
```

-Compare that to another country
-total deaths in the UK
```sql
SELECT location, date, total_cases, new_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
where location = 'United Kingdom'
order by 2 desc
```

### Question
UK vs US peak mortality rate?

-Peak death percentage in the UK (as of 4/30/21)
```sql
SELECT location, date, total_cases, new_cases, total_deaths, death_percentage
FROM (
  SELECT location, date, total_cases, new_cases, total_deaths, ROUND((total_deaths/total_cases)*100, 3) AS death_percentage
  FROM `promising-rock-425020-h8.covid19_global.covid19_deaths`
  WHERE location = 'United States'
) AS subquery
WHERE death_percentage < 15
ORDER BY death_percentage DESC;
```

### Question
Top 10 countries with the highest percentage of reported Covid cases.

-countries with highest percentage infected
```sql
SELECT 
    location, 
    population, 
    MAX(total_cases) AS highest_infection_count, 
    ROUND(MAX(total_cases) / population * 100, 2) AS percent_infected
FROM 
    `promising-rock-425020-h8.covid19_global.covid19_deaths`
GROUP BY 
    location, 
    population
ORDER BY 
    percent_infected DESC
LIMIT 10;
```

-same query, but filtering for only larger nations, population >50,000,000
```sql
SELECT
   location,
   population,
   MAX(total_cases) AS highest_infection_count,
   ROUND(MAX(total_cases) / population * 100, 2) AS percent_infected
FROM
   `promising-rock-425020-h8.covid19_global.covid19_deaths`
WHERE population > 50000000
GROUP BY
   location,
   population
ORDER BY
   percent_infected DESC
LIMIT 15
```

### Data manipulation
Need to tweak query; dataset treats continents as locations in some cases

-modified query resulting in countries, not regions
```sql
SELECT
   location,
   population,
   MAX(total_cases) AS highest_infection_count,
   ROUND(MAX(total_cases) / population * 100, 2) AS percent_infected
FROM
   `promising-rock-425020-h8.covid19_global.covid19_deaths`
WHERE population > 50000000 AND continent is not null
GROUP BY
   location,
   population
ORDER BY
   percent_infected DESC
LIMIT 15
``` 

### Question
Which regions/countries have the highest rate of vaccination?

-combining deaths table and vaccination table
```sql
SELECT *
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths` death
join `promising-rock-425020-h8.covid19_global.covid19_vac` vac
 on death.location = vac.location
 and death.date = vac.date
```

-vaccinations as a percentage of population
```sql
SELECT death.continent, death.location, death.date, death.population, vac.new_vaccinations,
 SUM(vac.new_vaccinations) over (partition by death.location order by death.location, death.date) as rolling_vac_total
FROM `promising-rock-425020-h8.covid19_global.covid19_deaths` death
join `promising-rock-425020-h8.covid19_global.covid19_vac` vac
on death.location = vac.location
and death.date = vac.date
Where death.continent is not null and vac.new_vaccinations is not null
Order by 2, 3;
```

-will now will export results from BigQuery and do final visualizations in Excel
