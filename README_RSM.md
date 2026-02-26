# Response Surface Methodology (RSM) for Expensive Simulations

A Jupyter notebook demonstrating how to replace an expensive physical simulator with a trained **Response Surface Model (RSM)** — a polynomial regression surrogate that enables fast optimization, parameter studies, and interpretable effect analysis.

---

## What Is Response Surface Methodology?

Response Surface Methodology is a collection of statistical and mathematical techniques for building an approximation of a complex, expensive system using a small number of carefully chosen experiments. Unlike purely black-box surrogates, RSM produces an **explicit polynomial equation** that you can inspect, interpret, and differentiate analytically.

The most common form is the **full second-order (quadratic) model**:

$$\hat{y} = \beta_0 + \sum_i \beta_i x_i + \sum_i \beta_{ii} x_i^2 + \sum_{i<j} \beta_{ij} x_i x_j$$

This captures linear trends, curvature effects, and pairwise interactions between variables — making it powerful enough for a wide range of smooth engineering responses while remaining fully interpretable.

---

## The Physical Problem

The notebook uses **1D steady-state heat conduction in a metal rod** as the case study. The goal is to predict the **maximum temperature** T_max based on three design variables:

| Variable | Symbol | Range | Units |
|---|---|---|---|
| Thermal conductivity | k | 10 – 400 | W/m·K |
| Heat source intensity | q | 1,000 – 10,000 | W/m³ |
| Rod length | L | 0.1 – 1.0 | m |

This is identical to the Kriging notebook, allowing direct comparison of the two surrogate approaches on the same problem.

---

## Notebook Structure

### Step 1 — Define the Expensive Simulation
The real simulator is defined as a Python function with a call counter. This makes it easy to see exactly how many expensive runs were consumed throughout the entire workflow.

### Step 2 — Design of Experiments: Central Composite Design (CCD)
RSM uses a **structured design** specifically built for polynomial fitting, rather than space-filling sampling. The CCD consists of three types of points: factorial corner points (2^k = 8), axial star points (2k = 6), and center point replicates (6). This gives 20 total runs arranged to support estimation of all quadratic and interaction terms with minimum variance.

### Step 3 — Fit the Response Surface Model
A full second-order polynomial is fit using ordinary least squares inside a scikit-learn pipeline. The pipeline handles input standardization, polynomial feature expansion (10 terms for 3 variables), and linear regression. All fitted coefficients are printed and interpreted.

### Step 4 — Validate the Surrogate
A 200-point test set is generated and compared against the surrogate. Validation includes a parity plot, residuals plot, and a coefficient bar chart showing which terms drive the response. Metrics reported are R², RMSE, MAPE, and 5-fold cross-validation R².

### Step 5 — Response Surface Visualization
2D contour maps compare the true simulator and the RSM surrogate side by side, with a third panel showing the absolute error across the design space. Training point locations are overlaid on the error map to show the relationship between data coverage and accuracy.

### Step 6 — Main Effects and 3D Surface Plots
A key advantage of RSM over Kriging is interpretability. This section plots the main effect of each variable (holding others at midpoint) and three 3D response surface plots showing pairwise interactions — directly readable from the model equation.

### Step 7 — Surrogate-Based Optimization
The RSM is used to solve a practical engineering problem: find the minimum thermal conductivity such that T_max stays below 150°C for a given heat source and rod length. 1,000 candidate designs are evaluated instantly using the surrogate, then the optimal design is confirmed with a single real simulator call.

### Step 8 — Cost Summary
A comparison table and log-scale bar chart quantify the computational savings. With 20 training runs and 1 verification call, the surrogate workflow achieves roughly a **476× speedup** over direct optimization.

---

## Results at a Glance

| Metric | Value |
|---|---|
| Design strategy | Central Composite Design (CCD) |
| Training simulator runs | 20 |
| Model terms | 10 (full quadratic, 3 variables) |
| Optimization evaluations | 1,000 (instant) |
| Total simulator calls | ~21 |
| Estimated speedup | ~476× |

