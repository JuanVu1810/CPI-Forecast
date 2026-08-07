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

## Planned Exploratory Data Analysis

Before fitting a multivariate forecasting model, the next version should include a leakage-aware EDA process for the CPI target and all external indicators. This is important because the downloaded datasets have different frequencies, different start dates, and different real-world publication timings.

The planned EDA steps are:

1. **Validate data integrity and time alignment**

   Check that each dataset has a valid date column, no duplicate timestamps, a monotonically increasing time index, and the expected frequency. CPI is quarterly, while unemployment, cash rate, commodity prices, exchange rates, and household spending are monthly, and oil prices are daily. These series need to be resampled to a common quarterly frequency before modelling.

2. **Audit missing values and usable history**

   Summarise the start date, end date, row count, missing values, and frequency of each variable. Some indicators do not cover the full CPI history: for example, household spending starts later than CPI, WTI and Brent oil prices start later than 1995, and some inflation expectation series contain many missing values. This audit will help decide whether to build one long-history model or several shorter-sample models.

3. **Avoid look-ahead bias**

   Missing values and frequency conversion must be handled without using future information. Back-filling should be avoided because it can leak future values into earlier quarters. Forward-filling or interpolation should only be used when it is economically reasonable and clearly documented.

4. **Inspect the CPI target**

   Plot the CPI index, quarterly CPI growth, and year-ended CPI growth. The EDA should check trend, seasonality, volatility, outliers, and structural breaks, especially around the Global Financial Crisis, COVID period, post-COVID inflation surge, and rapid RBA cash rate increases.

5. **Inspect each external variable**

   Plot each predictor over time and review its scale, distribution, outliers, and economic interpretation. Index variables such as wages, producer prices, commodity prices, and CPI may need differencing or percentage-change transformations. Rate variables such as unemployment, cash rate, and inflation expectations may be useful in levels or changes.

6. **Test stationarity and choose transformations**

   Apply Augmented Dickey-Fuller (ADF) and KPSS tests to CPI and candidate predictors. These tests should guide whether each series is modelled in levels, first differences, seasonal differences, percentage changes, or log changes.

7. **Explore lead-lag relationships**

   Use cross-correlation analysis to test whether external variables lead CPI inflation. Candidate lags should include 1-quarter, 2-quarter, and 4-quarter lags. This is especially important for variables such as cash rate, wage growth, producer prices, exchange rates, commodity prices, and oil prices, which may affect inflation with a delay.

8. **Test predictive usefulness**

   Use Granger causality tests to check whether lagged external variables add information beyond CPI's own past values. These tests should be treated as screening tools rather than final proof, but they can help justify which predictors should enter a SARIMAX model.

9. **Check relationship stability**

   Use rolling correlations to see whether relationships between CPI and candidate predictors are stable through time or only strong during unusual periods. Variables whose relationships reverse or disappear may be less reliable for forecasting.

10. **Check multicollinearity**

    Build a predictor correlation matrix and calculate variance inflation factors (VIFs) for candidate features. This is needed because commodity prices, oil prices, producer prices, and exchange rates may carry overlapping information.

11. **Audit feature availability**

    For each candidate predictor, document whether the value would actually be known at the forecast origin. Many macroeconomic indicators are published with a delay, and future values of external variables are unknown for an 8-quarter forecast unless they are separately forecast. The final SARIMAX setup should therefore distinguish between lagged historical features that are available at forecast time and future exogenous paths that would need their own assumptions or forecasts.

The EDA should finish with a clean quarterly modelling dataset, a variable coverage table, transformation decisions, candidate lag choices, multicollinearity diagnostics, and a justified shortlist of external predictors for SARIMAX.

## Planned Interactive Interface

The project should use **Streamlit** as the first interactive user interface. Streamlit is a good fit because this is primarily a data science and forecasting project where users need to explore datasets, view EDA charts, choose model settings, and inspect forecast outputs.

A future Streamlit dashboard could include:

1. **Overview**

   Summarise the project goal, CPI forecasting objective, model choices, and key findings.

2. **Data Explorer**

   Display the available CPI, ABS, RBA, and market datasets, including date ranges, frequencies, row counts, missing values, and column descriptions.

3. **EDA Dashboard**

   Show CPI trends, quarterly and year-ended inflation, seasonal patterns, external indicator plots, correlation heatmaps, lag-correlation results, and stationarity test summaries.

