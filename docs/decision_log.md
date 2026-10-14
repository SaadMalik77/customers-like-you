Date | Decision | Reason | Evidence
Date | Decision | Reason | Evidence
--- | --- | --- | ---
Day 1 | Load and stack both sheets | Two years needed for history and time split | 1,067,371 rows; identical columns
Day 1 | Read Invoice, StockCode, Description, Customer ID as strings | IDs are labels; avoids int/str mixing and float IDs | `dtype=` in `read_excel`
Day 1 | Add pyarrow, save data as Parquet | Keeps dtypes (CSV re-guesses them); faster than Excel | requirements.txt; round-trip asserts
Day 2 | Normalise StockCode (strip, uppercase) | "85123a" and "85123A" are the same product | `clean_data`
Day 2 | Rule order: duplicates, cancellations, missing ID, qty, price, non-product | Rules overlap; specific rules first so the log credits the right reason | Cleaning log
Day 2 | Drop exact duplicates (original 8 columns) | Double-recorded lines | -34,335
Day 2 | Drop cancellations ("C" invoices) | Not purchases | -19,104
Day 2 | Known limitation: original lines of cancelled orders kept | Matching them is out of scope; some returns count as purchases (memo risk) | —
Day 2 | Drop missing Customer ID | Can't place in customer x product matrix | -234,437 (23.1%)
Day 2 | Keep Qty <= 0 rule despite 0 removals | Already removed by earlier rules; kept as a safeguard | Log row = 0
Day 2 | Drop Price <= 0 | Free or adjustment lines | -70
Day 2 | Drop non-product codes (regex `^\d{5}`) | Postage, fees, adjustments, vouchers, tests | -2,848; 61 codes reviewed
Day 2 | Accept loss of rare DCGS/SP items | Real but rare (<= 31 raw rows); below product filter anyway | Code review table
Day 2 | Keep missing Descriptions and extreme quantities | Description unused; global outlier cut would leak test data (R2); log1p handles big orders | 776,577 clean rows
Day 3 | Test = last 2 months; cutoff = max date - 2 months | Brief requirement; computed, not hard-coded | Cutoff 2011-10-09 12:50
Day 3 | Drop cold-start test customers | No matrix row; keeps model and baseline on same customers | -378 of 2,420
Day 3 | Held-out = new products only, unique pairs, seen in train | Brief requirement; unseen products are guaranteed misses | 45,753 pairs; 1,916 customers
Day 3 | No test-period rows in train | Prevents leakage (R2) | R2 asserts pass
Day 3 | Purchase value = log1p(total quantity), aggregated first | Limits big-buyer dominance; aggregating avoids summing logs | `build_matrix`
Day 3 | Keep products with >= 10 customers (train only) | Too few buyers to learn from | 4,529 -> 3,687 products; 1 customer lost
Day 3 | Sorted index maps | Same row/column order every run (R6) | Index parquet files
Day 3 | Re-align test pairs to matrix | Can only evaluate known customers/products | 44,727 pairs; 1,913 eval customers
Day 4 | MB = bytes / 1e6; sparse size via `.nbytes` | Matches brief's example; index arrays use 4 bytes | Dense 161.43 MB, sparse 4.99 MB
Day 4 | Memo framing: storage win is vs dense, not vs sparse | k=50 factors (3.66 MB) barely smaller than sparse (4.99 MB) | Matrix report
Day 5 | ML1: Main model = Truncated SVD | A is 97.95% sparse (413,927 nnz, 4.99 MB); SVD works on it natively, while PCA centring makes it dense (161 MB). One fit gives product factors (Vᵀ) and customer factors (U·S) for the recommender. Component 1 = volume/popularity, useful in Q4. | ml.ipynb cell 1 (ML1 note); commit "ML1: method decision before fitting"
Day 5 | ML5: Comparison model = PCA (svd_solver="arpack"), same k and matrix | Brief requires the other method on the same matrix; arpack centres implicitly, so the dense matrix is never built. Tests the prediction: SVD wins hit@10/time/memory, PCA gives the cleaner customer map. | ml.ipynb ML5 section; PCA vs SVD table
Day 6 | ML4: baseline popularity = number of distinct train customers per product | Not units: a single wholesaler's bulk order shouldn't define a best seller; train only (R2) | ml.ipynb ML4 Chunk 4
Day 6| ML4: score customers in batches of 500 | Avoids building the 161 MB dense score matrix in one go | ml.ipynb ML4 Chunk 3
Day 6 | ML6: map on PCA comps 2-3, customers scaled to length 1; 3 groups | Comps 1-2 grouped by size (88/12/0.2%, all lifts < 1 in main group); comp 1 = size (corr 0.876). Scaling removes volume. 3 = best silhouette (0.548) | reports/figures/ml6_customer_map.png; reports/ml6_segments.csv