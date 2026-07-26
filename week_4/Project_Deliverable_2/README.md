# Name: Arun Bhaskar Gadde

# Course: 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

# Assignment: Project Deliverable 2: Regression Modeling and Performance Evaluation


## Overview

This deliverable builds on the cleaned Telco Customer Churn dataset from Deliverable 1
(7,043 customers, 21 attributes, zero missing values after cleaning) to develop and
evaluate regression models. Since the dataset's natural label (`Churn`) is categorical and
reserved for the classification deliverable, **`MonthlyCharges`** was selected as the
continuous target variable: a customer's monthly bill is driven by which services they
subscribe to and their contract terms, making it a genuine regression problem well suited
to feature engineering.

`TotalCharges` was deliberately excluded from the feature set. Deliverable 1's EDA found it
is very strongly correlated with `tenure` and `MonthlyCharges` (`TotalCharges` ≈ `tenure` ×
`MonthlyCharges`), so modeling it would be near-deterministic rather than a meaningful
regression task.

## Feature Engineering

- **Binary recoding** of Yes/No columns (`SeniorCitizen`, `Partner`, `Dependents`,
  `PhoneService`, `PaperlessBilling`, `gender`) into 0/1.
- **Collapsed three-level service columns** (e.g., `OnlineSecurity`: Yes / No / No internet
  service) into clean binary flags, since "No internet service" is just a special case of
  "No" driven by another column.
- **Engineered `num_addon_services`:** a 0-6 count of how many optional add-ons
  (OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV,
  StreamingMovies) each customer has, condensing six sparse columns into one "service
  breadth" signal.
- **Engineered `contract_length_months`:** `Contract` (Month-to-month / One year / Two year)
  mapped to its numeric length (1 / 12 / 24) to preserve its natural ordering for the linear
  models instead of one-hot encoding it.
- **One-hot encoding** of the remaining nominal columns: `InternetService` and
  `PaymentMethod`.
- **Dropped:** `customerID` (no predictive value), `TotalCharges` and `Churn` (excluded per
  the rationale above), and the raw columns replaced by their engineered versions.

## Models Built

| # | Model | Description |
|---|-------|-------------|
| 1 | Baseline Linear Regression | Trained on only 5 raw, unengineered columns — establishes a "before feature engineering" reference point |
| 2 | Multiple Regression | Linear Regression on the full 15-feature engineered set |
| 3 | Ridge Regression | L2-regularized, alpha selected via 5-fold `RidgeCV` |
| 4 | Lasso Regression | L1-regularized, alpha selected via 5-fold `LassoCV` |

All models were trained on an 80/20 train/test split (5,634 / 1,409 rows), with numeric
features standardized before fitting.

## Evaluation Results (Held-Out Test Set)

| Model | MAE | MSE | RMSE | R² |
|---|---|---|---|---|
| Baseline Linear Regression (raw features) | 22.630 | 760.650 | 27.580 | 0.160 |
| Multiple Regression (engineered features) | 1.973 | 7.102 | 2.665 | 0.992 |
| Ridge Regression (alpha≈0.869) | 1.973 | 7.102 | 2.665 | 0.992 |
| Lasso Regression (alpha≈0.003) | 1.972 | 7.101 | 2.665 | 0.992 |

## Cross-Validation Results (5-Fold, Full Dataset)

| Model | Mean CV R² | Std CV R² | Mean CV RMSE |
|---|---|---|---|
| Baseline Linear | 0.179 | 0.016 | 27.233 |
| Multiple Regression | 0.992 | 0.001 | 2.602 |
| Ridge | 0.992 | 0.001 | 2.602 |
| Lasso | 0.992 | 0.001 | 2.602 |

## Key Insights

- **Feature engineering drove almost all of the improvement.** Moving from 5 raw columns
  to the full 15-feature engineered set took R² from 0.160 to 0.992 — a far larger jump
  than anything regularization contributed afterward. `num_addon_services` and the
  `InternetService` one-hot columns (especially Fiber optic) were the strongest predictors,
  confirming that *what a customer subscribes to* — not how long they've been a customer —
  determines their monthly bill.
- **Multiple Regression, Ridge, and Lasso tied for best performance**, all reaching
  R² ≈ 0.992 on both the held-out test set and 5-fold cross-validation, with identical,
  very low fold-to-fold variance (std ≈ 0.001). This indicates the engineered feature set
  doesn't suffer from severe multicollinearity or overfitting at this sample size, so
  regularization didn't need to "fix" anything the base model was already doing well.
- **Lasso's main advantage was interpretability, not accuracy** — it matched the other
  models' performance while zeroing out one of the fifteen coefficients, offering a
  slightly leaner model.
- **Cross-validation validated the single train/test split.** The 5-fold CV metrics closely
  tracked the held-out test metrics for every model, giving confidence the reported
  performance isn't an artifact of an easy or lucky split.
- **Dataset connection to churn:** the strongest drivers of `MonthlyCharges` — fiber
  internet and a higher count of add-on services — were also flagged in Deliverable 1 as
  associated with *higher* churn. That link between premium, higher-billing customers and
  elevated churn risk is a thread worth carrying into the classification deliverable.

## Challenges and Decisions

- **Choosing a regression target:** the dataset has no natural continuous label suited to
  clean regression. `TotalCharges` was the obvious first choice but was rejected because
  it is near-deterministically derived from `tenure` and `MonthlyCharges`, which would have
  made the modeling exercise trivial. `MonthlyCharges` was chosen instead because it is
  driven by genuinely varied customer behavior (service choices), giving feature
  engineering real work to do.
- **Handling the "No internet/phone service" category:** several service columns encode a
  dependency on another column (e.g., you can't have `OnlineSecurity` without
  `InternetService`) through a third categorical level. Rather than one-hot encoding this
  as a separate category, it was collapsed into a simple binary flag so the model interprets
  it as "no service" rather than a distinct, unrelated class.
- **Deciding between one-hot encoding and ordinal mapping for `Contract`:** since contract
  length has a natural order (month-to-month < one year < two year), it was mapped to its
  numeric duration in months instead of one-hot encoded, so the linear models could use the
  ordering directly rather than treating the three contract types as unrelated categories.
- **Selecting alpha for Ridge and Lasso:** rather than picking regularization strength
  manually, `RidgeCV` and `LassoCV` were used to search a wide range of alpha values
  (0.001–1000) via cross-validation, so the reported results reflect a properly tuned
  alpha rather than a guess.
- **Interpreting near-identical model performance:** with all three engineered models
  reaching R² ≈ 0.992, the temptation was to overstate small differences between them.
  The write-up was revised to reflect that Ridge, Lasso, and plain Multiple Regression
  performed as a statistical tie on this dataset, rather than claiming one clearly "won."
