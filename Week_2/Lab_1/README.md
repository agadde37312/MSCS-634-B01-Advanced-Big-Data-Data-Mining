# Data Visualization, Preprocessing, and Statistical Analysis Lab

**Name:** Arun Bhaskar Gadde

**Course:** Advanced Big Data and Data Mining (MSCS-634-B01) - Summer Second Bi-term

**Assignment:** Lab 1 — Data Visualization, Data Preprocessing, and Statistical Analysis using Jupyter Notebook


## Purpose

This lab applies the core data analytics workflow — collection, visualization, preprocessing, and statistical analysis — to a retail sales dataset using Jupyter Notebook and Pandas. The goal was to practice identifying patterns and relationships in raw data through charts, clean messy real-world-style data (missing values, outliers), reduce and transform it for downstream analysis, and summarize it with descriptive and correlation statistics.

## Dataset

A synthetic retail sales dataset (500 orders, one year, four regions, five product categories) was generated with a fixed random seed so the notebook is fully self-contained and reproducible without relying on an external download. Realistic messiness — missing values in `CustomerRating`, `ShippingCost`, and `Region`, plus bulk-order outliers in `UnitsSold`/`Revenue` — was deliberately injected so the preprocessing steps in the lab had genuine problems to solve rather than working on already-clean data.

## Key Insights

**From visualizations:**
- Revenue scales with units sold, but the slope differs sharply by category — Electronics generates far more revenue per unit sold than Groceries or Toys due to price differences.
- A 7-day rolling average smooths out day-to-day revenue noise and reveals a few sharp spikes tied directly to the bulk-order outliers.
- North is the top-revenue region; West lags behind the other three.
- Unit price is right-skewed and multi-modal — the modes line up with product category price tiers, confirming price is category-driven rather than uniform.
- Electronics has the highest median revenue per order and the widest spread, with visible outliers above the box-plot whisker; Groceries and Toys are tightly clustered near the bottom.
- Electronics and Clothing together account for roughly half of all orders, while Toys and Furniture are the smallest categories.

**From statistical analysis:**
- `UnitsSold` and `Revenue` are strongly positively correlated, as expected since Revenue is derived from Units Sold × Unit Price.
- `UnitPrice` correlates with Revenue, but more moderately, since order size also drives revenue.
- `CustomerRating` shows little to no correlation with sales metrics — in this dataset, customer satisfaction is essentially independent of how much was ordered or spent.
- Central tendency and dispersion measures (mean, median, mode, IQR, variance, std dev) confirmed the right-skew visible in the histograms, particularly for `Revenue` and `UnitPrice`, where the mean sits noticeably above the median.

## Challenges and Decisions

- **Making "messy" data realistic without being arbitrary.** Rather than injecting missing values and outliers purely at random, they were placed in specific columns (`CustomerRating`, `ShippingCost`, `Region`) and as deliberate bulk orders (`UnitsSold`/`Revenue`) so the cleaning steps mirror the kinds of gaps and anomalies that show up in real retail data exports.
- **Choosing different imputation strategies per column instead of one blanket method.** `CustomerRating` (numeric, roughly normal) was filled with the mean; `ShippingCost` (numeric, scattered gaps in date-ordered data) was forward-filled; `Region` (categorical) was filled with the mode — each chosen to match the nature of that specific variable rather than defaulting to a single approach everywhere.
- **IQR vs. standard deviation for outlier detection.** IQR was chosen over the standard-deviation method because `UnitsSold` is right-skewed (Poisson-distributed) rather than normal, and IQR is more robust to skewed distributions.
- **Order of operations in preprocessing.** Missing values were handled before outlier detection, since a handful of missing `CustomerRating`/`ShippingCost` values could otherwise have been mistaken for statistical outliers or excluded silently from later calculations.
- **`fillna(method="ffill")` deprecation.** The originally written forward-fill call using `fillna(method="ffill")` raised a `TypeError` under the current pandas version, since the `method=` argument was removed. This was fixed by switching to the newer `.ffill()` accessor — a good reminder to always execute a notebook end-to-end rather than assuming code will run as written.
- **Column selection for dimension elimination.** `ShippingCost` was dropped in the data-reduction step because it wasn't relevant to the sales/revenue analysis being performed, illustrating that "less relevant" columns should be chosen based on the specific downstream analysis, not by default assumptions about which columns matter.
