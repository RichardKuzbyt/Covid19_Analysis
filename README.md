# Covid19_Analysis
A SQL analysis of up-to-date, real-time Covid-19 data from https://ourworldindata.org/covid-deaths

SELECT TOP (1000) [iso_code]
      ,[continent]
      ,[location]
      ,[date]
      ,[population]
      ,[total_cases]
      ,[new_cases]
      ,[new_cases_smoothed]
      ,[total_deaths]
      ,[new_deaths]
      ,[new_deaths_smoothed]
      ,[total_cases_per_million]
      ,[new_cases_per_million]
      ,[new_cases_smoothed_per_million]
      ,[total_deaths_per_million]
      ,[new_deaths_per_million]
      ,[new_deaths_smoothed_per_million]
      ,[reproduction_rate]
      ,[icu_patients]
      ,[icu_patients_per_million]
      ,[hosp_patients]
      ,[hosp_patients_per_million]
      ,[weekly_icu_admissions]
      ,[weekly_icu_admissions_per_million]
      ,[weekly_hosp_admissions]
      ,[weekly_hosp_admissions_per_million]
      ,[total_tests]
  FROM [Portfolio_Project].[dbo].[Covid_Deaths]
  
  
 Select*
From Portfolio_Project..Covid_Deaths
Where continent is not null
order by 3,4

--Select*
--From Portfolio_Project..Covid_Vaccinations
--order by 3,4

-- Select Data that we are going to be using

Select Location, date, total_cases, new_cases, total_deaths, population
From Portfolio_Project..Covid_Deaths
Where continent is not null
order by 1,2

--  Change data type

ALTER TABLE Portfolio_Project..Covid_Deaths
ALTER COLUMN total_cases int

ALTER TABLE Portfolio_Project..Covid_Deaths
ALTER COLUMN total_deaths int

-- Looking at Total Cases vs Total Deaths
-- Likelihood of dying if you contract Covid in UK

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as Death_Rate
From Portfolio_Project..Covid_Deaths
-- Where location like 'United Kingdom'
Where continent is not null
order by 1,2

-- Looking at Total Cases vs Population
-- Percentage of population that contracted Covid in UK

Select Location, date, total_cases,population, (total_cases/population)*100 as Contraction_Rate
From Portfolio_Project..Covid_Deaths
--  Where location like 'United Kingdom'
Where continent is not null
order by 1,2


--  Looking at Countries with Highest Infection Rate compared to Population

Select Location, population, MAX(total_cases) as Highest_Infection_Rate, MAX((total_cases/population))*100 as Contraction_Rate
From Portfolio_Project..Covid_Deaths
-- Where location like '%United Kingdom%'
Where continent is not null
Group by Location, population
order by Contraction_Rate desc

-- Showing Countries with the Highest Death Rate per Capita

Select Location, MAX(cast(Total_Deaths as int)) as Total_Death_Count
From Portfolio_Project..Covid_Deaths
-- Where location like '%United Kingdom%'
Where continent is not null
Group by Location
order by Total_Death_Count desc

-- Breakdown by continent

Select continent, MAX(cast(Total_Deaths as int)) as Total_Death_Count
From Portfolio_Project..Covid_Deaths
-- Where location like '%United Kingdom%'
Where continent is not null
Group by continent
order by Total_Death_Count desc

-- Showing continents with the Highest Death Count

Select continent, MAX(cast(Total_Deaths as int)) as Total_Death_Count
From Portfolio_Project..Covid_Deaths
-- Where location like '%United Kingdom%'
Where continent is not null
Group by continent
order by Total_Death_Count desc

--  Change data type

ALTER TABLE Portfolio_Project..Covid_Deaths
ALTER COLUMN new_deaths int

ALTER TABLE Portfolio_Project..Covid_Deaths
ALTER COLUMN new_cases float

--  Global numbers by day

SELECT date, SUM(new_cases) AS total_cases, 
    SUM(CAST(new_deaths AS int)) AS total_deaths, 
    CASE 
        WHEN SUM(new_cases) = 0 THEN 0 
        ELSE (SUM(CAST(new_deaths AS int)) * 100.0) / SUM(new_cases)
    END AS death_rate
FROM Portfolio_Project..Covid_Deaths
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY date;

-- Global number total

SELECT SUM(new_cases) AS total_cases, 
    SUM(CAST(new_deaths AS int)) AS total_deaths, 
    CASE 
        WHEN SUM(new_cases) = 0 THEN 0 
        ELSE (SUM(CAST(new_deaths AS int)) * 100.0) / SUM(new_cases)
    END AS death_rate
FROM Portfolio_Project..Covid_Deaths
WHERE continent IS NOT NULL
ORDER BY 1;

-- Looking at Total Population vs Vaccinations

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as bigint)) over (Partition by dea.location order by dea.location, dea.date) as Accumulation_People_Vaccinated
From Portfolio_Project..Covid_Deaths dea
Join Portfolio_Project..Covid_Vaccinations vac
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
order by 2,3

-- Use CTE

With PopvsVac (continent, location,date, population, new_vaccinations, Accumulation_People_Vaccinated)
as 
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as bigint)) over (Partition by dea.location order by dea.location, dea.date) as Accumulation_People_Vaccinated
From Portfolio_Project..Covid_Deaths dea
Join Portfolio_Project..Covid_Vaccinations vac
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3
)
Select*, (Accumulation_People_Vaccinated/population)*100
From PopvsVac

-- Temp table

DROP table if exists #Percent_Pop_Vacc
Create table #Percent_Pop_Vacc
(Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
Accumulation_People_Vaccinated numeric
)

Insert into #Percent_Pop_Vacc
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as bigint)) over (Partition by dea.location order by dea.location, dea.date) as Accumulation_People_Vaccinated
From Portfolio_Project..Covid_Deaths dea
Join Portfolio_Project..Covid_Vaccinations vac
On dea.location = vac.location
and dea.date = vac.date
--where dea.continent is not null
--order by 2,3

Select*, (Accumulation_People_Vaccinated/population)*100
From #Percent_Pop_Vacc

-- Create view to store data for later visualizations

Create view Percent_Pop_Vacc as 
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(cast(vac.new_vaccinations as bigint)) over (Partition by dea.location order by dea.location, dea.date) as Accumulation_People_Vaccinated
From Portfolio_Project..Covid_Deaths dea
Join Portfolio_Project..Covid_Vaccinations vac
On dea.location = vac.location
and dea.date = vac.date
where dea.continent is not null
--order by 2,3

Select*
From Percent_Pop_Vacc
