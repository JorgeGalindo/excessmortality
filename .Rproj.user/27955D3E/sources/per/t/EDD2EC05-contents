rm(list = ls())


library(tidyverse)
library(ggplot2)
library(ecpolstyle) #If not yet installed, run instead: devtools::install_github("esadeecpol/ecpolstyle")

countrykey <- read.csv("input/countrykey.csv",sep=";") #country organizer and merger

##First, read all three sources and produce mergeable datasets

#Our world in data
excessraw_owid <- read_csv("https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/excess_mortality/excess_mortality.csv") %>%
  filter(date<"2021-01-01") %>%
  replace_na(list(deaths_2020_all_ages=0,average_deaths_2015_2019_all_ages=0)) %>%
  mutate(deletekeep=case_when(
                    date=="2020-12-27" & deaths_2020_all_ages==0 ~ "delete",
                   TRUE ~ "keep"
                   )
         ) %>%
  filter(deletekeep=="keep") %>%       
  group_by(location) %>%
  mutate(last = last(date)) %>%
  ungroup() %>%
  filter(last > "2020-12-27") %>%
  group_by(location) %>%
  summarize(deaths_2020=sum(deaths_2020_all_ages),
            deaths_1519=sum(average_deaths_2015_2019_all_ages)) %>%
  ungroup() %>%
  mutate(country=location,
         excess_deaths_owid=deaths_2020-deaths_1519,
         excess_share_owid=(deaths_2020-deaths_1519)/deaths_1519*100) %>%
  select(country,excess_deaths_owid,excess_share_owid)

#The Economist
excessraw_economist <- read_csv("https://raw.githubusercontent.com/TheEconomist/covid-19-excess-deaths-tracker/master/output-data/excess-deaths/all_weekly_excess_deaths.csv") %>%
  filter(region_code=="0",start_date<"2021-01-01") %>%
  group_by(country) %>%
  mutate(last = last(start_date)) %>%
  ungroup() %>%
  filter(last > "2020-12-20") %>%
  group_by(country) %>%
  summarize(excess_deaths_econ=sum(excess_deaths),
            excess_share_econ=sum(excess_deaths)/sum(total_deaths)*100) %>%
  select(country,excess_deaths_econ,excess_share_econ)

#Karlinsky et al (2021); which has some countries featured by month and other by week, and thus needs two different dataframes
excessraw_akar_month <- read_csv("https://raw.githubusercontent.com/akarlinsky/world_mortality/main/world_mortality.csv") %>%
  filter(year != 2021, time_unit == "monthly") %>%
  group_by(country_name) %>%
  mutate(last = last(time)) %>%
  ungroup() %>%
  filter(last == 12) %>%
  replace_na(list(deaths=0)) %>%
  group_by(year,country_name) %>%
  summarize(deaths=sum(deaths)) %>%
  ungroup() %>%
  mutate(
    year=case_when(
      year==2020 ~ "post",
      TRUE ~ "pre"
    )
    ) %>%
  group_by(year,country_name) %>%
  summarize(deaths=mean(deaths)) %>%
  ungroup() %>%
  pivot_wider(names_from = year, values_from=deaths) %>%
  mutate(country_excess=country_name,
         excess_deaths_akar=post-pre,
         excess_share_akar=(post-pre)/pre*100) %>%
  select(country_excess,excess_deaths_akar,excess_share_akar)


excessraw_akar_week <- read_csv("https://raw.githubusercontent.com/akarlinsky/world_mortality/main/world_mortality.csv") %>%
  filter(year != 2021, time_unit == "weekly") %>%
  group_by(country_name) %>%
  mutate(last = last(time)) %>%
  ungroup() %>%
  filter(last == 53 | last == 52) %>%
  replace_na(list(deaths=0)) %>%
  group_by(year,country_name) %>%
  summarize(deaths=sum(deaths)) %>%
  ungroup() %>%
  mutate(
    year=case_when(
      year==2020 ~ "post",
      TRUE ~ "pre"
    )
  ) %>%
  group_by(year,country_name) %>%
  summarize(deaths=mean(deaths)) %>%
  ungroup() %>%
  pivot_wider(names_from = year, values_from=deaths) %>%
  mutate(country_excess=country_name,
         excess_deaths_akar=post-pre,
         excess_share_akar=(post-pre)/pre*100) %>%
  select(country_excess,excess_deaths_akar,excess_share_akar)

excessraw_akar <- rbind(excessraw_akar_month,excessraw_akar_week)

##Now, read IMF data -- should be downloaded by hand
econraw <- read.csv("input/WEO_Data.csv", sep=";") %>%
  filter(Subject.Descriptor == "Gross domestic product per capita, constant prices") %>%
  mutate(
    country_econ=Country,
    X2020=str_replace_all(X2020,",",""),
    X2019=str_replace_all(X2019,",",""),
    X2020=as.numeric(X2020),
    X2019=as.numeric(X2019)) %>%
  select(country_econ,X2019,X2020) %>%
  mutate(gdp_change=((X2020-X2019)/X2019)*100)

excess_econ <- countrykey %>%
  left_join(excessraw_akar) %>%
  left_join(excessraw_economist) %>%
  left_join(excessraw_owid) %>%
    left_join(econraw) %>%
  mutate(excess_deaths=case_when(is.na(excess_deaths_akar) ~ excess_deaths_econ,
                                 TRUE ~ excess_deaths_akar),
         excess_deaths=case_when(is.na(excess_deaths) ~ excess_deaths_owid,
                                 TRUE ~ excess_deaths),
         excess_share=case_when(is.na(excess_share_akar) ~ excess_share_econ,
                                 TRUE ~ excess_share_akar),
         excess_share=case_when(is.na(excess_share) ~ excess_share_owid,
                                 TRUE ~ excess_share)
        ) %>%
  filter(!is.na(excess_deaths))

##And finally, extract complementary demographic data
epidem_df <- read.csv("https://covid.ourworldindata.org/data/owid-covid-data.csv") 

dem_df <- epidem_df %>%
  filter(date=="2020-10-01",continent != "") %>% #date chosen at random, since we only want the complementary data
  mutate(alpha3=tolower(iso_code)) %>%
  select(alpha3,iso_code,population,population_density,median_age,aged_65_older,extreme_poverty,cardiovasc_death_rate,diabetes_prevalence,hospital_beds_per_thousand,human_development_index)
  
master_df <- excess_econ %>%
  left_join(dem_df) %>%
  mutate(excess_100k=excess_deaths/population*100000) %>%
  left_join(read_csv("input/paises.csv")) %>% #for country names in Spanish
  replace_na(list(name="no tiene")) %>% # for country names that are missing in Spanish
  mutate(pais=case_when(
    name=="no tiene" ~ country,
    TRUE ~ name
  )) %>%
  select(-name,-country_econ,-country_excess)

#Final merge and export
write_csv(master_df,"output/master_df.csv")

  
