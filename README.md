# Bakery Sales Forecasting — Time Series Model Comparison
## Project Overview

This project compares multiple time series forecasting models to predict daily bakery item sales. The dataset contains daily sales by product (unique_id) and includes the unit price of each item, enabling experiments with both univariate forecasting and exogenous-regressor (X) forecasting.

The primary goal is to identify which forecasting approach performs best for short-term demand planning, across multiple products.

## Data and Processing

The notebook uses a bakery sales dataset (daily_sales_french_bakery.csv) structured in a multi-time-series format compatible with StatsForecast:

- unique_id: product identifier (e.g., BAGUETTE, CROISSANT, etc.)

- ds: date (parsed as a datetime)

- y: daily sales quantity (target)

- unit_price: item price (used as an exogenous feature in certain models)

Key processing steps in the notebook include:

1. Date parsing

- Loaded the CSV using parse_dates=['ds'] to ensure proper time indexing.

2. Filtering for sufficient history

- Removed products with fewer than 28 observations.

3. Two modeling configurations

- Univariate forecasting (dropping unit_price)

- Exogenous forecasting using unit_price and engineered calendar features

4. Exploratory plotting

- Visualized sales trends for key products using utilsforecast.plotting.plot_series.

## Modelling Approach
### Forecast Horizon

All models forecast a 7-day horizon:

- horizon = 7

- Frequency: daily (freq="D")

- Weekly seasonality assumed in seasonal models (season_length = 7)

### Models Evaluated

The project compares:

1) Baseline Models (Univariate)

- Naive

- HistoricAverage

- WindowAverage (7-day)

- SeasonalNaive (weekly seasonality)

These serve as performance baselines for quick benchmarking.

2) ARIMA Family (AutoARIMA)

- ARIMA (non-seasonal)

- SARIMA (seasonal with weekly seasonality)

AutoARIMA was used to automatically identify model orders.

3) SARIMA with Exogenous Variables

Two exogenous approaches were explored:

- Price as exogenous (unit_price)

- Engineered time features

Generated using utilsforecast.feature_engineering.pipeline, including:

- Fourier terms (weekly seasonality)

- Calendar features (e.g., day/week/month)

## Evaluation
Holdout Test (Last-7-Days)

A simple, interpretable evaluation split was used:

Test set: last 7 days per product

Train set: remaining history

Performance was measured primarily using MAE (Mean Absolute Error), and averaged across series for summary reporting.

## Rolling Cross-Validation

To evaluate stability over time, rolling-origin CV was also used:

n_windows = 8

step_size = horizon (7 days)

refit = True

This produces multiple backtests and more reliable performance estimates than a single split.

### Metrics

Dinal evaluation includes:

- MAE, MSE, RMSE

- MAPE, sMAPE

- MASE (seasonality = 7)

- scaled CRPS (probabilistic accuracy)

## Results

1) Baseline Models (Holdout MAE, averaged)

| Metric | Naive | HistoricAverage | WindowAverage | SeasonalNaive |
| ------ | ----: | --------------: | ------------: | ------------: |
| MAE    | 6.108 |           5.228 |         5.012 |         4.614 |


2) Baselines vs ARIMA Models (Holdout MAE on selected products)
   
| Metric | ARIMA | SARIMA |  Naive | HistoricAverage | WindowAverage | SeasonalNaive |
| ------ | ----: | -----: | -----: | --------------: | ------------: | ------------: |
| MAE    | 7.430 |  5.926 | 10.318 |           8.396 |         7.997 |         8.096 |


3) Rolling Cross-Validation (MAE, averaged)
   
| Metric | SeasonalNaive | WindowAverage |  ARIMA | SARIMA |
| ------ | ------------: | ------------: | -----: | -----: |
| MAE    |        12.751 |        15.072 | 12.549 | 11.575 |


4) Exogenous Price Model vs SeasonalNaive (Final Metrics
   
| Metric      | SARIMA_exog_price | SeasonalNaive |
| ----------- | ----------------: | ------------: |
| MAE         |            11.572 |        12.751 |
| RMSE        |            14.928 |        16.692 |
| MSE         |           409.581 |       502.095 |
| MAPE        |             0.603 |         0.615 |
| sMAPE       |             0.248 |         0.298 |
| MASE        |             1.186 |         1.325 |
| Scaled CRPS |             0.199 |         0.229 |


Takeaway: Adding unit price as an exogenous regressor produced the best overall performance, improving both point accuracy (MAE/RMSE) and probabilistic accuracy (scaled CRPS) versus the seasonal baseline.

## Conclusion

- Weekly seasonality is a key pattern in bakery sales, and models that capture it (e.g., SeasonalNaive, SARIMA) outperform non-seasonal methods.

- SARIMA consistently achieved the best results among univariate models in both holdout testing and rolling cross-validation.

- Incorporating unit price as an exogenous feature further improved accuracy, making SARIMA + price exogenous the strongest approach tested.

This model comparison provides a practical framework for short-term demand forecasting in retail/food contexts, supporting decisions such as inventory planning, production scheduling, and pricing strategy evaluation.
