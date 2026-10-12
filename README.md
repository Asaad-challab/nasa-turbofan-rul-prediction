# Turbofan Remaining Useful Life Prediction — NASA C-MAPSS FD001

A reproducible machine-learning baseline for **remaining useful life (RUL)** prediction from simulated turbofan engine sensor data. The notebook investigates whether causal, short-window temporal features improve conventional Random Forest regression, and audits prediction overestimation relevant to maintenance decision-making.

## Dataset

This project uses the **FD001** subset of NASA's simulated C-MAPSS turbofan dataset: 100 complete run-to-failure training trajectories (20,631 observations) and 100 truncated test trajectories (13,096 observations). Each row contains an engine identifier, operating cycle, three operating settings, and 21 sensor measurements. The official `RUL_FD001.txt` provides one ground-truth RUL value per test engine at its final observed cycle.

**Data are not included in this repository.** In Kaggle, attach the dataset [`behrad3d/nasa-cmaps`](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps) so files are available under `/kaggle/input/datasets/behrad3d/nasa-cmaps/CMaps/`. If Kaggle mounts it at a different path, update the notebook's `DATA_DIR` variable.

## Method

1. Compute training labels as `RUL = final_cycle_for_engine - current_cycle`.
2. Split complete training engines into disjoint 80% development-training and 20% validation groups (`GroupShuffleSplit`, seed 42).
3. Compare Linear Regression and Random Forest using four instantaneous sensor features: `sensor_2`, `sensor_3`, `sensor_4`, `sensor_11`.
4. Evaluate two trailing features for Sensor 11: five-cycle rolling average and the difference from four cycles earlier. Features use only current/past sensor readings; initial missing differences are set to zero.
5. Freeze the six-feature Random Forest (`n_estimators=100`, `max_depth=10`, `min_samples_leaf=5`, `random_state=42`), retrain on all 100 original training engines, and predict at the last observed cycle of each of the 100 independent test engines.
6. Audit signed error (predicted RUL minus actual RUL) and frequency of overestimation.

## Results

The following scores were reported from the Kaggle experiment; run the notebook to recalculate them in your environment.

| Evaluation | MAE (cycles) | RMSE (cycles) | R² |
| --- | ---: | ---: | ---: |
| Validation: Linear Regression, four sensors | 32.070 | 40.927 | 0.611 |
| Validation: Random Forest, four sensors | 31.245 | 41.090 | 0.608 |
| Validation: RF + trailing mean | 30.132 | 39.959 | 0.630 |
| Validation: RF + trailing mean + recent change | **30.003** | **39.832** | **0.632** |
| **Official test: selected RF, one prediction per test engine** | **24.237** | **32.821** | **0.376** |

**Test error audit:** mean signed bias **+20.527 cycles**; RUL overestimated for **79%** of test engines; maximum overestimation **+105.863 cycles**. The test and validation rows are not directly comparable: validation metrics aggregate all observations in complete validation trajectories, while official test metrics use a single truncated endpoint per test engine.

## Reproduce

1. Open [`notebooks/FD001_RUL_Baseline.ipynb`](notebooks/FD001_RUL_Baseline.ipynb) in Kaggle or Jupyter.
2. Attach/download the referenced C-MAPSS dataset and set `DATA_DIR` accordingly.
3. Run all cells in order. Dependencies are in [`requirements.txt`](requirements.txt). The notebook uses Kaggle's standard scientific Python packages.

## Limitations

- FD001 is a **simulation dataset**, not field measurements from operational fleets.
- The six-feature approach is a baseline, not a state-of-the-art benchmark.
- Feature development used one engine-disjoint split; no repeated cross-validation or uncertainty intervals were established.
- The selected model overestimates remaining life frequently, which would require careful mitigation in maintenance use cases.
- Official test labels were used for evaluation, **not for fitting or further model tuning**.
- This repository does not establish deployability or safety guarantees.

## Repository contents

```text
nasa-turbofan-rul-prediction/
├── README.md
├── requirements.txt
├── .gitignore
└── notebooks/
    └── FD001_RUL_Baseline.ipynb
```