4. **Forecasting Interface**

   Allow users to select a model type, forecast horizon, training window, and candidate external variables. The interface should display forecasts with confidence intervals and make it easy to compare SARIMA and SARIMAX outputs.

5. **Model Evaluation**

   Present RMSE, MSE, MAE, benchmark comparisons, rolling-window validation results, and residual diagnostics.

FastAPI is not necessary for the first version of the interface because it does not provide a visual dashboard by itself. It would become useful later if the project needs a model-serving backend, such as an endpoint that returns CPI forecasts to another application.

If a more advanced multivariate forecasting framework is applied later, FastAPI should be considered as an optional deployment layer rather than a replacement for Streamlit. Streamlit would remain useful for exploration, EDA, model comparison, and portfolio demonstration. FastAPI would be useful if the trained model needs to be exposed through endpoints such as:

```text
POST /forecast
GET /model-metrics
GET /available-features
```

In that setup, Streamlit could act as the user-facing dashboard while FastAPI serves model predictions in the background.

A possible future structure is:

```text
.
├── app.py                     # Streamlit dashboard
├── src/
│   ├── data_processing.py      # Data loading, cleaning, merging, resampling
│   ├── eda.py                  # EDA summaries and plotting helpers
│   ├── modelling.py            # SARIMA, SARIMAX, and benchmark models
│   └── forecasting.py          # Forecast generation and evaluation helpers
├── dataset/
├── README.md
└── requirements.txt
```

The recommended development path is to build the Streamlit dashboard first, then add FastAPI only if the forecasting model needs to be served through an API.

## Advanced Forecasting Ideas From Recent Literature

Recent multivariate time-series forecasting research highlights three ideas that are relevant to a future version of this CPI project: multiscale temporal modelling, external data augmentation, and careful evaluation of how external variables improve forecasts.

Peng et al. (2025) propose MSP-EDA, a multivariate forecasting framework that combines multiscale patch representations with external data enhancement. Their model uses Fourier-based analysis to capture dominant global periodic patterns, wavelet-based analysis to capture local time-frequency variation, and attention mechanisms to learn temporal dependencies, cross-variable relationships, and the influence of external data.

The full MSP-EDA deep learning architecture is probably too complex for the current CPI dataset because the project has a relatively small number of quarterly observations. However, several ideas from the framework can still strengthen this project:

1. **Add multiscale CPI analysis**

   Analyse CPI at several time scales instead of only modelling the quarterly index level. Useful views include quarter-to-quarter inflation, year-ended inflation, rolling 2-year averages, rolling 4-year averages, and seasonal quarterly patterns.

2. **Add frequency-domain diagnostics**

   Use Fourier or periodogram analysis to check dominant CPI cycles and confirm whether quarterly seasonality is strong. Wavelet analysis can be treated as an optional advanced EDA extension for detecting local changes in inflation behaviour during periods such as COVID or the post-COVID inflation surge.

3. **Create an external-data quality score**

   Before modelling, score each external variable based on coverage, missingness, frequency alignment, publication delay, and economic relevance. This makes the choice of SARIMAX predictors more transparent.

4. **Evaluate forecasts by horizon**

   Since the project forecasts 8 quarters ahead, model accuracy should be reported separately for each forecast horizon, not only as one overall RMSE. This can show whether external variables help short-term CPI forecasts, longer-term CPI forecasts, or both.

5. **Run ablation studies by variable group**

   Compare the baseline SARIMA model against several SARIMAX variants to test which groups of external variables improve forecast accuracy. Candidate groups include labour market variables, price-pressure variables, monetary and exchange-rate variables, commodity and oil variables, and all selected variables combined.

6. **Analyse cross-variable relationships**

   Translate the paper's attention-based variable-relationship idea into interpretable diagnostics suitable for this project, such as lag correlations, Granger causality tests, predictor correlation matrices, VIF scores, and model coefficient interpretation.

These additions would make the next version more research-informed while keeping the modelling approach realistic for the available data size.

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

## References

Peng, S., Sun, W., Chen, P., Xu, H., Ma, D., Chen, M., Wang, Y., & Li, H. (2025). MSP-EDA: Multivariate time series forecasting based on multiscale patches and external data augmentation. *Electronics, 14*(13), 2618. https://doi.org/10.3390/electronics14132618
