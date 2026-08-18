---
title: "Figure2_Map_asr_DALYs_deciles"
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

``` R
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
year <- 2023
release <- 16 

# all ages
all_age_id <- 22 #all ages 
age_stnd_age_id <- 27 #age-standardized 
ages <- get_age_metadata(age_group_set_id = 24, release_id = release)
age_ids <- unique(ages$age_group_id)


# locations
global <- 1
location_set <- 91
locs <- get_location_metadata(location_set_id = location_set, release_id = release)

# measures
cases <- 6
deaths <- 1
dalys <- 2
# metrics
number <- 1
rate <- 3
percent <- 2
```

``` R
# pull data
asdrs <- get_outputs(topic = "cause", 
                         cause_id=plot_cause, 
                         measure_id=dalys, 
                         metric_id=rate, 
                         age_group_id=age_stnd_age_id, 
                         location_id=locs$location_id,
                         sex_id=plot_sex, 
                         release_id=release, 
                         year_id = year, 
                         location_set_id = location_set,
                         compare_version_id = dalynator_compare_version)
```

``` R
map_output_fp <- paste0(file_path, "M_Figure2_Map_asr_DALYs_deciles.pdf")

figure_title <- "Map of global age-standardised DALY rates"
legend_title <- "Age-standardised DALY rate per 100,000"


# --------------------------------------------------------------------------------

create_bins <- function(data, num_bins=10) {
  
  deciles <- quantile(data$mapvar, probs = seq(0, 1, length.out = num_bins + 1), na.rm = TRUE)
  return(deciles)
}

create_labels <- function(bins) {
  
  # turn the list of bins into "Quintile 1: XXX - YYY" and so on
  bins_rounded <- round(bins, 1)
  bins_rounded <- sprintf("%.1f", bins_rounded)
  lower_bounds <- bins_rounded[1:(length(bins_rounded))]
  upper_bounds <- bins_rounded[2:length(bins_rounded)]
  upper_bounds[1:(length(bins_rounded))] <- upper_bounds[1:(length(bins_rounded))]
  
  bounds_df <- data.frame(lower = lower_bounds, upper = upper_bounds)
  all_labels <-c()
  for(i in 1:nrow(bounds_df)) {
    all_labels <- c(all_labels, paste0(bounds_df$lower[i], " to < ", bounds_df$upper[i]))
  }
  
  return(all_labels)
}


# --------------------------------------------------------------------------------

asdrs <- merge(asdrs, locs[,.(location_id, is_estimate)], by = "location_id", all.x=T)

data_to_map <- asdrs

#create map variable
data_to_map$mapvar <- data_to_map$val*100000 #multiplying by 100,000 to convert rate


data_to_bin <- data_to_map[is_estimate==1] # keep just the data that we want to bin
bins <- create_bins(data_to_bin)
deciles_labels <- c(paste0(" < ", round(bins[2], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                        paste0("", round(bins[2], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[3], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                        paste0("", round(bins[3], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[4], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                        paste0("", round(bins[4], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[5], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0("", round(bins[5], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[6], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0("", round(bins[6], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[7], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0("", round(bins[7], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[8], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0("", round(bins[8], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[9], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0("", round(bins[9], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), " to < ", round(bins[10], 1) %>% format(big.mark=",", nsmall = 1, flag = "0"), ""),
                    paste0(">= ", round(bins[10], 1) %>% format(big.mark=",", nsmall = 1, flag = "0")))

# make the map
pdf(file = map_output_fp, height = 10, width = 18, pointsize = 10)
gbd_map(data_to_map,
        bins,
        inset=TRUE,
        labels = deciles_labels,
        legend.shift = c(10,-10),
        col.reverse=T,
        #title = figure_title,
        legend=TRUE,
        legend.title=legend_title,
        legend.cex = 1.5,
        sub_nat = "topic_wIND",
        na.color = "grey70"
)
dev.off()
```