---

## Requirements

```bash
pip install scikit-learn scipy numpy matplotlib
```

| Package | Purpose |
|---|---|
| `scikit-learn` | PolynomialFeatures, LinearRegression, Pipeline, cross_val_score |
| `scipy` | Latin Hypercube Sampling for the test set |
| `numpy` | Numerical computation |
| `matplotlib` | All plots including 3D surfaces |

Python 3.8 or higher is recommended.

---

## How to Run

1. Install the dependencies listed above.
2. Open `rsm_surrogate_model.ipynb` in Jupyter Lab or Jupyter Notebook.
3. Run all cells from top to bottom (`Kernel → Restart & Run All`).

Each section is self-contained and commented. You can modify the CCD alpha value, number of center points, polynomial degree, or temperature constraint and re-run to explore the effects.

---

## RSM vs Kriging — Choosing the Right Surrogate

Both notebooks solve the same physical problem, making comparison straightforward.

| Feature | RSM | Kriging |
|---|---|---|
| Model type | Polynomial regression | Gaussian Process |
| Interpretability | High — explicit coefficients | Low — black-box |
| Uncertainty estimate | No | Yes |
| Handles high nonlinearity | Limited to polynomial degree | Excellent |
| Training runs required | Few (20 with CCD) | More (30 with LHS) |
| Design strategy | Structured (CCD) | Space-filling (LHS) |
| Best for | Smooth, near-quadratic responses | Complex, highly nonlinear responses |
| Extrapolation behavior | Polynomial divergence | Honest uncertainty growth |

**Use RSM when** the response is expected to be smooth and well-approximated by a polynomial, you need interpretable coefficients (e.g., for reporting or regulatory purposes), or you have very few simulator runs to spare.

**Use Kriging when** the response has complex nonlinearity or interactions, you need uncertainty estimates for adaptive sampling, or the number of input variables is moderate and the response surface is irregular.

---

## Extending the Notebook

**Higher-order polynomials** — Try degree=3 for cubic RSM if the quadratic model shows systematic residuals, though this requires more training data to avoid overfitting.

**Ridge or Lasso regression** — Replace `LinearRegression` with `Ridge` or `Lasso` in the pipeline to regularize the coefficients when the design matrix is ill-conditioned.

**ANOVA decomposition** — Extend the coefficient analysis with an ANOVA table to formally test which terms are statistically significant and which can be dropped.

**Augmented design** — Add more runs to the CCD (e.g., face-centred points at intermediate levels) to improve accuracy in specific sub-regions of the design space.

**Comparison study** — Run both this notebook and the Kriging notebook on the same simulator, then compare R², RMSE, and cost side by side for a full surrogate benchmarking exercise.

---

## Key Concepts

**Response Surface Methodology (RSM)** — A framework combining designed experiments and polynomial regression to approximate and optimize expensive response functions with a minimal number of simulator evaluations.

**Central Composite Design (CCD)** — A structured experimental design for fitting quadratic response surfaces, composed of factorial corner points, axial star points, and center replicates. The face-centred variant (alpha=1) keeps all points within the original factor bounds.

**Polynomial Features** — The expansion of input variables into all polynomial and interaction terms up to a given degree. For 3 variables and degree 2, this yields 10 terms: 1 intercept, 3 linear, 3 quadratic, and 3 cross-product terms.

**Main Effects** — The individual influence of each input variable on the response, evaluated by varying that variable while holding all others fixed at their midpoints.

**Interaction Effects** — The combined influence of two variables on the response, visible when the effect of one variable depends on the level of another — captured by cross-product terms in the polynomial.

**Surrogate-based optimization** — An optimization workflow where a cheap surrogate replaces the expensive simulator during the search phase, with one or more verification calls to the real simulator to confirm the final optimum.
