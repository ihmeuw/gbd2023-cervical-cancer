---
title: "Cervical_Table3"
author: [NAME]
date: "2025-12-02"
output: html_document
editor_options: 
  markdown: 
    wrap: 72
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

``` r
# version_ids for make_aggregates() 
dalynator_version = 102
como_version = 1762
codcorrect_version = 528

# compare_version_ids for get_outputs()
dalynator_compare_version = 8352
como_compare_version = 8352
codcorrect_compare_version =  8352

###############################################################
# SET GLOBALS that may need modifications
###############################################################

cause_name <- "Cervical"
plot_cause <- 432 #cervical
cervical <- 432

females <- 2
plot_sex <- females 

###############################################################
# load libraries
library(data.table)
library(ggplot2)
library(gridExtra)
library(patchwork)

# Load Central Functions
source("[filepath]/make_aggregates.R")
source("[filepath]/get_draws.R")
source("[filepath]/get_age_metadata.R")
source("[filepath]/get_cause_metadata.R")
source("[filepath]/get_location_metadata.R")
source("[filepath]/get_covariate_estimates.R")
source("[filepath]/get_outputs.R")
source("[filepath]/get_population.R")
source("[filepath]/get_rei_metadata.R")

#Additional Globals
year <- year_late
year_early <- 1990
year_late <- 2023
release <- 16 

# all ages
all_age_id <- 22 #all ages 
age_stnd_age_id <- 27 #age-standardized 
ages <- get_age_metadata(age_group_set_id = 24, release_id = release)
age_ids <- unique(ages$age_group_id)


# locations
global <- 1
locs <- get_location_metadata(location_set_id = 35, release_id = release)
wbi_locs <- get_location_metadata(location_set_id=26, release_id=release)[level==1]
gbd_locs <- locs[level<=3]

# measures
cases <- 6
deaths <- 1
dalys <- 2
# metrics
number <- 1
rate <- 3
percent <- 2
```

``` r
#read in files
deaths_fhs <- fread(paste0("[FILEPATH]"))
deaths_chg <- fread(paste0("[FILEPATH]"))

forecasts_counts <- deaths_fhs[year_id==2050 & age_group_id==22]
sum_counts <- forecasts_counts[, .(mean_2050_deaths=signif(mean(mean_final), 3),
                                     lower_2050_deaths=signif(lower_final, 3),
                                     upper_2050_deaths=signif(upper_final, 3)),
                           by=c("location_name")]

forecasts_rates <- deaths_fhs[year_id==2050 & age_group_id==27]
sum_rates <- forecasts_rates[, .(mean_2050_asr=sprintf("%.1f",round(mean_final, 1)),
                           lower_2050_asr=sprintf("%.1f",round(lower_final, 1)),
                           upper_2050_asr=sprintf("%.1f",round(upper_final, 1))),
                           by=c("location_name")]

sum_change <- deaths_chg[, .(deaths_change_mean = sprintf("%.1f",round(deaths_change_mean*100, 1)),
                                     deaths_change_lower = sprintf("%.1f",round(deaths_change_lower*100, 1)),
                                     deaths_change_upper = sprintf("%.1f",round(deaths_change_upper*100, 1)),
                                     deaths_rate_change_mean = sprintf("%.1f",round(rate_change_mean*100, 1)),
                                     deaths_rate_change_lower = sprintf("%.1f",round(rate_change_lower*100, 1)),
                                     deaths_rate_change_upper = sprintf("%.1f",round(rate_change_upper*100, 1))),
                                 by=c("location_name", "sort_order")]

#merge everything together
deaths_pct_dif <- merge(sum_counts, sum_rates, by=c("location_name"))
deaths <- merge(deaths_pct_dif, sum_change, by=c("location_name"))

setorder(deaths, sort_order)

# Format values
deaths[, deaths_2050 := paste0(format(mean_2050_deaths, big.mark = ",", trim = TRUE), "\n(", 
                               format(lower_2050_deaths, big.mark = ",", trim = TRUE), " to ",
                               format(upper_2050_deaths, big.mark = ",", trim = TRUE), ")")]
deaths[, deaths_pc := paste0(formatC(deaths_change_mean, digits = 1, format = "f"), "\n(",
                               formatC(deaths_change_lower, digits = 1, format = "f"), " to ", 
                               formatC(deaths_change_upper, digits = 1, format = "f"), ")" )]
deaths[, asmr_2050 := paste0(formatC(mean_2050_asr, digits = 1, format = "f"), "\n(",
                             formatC(lower_2050_asr, digits = 1, format = "f"), " to ", 
                             formatC(upper_2050_asr, digits = 1, format = "f"), ")" )]
deaths[, asmr_pc := paste0(formatC(deaths_rate_change_mean, digits = 1, format = "f"), "\n(",
                             formatC(deaths_rate_change_lower, digits = 1, format = "f"), " to ", 
                             formatC(deaths_rate_change_upper, digits = 1, format = "f"), ")" )]
deaths_sub <- deaths[, .(location_name, deaths_2050, deaths_pc, asmr_2050, asmr_pc)]

# Rename columns
setnames(deaths_sub, old = "location_name", new = "Location Name")
setnames(deaths_sub, old = "deaths_2050", new = "Deaths in 2050")
setnames(deaths_sub, old = "deaths_pc", new = "% change, deaths, 2024 to 2050")
setnames(deaths_sub, old = "asmr_2050", new = "Age-standardised death rate in 2050")
setnames(deaths_sub, old = "asmr_pc", new = "% change, age-standardised death rate, 2024 to 2050")

fwrite(deaths_sub, paste0(file_path_outputs, "Cervical_SR_FHS_asmr_deaths_percent_change_formatted.csv"))
```
