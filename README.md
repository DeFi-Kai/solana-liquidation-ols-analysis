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
---
# SQL Workflow and Dataset Construction

All preprocessing was performed on Dune Analytics using raw Solana on-chain tables and custom decoding logic for Kamino Lend and Jupiter Lend.

## Base Tables

Three foundational tables were created:

### 1. Jupiter Lend Liquidations
- Decoded all liquidation events using `operate` instructions, discriminator `d96ad06374972a87`, and i128 debt and collateral fields.

### 2. Jupiter Lend Flash Loans
- Captured all internal flash-loan events *(not used directly in this OLS analysis).*

### 3. Kamino Lend Liquidations and Flash Loans
- The Kamino dataset originates from the pre-decoded table published by the Kamino team:
- `kamino_lend_solana.kamino_lending_call_liquidateobligationandredeemreservecollateralv2`

This table provides decoded instruction-level fields for each liquidation, including:
- liquidityAmount (raw debt repaid)
- call_data
- call_inner_instructions

Additional steps were performed to prepare the data for empirical analysis:
- Converted liquidityAmount to USD values using token metadata and hourly pricing.
- Mapped debt USD, symbols, and lending market, by extracting mint addresses from inner instructions.

## Combined Hourly Liquidation Table
A unified hourly dataset was constructed with the following schema:
```
hour
kamino_debt_repaid_usd_hour_scaled
jup_debt_repaid_usd_hour
total_debt_repaid_usd_hour_scaled
sol_price
```
This table merges Kamino and Jupiter liquidation flows into a single, time-aligned series used for the OLS regressions.

### Kamino Scaling Factor
Kamino’s raw liquidation notional (from liquidityAmount via Dune) overreported aggregate October 10 activity (23.5M debt repaid) relative to protocol-reported totals (19.9M debt repaid). To harmonize datasets, a scaling factor is applied:

```
scaling_factor = 19.9 / 23.5
```
This adjustment preserves the temporal structure of the decoded data while matching the correct order of magnitude.


## Liquidation Intensity Proxy

Debt repaid per hour is used as the proxy for liquidation intensity.


