Analysing public CovidData dataset (https://ourworldindata.org/covid-deaths) using GCP BigQuery and [create a dashboard with GMP Data Studio](https://datastudio.google.com/reporting/bafede8b-9b41-4820-b9ac-c983857d11a6), with focus on CovidDeaths and CovidVaccinations:

1. Exploring the CovidDeaths data
2. Exploring the vaccination data
3. Exploring data in DataStudio

**Skills used**: Joins, CTE's, Windows Functions, Aggregate Functions .

![BigQuery and Data Studio dashboard](https://github.com/NikoRadulescu/BigQuery-with-Data-Studio-Case-study-Covid19/blob/d8716f56b9511d66ef1711462015cc4f8033ebc4/screenshoot-dashboard.jpg)

# Step by step tutorial

### 1. Exploring the CovidDeaths data

``` 
SELECT
  *
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
```

``` 
SELECT
  location,
  date,
  total_cases,
  new_cases,
  total_deaths,
  population
FROM
  `portofolio-326818.CovidData.CovidDeaths`
ORDER BY
  location,
  date
``` 

**Looking at Total Cases vs. Total Deaths** 

This shows the likelihood of dying if you get infected with COVID19, in this case, in Spain

``` 
SELECT
  location,
  date,
  total_cases,
  total_deaths,
  (total_deaths / total_cases)*100 AS DeathPercentage
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  location LIKE '%Spain%'
  AND total_deaths >= 0
ORDER BY
  date
``` 

**Looking at the Total Cases vs Population**

This shows what percentage of the population got COVID19

``` 
SELECT
  location,
  date,
  Population,
  total_cases,
  (total_cases / population)*100 AS PercentPopulationInfected
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  location LIKE '%Spain%'
  AND total_deaths >= 0
ORDER BY
  date
``` 

**Looking at countries with Highest Infection Rate compared to Population**

``` 
SELECT
  location,
  population,
  MAX(total_cases) AS HighestInfectionCount,
  MAX((total_cases / population))*100 AS PercentPopulationInfected
FROM
  `portofolio-326818.CovidData.CovidDeaths`
GROUP BY
  location,
  population
ORDER BY
  PercentPopulationInfected desc
``` 

**Looking at Countries with Highest Death Count per Population**

``` 
SELECT
  location,
  MAX(total_deaths) AS TotalDeathCount
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
GROUP BY
  location
ORDER BY
  TotalDeathCount DESC
``` 

**Break things down by continent**

This shows the continents with the Highest Death Count per Population

``` 
SELECT
  continent,
  MAX(total_deaths) AS TotalDeathCount
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
GROUP BY
  continent
ORDER BY
  TotalDeathCount DESC
``` 

This does not show the real numbers so we have to use the following query to clean the data.

``` 
SELECT
  location,
  MAX(total_deaths) AS TotalDeathCount
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
 continent IS NULL
GROUP BY
  location
ORDER BY
  TotalDeathCount DESC
``` 

**Global numbers**

This shows the global death percentage, the total new cases and total new deaths

``` 
SELECT
  date,
  SUM(new_cases) AS TotalCases,
  SUM(new_deaths) AS TotalDeaths,
  SUM(new_deaths)/SUM(new_cases)*100 AS DeathPercentage
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
GROUP BY
  date
ORDER BY
  1,
  2
``` 

**This shows the total cases world wide**

``` 
SELECT
  SUM(new_cases) AS TotalCases,
  SUM(new_deaths) AS TotalDeaths,
  SUM(new_deaths)/SUM(new_cases)*100 AS DeathPercentage
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
ORDER BY
  1,
  2
``` 

### 2. Exploring the vaccination data

CovidVaccination table : select the data we are going to use

``` 
SELECT
  *
FROM
  `portofolio-326818.CovidData.CovidVaccination`
``` 

This shows the percentage of population that has received at least one Covid Vaccine

``` 
SELECT
  dea.continent,
  dea.location,
  dea.date,
  dea.population,
  vac.new_vaccinations
FROM
  `portofolio-326818.CovidData.CovidDeaths`dea
JOIN
  `portofolio-326818.CovidData.CovidVaccinations`vac
ON
  dea.location = vac.location
  AND dea.date = vac.date
WHERE
  dea.continent IS NOT NULL
  AND vac.new_vaccinations IS NOT NULL
ORDER BY
  1,
  2,
  3
``` 

** Looking at day by day Vaccinations **

``` 
SELECT
  dea.continent,
  dea.location,
  dea.date,
  dea.population,
  vac.new_vaccinations,
  SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS DayByDayVaccinationSum
FROM
  `portofolio-326818.CovidData.CovidDeaths`dea
JOIN
  `portofolio-326818.CovidData.CovidVaccinations`vac
ON
  dea.location = vac.location
  AND dea.date = vac.date
WHERE
  dea.continent IS NOT NULL
ORDER BY
  1,
  2,
  3
``` 

**Find out the percentage of people that are vaccinated in each country**

First we have to create a CTE (Common Table Expression)

``` 
WITH
  PopvsVac AS (
  SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,
    SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS DayByDayVacSum
  FROM
    `portofolio-326818.CovidData.CovidDeaths`dea
  JOIN
    `portofolio-326818.CovidData.CovidVaccinations`vac
  ON
    dea.location = vac.location
    AND dea.date = vac.date
  WHERE
    dea.continent IS NOT NULL )
SELECT
  *,
  (DayByDayVacSum / population)*100 as PercentageVacPeople
FROM
  PopvsVac
``` 

### 3. Exploring data in DataStudio

We will use the following tables

**First table**

``` 
SELECT
  date,
  SUM(new_cases) AS TotalCases,
  SUM(new_deaths) AS TotalDeaths,
  SUM(new_deaths)/SUM(new_cases)*100 AS DeathPercentage
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
GROUP BY
  date
ORDER BY
  1,
  2
``` 

**Second table**

``` 
SELECT
  continent,
  SUM(new_deaths) AS TotalDeathCount
FROM
  `portofolio-326818.CovidData.CovidDeaths`
WHERE
  continent IS NOT NULL
  AND location NOT IN ("World",
    "European Union",
    "International")
GROUP BY
  1
ORDER BY
  TotalDeathCount
``` 

**Third table**

``` 
SELECT
  location,
  population,
  MAX(total_cases) AS HighestInfectionCount,
  MAX((total_cases / population))*100 AS PercentPopulationInfected
FROM
  ´portofolio-326818.CovidData.CovidDeaths´
GROUP BY
  1,
  2
ORDER BY
  PercentPopulationInfected DESC
``` 

**Fourth table**

``` 
SELECT
  location,
  population,
  date,
  MAX(total_cases) AS HighestInfectionCount,
  MAX((total_cases / population))*100 AS PercentPopulationInfected
FROM
  ´portofolio-326818.CovidData.CovidDeaths´
GROUP BY
  1,
  2,
  3
ORDER BY
  PercentPopulationInfected DESC
``` 
