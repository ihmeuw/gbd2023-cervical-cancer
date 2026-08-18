------------------------------------------------------------------------

This README provides an overview of the R scripts and markdown files used for the **GBD 2023 Cervical Cancer Analysis**. These files facilitate the extraction, aggregation, formatting, and visualization of global health estimates, specifically focusing on incidence, mortality, and DALYs.

------------------------------------------------------------------------

## 📋 Project Overview

The scripts in this repository interface with IHME (Institute for Health Metrics and Evaluation) central databases to pull results from the **Global Burden of Disease (GBD) 2023** study. The analysis highlights the disproportionate burden of cervical cancer in low- and middle-income countries (LMICs) and provides forecasts through 2050 regarding WHO elimination targets.

### **Core Versions Used**

-   **Compare Version:** 8352
-   **CodCorrect (Mortality):** 528
-   **COMO (Incidence):** 1762
-   **Dalynator (DALYs):** 102

------------------------------------------------------------------------

## 📂 File Directory

### \*\*1. Analysis & Reporting

-   **`Cervical_Table1.md`**: Generates a standardized table showing incident cases, deaths, and age-standardized rates (ASIR/ASMR), including percent changes from 1990–2023.

### **2. Visualization & Mapping**

-   **`Cervical_Figure1.md`**: Produces line plots with uncertainty ribbons showing age-specific incidence and mortality rates, stratified by World Bank income levels.
-   **`Cervical_Figure2.md`**: Scripts for generating global choropleth maps, specifically binning Age-Standardized DALY rates into deciles for geographic comparison.

### **3. Forecasting** 

-   **`Cervical_Table2.md`**: Focuses on Future Health Scenarios (FHS). It formats 2050 forecasted incidence values, including ASIRs relative to the WHO elimination threshold (4 per 100,000).
-   **`Cervical_Table3.md`**: Focuses on Future Health Scenarios (FHS). It formats 2050 forecasted mortality values, including ASMRs relative to the WHO elimination threshold (4 per 100,000).

### **4. Sample Data Files (.csv)** 

-   **`Cervical_Figure_1_sample_data`**: Sample input data for Cervical_Figure_1
-   **`Cervical_Figure_2_sample_data`**: Sample input data for Cervical_Figure_2
-   **`Cervical_Table_1_sample_data`**: Sample input data for Cervical_Table_1
-   **`Cervical_Table_2_sample_data`**: Sample input data for Cervical_Table_2
-   **`Cervical_Table_3_sample_data`**: Sample input data for Cervical_Table_3

------------------------------------------------------------------------

## 🛠 Setup & Requirements

### **Dependencies**

To run these scripts, you need **R** and the following libraries:

``` r
install.packages(c("data.table", "ggplot2", "dplyr", "matrixStats", "gridExtra", "patchwork", "yaml"))
```

### **System Access**

These scripts utilize IHME central functions (e.g., `get_outputs`, `make_aggregates`).

------------------------------------------------------------------------

## 📊 Key Statistical Methods

-   **Uncertainty Intervals:** All estimates are reported with 95% Uncertainty Intervals (UIs) derived from the 2.5th and 97.5th percentiles of 1,000 draws.
-   **Age Standardization:** Rates are standardized using the GBD world standard population to allow for cross-country comparisons.
-   **Income Grouping:** Analysis is aggregated by World Bank Income Groups (Low, Lower-Middle, Upper-Middle, High) using location set ID 26.
-   **Statistical Note:** These files represent final estimates derived from 1,000 draws. When using these samples, the scripts typically bypass the "draw-level" aggregation and move straight to visualization, as the UIs (upper/lower) are already computed.

------------------------------------------------------------------------

## 🧪 Demo

### **Instructions to run on sample data**

1.  Open any of the `.md` scripts in RStudio.

2.  Locate the data loading section (usually the first code chunk).

3.  Comment out the internal IHME database functions (e.g., `get_outputs()` or `make_aggregates()`).

4.  Replace them with a local file read command using the provided sample files:

    R

    ```         
    # Example for Figure 1 dt_wbig <- fread("path/to/Cervical_figure1_wbig_inputs.csv") 
    ```

5.  Click **"Knit"** or run the code chunks sequentially.

### **Expected Output**

-   **Figures**: High-resolution plots (line graphs or maps) showing the burden of cervical cancer by age group or geography.

-   **Tables**: A formatted `.html` or `.csv` table containing means and 95% Uncertainty Intervals.

-   **Text**: Integrated narrative results with dynamically "plugged" values (e.g., "Approximately 85.8% of cases...").

### **Expected Run Time**

-   **Platform**: Standard desktop computer (8GB RAM, Quad-core processor).

-   **Time**: \< 1 minute. (Since the sample data is already aggregated, the "Knit" process only involves rendering graphics and text).

##  Instructions for Use

### **How to run the software on your own data**

To use these scripts with your own custom datasets, ensure your data follows the GBD standard structure or modify the scripts to match your headers:

1.  **Format your Data**: Your input `.csv` must include columns such as:

    -   `val`, `upper`, `lower` (numeric estimates).

    -   `location_name` or `location_id`.

    -   `measure_name` (e.g., "Deaths", "Incidence").

2.  **Modify Global Variables**: At the top of each script, update the `cause_name`, `year`, and `sex_id` variables to match your dataset.

3.  **Adjust Visualization Tiers**: If your data uses different regional groupings (e.g., WHO regions instead of World Bank Income Groups), update the `location_fill` and `location_colors` vectors in the `ggplot` sections to reflect your specific categories.

4.  **Execute**: Run the script. The helper functions `gbd_round_mean` and `gbd_round_val` will automatically format your raw numbers into publication-ready strings with parentheses.
