# K-Nearest Neighbors (KNN) vs. Radius Neighbors (RNN) — Wine Dataset

**Author:** Arun Bhaskar Gadde

**Course:** 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

**Notebook:** `KNN_RNN_Wine_Lab.ipynb`

## Purpose

This lab compares two distance-based classification methods — K-Nearest Neighbors (KNN) and Radius Neighbors (RNN) — on the scikit-learn Wine dataset (178 samples, 13 chemical properties, 3 wine classes). The goal is to see how each model's key parameter (k for KNN, radius for RNN) affects test accuracy, and to build intuition for when each approach is the better choice.

## Key Insights

**KNN (k = 1, 5, 11, 15, 21):**
- Accuracy was *lowest* at k=1 (77.8%) and jumped to its ceiling of 80.6% by k=5, then stayed completely flat through k=11, 15, and 21.
- k=1 underperforms because a single nearest neighbor is a high-variance vote — one mislabeled or borderline neighbor can flip the prediction. Once k reaches 5, there are enough votes to smooth that out.
- Accuracy not degrading even at k=21 (~15% of the training set) suggests the three wine classes are fairly well separated in this feature space.

**RNN (radius = 350, 400, 450, 500, 550, 600):**
- Accuracy was *highest* at the smallest radius tested (350: 72.2%) and steadily declined to 66.7% by radius 550-600 — the opposite of the "bigger neighborhood is better" assumption.
- Every test point had at least one neighbor at every radius tested (confirmed by explicitly counting empty neighborhoods), so the decline isn't from points being left without a vote — it's from a larger radius pulling in more distant, less relevant points from other classes and diluting the vote.

**KNN vs. RNN:** KNN's best accuracy (80.6%) clearly beat RNN's best (72.2%) on this dataset. The key difference is what each method holds fixed: KNN fixes the *number* of neighbors and lets distance vary, so it automatically adapts to how dense or sparse the data is in a given region. RNN fixes the *distance* and lets the neighbor count vary, which only works well if that distance means roughly the same thing everywhere in the feature space — and it doesn't here, because the data was deliberately left unscaled and one feature (`proline`, ranging up to 1680) dominates the Euclidean distance calculation.

**When to prefer each:** KNN is the safer default when feature density is uneven or features aren't on comparable scales, since it adapts automatically. RNN can be more competitive on properly scaled/standardized data, or in domains where a fixed distance has real physical meaning (e.g., geospatial data in meters) and where explicitly flagging outlier points with no nearby neighbors is itself valuable — a benefit that didn't come into play in this particular run, since every test point always had at least one neighbor.

## Challenges and Decisions

- **Making sense of the given radius range (350-600).** These values only make sense on *unscaled* data — pairwise distances in the raw Wine feature space range from about 2.6 to 1400 (mean ≈ 353, median ≈ 282), so a radius of 350-600 is a reasonable neighborhood size. On standardized data, these same values would be far too large and would include nearly every point. This was verified directly before writing the notebook, and the decision to leave the features unscaled was made deliberately (and explained in the notebook) rather than defaulting to standardization out of habit.

- **Handling test points with zero neighbors.** `RadiusNeighborsClassifier` raises an error by default if a test point has no training points within the given radius. This was handled with the `outlier_label` parameter, falling back to the majority training class — and the notebook explicitly counts how many test points actually triggered this fallback at each radius, rather than just assuming it wasn't an issue.

- **Letting the actual results override initial assumptions.** An earlier draft of the analysis assumed KNN accuracy would decrease with larger k and RNN accuracy would increase with a larger radius — reasonable-sounding guesses that turned out to be backwards once the code was actually run. The written observations in the notebook were rewritten to match the real computed output rather than the initial intuition, which is itself a useful reminder that these parameter/accuracy relationships are dataset-dependent and worth verifying empirically rather than assuming.
