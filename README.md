# tqqq-data-utils

Python utilities for building a synthetic pre-inception history for TQQQ and for estimating the hidden daily drag embedded in TQQQ-like leveraged ETF behavior.

I started this project because TQQQ only begins in 2010, which makes simple backtests vulnerable to inception bias and excludes stress regimes such as the 2008 financial crisis. The goal here is to extend the TQQQ tradeable history backward using Nasdaq-100 data, then test strategies against a longer synthetic series that is anchored to the ETF's actual post-inception performance.

The repository has two parts:

1. `synthetic_tqqq_data/` builds synthetic TQQQ price history from Yahoo Finance data.
2. `tqqq_fee_analysis/` estimates the daily and annual drag that best explains observed TQQQ performance over a chosen period.

## What This Repo Produces

### Raw market data

Written to `synthetic_tqqq_data/data/raw/`:

- `nasdaq_100_yahoo.csv` - Nasdaq-100 index data with `date, Open, Close`
- `tqqq_yahoo.csv` - actual TQQQ data with `date, Open, Close`

### Synthetic TQQQ data

Written to `synthetic_tqqq_data/data/processed/`:

- `tqqq_backfilled_0.0%.csv`
- `tqqq_backfilled_5.0%.csv`
- `tqqq_backfilled_10.0%.csv`

These files contain modeled TQQQ `date, open, close` values. The repository currently generates the `10.0%` version by default because `ANNUAL_FEE` is set to `10.0` in the backfill script.

### Fee analysis outputs

Written to `tqqq_fee_analysis/`:

- `yearly_tqqq_fee_summary.csv` - year-by-year solved drag from 2010 through 2025
- `tqqq_fee_validation.xlsx` - supporting validation workbook

## How The Synthetic Backfill Works

The backfill script downloads:

- `^NDX` as the market proxy
- `TQQQ` as the anchor series for the post-2010 period

It then rebuilds TQQQ backward from the first available actual TQQQ close using a simple leveraged model:

- close-to-close movement is approximated with a 3x Nasdaq-100 return minus an annual fee converted to a daily fee
- open prices are modeled separately using the overnight move from the previous close to the next open

The key idea is not that the output is a literal historical TQQQ tape. It is a controlled synthetic series that preserves the actual TQQQ history where it exists and extrapolates earlier behavior using a transparent rule set.

## How To Interpret The Data

- Raw CSVs are source data pulled from Yahoo Finance.
- Processed CSVs are synthetic and should be treated as model output, not as observed market history.
- The generated series is useful for strategy research, especially when you want to evaluate behavior through periods that TQQQ did not yet exist.
- The fee-analysis scripts are meant to help estimate a reasonable fee assumption for backfilling. They do not prove the exact issuer fee or all frictions in live trading.
- Negative solved drag in a year means the 3x model plus the chosen fee assumption still underperformed the actual TQQQ path over that period. That can happen because leveraged ETFs are path dependent.

## Project Layout

```text
synthetic_tqqq_data/
	data/
		raw/        downloaded Yahoo Finance data
		processed/  synthetic TQQQ backfills
	scripts/
		download_tqqq_backfill.py
	tests/
		test_raw_nasdaq_100_csv.py
		test_raw_tqqq_csv.py
		test_processed_tqqq_backfilled_csv.py

tqqq_fee_analysis/
	calculate_tqqq_fee_period.py
	calculate_tqqq_fee_yearly.py
	yearly_tqqq_fee_summary.csv
	tqqq_fee_validation.xlsx
```

## Environment Setup

This repo targets Python 3.11 on Windows and uses a local virtual environment in `.venv`.

### PowerShell

```powershell
Set-Location 'c:\Users\rohmi\tqqq-data-utils'
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Command Prompt

```bat
cd /d c:\Users\rohmi\tqqq-data-utils
python -m venv .venv
.venv\Scripts\activate.bat
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

The pinned dependencies are:

- `pandas==3.0.3`
- `numpy==2.4.6`
- `yfinance==1.4.1`

## Run The Backfill Generator

From the repository root:

```powershell
.\.venv\Scripts\python.exe synthetic_tqqq_data\scripts\download_tqqq_backfill.py
```

This downloads fresh Yahoo Finance data and writes:

- `synthetic_tqqq_data/data/raw/nasdaq_100_yahoo.csv`
- `synthetic_tqqq_data/data/raw/tqqq_yahoo.csv`
- `synthetic_tqqq_data/data/processed/tqqq_backfilled_10.0%.csv`

If you want a different synthetic fee assumption, change `ANNUAL_FEE` in `synthetic_tqqq_data/scripts/download_tqqq_backfill.py` and rerun the script. The output filename is derived from that value.

## Run The Fee Analysis

### Year-by-year drag summary

```powershell
.\.venv\Scripts\python.exe tqqq_fee_analysis\calculate_tqqq_fee_yearly.py
```

Optional date range:

```powershell
.\.venv\Scripts\python.exe tqqq_fee_analysis\calculate_tqqq_fee_yearly.py --start-year 2015 --end-year 2025
```

### Single-period drag estimate

```powershell
.\.venv\Scripts\python.exe tqqq_fee_analysis\calculate_tqqq_fee_period.py --start-date 2020-01-01 --end-date 2020-12-31
```

The fee scripts solve for a constant daily drag `g` that makes the modeled 3x Nasdaq-100 path match observed TQQQ performance over the selected period.

## Run The Tests

Run the test suite from the repository root:

```powershell
.\.venv\Scripts\python.exe -m unittest discover -s synthetic_tqqq_data\tests
```

The tests verify:

- the raw Nasdaq-100 CSV shape and sample rows
- the raw TQQQ CSV shape and sample rows
- the processed synthetic CSVs for the `0.0%`, `5.0%`, and `10.0%` fee variants

## Important Caveats

- The backfilled data is synthetic. It is useful for research, but it is not an official historical record of TQQQ.
- The model uses a fixed 3x leverage assumption and a simplified fee drag conversion.
- Yahoo Finance is an external dependency; data availability and adjustments can change over time.
- The fee analysis is descriptive, not a guarantee of future tracking behavior.
- The repository currently stores generated CSVs in version control so the tests can validate against concrete artifacts.

## Reproducibility Note

The repository has been set up and validated with a local `.venv` on Windows. If you regenerate the data, expect the processed CSVs and summary outputs to change when Yahoo Finance data or the chosen fee assumption changes.