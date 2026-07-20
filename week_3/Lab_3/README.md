# K-Means vs. K-Medoids Clustering — Wine Dataset

**Author:** Arun Bhaskar Gadde

**Course:** 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

**Notebook:** `KMeans_KMedoids_Wine_Lab.ipynb`

## Purpose

This lab applies two clustering algorithms — K-Means and K-Medoids — to the scikit-learn Wine dataset (178 samples, 13 standardized chemical properties, 3 known cultivars) and compares them using the Silhouette Score (cluster tightness/separation) and the Adjusted Rand Index, or ARI (agreement with the true wine classes). The goal is to understand how the two algorithms differ in what they optimize, how that shows up in the results, and when each is the better choice.

## Key Insights

**Metrics:**
| Metric | K-Means | K-Medoids |
|---|---|---|
| Silhouette Score | 0.285 | 0.268 |
| Adjusted Rand Index (ARI) | 0.898 | 0.741 |

- **Silhouette Score:** K-Means edged out K-Medoids, but the gap is small — both produce reasonably well-separated clusters. This matches what each algorithm actually optimizes: K-Means directly minimizes squared distance to a cluster mean (exactly what Silhouette rewards), while K-Medoids minimizes distance to an actual data point, a slightly less flexible objective.

- **ARI:** the gap here is more noticeable. K-Means (0.898) came very close to perfectly reproducing the three true wine cultivars from unsupervised clustering alone. K-Medoids (0.741) was clearly good but noticeably behind K-Means on this dataset.

- **Cluster centers:** K-Means centroids are synthetic means that can land in "empty space" with no matching real sample. K-Medoids medoids are always real data points, which makes them directly interpretable as "here is a representative example of this cluster" — useful when explainability matters more than squeezing out maximum accuracy.

- **When to prefer each:** K-Means is the more practical default for clean, fully numeric, roughly spherical data like this — it's cheaper to compute and won a bit on both metrics. K-Medoids becomes more attractive when the data has real outliers (medoids resist being pulled off-center the way a mean can be), when a non-Euclidean distance metric is needed (K-Medoids works with any distance metric; K-Means is mathematically tied to Euclidean distance), or when a genuinely interpretable, real-example cluster center matters more than the last bit of accuracy.

## Challenges and Decisions

- **No maintained K-Medoids implementation was available.** The standard third-party package for K-Medoids in Python (`scikit-learn-extra`) is unmaintained and fails to import under current NumPy versions (it was compiled against NumPy 1.x and is binary-incompatible with NumPy 2.x). Rather than fight a broken dependency, K-Medoids was implemented from scratch using the classic "alternating" PAM-style heuristic (assign points to the nearest medoid, then update each medoid to the point that minimizes total in-cluster distance, repeat until stable). This was verified against a small manual test before being used in the full notebook.

- **An unfair first comparison, caught and fixed.** The first version of the from-scratch K-Medoids used only a single random initialization, and scored far worse than expected (ARI ≈ 0.34) — but this wasn't a fair test, since scikit-learn's `KMeans` restarts 10 times by default (`n_init=10`) and silently keeps the best result. Once K-Medoids was given the same 10-restart treatment, its ARI jumped to 0.74. This is itself a real, worthwhile finding (K-Medoids' alternating heuristic is more initialization-sensitive than K-Means, and benefits proportionally more from multiple restarts) and is documented directly in the notebook rather than quietly fixed and forgotten.

- **Standardization was essential, not optional.** Clustering has no train/test split to fall back on, so every point's distance to every candidate center matters directly. Without z-score standardization, the `proline` feature (raw values up to 1680) would have dominated the distance calculation and effectively decided the clusters on its own, regardless of the other 12 chemical properties.

- **PCA for visualization only.** Since the data has 13 dimensions, PCA was used to project down to 2 components purely for plotting. Both algorithms were run — and both sets of performance metrics were computed — on the full 13-dimensional standardized data; PCA never influenced the actual clustering, only how the results are drawn.
