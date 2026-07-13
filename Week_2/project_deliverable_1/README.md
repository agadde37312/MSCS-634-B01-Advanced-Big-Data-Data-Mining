# MSCS-634 Advanced Big Data and Data Mining — Project Deliverable 1

## Data Collection, Cleaning, and Exploration

**Author:** Arun Bhaskar Gadde


## Dataset Summary

This deliverable uses the **Telco Customer Churn** dataset (IBM sample dataset, commonly hosted on Kaggle and GitHub), which contains **7,043 customer records across 21 attributes** — well above the project's minimum of 500 records / 8-10 attributes.

Each row represents one telecom customer and includes:
- **Demographics:** gender, senior citizen status, partner, dependents
- **Account info:** tenure, contract type, paperless billing, payment method, monthly charges, total charges
- **Services subscribed:** phone service, multiple lines, internet service type, online security, online backup, device protection, tech support, streaming TV, streaming movies
- **Target variable:** `Churn` — whether the customer left the company (Yes/No)

This dataset was chosen because it sits squarely in the "customer behavior" domain suggested for the project, mixes numeric and categorical attributes in a way that supports every technique required across all four deliverables (regression, classification, clustering, and association rule mining), and includes realistic data-quality quirks that provide genuine cleaning practice rather than a dataset that's already spotless.

## Key Insights from Analysis

- **Overall churn rate is ~26.5%**, a moderate class imbalance that later classification work will need to account for.
- **Contract type is highly predictive of churn:** month-to-month customers churn far more often than one-year or two-year contract customers.
- **Tenure is bimodal** — customers tend to either leave very early (0-5 months) or stay for a very long time (near 72 months), with fewer customers in between. Low-tenure customers are the highest churn risk.
- **New, higher-paying customers churn most:** churned customers cluster in the low-tenure, higher-MonthlyCharges region of the tenure-vs-charges scatter plot.
- **Fiber optic internet customers churn more than DSL or no-internet customers**, despite being the premium service tier — a pattern worth investigating further in later deliverables.
- **TotalCharges is strongly correlated with both tenure (0.83) and MonthlyCharges (0.65)**, since it accumulates from monthly billing over time. This near-collinearity will need to be handled carefully during regression feature selection in Deliverable 2.
- Several **binary service-subscription columns** (OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies) are well suited for association rule mining in Deliverable 4, to uncover which add-on services are commonly purchased together.

## Data Cleaning Steps

1. **Hidden missing values:** `TotalCharges` was loaded as a text column rather than numeric. Converting it to numeric revealed 11 unparseable (blank) entries, all belonging to customers with `tenure = 0` — i.e., brand-new customers who hadn't been billed yet. These were imputed as `0`, which is consistent with their billing history rather than an arbitrary fill value.
2. **Duplicates:** checked for both fully duplicated rows and duplicated `customerID` values — none were found. `customerID` was set as the DataFrame index since it's a unique identifier with no analytical value as a feature.
3. **Inconsistent categorical labels:** all categorical columns were inspected for typos, casing mismatches, or stray whitespace — none were found. `SeniorCitizen` (originally 0/1) was recoded to `"No"`/`"Yes"` for consistency with the other binary Yes/No columns.
4. **Noisy data / outliers:** boxplots and an IQR check were run on `tenure`, `MonthlyCharges`, and `TotalCharges`. No outliers were flagged for tenure or MonthlyCharges; the few flagged in TotalCharges reflect legitimate long-tenured, high-paying customers rather than noise, so no rows were removed — deliberately, since discarding them would bias the dataset against a segment central to churn analysis.

## Challenges and Decisions

- **Silent missing data:** the missing values in `TotalCharges` weren't visible via `.isnull()` until the column was converted from text to numeric — a good reminder to check dtypes carefully rather than trusting `.info()` at face value on a first pass.
- **Deciding how to impute `TotalCharges`:** rather than dropping the 11 affected rows or filling with the column mean (which would fabricate billing history), the missing values were filled with `0` because all 11 cases had `tenure = 0`, making zero the factually correct value rather than a statistical guess.
- **Distinguishing genuine outliers from meaningful business patterns:** the IQR check flagged a handful of high `TotalCharges` values, but these came from real long-tenured, high-value customers rather than data errors. The decision was made to keep them, since removing them would distort the churn analysis this dataset is meant to support.
- **Anticipating downstream needs while cleaning:** cleaning decisions (e.g., not dropping "outlier" high-value customers, recoding SeniorCitizen for consistency) were made with Deliverables 2-4 in mind, since regression, classification, clustering, and association rule mining all depend on this same cleaned dataset.
