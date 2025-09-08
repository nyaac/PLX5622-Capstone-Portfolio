# Dose-Dependent Microglial Depletion with PLX5622 in Mice— Code & Analysis Portfolio

**Description:** A reproducible workflow of my capstone project: dose-dependent microglial depletion in wild-type mice treated with PLX5622 (CSF1R inhibitor), quantified from Iba1-stained whole-brain images using QuPath 0.5.1 (object classifier).

## Highlights
- **Primary analyses:** One-way ANOVA (female doses); Two-way ANOVA (shared 0/900/1200, DosexSex); Threshold analysis.
- **Secondary analyses:** Brown–Forsythe, OLS with HC3, AIC checks, Spearman.
- **Tech:** Python (pandas, numpy, matplotlib, seaborn, scipy, statsmodels), macOS (Apple M2).

## Contents
- `01_data/01_raw` - exported QuPath CSVs files for each mouse
- `01_data/02_clean` - cleaned CSV file that contains Mouse_ID, Microglial_Count, Dose, Sex *final_microglial_counts_cleaned.csv*; also contains CSV and test files from statistical analyses performed.
- `02_analysis/runbook.md` — how to reproduce with existing code (commands, not code).
- `02_analysis/01_preprocessing`, `02_analysis/02_primary` and `02_analysis/03_secondary` — scripts/notebooks
- `02_analysis/04_visualizations` - visuals 

## Data availability
- Females: all doses.
- Males: shared doses (0, 900, 1200). One male lacked Iba1 signal (excluded).
- Raw QuPath exports are not tracked (`data/raw/` is gitignored).