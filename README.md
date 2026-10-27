# Credit Portfolio Risk Simulation & Stress Testing

A portfolio-level credit risk model built on the Lending Club dataset (1.3M+ loans), combining probability-of-default (PD) modeling, Monte Carlo simulation of correlated defaults, and macroeconomic stress testing — the same core techniques used in bank credit risk and regulatory capital teams.

## Overview

Most public credit risk projects stop at predicting whether a single loan will default. This project goes a step further: it models what happens to an **entire loan portfolio** when defaults are correlated through a shared economic cycle, and quantifies how much capital a lender would need to hold to survive a severe recession.

**Key results:**

| Metric | Baseline Scenario | Stressed (Recession) Scenario |
|---|---|---|
| Expected Loss | $3.96B | $7.41B (+87%) |
| VaR (99%) | $9.73B | $13.74B (+41%) |
| Expected Shortfall | $10.61B | $14.43B |
| Economic Capital | $5.77B | — |

## Methodology

**1. Data & target definition**
- Source: Lending Club loan-level data (2.26M loans, filtered to 1.3M with a known final outcome)
- Target: binary default flag (`Charged Off` / `Default` = 1, `Fully Paid` = 0)
- Overall historical default rate: ~20.1%

**2. Probability of Default (PD) estimation**
- *Baseline approach:* empirical default rate by Lending Club grade (A–G), which produced a clean, monotonically increasing PD curve from 6.1% (grade A) to 50.1% (grade G) — a useful sanity check that the grading system is well-calibrated.
- *Upgraded approach:* logistic regression using loan amount, interest rate, income, DTI, term, grade, home ownership, purpose, and employment length.
- The logistic regression improved discriminatory power from **AUC 0.678 → 0.703** (Gini 0.357 → 0.405), a meaningful lift for loan-level risk ranking, even though portfolio-level totals were similar between the two approaches — highlighting the difference between aggregate risk and individual-loan discrimination.

**3. Correlated default simulation (Vasicek single-factor model)**
- Rather than treating defaults as independent, each loan's PD is adjusted by a shared systematic risk factor (Z), representing the state of the broader economy, using an asset correlation of ρ = 0.15.
- 10,000 Monte Carlo simulations were run, each drawing a random economic state and simulating defaults across the full portfolio, producing a full portfolio loss distribution rather than a single point estimate.

**4. Risk metrics**
- **Expected Loss (EL):** average simulated loss — the loss level a lender should "price in"
- **Value at Risk (VaR 99%):** the loss level exceeded only 1% of the time
- **Expected Shortfall (ES):** average loss in the worst 1% of scenarios — captures tail severity beyond VaR
- **Economic Capital:** VaR minus Expected Loss — the buffer needed above expected losses to survive a 99th-percentile bad year

**5. Stress testing**
- The economic factor's distribution was shifted to represent a severe recession (mean shifted to −1.5 standard deviations), consistent with the scenario-shock approach used in regulatory stress tests (e.g. CCAR).
- Result: Expected Loss nearly doubled (+87%) while VaR rose more moderately (+41%) — because a shared downturn compresses previously "tail" outcomes into the "typical" range, rather than uniformly scaling all losses.



## Key Assumptions & Limitations

- Exposure at default is approximated using original loan amount (a simplification — some principal may already be repaid in practice)
- Asset correlation (ρ = 0.15) and the stress shock magnitude (−1.5σ) are illustrative, standard-range assumptions rather than fitted to a specific historical crisis
- The logistic regression uses a straightforward feature set; further gains are possible with more granular features or gradient boosting, at some cost to interpretability

## Tools Used

Python, pandas, NumPy, SciPy, scikit-learn, Matplotlib

## What This Project Demonstrates

- End-to-end credit risk pipeline: from raw loan data to portfolio-level capital estimates
- Understanding of correlated default modeling (Vasicek / single-factor framework), used in Basel IRB capital formulas
- Practical model risk thinking: comparing a simple baseline against a more sophisticated model and correctly interpreting where the improvement does and doesn't matter
- Stress testing methodology consistent with regulatory frameworks (CCAR-style scenario shocks)
