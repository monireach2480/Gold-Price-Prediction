# Gold Price Prediction

Final project for Data Analysis 1 (Dr. HAS Sothea, Fall 2026).

Team: Len Monireach (2024498) and Dy David (2024583).

We took 25 years of daily gold futures prices and asked how far tomorrow's price can be predicted from the price history alone. The answer has three parts:

| Question | Answer | Evidence on 1,029 unseen test days |
|---|---|---|
| Tomorrow's price | No better than today's price plus the long-run drift | All eight forecasts are within 0.6% of the naive forecast (RMSE of returns), and none beats the drift |
| Tomorrow's direction | No | Accuracy of 55.0% to 56.1% against 55.6% for always saying "up" |
| The size of tomorrow's move | Partly | A ridge model explains 11% of the variance (6% through 2025), and a 95% band scaled by it covers 93% to 96% of days in each full test year |

Along the way we found that the "cleaned" Kaggle file needed checking: 7 of its 20 columns are redundant, its 365-day average covers 1.45 years, 430 early rows have a close outside the day's own high-low range, and all 520 missing values are rolling-window warm-up rows that should not be imputed.

## What is in this folder

```
gold_price_analysis.ipynb    the whole analysis, with the outputs of a full run
data/                        the dataset (CSV, 6,383 rows, 20 columns)
figures/                     the 28 figures the notebook saves
results/                     every table as CSV, plus metrics.json with the numbers quoted in the report
report/                      the final report (PDF and Word)
requirements.txt             Python packages and the versions we used
```

The notebook follows the sections of the report: data preprocessing, exploratory analysis, model development, results, conclusions.

## Run it

You need Python 3.13 (the version we tested with). The commands below are for PowerShell on Windows.

```powershell
# create and activate a virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1

# install the packages
python -m pip install --upgrade pip
pip install -r requirements.txt

# open the notebook
jupyter lab gold_price_analysis.ipynb
```

If PowerShell refuses to run the activation script, allow scripts for the current window and try again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

To run the notebook from top to bottom without opening it:

```powershell
jupyter nbconvert --to notebook --execute --inplace gold_price_analysis.ipynb
```

A full run takes a few minutes. It rewrites `figures/` and `results/`. The random seed is fixed at 42, so with the package versions in `requirements.txt` you should get the same numbers as the report.

## Data

`data/gold_data_daily_comprehensive_cleaned.csv` is the file "Gold Price Historical Data (2000-2026)" from Kaggle:
https://www.kaggle.com/datasets/hamzasamiullah/gold-price-historical-data-2000-2026

It holds daily prices of COMEX gold futures (Yahoo Finance ticker GC=F) from 30 August 2000 to 6 February 2026. Check the licence on the Kaggle page before making this repository public. If you would rather not redistribute the file, delete it from `data/` and download it from the link above.

## Method in short

- 37 features built from the price history, all known at the close of day t, in six families: recent returns, momentum, trend, volatility, oscillators, calendar. A test that scrambles the future confirms that none of them looks ahead.
- Three targets for day t + 1: the return (converted to a price), the direction, and the absolute return.
- Chronological split: train on 20 February 2002 to 31 December 2021, test on 3 January 2022 to 5 February 2026.
- Tuning by grid search over five expanding-window folds inside the training years.
- Benchmarks: naive, drift, ARIMA, always-up, training mean, Volatility_30d, EWMA. Models: linear regression, ridge, lasso, logistic regression, random forest, XGBoost.
- Comparisons use Diebold-Mariano and McNemar tests, and every headline result is repeated without the most extreme day (30 January 2026).


