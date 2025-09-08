# PLX5622 Capstone — Runbook

## Preprocessing (QuPath → clean data)
Inputs: QuPath CSVs in `data/raw/` (not tracked in git)
Open & Run: *02_analysis/01_preprocessing/microglial_counts.ipynb → Run All*

What it does:
* Reads all CSVs in *01_data/01_raw/Total Counts*
* Keeps only rows where Classification == "Iba1"
* Extracts Mouse_ID from the Image column (HH#### pattern)
* Aggregates per mouse to Microglial_Count
* Maps Dose (mg/kg) and Sex (Female/Male) from fixed dictionaries

***Output:***
**CSV:**
* *01_data/02_clean/final_microglial_counts_cleaned.csv*

**Figures:**
* *02_analysis/04_visualizations/counts/female_counts_point.png*
* *02_analysis/04_visualizations/counts/female_counts_strip.png*
* *02_analysis/04_visualizations/counts/male_counts_point.png*
* *02_analysis/04_visualizations/counts/male_counts_strip.png*

## Primary analyses
1) One-way ANOVA (all female doses)
   Open & Run: *02_analysis/02_primary/primary.ipynb → section One-way ANOVA (Females)*

   **Steps:**
   * Load *01_data/02_clean/final_microglial_counts_cleaned.csv*
   * Filter *Sex == 'Female' (female_df)* 
   * Run one-way ANOVA across female doses
   * Tukey HSD post-hoc; convert to Tukey DataFrame *(tukey_df)*

   ***Output:***
      **CSV:**
      * *01_data/02_clean/oneway_ANOVA.csv*
      * *01_data/02_clean/tukey.csv*
   
      **Figures:**
      * *02_analysis/04_visualizations/counts/female_counts_point.png*
      * *02_analysis/04_visualizations/counts/female_counts_strip.png*
      * *02_analysis/04_visualizations/primary/tukey_hsd_females.png*
      * *02_analysis/04_visualizations/primary/tukey_mean_sem_females.png*

   **Notes:** *First two visuals show female mouse microglial counts vs dose; Tukey table shows which dose pairs differ. If 'Significant' == True, tukey pairs highlight blue.*

2) Two-way ANOVA (shared doses (0/900/1200) for male and female mice)
   Open & Run: *02_analysis/02_primary/primary.ipynb → section Two-way ANOVA*
  
  **Steps:**
   * Load *01_data/02_clean/final_microglial_counts_cleaned.csv*
   * Filter for the shared doses *(['Dose']. isin([0, 900, 1200]))*
   * Fit two-way ANOVA with factors *Sex, Dose, and DosexSex interaction*
   * Visualize the dose effects by sex 
   
   ***Output:***
      **CSV:**
      * *01_data/02_clean/twoway_ANOVA.csv*

      **Figures:**
      * *02_analysis/04_visualizations/primary/shared_counts_overlay.png*
      * *02_analysis/04_visualizations/primary/shared_counts_strip.png*

   **Notes:** *Overlay plot visualize sex dfferences across the shared doses; check interaction to assess sex-specific dose effects.*

3) Threshold analysis (all female doses)
   Open & Run: *02_analysis/02_primary/primary.ipynb → section Threshold Analysis*
  
  **Steps:**
   * Load *01_data/02_clean/final_microglial_counts_cleaned.csv*
   * Use the exisiting *female_df* created in One-Way ANOVA section
   * Group by *'Dose'* and compute for *'Microglial Count: 'mean', 'std', 'min', 'max', 'count'*
   * Compute percentage depltion from Vehicle (0 mg/kg) as:
   $$ 
   \%\ \text{depletion} = 100 \times \frac{\text{vehicle mean} - \text{dose mean}} {\text{vehicle mean}}
   $$
   * Round values to two decimals

   ***Output:***
      **CSV:**
      * *01_data/02_clean/threshold_analysis.csv*

      **Figures:**
      * *02_analysis/04_visualizations/primary/shared_counts_overlay.png*
      * *02_analysis/04_visualizations/primary/shared_counts_strip.png* 
   
   **Notes:** *Requires vehicle group to compute depletion. Positive % depletion from vehicle means counts are lower than vehicle; negative means higher than vehicle*


## Secondary analyses
1) Per-Dose Summary (female and male mice)
   Open & Run: *02_analysis/03_secondary/secondary.ipynb → section Per-dose summary*

   **Steps:**
   * Load *01_data/02_clean/final_microglial_counts_cleaned.csv*
   * Create *female_df* and *male_df*
   * Run and apply the function *summarize_counts* to *female_df* and *male_df* (seperately) to create per-dose tables
   * Save each table to the clean data folder

   ***Outputs:***
      **CSV:**
      * *01_data/02_clean/per_dose_female_summary.csv*
      * *01_data/02_clean/per_dose_male_summary.csv*

      **Figures:**
      * *02_analysis/04_visualizations/secondary/females_counts_median_IQR.png*
      * *02_analysis/04_visualizations/secondary/male_counts_median_IQR.png*

   **Notes:** *Per-Dose summary describes each dose: center (mean/median) and spread (SD, CV, 95% CI). Shows the shape of the data.*

2) Brown-Forsythe (variance check; females & males)
   Open & Run: *02_analysis/03_secondary/secondary.ipynb → section Variability within groups (Brown–Forsythe / Levene median-centered)* 

   **Steps:**
   * Use *female_df* and *male_df* created from the Per-Dose Summary, group *Microglial Counts* by dose level
   * Run once per sex, record W (test statistic) and p-value
   * Save W and p-value to CSV file

   **Outputs:**
      **CSV:**
      * *01_data/02_clean/brown_forsythe_female.csv*
      * *01_data/02_clean/brown_forsythe_male.csv*

      **Figures:**
      * *02_analysis/04_visualizations/secondary/females_within_dose_CV.png*
      * *02_analysis/04_visualizations/secondary/females_within_dose_SD.png*
      * *02_analysis/04_visualizations/secondary/males_within_dose_CV.png*
      * *02_analysis/04_visualizations/secondary/males_within_dose_SD.png*

   **Notes:** *Requires ≥2 dose groups with ≥2 observations each; otherwise write NaN for W and p-value.

3) OLS (linear regression) with HC3 robust SEs
   Open & Run: *02_analysis/03_secondary/secondary.ipynb → sectionDose trend (linear OLS, HC3 robust SE, quadratic check, Spearman)* 

   **Steps:**
   * Fit linear model on *female_df* and *male_df* created from the Per-Dose Summary
   * Compute HC3 robust stats for the Dose term; compute AIC (model selectiom; linear vs quadratic) Spearman (rank correlation; monotonic relationship)
      * AIC: compares linears vs quadratic AIC. Lower AIC is better; ΔAIC > 2 typically favors the lower-AIC model.
      * Spearman: checks for monotonic relationship between dose and counts without assuming linearity or normality.
   * Run and apply function created to save OLS Regression Results to a text file *save_ols*

   **Outputs:**
      **Files:**
      * *01_data/02_clean/ols_female_summary.txt*
      * *01_data/02_clean/ols_male_summary.txt*

      **Figures:**
      * *02_analysis/04_visualizations/secondary/male_counts_mean_CI.png*
      * *02_analysis/04_visualizations/secondary/females_counts_mean_CI.png*

   **Notes:** Used text files instead of CSV for readability.