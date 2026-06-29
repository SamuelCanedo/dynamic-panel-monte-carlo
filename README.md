# Econometrics: Dynamic Panel Data Monte Carlo Simulation (Nickell Bias)

This repository hosts a high-performance econometric simulation framework designed to analyze, visualize, and benchmark the finite-sample performance of estimators in dynamic panel data models. Specifically, the infrastructure uses **Monte Carlo simulations** to quantify the mathematical breakdown of standard estimators under the presence of **Nickell Bias** (O(1/T)) in autoregressive AR(1) environments.

---

## Analytical Context & Objective

In dynamic panel data frameworks featuring lagged dependent variables ($y_{i,t-1}$), the standard Fixed Effects (FE) / Within Estimator becomes endogenously biased due to the mechanical correlation between the transformed lagged variable and the mean-deviated error term. This script layer evaluates the asymptotic and finite properties of three core estimators:
1. **Fixed Effects (FE / Within-Group)**: Captures unobserved individual heterogeneity but experiences severe negative bias when the time dimension ($T$) is small, regardless of cross-sectional size ($N$).
2. **Pooled Ordinary Least Squares (OLS)**: Ignores individual fixed effects ($\alpha_i$), leading to severe positive bias due to omitted variable correlation.
3. **Anderson-Hsiao (IV / GMM Baseline)**: Mitigates endogeneity by first-differencing the AR(1) model and instrumenting the endogenous lag with a second-order lag primitive ($y_{i,t-2}$).

---

## Infrastructure Architecture

The codebase follows an enterprise modular layout separating data generation, mathematical estimation, iterative batch routing, and graphic visualization:

* `dgp.py` (**Data Generating Process**): Generates synthetic dynamic panel spaces based on the stationarity rules of an AR(1) data structure:
  
  $$y_{it} = \gamma y_{it-1} + \alpha_i + u_{it}$$
  
  Features an adjustable burn-in window (`burn_in=50`) to remove initial state dependence and ensure realistic time-series convergence before restructuring into long-format data structures.
* `estimators.py` (**Estimation Layer**): Handles high-dimensional regressions utilizing `linearmodels` backends. It processes multi-index entity groups to return computed scalar coefficients for Pooled OLS, Panel OLS with entity effects, and Two-Stage Least Squares Instrumental Variables (`IV2SLS`).
* `monte_carlo.py` (**Simulation Matrix Engine**): Orchestrates the grid search array across customizable parameter boundaries ($N \times T$). Executes automated replication loops (`replications=100`) with seed locking for strict scientific reproducibility.
* `graphs.py` (**Visualization Engine**): Generates publishing-grade diagnostic charts via `matplotlib` and `seaborn` to contrast empirical convergence paths against asymptotic theory.

---

## Simulation Grid & Hyperparameters

The default experimental setup checks model constraints across the following dimensions:

| Dimension / Parameter | Value Spectrum | Functional Objective |
| --- | --- | --- |
| **Cross-Sections ($N$)** | `[50, 100, 200]` | Evaluates small vs. large sample properties |
| **Time Horizons ($T$)** | `[5, 10, 20]` | Captures $O(1/T)$ attenuation trends |
| **True Lag Vector ($\gamma$)** | *User Defined* | Ground truth autoregressive persistence parameter |
| **Replications** | 100 | Statistical sample iterations per grid cell |
| **Randomization Seed** | `42` | Locks pseudo-random initialization matrices |

---

## Empirical Simulation Results

The extracted matrix highlights the stark trade-offs between estimation consistency and finite-sample variances:

| Cross-Sections ($N$) | Time Horizons ($T$) | FE Mean Bias | Pooled OLS Mean Bias | Anderson-Hsiao (IV) Bias |
| :--- | :--- | :--- | :--- | :--- |
| **50** | 5 | -0.4030 | +0.3731 | +0.0596 |
| **50** | 10 | -0.1775 | +0.3728 | +0.0072 |
| **50** | 20 | -0.0874 | +0.3716 | +0.0041 |
| **100** | 5 | -0.4137 | +0.3727 | +0.0109 |
| **100** | 10 | -0.1809 | +0.3704 | +0.0154 |
| **100** | 20 | -0.0813 | +0.3740 | +0.0050 |
| **200** | 5 | -0.4098 | +0.3731 | -0.0003 |
| **200** | 10 | -0.1841 | +0.3746 | -0.0001 |
| **200** | 20 | -0.0841 | +0.3738 | +0.0008 |

### Key Analytical Takeaways
* **Fixed Effects Inconsistency**: As $T$ doubles (from 5 to 10, and 10 to 20), the negative Fixed Effects bias is cut roughly in half ($\approx -0.41 \rightarrow -0.18 \rightarrow -0.08$), visually confirming the analytical $O(1/T)$ rate of Nickell's structural prediction.
* **Pooled OLS Failure**: Pooled OLS bias remains consistently locked around $+0.37$, completely unaffected by increases in either $N$ or $T$, due to the permanent omitted variable bias caused by ignoring $\alpha_i$.
* **Instrumental Variable Domination**: The Anderson-Hsiao estimator achieves near-zero bias even at short panels ($T=5, N=200 \rightarrow \text{Bias}: -0.0003$), highlighting the structural importance of correct instrument choices in dynamic settings.

---

## Quickstart & Operational Usage

### 1. Execute the Simulation Array
To run the full grid simulation and export the empirical performance results into a structured `.csv` ledger:

```python
from monte_carlo import run_monte_carlo

# Define testing spectrum dimensions
N_sizes = [50, 100, 200]
T_sizes = [5, 10, 20]
true_gamma = 0.5  # Adjust persistence vector based on research targets

results_df = run_monte_carlo(N_sizes, T_sizes, true_gamma=true_gamma, replications=100)
results_df.to_csv('monte_carlo_results.csv', index=False)
```
Author: Samuel Antonio Cañedo Pérez

Research Focus: Selected Topics on Panel Data (Fall 2026)
