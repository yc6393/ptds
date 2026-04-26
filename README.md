# Analysis of Financial Time Series

Course project for **NYU CSCI-US 381 — Programming Tools for Data Scientists**, Spring 2025.

**Authors:** Yuri Chentsov, Amish Vandse, Benson Yu

A statistical and machine-learning study of four financial time series — the **S&P 500**, **US 10-Year Treasury yield**, **WTI crude oil**, and **Bitcoin** — over a roughly ten-year window pulled live from the [FRED (Federal Reserve Economic Data)](https://fred.stlouisfed.org/) API. The project covers return distributions, static and rolling correlations, market-regime detection with K-Means clustering, and supervised regime classification with a Support Vector Machine.

## Project structure

```
ptds/
├── PTDS_Course_Project_Final.ipynb     # Main Jupyter notebook (analysis + write-up)
├── ptds_project_data.csv               # Cached daily price/yield data from FRED
├── PTDS_Course_Project.zip             # Archived snapshot of the deliverable
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Data

Series pulled from FRED:

| Column     | FRED ID       | Description                                     |
| ---------- | ------------- | ----------------------------------------------- |
| `SP500`    | `SP500`       | S&P 500 index (price)                           |
| `US_10Y`   | `DGS10`       | 10-Year US Treasury constant-maturity yield (%) |
| `Oil`      | `DCOILWTICO`  | Crude oil prices, WTI (USD per barrel)          |
| `Bitcoin`  | `CBBTCUSD`    | Bitcoin / USD spot price (Coinbase)             |

The CSV in this repo is a cached snapshot (~2,600 daily observations through April 2026). The notebook will pull a fresh copy if you provide a FRED API key.

## Requirements

- Python 3.12 (the notebook was developed against this; 3.10+ should also work)
- Jupyter Notebook or JupyterLab
- Packages listed in `requirements.txt`:
  - `pandas`, `numpy`
  - `matplotlib`, `seaborn`
  - `scikit-learn`
  - `scipy`
  - `fredapi`

## Setup

Clone and install:

```bash
git clone https://github.com/yc6393/ptds.git
cd ptds
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### FRED API key

The notebook fetches data through `fredapi`, which needs a personal API key. Get one for free from the [FRED API key page](https://fred.stlouisfed.org/docs/api/api_key.html), then expose it as an environment variable rather than hard-coding it:

```bash
export FRED_API_KEY="your_key_here"            # Windows (PowerShell): $Env:FRED_API_KEY="your_key_here"
```

In the notebook this becomes:

```python
import os
from fredapi import Fred
fred = Fred(api_key=os.environ["FRED_API_KEY"])
```

> **Security note.** Earlier commits of this repository include a hard-coded FRED API key. Treat that key as compromised, [revoke and rotate it](https://fred.stlouisfed.org/docs/api/api_key.html), and never commit secrets back to the repo.

## Usage

Launch Jupyter and open the notebook:

```bash
jupyter notebook PTDS_Course_Project_Final.ipynb
```

Run the cells in order. Section 1 will fetch fresh data from FRED and overwrite `ptds_project_data.csv`. If FRED is unreachable (the notebook notes occasional proxy/auth errors), re-running the cell or restarting the kernel usually clears it; alternatively, skip the fetch and load the cached CSV directly:

```python
data = pd.read_csv("ptds_project_data.csv", index_col=0, parse_dates=True)
```

## Notebook contents

1. **Historical data upload** — fetches daily series from FRED and writes a CSV cache.
2. **Returns calculation and normalization** — computes daily, weekly, and monthly returns; standardizes them with `StandardScaler`.
3. **Analysis of return distributions** — descriptive statistics, skewness, kurtosis, Jarque-Bera and Shapiro-Wilk normality tests, and distribution plots with normal overlays.
4. **Correlation analysis** — static correlation matrix, rolling correlations across 1-month, 3-month, and 1-year windows, and pairwise scatter plots colored by year.
5. **ML for market regime detection** — K-Means clustering of returns into low / medium / high-volatility regimes, regime-labeled price and return visualizations, and an SVM classifier benchmarked with `train_test_split` and a classification report. Cluster quality is assessed with the silhouette score.
6. **Conclusion** — synthesis of the statistical and ML findings.

## Selected findings

A few highlights from the write-up inside the notebook:

- **Volatility ranking flips with horizon.** Oil is the most volatile asset on a daily basis, but Bitcoin overtakes it at weekly and monthly horizons. The S&P 500 is the least volatile across all timeframes.
- **Distributions are not stable across years.** Pairwise scatter plots colored by year show that both individual return distributions and pairwise relationships shift meaningfully between calendar years.
- **Correlations are not stable either.** Static correlations are all positive but weak (< 0.3); rolling correlations swing widely — for example, S&P 500 vs US 10Y yields have ranged from roughly +0.6 to -0.2 depending on the period.
- **Three clear volatility regimes** emerge from K-Means clustering: a calm baseline, a periodic stress regime, and a crisis regime concentrated around the COVID shock. Silhouette scores are modest, which is consistent with the gradual, overlapping nature of regime transitions in real markets.

## License

This project is released under the [MIT License](LICENSE). FRED data is provided by the Federal Reserve Bank of St. Louis under its own [terms of use](https://fred.stlouisfed.org/legal/).
