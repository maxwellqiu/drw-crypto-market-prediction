# DRW – Crypto Market Prediction (Baseline)

A reproducible baseline solution for Kaggle community competition **“DRW - Crypto Market Prediction”**.
This repo is designed for **interview practice**: clear problem framing, clean pipeline, and reproducible submissions.

---

## Problem

Given minute-level crypto market data (public market stats + anonymized proprietary features), build a model to predict the **future price movement signal** `label`.

**Metric:** Pearson correlation between true labels and predicted values on the private test set.

---

## Dataset Summary

Files (from Kaggle competition data):

- `train.parquet`: features + `label`
- `test.parquet`: features (labels set to 0), **timestamps are masked/shuffled** to prevent “future peeking”
- `sample_submission.csv`: required submission format

Key columns:
- `timestamp`: minute index (train is real time; test is masked/shuffled)
- `bid_qty`, `ask_qty`, `buy_qty`, `sell_qty`, `volume`: public microstructure/volume stats
- `X_1 ... X_780`: anonymized proprietary features
- `label`: target to predict

---

## Important Constraint (Why this baseline is “row-wise”)

In the **test set**, timestamps are masked and shuffled.  
Therefore this baseline **does NOT rely on time order** (no rolling windows, lags, or sequence features on test).  
Instead, it uses **row-wise features only** so train/test feature computation is consistent.

---

## Approach (Baseline)

### Features
- All original features (`bid_qty`, `ask_qty`, `buy_qty`, `sell_qty`, `volume`, and `X_*`)
- Additional **row-wise microstructure features**:
  - Bid/Ask imbalance
  - Trade imbalance (buy vs sell)
  - Volume-to-depth ratio
  - Buy/Sell ratios
  - Volume per trade

Missing values: median imputation.

### Model
- **LightGBM Regressor**
- Output is a continuous signal (good fit for Pearson correlation).

### Ensemble
- 3 different random seeds
- Average predictions
- Z-score normalization of predictions (does not change Pearson, but stabilizes scale)

---

## Results
One late submission produced:
- **Public Score:** 0.05778  
- **Private Score:** 0.04392  

(Leaderboard scores may vary by split/time.)
