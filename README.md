# SQL-Project-data-exploration
## Data Cleaning Queries

/*
note: the covid stats in this dataset are from 1st jan 2020 to 30 april 2021
COVID 19 DATA EXPLORATION
SKILLS USED - joins,cte's,temp tables,windows functions,
aggregate functions,creating views,datatypeconversion
*/



--select * from CovidDeaths
--where continent is not null
--order by 3,4




           --select data that we are going to use

--select location,date,total_cases,new_cases,total_deaths,population
--from CovidDeaths
--where continent is not null
--order by 1,2



                  -- total cases vs total deaths
        --  **shows likelihood of dying if you contract covid in your country


--select location,date,total_cases,total_deaths,(total_deaths/total_cases)*100
--as DeathPercentage
--from CovidDeaths
--where location like '%india%'
--and continent is not null
--order by 1,2


      --  TOTAL CASES VS POPULATION
	  --   Shows what percentage of population are infected with covid


--select location,date,population,total_cases,(total_cases/population)*100
--as InfectedPopulationPercent
--from CovidDeaths
--where location like '%india%'
--and continent is not null
--order by 1,2


            --countries with highest infection rate compared to population

--Select location, population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
--From CovidDeaths
--Group by location,population
--order by PercentPopulationInfected desc



-- Countries with Highest Death Count per Population


--select location,MAX(cast(total_deaths as int) ) as TotalDeathCount
--from CovidDeaths
--where continent is not null
--group by location
--order by TotalDeathCount desc


-- EXPLORING DATA BY CONTINENT

-- Showing contintents with the highest death count per population

--select continent,MAX(cast(total_deaths as int)) as TotalDeathCount
--from CovidDeaths
--where continent is not null
--group by continent
--order by TotalDeathCount


--DAILY COVID STATS WORLDWIDE

--select date,sum(new_cases) as total_cases,sum(cast(new_deaths as int)) as totaldeaths
--,sum(cast(new_deaths as int))/sum(new_cases)*100 as deathPercent
--from CovidDeaths
--where continent is not null
--group by date
--order by 1,2

       -- Total Population vs Vaccinations
      -- Shows Percentage of Population that has recieved at least one Covid Vaccine

--Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--From PortfolioProject..CovidDeaths dea
--Join PortfolioProject..CovidVaccinations vac
--	On dea.location = vac.location
--	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3


-- Using CTE to perform Calculation on Partition By in previous query



--With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
--as
--(
--Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--From PortfolioProject..CovidDeaths dea
--Join PortfolioProject..CovidVaccinations vac
--	On dea.location = vac.location
--	and dea.date = vac.date
--where dea.continent is not null 
----order by 2,3
--)
--Select *, (RollingPeopleVaccinated/Population)*100 as RollingVaccinationPercent
--From PopvsVac



-- Using Temp Table to perform Calculation on Partition By in previous query

--DROP Table if exists #PercentPopulationVaccinated
--Create Table #PercentPopulationVaccinated
--(
--Continent nvarchar(255),
--Location nvarchar(255),
--Date datetime,
--Population numeric,
--New_vaccinations numeric,
--RollingPeopleVaccinated numeric
--)

--Insert into #PercentPopulationVaccinated
--Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
--, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
----, (RollingPeopleVaccinated/population)*100
--From PortfolioProject..CovidDeaths dea
--Join PortfolioProject..CovidVaccinations vac
--	On dea.location = vac.location
--	and dea.date = vac.date
----where dea.continent is not null 
----order by 2,3

--Select *, (RollingPeopleVaccinated/Population)*100
--From #PercentPopulationVaccinated




---- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
