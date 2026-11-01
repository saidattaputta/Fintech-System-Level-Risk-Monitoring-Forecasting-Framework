# System-Level Financial Risk Monitoring & Forecasting Framework

A notebook-based quantitative analysis of the PaySim mobile-money transaction simulator. The project investigates transaction-level balance consistency, constructs origin/destination user-time exposure records, builds a step-level composite risk index, and explores risk-state persistence and next-row risk-index regression.

This is a research and analytical prototype using synthetic transaction data—not a deployed bank monitoring or fraud-detection system.

## Research Question

Can account-balance movements and transaction-level signals be aggregated into interpretable monitoring indicators, and do the resulting risk-index observations contain information about subsequent risk-index values?

## Dataset and Scale

- **Dataset:** PaySim (Kaggle dataset ID: ealaxi/paysim1), downloaded with KaggleHub
- **Raw transaction rows:** 6,362,620
- **Raw columns:** 11
- **Analysis sample used in the risk-intensity notebook:** 418,269 user-time exposure rows
- **Source fields:** transaction type, amount, origin/destination balances, simulation step, fraud label, and flagged-fraud indicator

The notebooks use Kaggle input paths and intermediate Parquet files to pass data between stages. Those intermediate files are not committed in this GitHub repository.

## Results and Evidence

The figures below are taken from saved notebook outputs. They were not regenerated during this README update.

### 1. Transaction validation and exposure construction

Notebook: 01_system_risk_signal_engineering.ipynb

| Diagnostic | Saved result |
|---|---:|
| Transaction rows examined | 6,362,620 |
| Selected analysis columns | 8 |
| Missing values in the selected 8-column frame | 0 |
| Duplicate rows before removal | 543 |
| Duplicate rows after the notebook's removal step | 0 |
| Origin balance arithmetic mismatches | 5,413,997 (85.09%) |
| Destination balance arithmetic mismatches | 4,736,674 (74.45%) |
| Rows where amount exceeds origin opening balance | 4,079,080 (64.11%) |

These are data diagnostics, not fraud labels. A balance equation mismatch does not by itself establish fraud: transaction type, simulator conventions, and balance-update behavior can affect arithmetic reconciliation. The notebook compares mismatch counts with the fraud label, then reconstructs a unified origin/destination user-time exposure table.

The origin/destination role analysis reports 6,353,307 unique origin identifiers and 2,722,362 unique destination identifiers, with 1,769 identifiers appearing in both roles. The notebook then stacks origin and destination activity into a unified user-time exposure view.

### 2. Step-level risk intensity

Notebook: 02_risk_intensity_forecasting_model.ipynb

| Measure | Saved result |
|---|---:|
| User-time development sample | 418,269 rows |
| Rows with drawdown-style ratio above 20% | 4.53% |
| Average step-level high-drawdown rate after minimum-volume filter | 4.50% |
| Average step-level fraud rate after minimum-volume filter | 0.1311% |
| Step groups retained after requiring at least 100 observations | 317 |

The notebook builds a step-level table containing transaction count, high-drawdown rate, fraud rate, and average drawdown-style ratio. It then constructs a composite index by adding drawdown-rate and fraud-rate amplitudes relative to their respective averages.

The five highest saved composite-risk-index rows are:

| Simulation step | Transactions | Drawdown rate | Fraud rate | Composite risk index |
|---:|---:|---:|---:|---:|
| 25 | 105 | 5.71% | 1.90% | 15.803 |
| 617 | 172 | 6.40% | 1.74% | 14.729 |
| 428 | 176 | 4.55% | 1.70% | 14.016 |
| 670 | 137 | 4.38% | 1.46% | 12.112 |
| 659 | 174 | 8.05% | 1.15% | 10.559 |

The index is an exploratory, label-informed heuristic: fraud rate is one of its explicit components. It should not be described as an unsupervised risk score or as evidence of a causal relationship.

**Output-label discrepancy:** the notebook's printed descriptions for the two fraud-rate subsets are reversed relative to the Boolean masks. The code's greater-than-0.20 drawdown subset has a saved fraud rate of 0.00%; the less-than-or-equal-to-0.20 subset has a saved fraud rate of approximately 0.137%. The masks—not the printed labels—define these comparisons.

### 3. Risk-state and regression exploration

Notebook: 03_shock_decay_analysis_and_monitoring_framework.ipynb

