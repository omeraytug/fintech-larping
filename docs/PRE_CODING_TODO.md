# Fintech Larper --- Pre-Coding TODO

This document tracks the remaining research and decisions required
before implementing the main forecasting pipeline.

## 1. Choose the stock basket

- [x] Choose basket size.
- [x] Select stocks.
- [x] Define sector and market-cap diversity.

**Decision:** NVDA, FDS, XOM, HAE, JJSF

## 2. Choose the historical data source

- [x] Choose data source.
- [x] Validate historical coverage and required fields.
- [x] Decide how corporate actions are handled.
- [x] Decide where fetched data is stored.

**Decision:** Yahoo Finance via `yfinance`, persisted in PostgreSQL.

## 3. Define exactly what is being forecast

- [x] Choose the target.
- [x] Confirm `t+1`, `t+5`, and `t+20` evaluation horizons.
- [x] Confirm full `t+1...t+20` trajectory forecasting.
- [x] Define horizons as trading sessions.

**Decision:** Future daily `Close`.

## 4. Define model information levels

- [x] Define V1.
- [x] Define V2.
- [x] Define the purpose of V3.
- [x] Keep future `Close` as the target across all versions.

**V1:** OHLC\
**V2:** OHLCV\
**V3:** OHLCV + richer/model-appropriate information; exact regressors
remain open.

## 5. Choose the baselines

- [x] Naive/random walk.
- [x] Drift.
- [x] ARIMA.

## 6. Choose the general-purpose foundation model

- [x] Select model.
- [x] Decide whether it is zero-shot or fine-tuned.
- [x] Confirm multi-horizon forecasting.
- [x] Confirm V1/V2/V3 direction.

**Decision:** TimesFM-3, frozen/zero-shot.

## 7. Choose the finance-specific foundation model

- [x] Select finance-specific foundation model.
- [x] Confirm zero-shot forecasting.
- [x] Confirm transfer-learning/fine-tuning path.
- [x] Confirm V1 = OHLC.
- [x] Confirm V2 = OHLCV.
- [x] Confirm future `Close` remains the evaluated target.
- [ ] Choose Kronos checkpoint/size.
- [ ] Choose fine-tuning strategy.
- [ ] Define leakage-safe fine-tuning procedure.
- [ ] Define V3 adaptation approach when V3 is implemented.

**Decision:** Kronos.

**Experiment:** Compare pretrained zero-shot Kronos against Kronos
adapted/fine-tuned on the target-stock training data.

## 8. Choose the custom-model direction

- [x] Decide whether to use one or two custom models.
- [x] Choose the tree/tabular model.
- [x] Choose the neural forecasting model.
- [x] Confirm both models can produce the `t+1...t+20` forecast trajectory.
- [x] Confirm compatibility with the V1 → V2 → V3 progression.

**Selected custom models:**

- **XGBoost** — engineered features + supervised tree learning.
- **N-HiTS** — neural forecasting trained from random initialization.

**Fallbacks:** LightGBM and LSTM.

Detailed training and evaluation choices are deferred to the experiment-design decisions.

## 9. Define the walk-forward evaluation

- [x] Choose the historical start date.
- [x] Choose the initial training period.
- [x] Choose the walk-forward evaluation period.
- [x] Choose the walk-forward step size.
- [x] Choose expanding vs. rolling training windows.
- [x] Define the forecast trajectory at each origin.
- [x] Define the primary evaluation horizons.
- [x] Establish the no-future-information rule.

**Selected setup:**

- Historical start: **2016**
- Initial training: **2016–2021**
- Walk-forward evaluation: **2022–October 2, 2026**
- Step: **20 trading sessions**
- Window: **expanding**
- Forecast: **full `t+1...t+20` trajectory**
- Primary horizons: **`t+1`, `t+5`, `t+20`**

## 10. Choose evaluation metrics

### Point forecasts

- [ ] Choose point metrics.
- [ ] Decide how to compare stocks with different price scales.
- [ ] Decide reporting at `t+1`, `t+5`, and `t+20`.
- [ ] Decide aggregation across stocks.

### Probabilistic forecasts

- [ ] Choose probabilistic/calibration metrics.
- [ ] Decide how interval width/sharpness is evaluated.

**Selected metrics:** TBD

## 11. Define the uncertainty experiment

- [ ] Choose uncertainty representation.
- [ ] Choose interval/quantile levels.
- [ ] Decide how point-only models are handled.
- [ ] Define coverage/calibration evaluation.
- [ ] Define interval-width evaluation.

**Uncertainty setup:** TBD

## 12. Write the V1 experiment specification

- [ ] Fill in final date ranges.
- [ ] Add the selected custom model.
- [ ] Add the final backtesting design.
- [ ] Add point and probabilistic metrics.
- [ ] Review for look-ahead leakage.
- [ ] Confirm fair model comparison.
- [ ] Commit the specification before main modeling experiments.

```text
Assets: NVDA, FDS, XOM, HAE, JJSF
Data source: Yahoo Finance / yfinance
Date range: TBD

Target: Close
Forecast: full t+1...t+20 trajectory
Key horizons: t+1, t+5, t+20

V1 inputs: OHLC

Baselines: Naive/random walk, Drift, ARIMA
General foundation model: TimesFM-3 (zero-shot)
Finance foundation model: Kronos (zero-shot + fine-tuned)
Custom model: TBD

Backtesting method: TBD
Point metrics: TBD
Probabilistic metrics: TBD
```

## Not deciding yet

- MLflow architecture
- Docker image design
- Cloud provider/infrastructure
- Terraform structure
- Kubernetes architecture
- Helm
- Argo CD
- Argo Workflows
- Production deployment architecture
- Model registry strategy
- Continuous-training cadence
- Drift-triggered retraining

These will be introduced when the project creates a real reason to use
them.

## Immediate order of work

```text
Choose custom model (#8)
        ↓
Design walk-forward evaluation (#9)
        ↓
Choose metrics (#10)
        ↓
Define uncertainty experiment (#11)
        ↓
Write V1 experiment specification (#12)
        ↓
Start implementation
```

**Next task:** Choose the custom model for Decision #8.
