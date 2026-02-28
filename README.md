# Caf-_Sales_Analysis

# ☕ Café Sales — Exploratory Data Analysis

> End-to-end data cleaning, validation, and exploratory analysis of 10,000 café transactions using Python. Covers missing-value imputation, feature engineering, and business-insight visualisation.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Project Structure](#project-structure)
- [Pipeline Summary](#pipeline-summary)
- [Key Findings](#key-findings)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Notebook Walkthrough](#notebook-walkthrough)
- [Design Decisions](#design-decisions)
- [Business Recommendations](#business-recommendations)
- [License](#license)

---

## Project Overview

This project performs a comprehensive EDA on a real-world café sales dataset. The primary goals are:

1. **Clean and validate** raw transactional data with multiple data quality issues (sentinel strings, mixed date formats, missing values across several columns)
2. **Impute missing values** using statistically appropriate strategies — not just convenience defaults
3. **Engineer time-based features** to enable temporal analysis
4. **Surface business insights** through structured aggregations and eight purpose-built visualisations

The project is aimed at intermediate Python practitioners and follows standard data-science project conventions.

---

## Dataset

| Property | Detail |
|---|---|
| File | `cafe_sales.xlsx` |
| Rows | 10,000 transactions |
| Columns | 8 |
| Period | Full year 2023 |
| Source | Synthetic café POS data |

### Columns

| Column | Type | Description |
|---|---|---|
| `Transaction ID` | `str` | Unique transaction identifier |
| `Item` | `str` | Menu item purchased |
| `Quantity` | `float` | Number of units |
| `Price Per Unit` | `float` | Unit price in GBP |
| `Total Spent` | `float` | Transaction value (may be missing or require recalculation) |
| `Payment Method` | `str` | Cash / Credit Card / Digital Wallet |
| `Location` | `str` | In-store / Takeaway |
| `Transaction Date` | `datetime` | Date of purchase (mixed raw formats) |

### Data Quality Issues Found

| Column | Issue | Volume |
|---|---|---|
| All columns | `'ERROR'` and `'UNKNOWN'` used as sentinel strings | Widespread |
| `Item` | True NaN after sentinel removal | 9.7% |
| `Payment Method` | True NaN after sentinel removal | 31.8% |
| `Location` | True NaN after sentinel removal | 39.6% |
| `Transaction Date` | Mixed formats + NaN | 1.6% |
| `Quantity` / `Price Per Unit` / `Total Spent` | Stored as `object`, NaN after cast | Sparse |

---

## Project Structure

```
cafe-sales-eda/
│
├── cafe_sales.xlsx                  # Raw data (not committed — see note below)
├── cafe_sales_analysis.ipynb        # Main analysis notebook
├── README.md                        # This file
└── requirements.txt                 # Python dependencies
```

> **Note:** Raw data files are not committed to version control. Place `cafe_sales.xlsx` in the project root before running the notebook.

---

## Pipeline Summary

```
Raw Excel
    │
    ├── Stage 1 │ Load & inspect  →  shape, dtypes, unique values, null audit
    │
    ├── Stage 2 │ Clean & validate
    │               ├── Replace sentinel strings ('ERROR', 'UNKNOWN') → NaN
    │               ├── Cast numeric columns (Quantity, Price, Total Spent) → float
    │               ├── Parse Transaction Date (two mixed formats)
    │               ├── Impute categoricals (Item, Payment Method, Location)
    │               │       └── proportional random sampling  ← see Design Decisions
    │               ├── Impute dates → median date
    │               ├── Recalculate Total Spent = Qty × Price where recoverable
    │               └── Fill remaining numeric NaN → item-level median
    │
    ├── Stage 3 │ Feature engineering  →  Month, Month Name, Day of Week, Quarter
    │
    ├── Stage 4 │ EDA aggregations  →  revenue by item / month / location / DoW / payment
    │
    ├── Stage 5 │ Visualisations  →  8 charts (bar, pie, donut, line, heatmap)
    │
    ├── Stage 6 │ KPI summary
    │
    └── Stage 7 │ Business recommendations
```

---

## Key Findings

| Metric | Value |
|---|---|
| Total Revenue | £88,952 |
| Total Transactions | 10,000 |
| Avg Transaction Value | £8.93 |
| Top Revenue Item | Salad (£17,320) |
| Top Location | In-store |
| Best Month | June (£8,073) |
| Busiest Day | Monday |

- **Salad** generates the most revenue despite not being the most frequently ordered item — it commands the highest average basket value
- **Juice** leads on transaction volume but with a lower unit price, ranking lower on total revenue
- Revenue peaks in **June**, dips in **January–February** — a clear seasonal pattern
- **Payment methods are almost perfectly split** (~33% each), making all three channels equally business-critical
- **In-store vs Takeaway** revenue is nearly identical, so neither channel should be deprioritised

---

## Tech Stack

| Library | Version | Purpose |
|---|---|---|
| `pandas` | ≥ 2.0 | Data loading, cleaning, aggregation |
| `numpy` | ≥ 1.25 | Numeric operations, random imputation |
| `matplotlib` | ≥ 3.7 | Base plotting layer |
| `seaborn` | ≥ 0.13 | Statistical visualisation, heatmap |
| `openpyxl` | ≥ 3.1 | Excel file engine for pandas |

---

## Getting Started

### Prerequisites

- Python 3.10+
- `pip` or `conda`

### Installation

```bash
# 1. Clone the repo
git clone https://github.com/SyusufWaliyyi/Cafe
_Sales_Analysis.git
cd Cafe_Sales_Analysis

# 2. Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch the notebook
jupyter lab cafe_sales_analysis_v2.ipynb
```

### `requirements.txt`

```
pandas>=2.0
numpy>=1.25
matplotlib>=3.7
seaborn>=0.13
openpyxl>=3.1
jupyter>=1.0
```

---

## Notebook Walkthrough

| Stage | Cells | What it does |
|---|---|---|
| **0 — Config** | 1 | Imports, global constants, matplotlib / seaborn theme |
| **1 — Load & Inspect** | 4 | Read Excel, `.info()`, raw unique values, null audit |
| **2 — Clean & Validate** | 9 | Full cleaning pipeline with before/after verification at each step |
| **3 — Feature Engineering** | 1 | Extracts Month, Month Name, Day of Week, Quarter from date |
| **4 — EDA Aggregations** | 4 | All groupby summaries computed and printed |
| **5 — Visualisations** | 8 | One chart per cell for easy iteration and reuse |
| **6 — KPI Summary** | 1 | Formatted metrics dictionary printed as a summary table |
| **7 — Recommendations** | Markdown | Business action points in a structured table |

---

## Design Decisions

### Why proportional random sampling for categorical imputation?

The standard approach of filling missing categoricals with the **mode** is only appropriate when one category clearly dominates. That isn't the case here:

| Column | Missing | Distribution | Mode fill effect |
|---|---|---|---|
| `Payment Method` | 31.8% | Cash / Credit Card / Digital Wallet ≈ 33% each | Artificially inflates Digital Wallet by ~10pp |
| `Location` | 39.6% | Takeaway 50.1% vs In-store 49.9% | Gives Takeaway a false ~10pp lead |
| `Item` | 9.7% | 8 items ranging 1,089–1,171 transactions | Inflates Juice at the expense of the other 7 |

The fix is to compute the **observed probability** of each category from non-null rows and use `np.random.default_rng` to sample replacements in those exact proportions. This preserves the natural distribution and avoids introducing bias into any downstream analysis.

```python
def impute_categorical_proportional(series, seed=42):
    rng          = np.random.default_rng(seed)
    null_mask    = series.isna()
    value_counts = series.dropna().value_counts(normalize=True)
    fill_values  = rng.choice(value_counts.index, size=null_mask.sum(), p=value_counts.values)
    result       = series.copy()
    result.loc[null_mask] = fill_values
    return result
```

A `seed` parameter is accepted so the imputation is fully reproducible.

### Why item-level medians for numeric imputation?

A single global median for `Price Per Unit` would be meaningless — a Coffee and a Salad have very different price points. Using `groupby('Item').transform('median')` ensures each gap is filled with a value that is plausible *for that specific item*.

### Why recalculate before imputing `Total Spent`?

Where both `Quantity` and `Price Per Unit` are already known, `Total Spent` can be recovered exactly as `Qty × Price`. Deterministic recovery is always preferable to statistical imputation. Imputation is only used for the rows where the underlying components are also missing.

---

## Business Recommendations

| # | Finding | Recommendation |
|---|---|---|
| 1 | Salad is the top revenue item | Bundle Salad + Juice as a meal deal — combine the highest revenue item with the highest volume item |
| 2 | Revenue dips in Jan–Feb | Launch a winter loyalty campaign (stamp cards, hot-drink discounts) to smooth seasonal volatility |
| 3 | In-store and Takeaway revenue nearly equal | Don't neglect the Takeaway channel — faster payment flow and better packaging could push it ahead |
| 4 | Monday is the busiest day | Investigate the driver (office crowd?) and explore B2B catering partnerships |
| 5 | Payment split is exactly 33% each | All three terminal types must be reliable; consider Digital Wallet incentives for throughput speed |
| 6 | June peaks | Plan seasonal specials (iced drinks, salad promotions) and staff up ahead of the summer surge |

---

## License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE) for details.

---

*Built with Python · pandas · seaborn · matplotlib*
