# Pakistan-inflation-forecasting
The purpose of creating this is to get to know about the current situation of Pakistan, and how people are affected by it and how much it have increased (inflation) by a decade. Will be Forecasting Pakistan monthly CPI inflation using econometrics and machine learning models.

Forecasting Pakistan's monthly CPI inflation (year-on-year) one month ahead, using econometric and machine learning models.

Author: Usman | Data through: Aug 2026 (Sep 2026 CPI added from monthly data)

# Why this project

Pakistan's inflation went from above 35% in 2023 to close to zero in 2025, and is back around 10% in 2026. Swings like this make forecasting hard and important. I wanted to know: can a model beat the simplest forecast ("next month = this month"), and do the policy rate, the rupee and oil prices actually help?

# Headline results
A simple model that uses the base effect cuts the average forecast error by roughly 35-40% compared with "same as today".
The best model, SARIMAX with oil, rupee and policy-rate inputs, has the lowest error in the two most recent test windows. The gain over the simple version is modest and is not statistically tested.
Models that predicted year-on-year inflation directly (regularised regression, random forest) did not reliably beat the baseline at 1, 3 or 6 months ahead.

Average miss (MAE, percentage points) on the last N months, one month ahead:

# Model	Last 12 mo	Last 24 mo	Last 36 mo
Baseline: same as today	1.18	1.23	1.74
Ridge, predicting YoY directly (all features)	1.15	1.09	1.87
Base effect + 12-month average monthly price change	0.80	0.80	1.02
Base effect + Ridge (monthly history only)	0.82	0.91	1.07
Base effect + Ridge (+ oil, rupee, policy)	0.73	0.75	1.02
Base effect + SARIMA (monthly history only)	0.83	0.89	1.03
Base effect + SARIMAX (+ oil, rupee, policy)	0.69	0.73	1.02

A random forest scored 1.49 over the last 24 months, worse than the baseline (1.23).

# The key idea: the base effect

Year-on-year inflation changes each month because a new month replaces the same month from last year. The old month is already known, so only the new month's price change has to be forecast:

YoY(next) = (1 + YoY(now)) x (1 + MoM(new)) / (1 + MoM(same month last year)) - 1

Example from the data: YoY inflation was 11.1% in June 2026, 9.2% in July and 11.1% in August. July 2025 had a +2.9% monthly jump that dropped out of the window, and August 2025 had a -0.6% fall that dropped out. No shock was needed to explain the swings.

Checking the formula: if the next month's price change is known exactly, the formula reproduces the next YoY with an average error of about 0.04 points (rounding only). All remaining forecast error comes from guessing the new month's price change.

# Data
Series	Source	Frequency
CPI National, YoY and MoM (base 2015-16)	State Bank of Pakistan, EasyData	Monthly
SBP policy (target) rate	State Bank of Pakistan, EasyData	Changes at policy meetings, filled forward to monthly
PKR/USD exchange rate	State Bank of Pakistan, EasyData	Monthly
Brent crude oil price (USD per barrel)	FRED (series POILBREUSDM)	Monthly

Sample: Jul 2016 to Aug 2026 (122 months, about 110 usable after building lags). Brent is built into an oil-price-in-rupees feature (Brent x PKR/USD).

# Method
Clean and merge the four sources into one monthly table (data/processed/monthly.csv).
Features (data/processed/features.csv): inflation lags, monthly changes in the rupee and in oil priced in rupees, and the change in the policy rate. Every feature uses only information available at the forecast date.
Walk-forward testing: for each test month, models train only on earlier months and forecast the next one. For longer horizons the training set excludes outcomes not yet known at the forecast date.
Baseline: next month's inflation equals this month's.
Direct models: Ridge regression and random forest predicting YoY inflation at 1, 3 and 6 months ahead.
Base-effect models: forecast next month's monthly price change (12-month average, Ridge, SARIMA, SARIMAX with lagged oil, rupee and policy-rate inputs), then convert it to YoY with the formula above.
# Live forecast (pre-registered)
	
Latest YoY used	Sep 2026: about 10.2% (derived from the monthly data, because the SBP download was unavailable)
Method	Base effect + 12-month average monthly price change
Forecast for Oct 2026	9.3% (typical miss about 0.8 points)
Actual (SBP release)	to be added after the release
Limitations
Small sample: about 10 years of monthly data, with a large regime change in 2023-2025.
The 12, 24 and 36-month test windows overlap, so they are one result viewed three ways, not independent confirmations.
Differences between the best base-effect models are small and are not tested for statistical significance.
The policy rate responds to inflation, so its predictive value is not evidence that it causes inflation.
Monthly price changes are published to one decimal place, which adds rounding noise.
The Brent series is published with a lag, so its latest month is often missing.

# Repository structure
data/raw/          original downloads (SBP EasyData, FRED)
data/processed/    monthly.csv (merged), features.csv (model inputs)
notebooks/         01_data_check.ipynb (cleaning, features, models, tests)
src/               reserved for reusable code

# Reproduce
what i did is pip install pandas numpy matplotlib scikit-learn statsmodels
At the end i would say that i  Open notebooks/01_data_check.ipynb in VS Code or Jupyter. The cells follow the order of the analysis: cleaning and merging, exploration, features and baseline, direct models, horizon test, base-effect model, SARIMA/SARIMAX.
