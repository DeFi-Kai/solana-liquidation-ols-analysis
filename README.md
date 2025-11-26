# solana-liquidation-ols-analysis

This repository provides two analysis-ready datasets and a minimal Python workflow for estimating OLS regressions that test how liquidation activity relates to short-term volatility on Solana. All preprocessing was performed on Dune Analytics using raw on-chain data and reconstructed liquidation flows from Kamino Lend and Jupiter Lend.

## Dataset Overview

### 1. SOL Volatility Regression Dataset

Used to estimate the effect of system-wide liquidations on SOL volatility.
- hour – UTC timestamp
- price – SOL hourly price
- log_return – hourly log return
- vol_abs – absolute return
- vol_sq – squared return (realized volatility estimator)
- total_debt_repaid_usd_hour – total liquidations (Kamino + Jup Lend)
- sol_volume_usd_hour – SOL trading volume (hourly)

### 2. JLP Volatility Panel Dataset

Used to estimate the effect of system-wide liquidations on JLP volatility.
- hour – UTC timestamp
- price – JLP hourly price
- log_return – hourly log return
- vol_abs – absolute return
- vol_sq – squared return (realized volatility estimator)
- total_debt_repaid_usd_hour – total liquidations (Kamino + Jup Lend)
- sol_volume_usd_hour – JLP trading volume (hourly)

## Contents

```
/data
    sol_regression.csv      # SOL volatility dataset
    jlp_regression.csv      # JLP volatility panel

/code
    ols_sol.py              # OLS for SOL dataset
    ols_jlp.py              # OLS for JLP dataset
```

## Methodology
Each dataset is used to estimate an OLS model of the form:

`volatility_t = α + β * liquidations_t + γ * volume_t + ε_t`

Where `volatility_t` is either:
- vol_abs (absolute returns)
- vol_sq (squared returns)

and `liquidations_t` is:
- total_debt_repaid_usd_hour

Control variables:
- sol_volume_usd_hour for the SOL regression
- jlp_volume_usd_hour for the JLP regression


## Running the Models
```
python code/ols_sol.py
python code/ols_jlp.py
```


