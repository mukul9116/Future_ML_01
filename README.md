# Sales & Demand Forecasting - Store Item Demand Forecasting

## Problem Statement
Predict the daily sales of different items from various stores based on historical sales data.

## Data
Raw data is not included in this repo (see .gitignore).
Dataset-link: https://www.kaggle.com/competitions/store-sales-time-series-forecasting
To reproduce, download the CSVs and place them in Google Drive at FUTURE_ML_01/data/, or locally in data/ if running outside Colab.
## Dataset
- **Source:** Kaggle - [Corporación Favorita Grocery Sales Forecasting]
- **Files:** train.csv, test.csv, sample_submission.csv, oil.csv, transactions.csv, stores.csv, holiday_events.csv
- **Size:** ~913K rows for training and ~450K rows for testing
- **Date Range:** 2013-2017
- **Stores:** 10
- **Items:** 50

## Scope
Investigate the dataset, study its structure, clean and analyze the data, create meaningful features, and develop forecasting models to predict future daily sales.

## Approach (Outline)
Loading Data -> Exploring Data -> Exploratory Data Analysis -> Feature Creation -> Modeling -> Evaluation -> Visualization

## Progress (Day 1 of 10)
- Created project repository
- Downloaded dataset from Kaggle
- Set up environment
- Loaded dataset
- Investigated dataset

## Day 2 Findings (EDA)
- No missing values in train/test/stores/holidays/transactions
- oil.csv file has 43 dcoilwtico missing values - structural missing (market closed during weekends and market holidays) and not random missing
- All 54 stores are there from the first day (2013-01-01) of the dataset
- Zero-sales pattern (present in 31% of train records) is NOT random; it is because of the product family not yet launched in the specific store (product families like "BOOKS/BABY CARE" started selling in some stores after few years of dataset)

## Day 3 Results (Time Series Analysis)
- Total sales clearly have an upward trend between 2013-2017
- Seasonality is evident; sales on weekends are higher compared to weekday sales
- ADF test on sales data for months: p-value = 0.26707 (>0.05), therefore the series is non-stationary
- Conclusion: trend and seasonality exist, and the trend is non-stationary, so we use SARIMA models instead of ARIMA.

## Findings of Day 4 (Feature Engineering)
- Created lag features (sales_lag_1, sales_lag_7) by using store_nbr+family to prevent any contamination across products
- Created rolling_mean_7 but shifted first because we don't want today's information to be part of the feature itself as it can cause to data-leakage 
- Created date features: month, year, and is_weekend
- Created is_launched / days_since_launched, is_nan for never launched product-store combinations will require a filling of missing values (probably 0) prior to modeling.
- lesson learned: Rerunning notebook cells out of sequence on existing data causes silent duplicate column problems - always restart from scratch before final run

## Day 5 Results (Train/Test split and Baselines)
- Created chronological train/validation split (last 3 months as validation set) - absolutely necessary to prevent leakage, since shuffling will allow the model to see the future
- Verified that test.csv (Kaggle's file) does not have the sales column - cannot use this data for evaluation, and will split the train.csv file instead
- Cannot use MAPE because of the ~31% zero-sales records that cause division by zero
- Simple baseline (yesterday=today): MAE = 128.1, RMSE = 469.3
- Moving average baseline (rolling_mean_7): MAE = 109.0, RMSE = 395.3 - outperforms simple baseline in both cases
- Significant difference between RMSE and MAE in simple baseline indicates occasional large errors (probably caused by weekday-to-weekend transition), not small consistent errors
- These are the baselines that we need to build the model.

## Day 6 Results (Choosing best and comparing Models for Prediction)
- Created dummy variables for family using pd.get_dummies (with drop_first=True); forced alignment of test set columns by using reindex to solve the problem with train/test column mismatch
- Linear Regression: test MAE = 94.76, RMSE = 306.23, R2 = 0.9469; gave ~1M non-sensical negative predictions on test set (min = -35.9)
- Random Forest (n_estimators = 50, max_depth = 10): test MAE = 74.99, RMSE = 273.90, R2 = 0.9575; made no negative predictions, tree-based models cannot extrapolate outside range of values observed
- Both models outperform Day 5 results (baseline naive MAE = 128.1, baseline moving avg MAE = 109.0)
- Random Forest is our best model for now.

## Day 7 Findings (SARIMA)
- Fit SARIMA order(1,1,1)xseasonal_order(1,1,1,7) on Store 1/BEVERAGES (chosen for low zero-rate, high volume - fair test for seasonal modeling)
- SARIMA test: MAE 232.73, RMSE 313.79, R2 0.6655
- Re-evaluated Random Forest on the SAME series alone (not all combos): MAE 361.51, RMSE 485.03, R2 0.1932
- SARIMA outperforms RF on this specific high-volume, strongly-seasonal series
- Key insight: RF's earlier "better" overall numbers were an average across 1,782 combos of very different scale/sparsity - not a fair comparison to a single-series SARIMA model
- Practical implication: model choice should depend on series characteristics - SARIMA/per-series models for high-volume consistent sellers, global ML models for the sparse long tail

## Model Selection Reasoning: Random Forest and SARIMA

In this project, two distinct approaches were tested:

- **Random Forest** – one global model trained for all 1,782 store-family
  combinations simultaneously. Easy to train/maintain on a large scale, yet can
  only capture seasonal trends indirectly (with features such as is_weekend),
  and average error metrics on a highly heterogeneous dataset of high-volume
  and low-volume items.

- **SARIMA** – fitted on each individual series explicitly modeling the
  trend and seasonality. In one specific example of a high-volume, consistently
  selling series (Store 1/BEVERAGES), SARIMA clearly beat Random Forest
  (MAE 232.73 vs. 361.51) on the same series. However, SARIMA cannot be used
  directly on this dataset of 1,782 combinations without fitting a new model
  for each series, which requires substantially more effort than one global
  model.

**Conclusion:** there is no "best" model here – it depends on the series.
For the purposes of this project, Random Forest was chosen as the global model,
(adding robustness with sklearn Pipeline and OneHotEncoder,
cross-validation with TimeSeriesSplit). In the real-world scenario,
a hybrid solution is preferred: dedicated per-series models (SARIMA or alike)
for flagship high-volume items when the additional accuracy is worth it,
and global

## Day 8 Results (Analysis & Tuning)
- Switched from pd.get_dummies & manual reindexing to an appropriate sklearn Pipeline (ColumnTransformer + OneHotEncoder(handle_unknown='ignore'))
- Updated pipeline: test MAE 74.96, RMSE 273.46, R2 0.9577 – basically the same as Day 6 manual approach, which proves that the change has improved robustness without reducing predictability
- Performed TimeSeriesSplit (5 folds) on a 200K sample: mean MAE 68.03, standard deviation 11.38
- The MAE increased in later folds – due to the overall increase in sales, rather than deterioration of the model's ability
- Model selection: Random Forest through Pipeline, due to robustness and decent cross-validation results