# Weekly Sales Forecasting Pipeline

A Python-based forecasting pipeline that processes online retail transaction data, generates weekly sales aggregates, and compares three forecasting approaches — Linear Trendline, Recursive Moving Average, and Facebook Prophet — to support data-driven business decisions.

---

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Data Source](#data-source)
- [Pipeline Structure](#pipeline-structure)
- [How to Run](#how-to-run)
- [Outputs](#outputs)
- [Methodology](#methodology)
- [Results Interpretation](#results-interpretation)

---

## Overview

This project takes two years of online retail transaction data (2009–2011), cleans and aggregates it to a weekly level, then builds and evaluates three forecasting models:

1. **Linear Trendline** — captures long-term directional growth
2. **Recursive 4-Week Moving Average** — smooths short-term noise
3. **Prophet** — decomposes trend, seasonality, and uncertainty intervals

The pipeline also supports SKU-level forecasting for the top 15 products by revenue.

---

## Requirements

Install dependencies via pip:

```bash
pip install pandas numpy matplotlib prophet openpyxl
```

| Package | Purpose |
|---------|---------|
| `pandas` | Data manipulation and time-series aggregation |
| `numpy` | Numerical operations, linear regression |
| `matplotlib` | Visualization (scatter plots, trendlines, forecasts) |
| `prophet` | Automated time-series forecasting with seasonality |
| `openpyxl` | Reading `.xlsx` source files |

---

## Data Source

This project uses the **Online Retail II** dataset from the UCI Machine Learning Repository.

> **Citation:** Chen, S. (2019). *Online Retail II* [Dataset]. UCI Machine Learning Repository.  
> https://archive.ics.uci.edu/dataset/502/online+retail+ii

The pipeline expects an Excel file named **`online_retail_II.xlsx`** with two sheets:

| Sheet | Period |
|-------|--------|
| `Year 2009-2010` | First year of transactions |
| `Year 2010-2011` | Second year of transactions |

**Expected columns:** `Invoice`, `Price`, `Customer ID`, `Description`, `Quantity`, `UnitPrice`, `InvoiceDate`

---

## Pipeline Structure

### 1. Data Ingestion & Cleaning

```python
s1 = pd.read_excel('online_retail_II.xlsx', sheet_name='Year 2009-2010')
s2 = pd.read_excel('online_retail_II.xlsx', sheet_name='Year 2010-2011')
df = pd.concat([s1, s2], ignore_index=True)
```

**Cleaning steps:**
- Renames columns to standard names (`InvoiceNo`, `UnitPrice`, `CustomerID`)
- Drops rows with missing `Description`
- Filters out negative/zero `Quantity` and `UnitPrice`
- Strips whitespace from product descriptions
- Excludes non-product lines (postage, fees, samples, etc.)
- Computes `Sales = Quantity × UnitPrice`

### 2. Weekly Aggregation

Aggregates cleaned transactions to weekly totals:

```
weekly_total_sales_2yr.csv
```

Also extracts the **Top 15 products by total revenue** and aggregates them weekly:

```
weekly_top15_products_2yr.csv
```

### 3. Forecasting Models

#### A. Linear Trendline
- Fits a least-squares linear regression on weekly sales
- Extrapolates 16 weeks (~4 months) into the future
- **Output:** `scatter_trendline_extrapolated.png`

#### B. Recursive 4-Week Moving Average
- Computes a rolling 4-week mean on historical data
- Recursively forecasts by appending each new prediction back into the window
- **Forecast horizon:** 4 weeks

#### C. Prophet Model
- Configured with `yearly_seasonality=True`, `weekly_seasonality=False`
- Uses additive seasonality mode
- Generates 95% prediction intervals via Monte Carlo simulation
- **Forecast horizon:** 26 weeks (~6 months)
- **Outputs:** Main forecast plot + component decomposition plot

### 4. Backtest Comparison

Holds out the **last 12 weeks** of actual data as a blind test set. Each model is trained only on earlier data, then evaluated against the held-out period.

**Metrics computed:**
- **MAE** (Mean Absolute Error) — average magnitude of errors
- **MAPE** (Mean Absolute Percentage Error) — error relative to actual values

**Output:** `three_way_comparison.csv`

### 5. Product-Level Analysis

Ranks all products by total revenue and isolates the #1 best-selling product for potential individual SKU forecasting.

---

## How to Run

1. Place `online_retail_II.xlsx` in the working directory.
2. Run the script sections sequentially (or as a single notebook/script).
3. Generated CSVs and PNGs will appear in the working directory.

```bash
python forecasting_pipeline.py
```

---

## Outputs

| File | Description |
|------|-------------|
| `weekly_total_sales_2yr.csv` | Weekly total sales across all products |
| `weekly_top15_products_2yr.csv` | Weekly sales, quantity, price, and orders for top 15 SKUs |
| `scatter_trendline_extrapolated.png` | Scatter plot of actual sales with linear trendline and 16-week projection |
| `three_way_comparison.csv` | Side-by-side actual vs. predicted values for the 12-week holdout |
| *(displayed)* | Prophet forecast plot with 95% confidence band |
| *(displayed)* | Prophet component plot (trend + yearly seasonality) |

---

## Methodology

### Why Three Models?

| Model | Strength | Weakness |
|-------|----------|----------|
| **Linear Trendline** | Simple, interpretable slope (£/week growth) | Assumes constant growth forever; misses seasonality |
| **Moving Average** | Smooths noise; tracks recent momentum | Flat-lines after the last window; no seasonality |
| **Prophet** | Detects yearly cycles, trend shifts, holidays; gives uncertainty bands | Requires more data; slightly more complex |

### Seasonality Handling

- The business exhibits strong **Nov/Dec holiday peaks** and **post-holiday troughs**.
- Prophet automatically detects this yearly cycle via Fourier series decomposition.
- Simple methods treat Christmas week identically to March, missing the core revenue cycle.

### Train/Test Split

- **Training data:** All weeks except the final 12
- **Test data:** Final 12 weeks (held out from all models)
- This ensures a fair, out-of-sample comparison of predictive accuracy.

---

## Results Interpretation

After running the comparison block, you will see output like:

```
Trendline   MAE:    123,456   MAPE:  25.3%
MovingAvg   MAE:     98,765   MAPE:  19.8%
Prophet     MAE:     45,210   MAPE:   8.4%
```

**Typical expectation:** Prophet achieves the lowest MAE/MAPE because it captures seasonality and trend changes that linear and MA methods miss.

### Business Actions Enabled

| Decision | How Forecast Helps |
|----------|-------------------|
| **Inventory** | Order stock ahead of predicted Nov/Dec peaks |
| **Staffing** | Scale warehouse/customer-service capacity to forecasted demand |
| **Cash Flow** | Tie purchasing schedules to predicted revenue curves |
| **Marketing** | Promote SKUs where forecast underperforms current inventory |
| **Discontinuation** | Flag products with flat/negative trend components |

---

## Extending the Pipeline

- **Add holidays:** Feed a custom `holidays` DataFrame into Prophet for Black Friday, Christmas, etc.
- **SKU-level Prophet:** Loop the Prophet block over each of the Top 15 products for individual product forecasts.
- **Longer horizon:** Increase `forecast_periods` in Prophet or `extra_weeks` in the trendline.
- **Cross-validation:** Use Prophet's built-in `cross_validation` and `performance_metrics` for robust error estimation.

---

## License

This project is provided as-is for educational and analytical purposes.
