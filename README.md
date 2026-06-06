# NECC Egg Price Time Series Data

This project collects and prepares daily egg price data from the NECC egg price website:

https://www.e2necc.com/home/eggprice

The goal is to create a clean daily time-series dataset that can be used for market comparison, trend analysis, interpolation, forecasting, and other data analysis work.

## Project Files

- `scraper.py`  
  Downloads monthly HTML reports from the NECC website, parses daily egg prices, creates a complete daily time-series grid, and saves the prepared CSV.

- `necc_egg_prices_daily.csv`  
  Final prepared dataset. It contains daily price data for each market and price category.

- `raw_html/`  
  Cached monthly HTML pages downloaded from the website. These files make it possible to rebuild the dataset without downloading every page again.

- `Egg_data_analysis.ipynb`  
  Jupyter notebook used for analysis on the prepared egg price dataset. This notebook can be used for exploratory data analysis, visualizations, market-wise comparison, missing value inspection, interpolation checks, and trend analysis.

## Dataset Summary

Current prepared CSV summary:

- Date range: `2009-01-01` to `2026-05-31`
- Total rows: `241,680`
- Markets: `37`
- Market/category pairs: `38`
- Raw missing prices: `44,821`
- Missing values after filling: `0`

## CSV Columns

The final CSV contains these columns:

- `date` - Daily date of the observation.
- `year` - Year extracted from the date.
- `month` - Month extracted from the date.
- `day` - Day extracted from the date.
- `market` - Egg market or zone name.
- `category` - Price category, such as `NECC` or `PREVAILING`.
- `price` - Original scraped price from the website. Missing website values are kept as `NA`.
- `price_filled` - Analysis-ready price after filling missing values.
- `fill_method` - Method used to produce `price_filled`.

## Data Preparation Steps

The data preparation pipeline performs the following steps:

1. Downloads NECC monthly reports from the website.
2. Saves raw monthly HTML files into `raw_html/`.
3. Parses daily prices from each monthly table.
4. Converts the website's month-style report into daily records.
5. Builds a complete daily grid for every observed `date + market + category` pair.
6. Keeps missing website values as `NA` in the original `price` column.
7. Adds a filled time-series column named `price_filled`.
8. Adds a `fill_method` column so filled values remain traceable.

## Missing Value Filling Strategy

The original `price` column is never overwritten. Missing values are filled into a separate column named `price_filled`.

The filling strategy is:

1. `observed`  
   The value came directly from the website.

2. `linear_interpolation`  
   Used only for short internal gaps of up to 15 consecutive missing days.

3. `previous_price`  
   Used for missing values where the previous known price is the most practical estimate.

4. `next_price`  
   Used for leading missing values before the first available price for a market/category pair.

This keeps the dataset usable for analysis while preserving transparency about which values are real and which values are estimated.

Current fill-method counts:

```text
observed                196,859
next_price               33,529
previous_price           11,042
linear_interpolation        250
```

## Running the Scraper

Install the required Python packages:

```bash
pip install pandas requests beautifulsoup4
```

Run the scraper:

```bash
python3 scraper.py
```

Run for a custom date range:

```bash
python3 scraper.py --start-date 2024-01-01 --end-date 2024-12-31
```

Force fresh downloads instead of using cached HTML:

```bash
python3 scraper.py --no-cache
```

Change the interpolation limit:

```bash
python3 scraper.py --interpolation-limit-days 7
```

## Analysis Notebook

`Egg_data_analysis.ipynb` is intended for analysis after data preparation. Typical notebook work includes:

- Loading `necc_egg_prices_daily.csv`
- Checking missing values by market and category
- Comparing `price` vs `price_filled`
- Plotting price trends over time
- Studying market-wise price movement
- Detecting seasonal patterns
- Preparing the dataset for forecasting or machine learning

For analysis, use `price` when you need only original website values, and use `price_filled` when you need a continuous daily time series.
