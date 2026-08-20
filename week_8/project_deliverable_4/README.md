**Name:** Arun Bhaskar Gadde

**Course:** 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

**Assignment:** Project Deliverable 4: Final Insights, Recommendations, and Presentation

## Overview

This deliverable is the final phase of a four-part data mining project built on the
**Telco Customer Churn** dataset (IBM sample dataset, 7,043 customers, 21 attributes).
It consolidates the full pipeline — data cleaning, EDA, feature engineering, regression,
classification, clustering, and association rule mining — into a single notebook, all
built on the same customer population.

**A note on continuity:** In the originally submitted Deliverable 3, classification,
clustering, and association rule mining were built on a separate synthetic
patient-vitals dataset rather than the Telco data used in Deliverables 1 and 2. For this
final deliverable, those three techniques were **rebuilt on the actual Telco Customer
Churn dataset** so the whole project tells one connected story about the same customers,
as this deliverable requires.

## Dataset Summary

- **7,043 customers, 21 attributes** — demographics, account info, services subscribed,
  and the `Churn` target (Yes/No).
- Chosen because it mixes numeric and categorical attributes in a way that supports
  regression, classification, clustering, and association rule mining on one dataset,
  and includes a realistic data-quality issue for genuine cleaning practice.

## Project Steps

1. **Data Cleaning:** Recovered 11 hidden missing values in `TotalCharges` (stored as
   text; blanks belonged to 0-tenure customers and were imputed as 0). No duplicate
   rows or customer IDs found. No outliers removed — flagged high-value customers were
   judged to be a legitimate business segment, not noise.
2. **EDA:** Found a ~26.5% churn rate, bimodal tenure, and strong churn concentration
   among month-to-month, low-tenure, higher-paying, and Fiber optic customers.
3. **Feature Engineering:** Built `num_addon_services` (0-6 count of add-ons) and
   `contract_length_months` (ordinal contract mapping), collapsed three-level service
   columns to binary flags, and one-hot encoded remaining nominal columns.
4. **Regression:** Predicted `MonthlyCharges`. Feature engineering took R² from 0.160
   (raw features) to 0.992 (engineered features); Ridge and Lasso matched but didn't
   exceed Multiple Regression's accuracy.
5. **Classification:** Predicted `Churn` with a class-weighted, tuned Decision Tree
   (GridSearchCV) vs. k-NN. The tuned tree reached 75% accuracy with 0.78 recall on the
   churn class (AUC 0.821), outperforming k-NN (AUC 0.777) on the metric that matters
   most for retention outreach.
6. **Clustering:** K-Means (k=3, chosen via silhouette score) segmented customers into
   "Loyal Long-Term" (13.4% churn), "Stable Value" (2.0% churn), and "New & At-Risk"
   (42.5% churn, the largest segment).
7. **Association Rule Mining:** Apriori and FP-Growth (cross-validated against each
   other, both found 275 identical itemsets) surfaced service-bundling patterns (e.g.,
   OnlineSecurity + StreamingMovies → TechSupport, lift ≈ 2.9) and churn-risk rules
   (month-to-month + streaming services → churn, lift ≈ 2.0).

## Major Findings

- **The same at-risk customer profile — new, month-to-month, low add-on adoption —
  emerged independently from EDA, classification, clustering, and association rule
  mining.** Four different techniques converging on the same finding is stronger
  evidence than any one method alone.
- **What a customer subscribes to predicts billing far better than how long they've
  been a customer**, and **contract length is the strongest lever tied to retention**
  across every technique.
- Regularization (Ridge/Lasso) did not outperform plain Multiple Regression on this
  dataset — reported as a tie rather than overstating a difference that wasn't there.

## Recommendations

1. Target the "New & At-Risk" cluster (the largest segment, 42.5% churn) with an
   early-tenure retention program.
2. Use contract-length incentives (e.g., discounted upgrade to a 1-year contract) as the
   primary retention lever, not price discounts alone.
3. Investigate the Fiber optic churn gap with a customer survey or support-ticket
   review, since the dataset alone can't distinguish price sensitivity from service
   quality issues.
4. Use the mined association rules as a transparent, explainable basis for add-on
   cross-sell prompts.

## Ethical Considerations

- The dataset is de-identified and publicly released; `customerID` was used only as an
  index, never as a model feature.
- `gender` and `SeniorCitizen` were included as features but confirmed to be minor
  contributors (not primary drivers) to model predictions; a fairness audit is
  recommended before any real-world deployment.
- Class imbalance (~26.5% churn) was addressed with `class_weight="balanced"` and by
  evaluating churn-class recall/F1 rather than raw accuracy, so the model's real-world
  usefulness isn't hidden behind a misleadingly high accuracy figure.
- A Decision Tree and association rules were deliberately favored for their
  explainability, since a retention team needs to understand *why* a customer was
  flagged.

## Files

- `MSCS634_Deliverable4_Final.ipynb` — full, executed notebook with all code, outputs,
  and visualizations across all four techniques
- `MSCS634_Deliverable4_Report.docx` — comprehensive written report (title page,
  introduction, data preparation, modeling, evaluation, insights, ethical
  considerations, recommendations, references)
- `MSCS634_Deliverable4_Presentation.pptx` — slide deck with speaker notes
- `data/Telco-Customer-Churn.csv` — raw dataset
- `data/Telco-Customer-Churn-Cleaned.csv` — cleaned dataset used across all sections
- `README.md` — this file

## How to Run

```bash
pip install pandas numpy matplotlib seaborn scikit-learn mlxtend
jupyter notebook MSCS634_Deliverable4_Final.ipynb
```
