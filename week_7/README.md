# Name: Arun Bhaskar Gadde

# Course: 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

# Project Deliverable 3: Classification, Clustering, and Pattern Mining


## Overview

This deliverable builds on the HealthTrack capstone by applying classification,
clustering, and association rule mining to a synthetic patient-vitals dataset
(heart rate, blood pressure, SpO2, temperature, respiratory rate, age) with a
clinically-inspired `risk_label` (High / Low). The full, executed analysis is
in `project_deliverable3.ipynb`.

## What's in the notebook

1. **Dataset generation** — 900 synthetic patients with vitals and a risk
   label derived from a noisy combination of realistic clinical thresholds
   (so the classification task isn't trivially easy).
2. **Classification models** — Decision Tree and k-Nearest Neighbors (k=5),
   trained to predict `risk_label` from vitals.
3. **Hyperparameter tuning** — `GridSearchCV` over the Decision Tree's
   `max_depth`, `min_samples_split`, `min_samples_leaf`, and `criterion`,
   optimizing 5-fold cross-validated F1 score.
4. **Evaluation** — confusion matrices, ROC curves, accuracy/F1, a full
   classification report, and cross-validation, comparing the tuned tree
   against k-NN.
5. **Clustering** — K-Means (k chosen via elbow method + silhouette score),
   visualized with PCA, with cluster vitals-profiles interpreted as
   "stable" / "borderline" / "elevated-risk" patient groups.
6. **Association rule mining** — vitals binned into clinical categories,
   then mined with both **Apriori** and **FP-Growth**, filtered to rules
   predicting `Risk_High` / `Risk_Low`, ranked by lift.

## Key insights

- **Classification & tuning:** The tuned Decision Tree reached **85% overall
  accuracy**, but the more important number is the **High-risk class**
  performance: precision 0.65, recall 0.57, F1 0.60 on the held-out test set,
  and a mean 5-fold cross-validation F1 of **0.63** for that class (which is
  consistent with the test result). k-NN had similar accuracy (82%) but a
  much lower recall on High-risk patients (0.24), meaning it missed far more
  true high-risk cases than the Decision Tree. This matters for a monitoring
  system, where missing a true high-risk patient is more costly than a false
  alarm.
- **Clustering:** Patients grouped into roughly three clusters whose vitals
  averages line up with an informal triage system — a mostly-stable group,
  a borderline group with mildly elevated vitals, and a smaller group with
  clearly abnormal vitals and a much higher share of High-risk labels —
  even though the clustering never saw the risk label itself.
- **Association rule mining:** The strongest rules linking vitals to risk
  consistently involve **low SpO2** combined with a normal heart rate/
  temperature profile, with lift values around **4–5**, meaning patients
  with that combination are 4–5x more likely to be labeled High risk than
  chance would predict. This gives a transparent, clinician-checkable
  explanation for the model's behavior.

## Real-world relevance

- The tuned classifier could serve as an automated first-pass triage flag
  in HealthTrack, with the confusion matrix guiding whether the team wants
  to trade more false alarms for fewer missed high-risk patients.
- The cluster profiles could help prioritize nursing attention toward the
  "borderline" group before they escalate to "elevated-risk."
- The association rules act as a plain-language audit trail — if the model
  flags a patient, the underlying rule (e.g., low SpO2 + fever) can be shown
  to a clinician for quick verification, which matters for trust in any
  automated alerting system.

## Challenges encountered and how they were addressed

- **Trivial separability risk:** An early version of the risk label used a
  single clean threshold, which made classification too easy (>95%
  accuracy) and unrealistic. This was fixed by deriving the label from a
  weighted, noisy combination of several vitals instead.
- **Misleading cross-validation score:** `cross_val_score`'s default F1
  scorer treated the majority class ("Low") as positive, which produced an
  inflated CV score (~0.91) that did not match the test-set F1 for the
  class we actually care about (High risk, ~0.60). This was caught and
  fixed by building an explicit scorer with `pos_label` set to the High-risk
  class, after which the CV and test scores became consistent (~0.63 vs
  ~0.60).
- **Choosing k for clustering:** The elbow plot alone did not have a sharp
  "elbow," so silhouette scores were used alongside it to settle on k=3.
- **Binning for association rules:** Continuous vitals had to be converted
  into clinically sensible categories (rather than arbitrary quartiles) so
  the mined rules would be interpretable to a non-technical audience.

## How to run

```bash
pip install -r requirements.txt
jupyter notebook project_deliverable3.ipynb
```

## Files

- `project_deliverable3.ipynb` — full, executed notebook with code, outputs,
  and visualizations
- `requirements.txt` — Python dependencies
- `README.md` — this file
