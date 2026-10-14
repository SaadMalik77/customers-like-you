# Customers Like You – GiftLane recommendations and customer map

## LLM help disclosure
-- LLM help was taken in understanding the assignment and its tasks. 
-- In code, most of the code, i have written myself. Help was taken in following aspects:
    1. There were numerous built-in functions from the modules that i had to ask LLM for, since I havent worked with them before.
    2. Structure of the folders
    3. Note() and apply_rule() functions in build_matrix.ipynb was suggested by LLM during debugging. I used them to generate reports
    4. In ml.ipynb, plot was drawn with the help of LLM for syntax.
    5. LLM help was taken in debugging.
    6. Important: Due to shortage of time and doing this project solo, i used LLM for coding ML4, ML5 and ML6. However, I made sure i understand what it does and analyze its results.
    7. I used my results from my code and analysis to formulate a memo with the help of LLM. Analysis was mine. Help was taken to formulate a formal document.

## Data
- Source: UCI Online Retail II — https://archive.ics.uci.edu/dataset/502/online+retail+ii
- Citation: Chen, D. (2012). DOI 10.24432/C5CG6D. Licence CC BY 4.0
- File: data/raw/online_retail_II.xlsx (43.5 MB, sheets "Year 2009-2010", "Year 2010-2011")
- Rows: 1,067,371
- To reproduce: download the zip, place the .xlsx in data/raw/, run `python src/build_matrix.py`


Mini-project: PCA vs Truncated SVD on the UCI Online Retail II dataset.
Main model: **Truncated SVD (k = 50)**. Comparison: PCA (arpack), same k, same matrix.

## Team and roles
Solo project: I hold both roles.
- **Data Engineer:** DE1–DE5 (profile, cleaning, time split, sparse matrix, matrix report)
- **ML Engineer:** ML1–ML6 (method choice, k, recommendations, evaluation, PCA vs SVD, customer map)
- Joint: J1 targets, J2 memo, J3 slides
- Stretch tasks DE6 (build_matrix.py) and ML7 (purchase-value sensitivity) not done (optional for individuals).

## Success targets (J1, agreed Day 1) and results
| # | Target | Result | Met? |
|---|---|---|---|
| 1 | Stored factors much smaller than the full matrix | 3.66 MB vs 161.43 MB dense (44× smaller); sparse matrix is already 4.99 MB | ✅ vs dense |
| 2 | Nightly job is fast (< 60 s) | 1.35 s total (fit 0.08 s, also-bought table 0.53 s, score all customers 0.74 s) | ✅ |
| 3 | Recommendations beat the best-seller baseline on held-out purchases (hit@10) | SVD 21.5% vs baseline 26.9% (−5.4 points) | ❌ honestly lost; see memo |
| 4 | Customer map shows groups marketing can name | 3 taste segments: Bags & lunchware buyers, Home décor shops, Kids' crafts & retro baking | ✅ |

## Data (Step 0)
- Source: Chen, D. (2012). *Online Retail II* [Dataset]. UCI Machine Learning Repository.
  https://doi.org/10.24432/C5CG6D. Licence CC BY 4.0.
- Download: https://archive.ics.uci.edu/dataset/502/online+retail+ii
- Put `online_retail_II.xlsx` in `data/raw/`. The `data/` folder is in `.gitignore` (raw data is not committed).
- **Row count on load: 1,067,371** (sheet "Year 2009-2010": 525,461; sheet "Year 2010-2011": 541,910).

## How to run
Requires Python 3.14. All package versions are pinned in `requirements.txt`.

```bash
# 1. Create the environment
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 2. Rebuild the matrix from the raw file (R6: one command)
jupyter nbconvert --to notebook --execute --inplace notebooks/build_matrix.ipynb

# 3. Run the analysis
jupyter nbconvert --to notebook --execute --inplace notebooks/ml.ipynb
```
The project path may contain spaces; run commands from the repository root.

## Reproducibility
- Fixed seed `SEED = 42` for TruncatedSVD, PCA, and KMeans.
- Time-based split: cutoff 2011-10-09 12:50 (last 2 months = test). Test purchases are held out
  before any model is fitted (R2); leak checks are asserted in the build notebook.
- k = 50 was chosen from the error curve and storage before any test hit rate was computed.

## Repository structure
```
notebooks/
  build_matrix.ipynb   DE1–DE5: load, clean, split, build matrix, reports
  ml.ipynb             ML1–ML6: method, k, recommendations, evaluation, comparison, map
data/                  (not committed)
  raw/                 online_retail_II.xlsx
  processed/           transactions_clean.parquet, train.parquet, test_pairs.parquet,
                       test_pairs_matrix.parquet, A_train.npz, customer_index.parquet,
                       product_index.parquet
reports/
  cleaning_log.csv          DE2: rows removed per rule
  split_report.csv          DE3: train/test counts
  matrix_report.csv         DE5: shape, sparsity, memory
  ml2_svd_curve.csv         ML2: error, time, size per k
  also_bought_k50.csv       ML3: 10 similar products for every product (product pages)
  ml4_results.csv           ML4: model vs baseline hit@10
  ml4_nightly_timing.csv    ML4: nightly job timing
  ml4_sensitivity_k.csv     ML4: hit@10 for every k (report only)
  ml5_pca_vs_svd.csv        ML5: comparison table
  ml6_segments.csv          ML6: segment sizes, countries, products
  figures/
    ml2_error_vs_k.png
    ml6_customer_map.png
docs/
  decision_log.md           R1: every decision with its reason and evidence
memo.pdf                    J2: one-page memo for the stakeholder
slides.pdf                  J3: presentation
```

## Key numbers
| Item | Value |
|---|---|
| Clean rows | 776,577 (from 1,067,371) |
| Matrix | 5,473 customers × 3,687 products, 413,927 non-zeros, 97.95% sparse |
| Purchase value | log(1 + total quantity); products bought by ≥ 10 customers |
| Evaluation | 1,913 customers, 44,727 held-out (customer, product) pairs |
| Rebuild error at k = 50 | SVD 0.790, PCA 0.789 (same scale) |
| Hit@10 | SVD 21.5%, PCA 21.0%, best-seller baseline 26.9% |

## Privacy (R4)
No customer IDs appear in the memo, the slides, any figure, or any file in `reports/`.
Customers are referred to by segment name or row index only.