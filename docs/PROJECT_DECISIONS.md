# Fintech Larper --- Project Decisions

This file is a concise record of decisions made while working through
`PRE_CODING_TODO.md`. Detailed research and experimentation belong
elsewhere.

## 1. Stock Basket

**Decision:** Use 5 individual U.S. stocks from different sectors, with
a market-cap mix of 2 large, 2 mid, and 1 small.

Ticker Company Sector Size

---

`NVDA` NVIDIA Information Technology Large
`FDS` FactSet Research Systems Financials Mid
`XOM` Exxon Mobil Energy Large
`HAE` Haemonetics Healthcare Mid
`JJSF` J&J Snack Foods Consumer Staples Small

Stocks were selected for sector/size diversity, adequate historical
data, and reasonable liquidity. Size labels refer to the approximate
classification at selection time (October 2026).

## 2. Historical Data Source

**Decision:** Use Yahoo Finance via `yfinance`.

Available OHLCV, adjusted close, dividends, and split data were
validated for all five stocks. Market data will be ingested, validated,
and persisted in PostgreSQL.

```text
Yahoo Finance → yfinance → ingestion → validation → PostgreSQL → ML pipeline
```

Corporate actions will be retained. Yahoo historical `Close` is
split-adjusted; `Adj Close` additionally reflects distributions such as
dividends.

## 3. Forecast Target

**Decision:** Forecast daily `Close` for each stock.

- Predict the complete trajectory from `t+1` through `t+20`.
- Evaluate particularly at `t+1`, `t+5`, and `t+20`.
- Horizons refer to U.S. market trading sessions.
- Keep `Close` as the forecasting target across V1, V2, and V3.
- Retain predictive uncertainty where supported; exact uncertainty
  evaluation is Decision #11.

## 4. Model Inputs

**V1 --- OHLC**

Historical Open, High, Low, and Close are available to the model.

**V2 --- OHLCV**

V1 plus historical Volume.

**V3 --- richer/model-appropriate information**

OHLCV plus more complex information or regressors appropriate to each
model. Exact V3 inputs do not need to be identical across model families
and will be selected later.

The forecasting target remains future `Close` in every version.

## 5. Baselines

**Decision:** Use naive/random walk, drift, and ARIMA.

All baselines will use the same walk-forward evaluation framework and
forecast horizons as the main models.

## 6. General-Purpose Foundation Model

**Decision:** Use **TimesFM-3**.

- Frozen pretrained model; no fine-tuning.
- Same checkpoint across V1, V2, and V3.
- V1 uses OHLC.
- V2 uses OHLCV.
- V3 may use richer supported covariates.
- Forecast the complete trajectory through `t+20`.
- Retain native probabilistic/quantile forecasts.
- Pretrained weights have non-commercial usage restrictions;
  acceptable for the current educational/research project.

## 7. Finance-Specific Foundation Model

**Decision:** Use **Kronos**.

Kronos will be evaluated in two modes:

1.  **Zero-shot:** use the pretrained finance-specific model without
    weight updates.
2.  **Transfer learning:** fine-tune/adapt the pretrained model on the
    target-stock training data and compare against its zero-shot
    performance.

Input progression:

- **V1:** OHLC.
- **V2:** OHLCV.
- **V3:** OHLCV plus richer/model-appropriate information through an
  appropriate adaptation strategy.

Only future `Close` is evaluated, even if Kronos internally forecasts
additional K-line fields.

The exact Kronos checkpoint size, fine-tuning strategy, and V3
adaptation design are not locked yet. The project's own leakage-safe
training procedure will be used rather than blindly relying on the
official CSV fine-tuning pipeline.

## 8. Custom Models

**Decision:** Use **XGBoost** and **N-HiTS** as the two custom models.

Both models will be trained without external pretrained weights.

### XGBoost

XGBoost will represent the conventional supervised/tabular ML approach.

- Uses explicitly engineered temporal features.
- Predicts the future `Close` trajectory through `t+20`.
- Supports the V1 → V2 → V3 information progression.

### N-HiTS

N-HiTS will represent the neural forecasting approach trained from scratch.

- Trained from random initialization using NeuralForecast.
- Directly predicts the future `Close` trajectory through `t+20`.
- Supports historical, future, and static exogenous variables for the V1 → V2 → V3 progression.

LightGBM and LSTM are retained as fallback alternatives but are not part of the primary experiment.

The exact feature engineering, internal target representation, training scope, normalization, lookback lengths, and training procedure will be decided during the experiment-design stage.

## 9. Walk-Forward Evaluation

**Decision:** Use an expanding-window walk-forward evaluation with 20-trading-session forecast blocks.

### Timeline

- **Historical start:** First U.S. trading session of 2016.
- **Initial training period:** 2016–2021.
- **Walk-forward evaluation period:** First trading session of 2022 through October 2, 2026.
- **Walk-forward step:** 20 trading sessions.

### Procedure

At each forecast origin:

1. Use all available data from the fixed 2016 start through the current forecast origin.
2. Produce the complete `t+1...t+20` forecast trajectory.
3. Evaluate the forecast once the next 20 trading sessions are observed.
4. Append those 20 actual observations to the available history.
5. Refit or update trainable models as required.
6. Advance to the next 20-session forecast block and repeat.

The training window is **expanding**, meaning the historical start remains fixed at 2016 while newly observed data is added after each walk-forward block.

Performance will be evaluated across the complete `t+1...t+20` trajectory, with `t+1`, `t+5`, and `t+20` used as the primary reporting horizons.

### Information Availability

Each historical forecast must reproduce the information that would have been available at its forecast origin.

Historical regressors may only contain observations available up to that origin. Future regressor values must not be used unless they were genuinely known in advance at the time of forecasting.

The exact metrics, uncertainty evaluation, and model-selection procedure are handled in later experiment-design decisions.

## 10. Evaluation Metrics

_Not decided yet._

## 11. Uncertainty Experiment

_Not decided yet._

## 12. V1 Experiment Specification

_Not decided yet._