The notebook loads the 317-row risk-index table, defines the 85th-percentile threshold as **2.8704**, and derives a next-row elevated-risk label. After dropping the final row, the saved label distribution is:

| Next-row label | Rows |
|---|---:|
| Not elevated | 269 |
| Elevated | 47 |
| Total | 316 |

The first linear regression experiment reports:

| Metric | Training | Test |
|---|---:|---:|
| R² | 0.9963 | 0.8385 |

| Test metric | Saved value |
|---|---:|
| MAE | 0.06466 |
| RMSE | 0.07625 |

A second experiment adds three lag features and a risk-change feature:

| Metric | Training | Test |
|---|---:|---:|
| R² | 0.9964 | 0.9092 |

| Test metric | Saved value |
|---|---:|
| MAE | 0.04858 |
| RMSE | 0.05706 |

**Critical forecasting caveat:** notebook 2 sorts the risk-index table in descending risk-index order before saving it. Notebook 3 then applies shift operations, creates lag features, and splits the sorted rows by position. The notebook also reports zero transitions into the elevated-risk state, and its 95-row test partition contains no positive next-risk labels. Therefore, these regression metrics describe an index-sorted sequence, not a validated chronological forecast of future financial risk. Treat them as exploratory regression diagnostics only.

The same notebook reports that the sorted risk-index values fall below half of the selected peak after 16 index positions. Because the sequence is risk-ranked rather than time-ordered, this is an index-position decay measure, not a real-time shock half-life.

## Methodology and Architecture

The three-stage workflow is:

1. **Signal engineering:** inspect raw transaction data, check balance consistency, compare discrepancies with fraud labels, and construct a unified user-time exposure view.
2. **Risk intensity:** derive a drawdown-style feature, aggregate risk statistics by simulation step, and construct a composite risk-index heuristic.
3. **Persistence and regression exploration:** threshold the risk index, create shifted labels and lag features, run linear regression experiments, and inspect index-position decay.

## Notebook Guide

| Notebook | Responsibility |
|---|---|
| 01_system_risk_signal_engineering.ipynb | Dataset checks, balance diagnostics, origin/destination role analysis, and unified user-time exposure construction |
| 02_risk_intensity_forecasting_model.ipynb | Drawdown-style measure, step-level aggregation, composite risk index, and high-risk table export |
| 03_shock_decay_analysis_and_monitoring_framework.ipynb | Risk threshold labels, lag features, regression metrics, and index-position decay analysis |

## Technology

Python, pandas, NumPy, scikit-learn, PyArrow, Jupyter Notebook, and KaggleHub.

## Reproducing the Analysis

The notebooks were developed around Kaggle dataset and notebook outputs. To reproduce them:

1. Obtain access to the PaySim dataset through Kaggle.
2. Install the packages imported by the notebooks.
3. Run notebook 1 to create the unified user-time exposure data.
4. Export or adapt the intermediate Parquet inputs expected by notebook 2.
5. Run notebook 2 to build the step-level risk table.
6. Supply the resulting Parquet table to notebook 3.

Several notebook cells refer to Kaggle-specific input paths and intermediate artifacts not committed here. Update those paths locally; the repository alone does not contain every intermediate input needed for a sequential rerun.

## Repository Contents

- 01_system_risk_signal_engineering.ipynb
- 02_risk_intensity_forecasting_model.ipynb
- 03_shock_decay_analysis_and_monitoring_framework.ipynb
- README.md

## Limitations and Research Caveats

- PaySim is simulated mobile-money data; findings may not generalize to real financial institutions.
- Balance mismatches and amount-to-balance checks are diagnostics, not proof of fraud.
- The composite risk index uses the fraud label in its calculation and is therefore not an independent unsupervised early-warning score.
- The risk-index table is sorted by score before the third notebook shifts rows; its current lag features are not based on chronological time.
- The third notebook's test partition contains no positive next-risk labels, limiting any classification interpretation.
- The regression scores require chronological, leakage-safe validation and baseline comparisons before they can support forecasting claims.
- The reported 16-position decay result is not a time-based half-life.

## Recommended Next Research Steps

Rebuild the risk-index sequence in chronological step order, separate predictive features from fraud labels, evaluate with a time-based holdout, compare against persistence/naïve baselines, and report classification metrics only where the test set contains both classes.

---

**Sai Datta Putta**

[LinkedIn](https://www.linkedin.com/in/sai-datta-putta/)
