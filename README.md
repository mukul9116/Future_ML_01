# Sales & Demand Forecasting — Store Item Demand Forecasting

## Problem Statement
Predict the daily sales of different items from various stores based on historical sales data.

## Dataset
- **Source:** Kaggle - Store Item Demand Forecasting Challenge
- **Files:** train.csv, test.csv, sample_submission.csv
- **Size:** ~913K rows for training and ~450K rows for testing
- **Date Range:** 2013-2017
- **Stores:** 10
- **Items:** 50

## Scope
Investigate the dataset, study its structure, clean and analyze the data, create meaningful features, and develop forecasting models to predict future daily sales.

## Approach (Outline)
Loading Data -> Exploring Data -> Exploratory Data Analysis -> Feature Creation -> Modeling -> Evaluation -> Visualization

## Progress (Day 1 of 30)
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