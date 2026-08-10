# Name: Arun Bhaskar Gadde

# Course: 2026 Summer - Advanced Big Data and Data Mining (MSCS-634-B01) - Second Bi-term

# Lab 6: Association Rule Mining with Apriori and FP-Growth

## Purpose

This lab applies two frequent-itemset mining algorithms — Apriori and FP-Growth — to a
real, publicly available transactional dataset, then generates and interprets
association rules from the results. The goal is to practice the full market-basket
analysis workflow (cleaning transactional data, mining frequent itemsets, comparing
algorithm efficiency, generating rules with support/confidence/lift, and visualizing
results with Seaborn) on data messy enough to require real preprocessing decisions.

**Dataset:** UCI Machine Learning Repository's Online Retail Dataset — 541,909
transaction line items from a UK-based online gift retailer (Dec 2010 – Dec 2011). The
UCI repository and Kaggle were not directly reachable from this notebook's execution
environment, so the dataset was pulled from a public GitHub mirror of the same,
unmodified file (row count and columns match the original UCI dataset exactly).

## Key Insights

- **FP-Growth was about 13x faster than Apriori** on this dataset at the same 2%
  minimum support threshold (Apriori: ~16.7s, FP-Growth: ~1.3s), while finding the
  **exact same 385 frequent itemsets**. This matches the theoretical expectation:
  Apriori repeatedly re-scans the transaction data to generate and test candidate
  itemsets at each itemset length, while FP-Growth compresses the data into a single
  FP-tree once and mines patterns directly from it, with no candidate generation step.
- **The strongest association rules were between product color/pattern variants**, not
  unrelated products. The highest-lift rules connected the "Regency Teacup and Saucer"
  sets in pink, green, and rose patterns (lift up to ~18), and the two "Gardeners
  Kneeling Pad" designs (lift ~15.6). This is a realistic and useful pattern: customers
  buying one variant of a themed product are unusually likely to buy a matching variant
  in the same order — a genuine cross-sell/bundling signal a retailer could act on.
- **153 rules met the 30% minimum confidence threshold**, and nearly all of them had a
  lift well above 1 (see the confidence-vs-lift scatter plot in the notebook), meaning
  the itemsets that survived the 2% support filter mostly represent real, non-random
  associations rather than coincidental co-purchases.
- The **co-occurrence heatmap** made the "Jumbo Bag" and "Jumbo Shopper/Storage" product
  families visibly stand out as a cluster before any formal rule mining was even run —
  a useful sanity check that the later Apriori/FP-Growth results lined up with what the
  raw co-occurrence counts already suggested.

## Challenges and Decisions

- **Dataset access:** UCI and Kaggle were not reachable from this environment, so the
  dataset was sourced from a public GitHub mirror instead. The file was verified to
  match the original UCI dataset's known shape (541,909 rows, 8 columns) before use.
- **Encoding:** The raw CSV failed to load with pandas' default UTF-8 encoding;
  `encoding="latin1"` was required.
- **Item dimensionality:** With 4,065 distinct item descriptions, a full invoice-by-item
  matrix (~82 million cells) was unnecessarily large, since most items are far too rare
  to ever reach a usable support threshold. Items purchased in fewer than 50 distinct
  invoices were dropped before building the matrix, reducing it to 2,185 items without
  changing which itemsets would qualify as "frequent" at the support levels used here.
- **Choosing a support threshold:** 5% support returned almost no multi-item itemsets;
  1% support produced a very large number of itemsets and took noticeably longer to
  mine. 2% was chosen as the point that produced a meaningful number of multi-item
  itemsets while keeping runtime reasonable for both algorithms.
- **Fair algorithm comparison:** Apriori and FP-Growth were run on the identical basket
  matrix with the identical minimum support, and their resulting itemsets were
  explicitly checked for set-equality in code before comparing runtimes, so the timing
  comparison reflects a genuinely fair, apples-to-apples test.

## Files in This Repository

- `Lab6_Association_Rule_Mining.ipynb` — the full, executed notebook (every cell has
  real output from an actual run against the dataset; re-run "Restart & Run All" to
  reproduce).
- `README.md` — this file.
- `data/online_retail.csv` — the dataset used (UCI Online Retail Dataset, via GitHub mirror).


## How to Reproduce

```bash
pip install pandas numpy matplotlib seaborn mlxtend
jupyter notebook Lab6_Association_Rule_Mining.ipynb
# then: Kernel > Restart & Run All
```
