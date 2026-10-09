# Framingham Heart Disease: Statistical Inference

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![Statsmodels](https://img.shields.io/badge/Statsmodels-Logit-orange.svg)
![Seaborn](https://img.shields.io/badge/Seaborn-Data%20Viz-green.svg)
![Data Source](https://img.shields.io/badge/Data%20Source-Kaggle-yellow.svg)

The primary objective of this project is **not predictive classification or machine learning scoring**, but rather **statistical inference and coefficient interpretation**, by leveraging multiple logistic regression and clinical scaling.  This study isolates and quantifies how specific demographic, behavioral, and clinical risk factors influence the odds of heart disease over a 10-year horizon.

## Dataset Description

The dataset includes over 4,000 patient records with 15 attributes categorized into demographic, behavioral, and medical risk factors.
The target variable is `TenYearCHD` (binary: 1 = 10-year risk of coronary heart disease, 0 = No risk).

## Data Preprocessing & Cleaning

* **Missing Values:** Rows containing missing values (582 rows, representing 14% of the dataset) were dropped to ensure a clean, stable sample of 3,656 observations for statistical modeling.
* **Duplicates:** Verified zero duplicate rows in the dataset.

## Collinearity & Feature Selection

1. To prevent multicollinearity distortion, pairwise correlations were examined:
* `cigsPerDay` showed high correlation with `currentSmoker` (r = 0.77)
* Blood pressure variables (`diaBP`, `sysBP`, and `prevalentHyp`) showed strong collinearity (r = 0.62 to 0.79)
* `glucose` and `diabetes` showed correlation (r = 0.61)

2. Then, redundant features were selectively dropped :  
* Dropped `currentSmoker` in favor of `cigsPerDay` (which captures continuous smoking volume and naturally includes non-smokers at zero)
* Dropped `prevalentHyp` as blood pressure is directly tracked via `sysBP` and `diaBP`
* Kept `glucose` and `diabetes` as even though these variables are correlated, diabete brings additional information about chronic situation (while glucose measurement may be subject to temporary patient fluctuation).

3. Lastly, backward elimination were performed comparing three methods :  
**AIC**, **P-values (threshold < 0.05)**, and **Lasso Regularization (L1)**. 
The core 6 predictors selected unanimously across the three methods are: **`male`**, **`age`**, **`cigsPerDay`**, **`totChol`**, **`sysBP`**, and **`glucose`**.

## Logistic Regression & Clinical Scaling

A logistic regression model was fitted using `statsmodels`.  
To provide meaningful clinical interpretations, continuous coefficients were scaled to realistic medical step sizes:
* **Age:** 10-year increase
* **Cigarettes per day:** 20 cigarettes (1 pack)
* **Total Cholesterol:** 50 mg/dL increase
* **Systolic Blood Pressure:** 10 mm Hg increase
* **Glucose:** 10 mg/dL increase

## Key Findings & Interpretation

Holding all other features constant, the quantified impact on heart disease odds is as follows : 
* Men have 75% higher odds of heart disease than women (CI95%:[42%;116%]).  
* For a 10-year increase in age, the odds of having a heart disease are higher by 93% (CI95%:[70%;119%])
* For an increase of 1 pack of cigarettes per day, the odds of having a heart disease are higher by 47% (CI95%:[25%;73%])
* For a 50-mg/dL increase in total cholesterol, the odds of having a heart disease are higher by 12% (CI95%:[0%;25%])
* For a 10-mm Hg increase in systolic blood pressure, the odds of having a heart disease are higher by 19% (CI95%:[14%;24%])
* For a 10-mg/dL increase in glucose level, the odds of having a heart disease are higher by 8% (CI95%:[4%;11%])

In other words, if we take the example of the `cigsPerday` variable:   
if we compare two otherwise identical men (same age, same cholesterol, same blood pressure etc.) smoking 1 pack of cigarettes difference, the man who smokes 1 additional pack per day has odds of heart disease higher by 47% compared to the other man.

## Model Evaluation & Diagnostics

1. Calibration Analysis
To verify if predicted probabilities match real observed frequencies, a reliability diagram was constructed using quantile-based binning (`pd.qcut`) paired with binomial 95% confidence intervals. The analysis confirmed that the model's calibration is reliable across both low- and high-probability subgroups.

2. Discrimination (ROC-AUC Score)
The model's overall ranking ability was evaluated via ROC curve analysis, achieving an **ROC-AUC score of 0.736**, demonstrating strong discrimination capability.
