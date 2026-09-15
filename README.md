# GLM-Based Insurance Pricing Model

> **Language:** Python · **Methods:** Generalised Linear Models (Poisson, Gamma), Gini, Risk Ratio · **Domain:** Non-life Insurance Pricing (Actuarial Science)

## Overview

This project develops a **multiplicative Generalised Linear Model (GLM)** to price business travel insurance policies. The model is structured as a pure premium framework that separates risk into two independent components:

$$\text{price} = \gamma_0 \prod_{k=1}^{M} \gamma_{k,i}$$

where $\gamma_0$ is the base premium and $\gamma_{k,i}$ is the relativty for variable $k$, group $i$. This approach, standard in the non-life insurance industry, provides transparent, auditable, and regulatorily interpretable pricing.

The dataset covers business travel insurance for corporate clients, including claims from 2018–2022 (~149k training contracts, ~79k validation and evaluation records).

## Business Context

The insurance product covers employees travelling for business:

- **Covered losses:** Lost luggage, travel delays, medical expenses, cancellations
- **Client type:** Corporate policyholders (B2B)
- **Key challenge:** Risk varies significantly with company characteristics — modelling must capture non-linear risk relationships while remaining statistically stable

## Methodology

### 1 — Exploratory Data Analysis

- Distribution analysis of claim counts and average claim costs
- Risk trend analysis by year (frequency, severity, yearly claim cost)
- Variable profiling: `NumberOfPersons`, `CompanyAge`, `FinancialRating`, `TravellingArea`, `Activity`, `DangerousAreas`

### 2 — Variable Grouping

Continuous variables are binned into **risk-homogeneous groups** using domain knowledge and statistical criteria:

| Variable | Rationale |
|----------|-----------|
| `NumberOfPersons` | Non-linear risk — very small and very large companies behave differently |
| `CompanyAge` | New companies and mature companies have distinct risk profiles |
| `FinancialRating` | Credit quality proxies for operational stability and claim-filing behaviour |
| `TravellingArea` | Geographic risk exposure (Nordics vs. World) |
| `Activity` | Industry sector affects travel frequency and risk type |
| `DangerousAreas` | Binary coverage decision with a strong pricing impact |

### 3 — GLM Modelling

Three model versions are developed and compared:

| Model | Features | Notes |
|-------|----------|-------|
| Model 1 | All 6 features, original groupings | Full baseline |
| Model 2 | 4 features (removed `TravellingArea`, `Activity`) | Reduced — tests feature importance |
| Model 3 | All 6 features, merged groups for `FinancialRating` and `CompanyAge` | Addresses sparsity and instability |

For each model, **separate GLMs** are fitted for:
- **Claim Frequency** — Poisson distribution with log link, exposure = contract duration
- **Claim Severity** — Gamma distribution with log link, weights = number of claims

The combined pure premium is: **frequency × severity × base level**.

### 4 — Model Validation

#### Statistical Tests
- **Likelihood-Ratio Test (LRT):** Tests whether adding/removing a feature significantly improves model fit.
- **AIC / BIC Stepwise Selection:** Automated feature selection to identify the optimal model under information criteria.
- **Wald Test:** Per-coefficient significance test for individual factor levels.

#### Business Tests
- **Gini Score:** Measures the model's ability to discriminate high-risk from low-risk policies (similar to AUC in classification). Computed via the Lorenz curve.
- **Risk Ratio (RR):** Ratio of observed to predicted claims — should be close to 1.0 for a well-calibrated model.

### 5 — Leveling (Pricing Calibration)

The base level $\gamma_0$ is set so that the total expected premium meets a **target risk ratio of 90%** — meaning total premiums are set 10% above expected claims to cover expenses and margin.

## Repository Structure

```
glm-insurance-pricing/
├── analysis.ipynb         # Main notebook — full pipeline
├── requirements.txt       # Python dependencies
├── data/
│   ├── train.csv          # Training data (2018–2022, ~149k rows)
│   ├── validation.csv     # Validation data (~79k rows, includes claims)
│   └── eval.csv           # Evaluation data (~79k rows, no claims column)
└── README.md              # This file
```

## Data Dictionary

| Column | Description |
|--------|-------------|
| `RiskYear` | Policy year (2018–2022) |
| `NumberOfPersons` | Number of insured employees |
| `CompanyAge` | Age of the insured company (years) |
| `FinancialRating` | Credit/financial rating of the company |
| `TravellingArea` | Geographic travel zone |
| `Activity` | Industry sector of the company |
| `DangerousAreas` | Whether coverage includes high-risk destinations |
| `Duration` | Exposure in policy-years |
| `NumberOfClaims` | Count of claims filed |
| `ClaimCost` | Total claim cost (SEK) |

> The evaluation set (`eval.csv`) omits claims columns — it is used to generate pricing predictions for an unseen future period.

## How to Run

### Prerequisites

```bash
pip install -r requirements.txt
```

### Run the notebook

```bash
jupyter notebook analysis.ipynb
# or
jupyter lab analysis.ipynb
```

Execute all cells sequentially. The full pipeline runs in under 2 minutes.

## Skills Demonstrated

- **Actuarial pricing:** Multiplicative GLM framework (frequency × severity)
- **Statistical modelling:** Poisson and Gamma regression with canonical log link
- **Model selection:** LRT, AIC/BIC, Wald tests, stepwise selection
- **Business validation:** Gini score (Lorenz curve), risk ratio calibration
- **Feature engineering:** Risk-homogeneous grouping with exposure-weighted aggregation
- **Python:** `statsmodels`, `pandas`, `numpy`, `matplotlib`, `scipy`

## References

1. de Jong, P. & Heller, G.Z. (2008). *Generalised Linear Models for Insurance Data*. Cambridge University Press.
2. Ohlsson, E. & Johansson, B. (2010). *Non-Life Insurance Pricing with Generalised Linear Models*. Springer.
3. Nelder, J.A. & Wedderburn, R.W.M. (1972). Generalised Linear Models. *Journal of the Royal Statistical Society*, Series A, 135(3), 370–384.
