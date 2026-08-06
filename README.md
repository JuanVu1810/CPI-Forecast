# CPI Forecast

This repository contains a university time-series forecasting project for Australian Consumer Price Index (CPI). The main notebook, `cpi_forecast_V1.ipynb`, builds and evaluates a quarterly CPI forecasting model using historical CPI observations from 1995 Q1 to 2022 Q4.

## Project Objective

The assignment goal was to forecast Australian CPI for the next 8 quarters and evaluate how well a time-series model can capture CPI trend and seasonality. CPI is an important inflation indicator, so the project frames the forecast as useful for economic planning, policy analysis, budgeting, and business decision-making.

## What `cpi_forecast_V1.ipynb` Does

The notebook follows a complete forecasting workflow:

1. Loads `CPI_train.csv`, which contains quarterly CPI observations.
2. Cleans the data by checking data types, missing values, and outliers.
3. Converts the `Quarter` column into a quarterly time-series index.
4. Performs exploratory data analysis with time-series plots, boxplots, and seasonal decomposition.
5. Tests stationarity using ACF, PACF, and Augmented Dickey-Fuller tests.
6. Applies first-order differencing to remove trend.
7. Applies seasonal differencing with lag 4 to handle quarterly seasonality.
8. Uses `pmdarima.auto_arima` to search for a suitable SARIMA model.
9. Compares the SARIMA model against a seasonal random walk benchmark using rolling-window validation.
10. Fits the final model and evaluates out-of-sample forecast accuracy.
11. Produces forecast outputs and confidence intervals.

The selected model in the notebook is:

```text
SARIMA(0, 1, 1)(0, 1, 1)[4]
```

This model was chosen because the CPI series has a clear upward trend and a repeating quarterly seasonal pattern. The notebook shows that first differencing removes the trend, while seasonal differencing at lag 4 handles the yearly seasonal cycle.

## Main Findings

The notebook found that SARIMA slightly improved on a simple seasonal random walk benchmark during rolling validation. On the 8-quarter test period, the final SARIMA model achieved a test MSE of about `46.25` and an RMSE of about `6.8` CPI points.

Residual diagnostics in the notebook suggest that the final model residuals are reasonably well behaved: there is no strong remaining autocorrelation, the residuals are approximately normal, and no obvious trend remains in the residual series.

## Current Update: `data_retrieval.py`

The project has been updated with a separate data collection script, `data_retrieval.py`. This script is not the original modelling notebook; it is a reproducible data pipeline for downloading additional Australian macroeconomic and market indicators that could support future versions of the CPI forecast.

The script can download data from:

- ABS through `readabs`
- RBA tables through `readabs`
- Market data through `yfinance`

The current script retrieves and saves:

- CPI index
- unemployment rate
- wage price index
- producer price index
- household spending
- RBA cash rate
- AUD/USD exchange rate
- inflation expectations
- commodity price indexes
- WTI crude oil futures
- Brent crude oil futures

Downloaded data is saved under `dataset/`, separated into `abs/`, `rba/`, and `market/` folders. A `download_manifest.json` file is also created to record the download time, package versions, selected year range, output files, row counts, and column names.

This update makes the project easier to extend from a univariate SARIMA model into a future multivariate forecasting project, where CPI could be modelled together with labour market, interest rate, exchange rate, commodity, and oil price indicators.

## Why Add More Variables?

The first version of the project uses only historical CPI values. This is useful for capturing trend and seasonality, but it limits the model because CPI is affected by broader economic conditions. During unusual periods, such as the post-COVID inflation surge, a univariate SARIMA model may underperform because it cannot observe external shocks or policy changes.

The additional variables are included because they represent possible drivers of inflation:

- **Unemployment rate:** captures labour market tightness. Lower unemployment can increase wage pressure and demand.
- **Wage Price Index:** measures wage growth, which can affect business costs and household spending.
- **Producer Price Index:** captures upstream price pressure before it reaches consumers.
- **Household spending:** measures demand-side pressure in the economy.
- **RBA cash rate:** represents monetary policy, which can influence inflation with a delay.
- **AUD/USD exchange rate:** affects import prices and imported inflation.
- **Inflation expectations:** captures forward-looking views about future inflation.
- **Commodity prices and oil prices:** capture energy, fuel, transport, and global supply-cost pressure.

Adding these variables should help the next version of the project move beyond "CPI depends only on past CPI" toward a more realistic economic forecasting model.

## Next Development Plan

The next stage of the project will extend `cpi_forecast_V1.ipynb` into a second modelling version. The planned improvements are:

1. **Build a merged modelling dataset**

   Combine CPI with the downloaded ABS, RBA, and market variables. Since the data comes at different frequencies, the monthly and daily variables will need to be converted to quarterly frequency before modelling.

2. **Create lagged economic features**

   Many economic variables affect inflation with a delay. For example, interest rate changes, wage growth, exchange rate movements, and oil price shocks may influence CPI one or more quarters later. The next version will create lagged features such as 1-quarter, 2-quarter, and 4-quarter lags.

3. **Compare SARIMA with SARIMAX**

   The current model is SARIMA, which only uses past CPI. The next model will test SARIMAX, which allows external variables. This will help evaluate whether macroeconomic indicators improve CPI forecast accuracy.

4. **Add stronger benchmark models**

   The project will compare SARIMAX against simpler benchmarks such as seasonal naive forecasting and possibly ETS/exponential smoothing. This makes the evaluation stronger because the final model must prove that it improves on simpler alternatives.

5. **Use rolling-window validation**

   Time-series models should be evaluated in chronological order. The next version will continue using rolling-window validation so the model is tested in a realistic forecasting setting.

6. **Interpret which variables matter**

   The final model should not only forecast CPI, but also explain which indicators appear useful. This will make the project more attractive for a CV because it connects modelling results to economic reasoning.

The goal of the next version is to turn the project from a univariate CPI forecasting assignment into a more complete inflation forecasting pipeline using real external economic indicators.

## Repository Structure

```text
.
├── cpi_forecast_V1.ipynb       # Main university forecasting notebook
├── CPI_train.csv               # Original CPI training data
├── CPI_forecast.csv            # Forecast output from the notebook
├── data_retrieval.py           # Updated data download pipeline
├── requirements-data.txt       # Packages needed for data retrieval
├── dataset/                    # Downloaded ABS, RBA, and market datasets
└── README.md                   # Project summary and development plan
```

## Running the Data Retrieval Script

Install the data retrieval dependencies:

```bash
python -m pip install -r requirements-data.txt
```

Run the script for an inclusive year range:

```bash
python data_retrieval.py 1995 2025
```

Or choose a custom output folder:

```bash
python data_retrieval.py 1995 2025 --output-dir dataset
```

## Notes

The notebook is the main submitted university project. The newer `data_retrieval.py` script is an update that improves reproducibility and prepares the repository for future model extensions using external economic indicators. The next modelling step is expected to be a new notebook or script that merges these datasets and tests whether external variables improve forecast performance.
