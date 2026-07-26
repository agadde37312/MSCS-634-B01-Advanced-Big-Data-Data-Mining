# Name: Arun Bhaskar Gadde

# Course: 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

# Assignment: Lab 4 - Regression Analysis with Regularization Techniques

## Purpose

This lab explores multiple regression techniques on the **Diabetes dataset**
(`sklearn.datasets.load_diabetes`), which contains 10 baseline physiological
measurements for 442 patients and a target representing disease progression
one year after baseline. The goals of the lab were to:

- Implement **Simple Linear Regression**, **Multiple Regression**, and
  **Polynomial Regression** models.
- Apply **Ridge (L2)** and **Lasso (L1)** regularization to observe how they
  help prevent overfitting.
- Evaluate every model using **MAE, MSE, RMSE, and R²**.
- Visualize predictions, residual patterns, and the effect of regularization
  strength (alpha) on model behavior.

## Repository Contents

- `regression_analysis.ipynb` – the full Jupyter Notebook with code, visualizations,
  and written analysis for all six lab steps.
- `README.md` – this file.

## Key Insights

- **Feature importance:** BMI, blood pressure, and the `s5` serum measurement
  are the strongest individual predictors of disease progression.
- **Simple vs. multiple regression:** Using all 10 features instead of BMI
  alone substantially improved R², confirming that disease progression is
  driven by a combination of factors rather than any single measurement.
- **Polynomial regression and overfitting:** Increasing the polynomial degree
  raised training R² (near-perfect fit at degree 3) but did **not** improve
  — and in some cases hurt — test R². The widening gap between training and
  test performance as degree increased is a textbook overfitting signature.
- **Regularization effect:** Applying Ridge and Lasso to the polynomial
  feature set recovered much of the generalization performance lost to
  overfitting. Ridge shrinks all coefficients toward zero without eliminating
  them; Lasso can zero out coefficients entirely, effectively performing
  feature selection. Alpha values that were too small behaved like plain
  (overfitting) regression, while alpha values that were too large caused
  underfitting — the best test R² occurred at a moderate alpha for both
  methods.
- **Overall takeaway:** For this dataset, a well-tuned Ridge or Lasso model
  applied to the polynomial features gave the best balance of accuracy and
  generalization, outperforming both the plain multiple regression model and
  the unregularized higher-degree polynomial models.

## Challenges and Decisions

- **Choosing the polynomial degree range:** Degree 2 and 3 were selected to
  clearly illustrate the transition from a reasonable fit to overfitting;
  higher degrees were avoided since the number of polynomial features grows
  quickly with 10 base features and would have made the model impractically
  large.
- **Scaling before regularization:** Ridge and Lasso penalize coefficient
  magnitude, so features were standardized (`StandardScaler`) after
  polynomial expansion to ensure the penalty was applied fairly across all
  features regardless of their original scale.
- **Selecting alpha values:** A range of alpha values (0.01 to 100) was
  tested for both Ridge and Lasso, and the alpha with the best test R² was
  selected for the final comparison, to avoid arbitrarily picking a single
  regularization strength.
- **Lasso convergence:** Lasso required a higher `max_iter` (20,000) to
  reliably converge on the expanded polynomial feature set without
  convergence warnings.
