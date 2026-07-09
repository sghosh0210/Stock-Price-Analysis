# Stock Market Dashboard — Indian Equities Tracker

A Power BI dashboard that tracks daily OHLCV price data for 13 NSE/BSE-listed
Indian stocks, sourced live from Yahoo Finance and refreshed through an
embedded Python script.

## Overview

<img width="984" height="550" alt="image" src="https://github.com/user-attachments/assets/60497e94-f679-457a-8ec5-87a9ead3917a" />


| | |
|---|---|
| **Data source** | Yahoo Finance (via the `yfinance` Python package) |
| **Automation** | Python script, run inside Power Query as a Python data source |
| **Dashboard** | Power BI (`Stock_market_dashboard.pbix`) |
| **Coverage** | 13 stocks · daily data from Jan 1, 2015 to present · filtered to current year in the report |
| **Refresh** | Manual or scheduled refresh in Power BI re-runs the Python script and pulls the latest prices |

## Stocks tracked

| Ticker | Company | Sector |
|---|---|---|
| TCS.NS | Tata Consultancy Services | IT Services |
| INFY.NS | Infosys | IT Services |
| WIPRO.NS | Wipro | IT Services |
| SBIN.NS | State Bank of India | Banking |
| HDFCBANK.NS | HDFC Bank | Banking |
| ICICIBANK.NS | ICICI Bank | Banking |
| AXISBANK.NS | Axis Bank | Banking |
| BANKBARODA.NS | Bank of Baroda | Banking |
| MARUTI.NS | Maruti Suzuki | Auto |
| HEROMOTOCO.NS | Hero MotoCorp | Auto |
| HONDAPOWER.NS | Honda Power | Auto |
| TMCV.BO | Tata Motors | Auto |
| RELIANCE.NS | Reliance Industries | Energy / Conglomerate |

## How it works

### 1. Data collection (Python + yfinance)

A Python script pulls historical OHLCV data for every ticker and stacks it
into a single table. This is the script embedded in the dashboard's Power
Query source:

```python
import datetime
import yfinance as yf
import pandas as pd

all_stock_symbol = [
    'TCS.NS', 'WIPRO.NS', 'INFY.NS', 'SBIN.NS',
    'HDFCBANK.NS', 'BANKBARODA.NS', 'ICICIBANK.NS', 'AXISBANK.NS',
    'TMCV.BO', 'MARUTI.NS', 'HEROMOTOCO.NS', 'HONDAPOWER.NS',
    'RELIANCE.NS'
]

empty_df = pd.DataFrame()

for ticker in all_stock_symbol:
    start_date = datetime.datetime(2015, 1, 1)
    end_date = datetime.datetime.now()
    meta = yf.Ticker(ticker)
    data = meta.history(start=start_date, end=end_date)
    data['Company_Name'] = ticker
    empty_df = pd.concat([empty_df, data])

stock_df = empty_df.copy()
stock_df = empty_df.reset_index()
```

Output columns: `Date, Open, High, Low, Close, Volume, Dividends, Stock Splits, Company_Name`.

Install the one dependency this needs:

```bash
pip install yfinance pandas
```

### 2. Importing into Power BI

The script is wired into Power BI as a **Python script data source**, so
Power BI runs it directly and loads `stock_df` as a table:

1. Power BI Desktop → **Get Data → More → Other → Python script**
2. Paste the script above into the Python script editor
3. In the Navigator, select the `stock_df` result
4. Power BI's Power Query then applies light transforms on top:
   - casts `Date` to date/time, `Open/High/Low/Close/Dividends` to decimal, `Volume/Stock Splits` to whole number, `Company_Name` to text
   - filters rows to `Date.IsInCurrentYear([Date])` so the report always shows the running year

A second table, `Meta_info` (ticker → company name, sector, logo URL), is
imported from an Excel workbook and joined to `stock_table` on
`Company_Name` to drive labels and icons in the report.

### 3. Refreshing the data

Because the source is a live Python script rather than a static file, hitting
**Refresh** in Power BI re-executes the script and pulls the latest Yahoo
Finance prices — no manual re-export needed. For hands-off updates, set up a
[scheduled refresh](https://learn.microsoft.com/power-bi/connect-data/refresh-scheduled-refresh)
in the Power BI service (note: Python-based data sources need a
[personal gateway](https://learn.microsoft.com/power-bi/connect-data/service-gateway-personal-mode)
with Python + `yfinance` installed to refresh in the cloud).

### 4. Dashboard

`Stock_market_dashboard.pbix` visualizes the resulting table with:

- Candlestick / HLOC charts per stock (custom visuals)
- A chiclet slicer to switch between companies
- Volume, price trend, and sector breakouts

## Repo structure

```
.
├── Stock_market_dashboard.pbix   # Power BI report (data model + visuals)
├── 01_data.ipynb           # standalone version of the Python collection script
└── README.md                    # this file
```

## Notes

- Data reflects whatever was cached the last time the report was refreshed — refresh in Power BI Desktop to pull current prices before relying on it for live decisions.
- `TMCV.BO` is the BSE listing for Tata Motors; all other tickers are NSE (`.NS`).
