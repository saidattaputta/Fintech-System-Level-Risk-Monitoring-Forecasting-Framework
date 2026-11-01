# System-Level Financial Risk Monitoring & Forecasting Framework

A notebook-based quantitative analysis using the PaySim mobile-money transaction dataset. The project moves from transaction-level data validation and user exposure feature construction to step-level risk-intensity analysis and exploratory forecasting of subsequent risk conditions.

## Research Objective

Explore whether transaction and account-balance behavior can be transformed into interpretable monitoring signals, and whether aggregated risk intensity contains information about future risk states.

This is an analytical research project using a synthetic financial transaction simulator, not a deployed bank fraud-detection or market-risk system.

## Data

The notebooks download the PaySim dataset through KaggleHub:

`ealaxi/paysim1`

The source records include transaction type and amount, origin/destination account balances, time step, and fraud labels.

## Research Pipeline

### 1. Transaction Signal Engineering

Notebook: `01_system_risk_signal_engineering.ipynb`

- Inspect transaction schema and data quality.
- Check missing values, duplicates, non-negative monetary/balance fields, and balance consistency.
- Compare balance inconsistencies with the fraud label.
- Construct a unified user-time exposure view from origin and destination roles.
- Derive rolling balance peaks and drawdown-style features for exploratory analysis.

### 2. Risk Intensity Analysis

Notebook: `02_risk_intensity_forecasting_model.ipynb`

- Load the derived development sample.
- Examine drawdown-style measures and fraud rates.
- Aggregate transaction count, high-drawdown rate, fraud rate, and average drawdown by simulation time step.
- Construct a composite step-level risk index from normalized drawdown and fraud-rate components.
- Export the resulting high-risk time-step table.

### 3. Risk Persistence and Forecasting Exploration

Notebook: `03_shock_decay_analysis_and_monitoring_framework.ipynb`

- Define elevated-risk states using a quantile-based threshold.
- Explore transitions into elevated-risk states.
- Build lagged risk-index features and examine next-step risk-index prediction using linear regression.
- Calculate regression metrics and inspect post-peak risk decay.

## Repository Contents

```text
.
├── 01_system_risk_signal_engineering.ipynb
├── 02_risk_intensity_forecasting_model.ipynb
├── 03_shock_decay_analysis_and_monitoring_framework.ipynb
├── .gitignore
└── README.md
```

## Technology

Python · pandas · NumPy · scikit-learn · PyArrow · Jupyter Notebook · KaggleHub

## Reproducing the Notebooks

The notebooks were developed around Kaggle datasets/notebook inputs and intermediate Parquet/Pickle artifacts. To reproduce:

1. Obtain access to the PaySim dataset through Kaggle.
2. Install the required Python packages used in the notebooks.
3. Run notebook 1 to construct the transaction/user-time features.
4. Export or adapt the intermediate data paths expected by notebook 2.
5. Run notebook 2 to create the step-level risk table.
6. Supply the resulting risk table to notebook 3 and run the forecasting/decay analysis.

Some notebook cells refer to Kaggle-specific input paths and generated intermediate artifacts that are not committed in this repository. Update those paths locally as needed.

## Important Methodological Notes

- PaySim is simulated mobile-money data; findings may not generalize to real financial institutions.
- The constructed composite risk index is an exploratory heuristic, not a validated regulatory or operational risk score.
- The notebooks use fraud labels in parts of the risk-index construction and analysis. This creates target leakage if the index is interpreted as a fully unsupervised early-warning signal.
- The third notebook explores time-step aggregates and linear regression; results require robust time-based validation and baseline comparison before being described as a reliable forecast.
- Rolling balance peaks and drawdown definitions should be validated against the intended account-level time ordering and duplicate user-step records.

## Author

Sai Datta Putta
