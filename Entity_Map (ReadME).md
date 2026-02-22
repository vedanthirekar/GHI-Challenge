# HIV 2013 ORS – Entity Map + Impact Score Computation

This notebook (`Hiv_2013_entity_map.ipynb`) loads the **HIV 2013 ORS** worksheet and reproduces the spreadsheet’s **Impact Score** calculations in Python.

Although the workbook contains many sections (country data, constants, regimen tables, notes/links), the core computation relies on a small set of **mapped entities**:

- Country-level inputs (DALYs, WHO group, treatment coverage, retention rate)
- First-line and second-line regimen tables (adult/child regimen shares + efficacies)
- A small set of constants used in the Excel formula
- The list of drugs (derived from columns named `Impact of …`)

The “entity map” in this notebook is encoded as **Excel-style row/column references** that point Python to the correct parts of the sheet.

---

## What this notebook outputs

- Adds one column per drug: `Computed Impact Score (<drug>)`
- Adds a per-country sum: `Computed Overall Treatment Impact`
- Exports the updated dataset to: `updated_HIV2013_nb.csv`

---

## Requirements

Python packages used:
- `pandas`
- `numpy`
- `requests` (only needed if you load the Google Sheet via the export URL)
- `openpyxl` (required by `pandas.read_excel` for `.xlsx`)

> If `pd.read_excel(url, …)` fails with permissions / 403 / no internet, download the sheet as `.xlsx` and load it from disk.

---

## How to run (high level)

1. **Load** the HIV 2013 sheet into a pandas DataFrame
2. **Fix headers** (the ORS template uses multi-row headers)
3. **Compute impact scores** for every `Impact of …` drug column by scanning regimen tables
4. **Compute overall impact** by summing drug impacts per country
5. **QA check**: compare computed vs. “Actual” columns in the ORS
6. **Export** to CSV

---

# Entity map definition (HIV 2013 ORS)

The ORS template is treated like an Excel grid. The notebook converts Excel column letters (e.g., `E`, `AS`, `AP`) to **0-based pandas column indices** using `_col_idx()`.

## 1) Country-level inputs (per-row)

These are read from each row `i` when computing impact scores:

| Excel column | Meaning in code | Notes |
|---|---|---|
| `E` | `WHO Group` (`A`/`B`) | Determines which constants + which regimen table rows to use |
| `G` | `Adult DALYs` | Base burden for adult contribution |
| `H` | `Children DALYs` | Base burden for child contribution |
| `I` | `Retention Rate` | Used for normalization at the end |
| `Q` | Adult coverage factor | “Adult” treatment coverage used in the Excel model |
| `T` | Child coverage factor | “Child” treatment coverage used in the Excel model |

## 2) Constants used by the Excel formula

The Excel model uses different constants for WHO Group **A** vs **B** and for first-line vs second-line regimens.

These constants are pulled from fixed cells in the sheet:

### WHO Group B constants (column `AP`)
- `AP5`  → First-line (adult/adolescent) constant  
- `AP6`  → Second-line (adult/adolescent) constant  
- `AP10` → First-line (child) constant  
- `AP11` → Second-line (child) constant  

### WHO Group A constants (column `AQ`)
- `AQ5`  → First-line (adult/adolescent) constant  
- `AQ6`  → Second-line (adult/adolescent) constant  
- `AQ10` → First-line (child) constant  
- `AQ11` → Second-line (child) constant  

> In code these are extracted once, then selected based on the row’s `WHO Group`.

## 3) Regimen tables (shared lookups)

The ORS contains two regimen “blocks”:

- **First-line regimens** (adult/adolescent + child shares + efficacies)
- **Second-line regimens** (adult/adolescent + child shares + efficacies)

### Regimen table columns

**First-line regimen block**
| Excel col | Meaning |
|---|---|
| `AS` | Regimen string (e.g., `AZT + 3TC + NVP`) |
| `AT` | Adult/adolescent regimen share (%) |
| `AU` | Adult/adolescent efficacy |
| `AV` | Child regimen share (%) |
| `AW` | Child efficacy |

**Second-line regimen block**
| Excel col | Meaning |
|---|---|
| `AX` | Regimen string |
| `AY` | Adult/adolescent regimen share (%) |
| `AZ` | Adult/adolescent efficacy |
| `BA` | Child regimen share (%) |
| `BB` | Child efficacy |

### Regimen table row ranges (Excel row numbers)

The notebook points to the regimen tables using fixed Excel row ranges:

- **WHO Group B regimen rows:** `8–20`
- **WHO Group A regimen rows:** `27–37`

These are set in:
- `b_regimen_excel_rows = range(8, 21)`
- `a_regimen_excel_rows = range(27, 38)`

## 4) Impact score formula implemented (per drug, per country)

For a given **country row** and a given **drug**, the notebook:

1. Selects the correct regimen rows and constants based on `WHO Group` (`A` vs `B`)
2. Scans every regimen row:
   - If the regimen string contains the drug token, it contributes to the drug’s impact
3. Adds up **adult** and **child** contributions across:
   - first-line regimens
   - second-line regimens
4. Applies the retention-rate normalization

### Contribution from one matching regimen

For each matching regimen, the contribution is computed as:

