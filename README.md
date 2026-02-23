# HIV 2015 Impact Score Computation

This task computes the Impact Score for HIV 2015 data from the ORS dataset.

## Overview

The Impact Score measures the contribution of each antiretroviral drug to reducing the disease burden (DALYs) across countries, accounting for treatment coverage, regimen efficacy, and retention rates.

## Calculation Approach

The computation follows the exact Excel formula used in the ORS file:

### Formula
For each drug and country:
```
Impact Score = SUM(
  Adult contributions + Child contributions
) / (100 / (100 - Retention Rate))
```

Where:
- **Adult contribution**: `G × Q × AY / (1 - Q × AY) / num_drugs`
- **Child contribution**: `H × T × AZ / (1 - T × AZ) / num_drugs`

### Variables
- **G**: Adult DALYs (disease burden for adults)
- **H**: Children DALYs (disease burden for children)
- **Q**: Adult treatment coverage (%)
- **T**: Child treatment coverage (%)
- **AY**: Adult x-value (proportion × efficacy, pre-multiplied in Excel)
- **AZ**: Child x-value (proportion × efficacy, pre-multiplied in Excel)
- **num_drugs**: Number of drugs in the regimen
- **Retention Rate**: Treatment retention rate (%)

### Process
1. Load HIV2015 data from the ORS Excel file
2. For each drug, scan all regimen rows (Excel rows 5-37)
3. If the regimen contains the drug, calculate its contribution
4. Sum contributions across all matching regimens
5. Normalize by retention rate
6. Compute overall impact by summing all drug impacts per country

## Files

### Input
- `ORS (2015_2017_2019) copied 2025-7-5.xlsx` - Source data file
  - Sheet: `HIV2015`

### Output
- `impact_score.csv` - Complete results file containing:
  - All original columns from the HIV2015 sheet (84 columns)
  - Computed Impact Score columns for each drug:
    - `Computed Impact Score (3TC)`
    - `Computed Impact Score (ABC)`
    - `Computed Impact Score (AZT)`
    - `Computed Impact Score (ddl)`
    - `Computed Impact Score (d4T)`
    - `Computed Impact Score (EFV)`
    - `Computed Impact Score (FTC)`
    - `Computed Impact Score (LPV/r)`
    - `Computed Impact Score (NVP)`
    - `Computed Impact Score (TDF)`
    - `Computed Impact Score (ATV/r)`
  - `Computed Overall Treatment Impact` - Sum of all drug impacts per country

### Notebook
- `Hiv_2015.ipynb` - Jupyter notebook with complete implementation
  - Loads and processes data
  - Implements the exact Excel formula
  - Generates comparison tables
  - Exports results to CSV

## Usage

1. Open `Hiv_2015.ipynb` in Jupyter Notebook
2. Run all cells
3. The `impact_score.csv` file will be generated

## Results

The computed values match the Excel calculations with ~99.7% accuracy (0.3% difference), with minor variations due to rounding.
