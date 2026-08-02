# Name: Arun Bhaskar Gadde

# Course: 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

# Assignment: Lab 5 - Clustering Techniques Using DBSCAN and Hierarchical Clustering

## Purpose

This lab applies two unsupervised clustering techniques — Agglomerative Hierarchical
Clustering and DBSCAN — to the Wine dataset from `sklearn.datasets`. The goal is to
practice the full clustering workflow (data exploration, standardization, model
fitting, parameter tuning, visualization, and evaluation) and to compare how the two
algorithms perform on the same, real dataset.

The Wine dataset contains 178 samples of chemical analysis results for wines grown in
the same region of Italy, from 3 different cultivars (grape varieties). The cultivar
label is not used during clustering — it is only used afterward, to check how well the
unsupervised clusters line up with the real grouping.

## Key Insights from the Clustering Results

- **Hierarchical Clustering was the clear winner on this dataset.** With `n_clusters=3`
  (chosen using both a silhouette-score sweep and the dendrogram), it reached a
  silhouette score of **0.277**, and — more importantly — a homogeneity score of
  **0.79** and completeness score of **0.78** against the true cultivar labels. That
  means its 3 clusters lined up closely with the 3 real grape varieties.
- **DBSCAN struggled on this dataset.** Across 15 combinations of `eps` and
  `min_samples`, DBSCAN never recovered the true 3-cluster structure — its best result
  found only **2** clusters, with a homogeneity score of just **0.03** against the true
  labels. Many parameter combinations produced degenerate results (0 or 1 clusters, or
  almost the entire dataset labeled as noise).
- **Why the gap?** DBSCAN assumes clusters are separated by regions of low density.
  The Wine dataset's 3 cultivars turned out to sit in the standardized feature space as
  fairly compact, evenly-sized, somewhat-overlapping groups — closer to what
  Hierarchical Clustering (or K-Means) is built for than what DBSCAN is built for. This
  was a genuinely useful finding: it's a concrete example of a real dataset where
  DBSCAN's flexibility (no need to pre-specify cluster count, ability to flag noise)
  didn't pay off, because the data's structure didn't match DBSCAN's core assumption.
- **The dendrogram was a useful sanity check.** Its largest vertical "jumps" (merge
  distances) lined up with a 3-cluster cut, which matched both the silhouette-score
  sweep and the known number of cultivars — three independent signals agreeing gave
  more confidence in the `n_clusters=3` choice than any single metric alone.

## Challenges and Decisions

- **Choosing DBSCAN's parameter grid.** An initial narrow grid around small `eps`
  values (which is what many DBSCAN tutorials default to) produced almost entirely
  noise on this standardized 13-feature dataset. The grid had to be widened
  (`eps` from 1.5 to 3.5) before any combination produced more than one usable
  cluster — a reminder that sensible `eps` ranges depend heavily on the number of
  features and how the data was scaled.
- **Handling degenerate DBSCAN results.** Several `(eps, min_samples)` combinations
  produced 0 or 1 clusters, for which `silhouette_score` is undefined. Rather than
  letting the notebook crash or silently skipping these, they're kept in the results
  table with `silhouette_score = None`, so the full parameter sweep — including its
  failures — is visible and honestly reported.
- **Choosing which points to compute silhouette score on.** For DBSCAN, silhouette
  score was computed only on non-noise points, since silhouette isn't meaningful for a
  point that has been explicitly excluded from every cluster. This is noted directly in
  the notebook so the metric isn't misread as covering the whole dataset.
- **Reporting results honestly.** An earlier draft of the analysis assumed DBSCAN would
  land "close to" Hierarchical Clustering's performance. After actually running the
  full parameter sweep, the real gap turned out to be much larger (homogeneity 0.79 vs.
  0.03). The write-up was revised to match the actual output rather than the initial
  assumption — the notebook's Step 4 discussion reflects the real, executed results.

## Files in This Repository

- `Wine_Clustering_Lab.ipynb` — the full, executed Jupyter Notebook (all cells have
  been run and contain real output — re-run "Restart & Run All" to reproduce).
- `README.md` — this file.


## How to Reproduce

```bash
pip install scikit-learn pandas numpy matplotlib scipy
jupyter notebook Wine_Clustering_Lab.ipynb
# then: Kernel > Restart & Run All
```
