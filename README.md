# woorifisa_final_ai
Seoul Commercial District Analysis & Business Type Recommendation

Final project of the Woori FISA AI Engineering Bootcamp (Team 3). A web service that analyzes Seoul's commercial districts and recommends the top 5 business types for a selected neighborhood, based on predicted next-quarter sales.

This repository contains my part of the project: data preprocessing and model implementation.

Problem

Someone planning to open a small business (for example, a retiree using a severance payout) has little accessible data on which business types are likely to do well in a given neighborhood. This project uses public commercial district data to give a data-driven recommendation per administrative dong.

Data
Source: Seoul Open Data Plaza (data.seoul.go.kr), public data
9 datasets: commercial district boundaries, resident population, floating population, working population, estimated sales, attraction facilities, apartments, stores, income/consumption
Period: 2017–2022 (6 years, quarterly)
About 500 columns in total, joined on shared keys (year, quarter, district type code, district code)
Data Processing (my role)
Merged 6 years of data for each of the 9 datasets, then joined them with the district boundary data to attach an administrative-dong code to every record.
Ran correlation analysis against quarterly sales to select meaningful features (e.g., sales count by age 40s, sales count by time of day, Saturday sales count, floating population by age and time).
Combined everything into a single modeling dataset: 25 columns, about 390,000 rows.
Handled missing values case by case:
Filled with the mean of the same service-type category when missing values were few relative to the data.
Dropped rows when an entire administrative dong had no data, since imputation would be meaningless.
Normalized features with MinMaxScaler.
Modeling (my role)

Target: quarterly sales amount. 17 independent variables.

Model	R²
Linear Regression	0.42
Ridge	0.40
Lasso	0.40
ElasticNet	0.40
Random Forest	[verify]
XGBoost	[verify]
XGBoost + SHAP feature selection (top 11 features)	0.939
Evaluated tree models with 5-fold cross-validation (cross_val_predict).
Used SHAP values to rank feature importance; keeping the top 11 features gave the best performance.
Troubleshooting: from regression to time series

The XGBoost model predicted the next quarter's sales from each quarter separately, so the top-5 recommendation returned the same business type multiple times. I restructured the data into a time series (built a period datetime column from year and quarter codes) and used Facebook Prophet to forecast next-quarter sales per business type, which produced a clean top-5 list.

Example: for retail in Dunchon 2-dong, Gangdong-gu, the recommended types were sporting goods, pharmacy, furniture, convenience store, and auto parts.

Limitations & What I'd Do Differently
Some features (e.g., sales counts) are closely tied to the target (sales amount), which may inflate R² through target leakage. I would remove sales-derived features and re-evaluate.
I would use time-based train/test splits instead of random cross-validation, since the task is forecasting future quarters.
Service (team)

The model was served in a Flask web app with a MapBox map: users pick a district and dong, view a commercial district report (environment, sales, population), and get business type recommendations in a popup. Deployed on AWS (EC2, RDS) with Docker, Nginx, and Redis caching, which cut report loading time from over 20 seconds.

Files
회귀모델링_HJ_통합본.ipynb — preprocessing, regression models, SHAP feature selection
🔨0911_HJ_시계열&XGBOOST로_예측👷‍♀️.ipynb — time series forecasting (Prophet) and XGBoost prediction
Tech Stack

Python, pandas, scikit-learn, XGBoost, SHAP, Prophet, Matplotlib, Seaborn
