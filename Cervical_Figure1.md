---
title: "Figure1_Global_mor_inc_rates_by_age_wbig"
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

all_ages <- 22
year <- 2023
release <- 16 

# measures
cases <- 6
deaths <- 1
dalys <- 2
# metrics
number <- 1
rate <- 3
percent <- 2
# locations
global <- 1
wbi_metadata <- get_location_metadata(location_set_id=26, release_id=release)[level==1]
wbi_locs <- as.vector(wbi_metadata$location_id)
wbi_set <- wbi_metadata[1, location_set_id]

colors_w_global = c("#000000","#e41a1c", "#377eb8", "#4daf4a", "#984ea3")
fill_w_global <- c("#808080", "#f27c81", "#8dbbe3", "#98df98", "#c49ecb")
location_colors <- setNames(colors_w_global, c("Global", "World Bank High Income", "World Bank Upper Middle Income", "World Bank Lower Middle Income", "World Bank Low Income"))  
location_fill <- setNames(fill_w_global, c("Global", "World Bank High Income", "World Bank Upper Middle Income", "World Bank Lower Middle Income", "World Bank Low Income"))  
loc_order <- names(location_fill)


ages <- get_age_metadata(age_group_set_id = 24, release_id = release)
ages <- ages[, age_midpoint := round(((age_group_years_start + age_group_years_end)/2),1)]
ages <- ages[age_group_years_end >15, .(age_group_id, age_group_alternative_name, age_midpoint, age_group_years_end)]
ages <- ages[, age_group_name_plot := gsub(" years", "", age_group_alternative_name)]
age_ids <- unique(ages$age_group_id)

order = c("0-4","5-9","10-14","15-19","20-24","25-29","30-34","35-39","40-44","45-49",
          "50-54","55-59","60-64","65-69","70-74","75-79","80-84","85-89","90-94","95+")
```

``` R
# pull data
dt_wbig <- get_outputs('cause',cause_id=cervical, release_id = release, year_id = year, sex_id = females,
                  age_group_id = ages$age_group_id, measure_id = c(cases, deaths), 
                  metric_id = rate, location_id = c(1,wbi_locs),
                  compare_version_id = como_compare_version, location_set_id = wbi_set)
dt_wbig <- merge(dt_wbig, ages, by="age_group_id")
# Remove rows where val is NA
dt_wbig <- dt_wbig[!is.na(val)]

#set location order
dt_wbig$location_name <- factor(dt_wbig$location_name, levels = loc_order)

measures <- unique(dt_wbig[, .(measure_name, measure_id)])

rate_max <- max(dt_wbig$upper)*100000

age_limits <- c("15-19", "20-24", "25-29", "30-34", "35-39", "40-44", "45-49", 
                "50-54", "55-59", "60-64", "65-69", "70-74", "75-79", "80-84", 
                "85-89", "90-94", "95+")

# create plots

plot_inc_rates <- ggplot(dt_wbig[measure_name=="Incidence"]) +
  geom_ribbon(aes(x = age_group_name_plot, ymin = lower*100000, ymax = upper*100000, fill=location_name, group=location_name), alpha=0.5) +
  geom_line(aes(x = age_group_name_plot, y = val*100000, color=location_name, group=location_name)) +
  scale_fill_manual(values = location_fill) +
  scale_color_manual(values = location_colors) +
  theme(
    panel.spacing = unit(1, units = "cm"), 
    strip.placement = "outside", 
    strip.background = element_rect(fill = "white"), 
    panel.grid.major = element_blank(), 
    panel.grid.minor = element_blank(),
    panel.background = element_blank(),
    axis.line = element_line(colour = "black"),
    legend.text = element_text(size = 10),
    legend.key.size = unit(.5, 'cm'),
    legend.title = element_blank(),
    legend.position = "none",
    axis.text.x = element_text(angle = 45, hjust = 1)
  ) +
  labs(x = "Age group (Years)", y = "Age-specific incidence rate (per 100 000)") +
  scale_y_continuous(limits = c(0, rate_max), expand = c(0,0)) +
  scale_x_discrete(limits = age_limits, expand = c(0,0))
print(plot_inc_rates)

plot_mor_rates <- ggplot(dt_wbig[measure_name=="Deaths"]) +
  geom_ribbon(aes(x = age_group_name_plot, ymin = lower*100000, ymax = upper*100000, fill=location_name, group=location_name), alpha=0.5) +
  geom_line(aes(x = age_group_name_plot, y = val*100000, color=location_name, group=location_name)) +
  scale_fill_manual(values = location_fill) +
  scale_color_manual(values = location_colors) +
  theme(
    panel.spacing = unit(1, units = "cm"), 
    strip.placement = "outside", 
    strip.background = element_rect(fill = "white"), 
    panel.grid.major = element_blank(), 
    panel.grid.minor = element_blank(),
    panel.background = element_blank(),
    axis.line = element_line(colour = "black"),
    legend.text = element_text(size = 10),
    legend.key.size = unit(.5, 'cm'),
    legend.title = element_blank(),
    legend.position = "bottom",
    axis.text.x = element_text(angle = 45, hjust = 1)
  ) +
  labs(x = "Age group (Years)", y = "Age-specific mortality rate (per 100 000)") +
  scale_y_continuous(limits = c(0, rate_max), expand = c(0,0)) +
  scale_x_discrete(limits = age_limits, expand = c(0,0))
print(plot_mor_rates)

#combine plots into panels
full_plot <- plot_inc_rates + plot_mor_rates + 
  plot_layout(ncol = 2, guides = "collect") & 
  theme(legend.position = "bottom")
print(full_plot)

#write out file
pdf(file = paste0(file_path, "M_Figure1_by_wbig_v8352.pdf"), height = 10, width = 12, pointsize = 10)
print(full_plot)
dev.off()
```
