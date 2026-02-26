# Kriging Surrogate Model for Expensive Simulations

A Jupyter notebook demonstrating how to replace an expensive physical simulator with a trained **Kriging (Gaussian Process Regression)** surrogate model — enabling fast, near-free optimization and parameter studies.

---

## What Is a Surrogate Model?

In engineering and science, simulations (FEM, CFD, molecular dynamics, etc.) can take minutes to hours per run. Running thousands of such simulations for optimization is often impractical or impossible.

A **surrogate model** (also called a metamodel or emulator) is a cheap, fast approximation trained on a small number of real simulator runs. Once trained, it replaces the simulator entirely for tasks like optimization, sensitivity analysis, and design space exploration.

**Kriging** (Gaussian Process Regression) is one of the most popular surrogate modeling techniques in engineering because it provides not just a prediction, but also an **uncertainty estimate** — telling you how confident the model is at any point in the design space.

---

## The Physical Problem

The notebook uses **1D steady-state heat conduction in a metal rod** as the case study. This is a realistic stand-in for any expensive thermal simulation (e.g., a finite element solver).

The goal is to predict the **maximum temperature** T_max in the rod based on three design variables:

| Variable | Symbol | Range | Units |
|---|---|---|---|
| Thermal conductivity | k | 10 – 400 | W/m·K |
| Heat source intensity | q | 1,000 – 10,000 | W/m³ |
| Rod length | L | 0.1 – 1.0 | m |

The underlying physics follows the analytical steady-state solution with a nonlinear correction term to simulate a realistic, complex solver response.

---

## Notebook Structure

### Step 1 — Define the Expensive Simulation
The real simulator is defined as a Python function. A call counter tracks how many times it is invoked, making it easy to see how many expensive runs were actually needed.

### Step 2 — Design of Experiments (Latin Hypercube Sampling)
Rather than sampling randomly, **Latin Hypercube Sampling (LHS)** is used to generate 30 training points that cover the design space efficiently. This is critical — a well-spread DoE means fewer simulator runs are needed to train an accurate surrogate.

### Step 3 — Train the Kriging Surrogate
A Gaussian Process Regressor is trained on the 30 simulator outputs using a **Matérn 5/2 kernel**, which is well-suited to smooth engineering response surfaces. Input features are standardized before training. Kernel hyperparameters are optimized automatically with multiple restarts.

### Step 4 — Validate the Surrogate
A separate test set of 200 points is generated and evaluated by both the real simulator and the surrogate. Three validation plots are produced: a parity plot, a residuals plot, and an uncertainty map. Reported metrics include R², RMSE, and MAPE.

### Step 5 — Response Surface Visualization
With one variable fixed, 2D response surfaces are plotted side-by-side for the true simulator and the surrogate. An error map highlights where the surrogate is most and least accurate relative to the training point locations.

### Step 6 — Surrogate-Based Optimization
The surrogate is used to solve a practical engineering problem: find the **minimum thermal conductivity** (lowest material cost) such that T_max stays below 150°C for a given heat source and rod length. 1,000 candidate designs are evaluated instantly using the surrogate, then the optimal design is verified with a single real simulator call.

### Step 7 — Cost Summary
A comparison table and bar chart quantify the computational savings. With a realistic assumption of 10 minutes per simulator run, the surrogate workflow achieves roughly a **300× speedup** over direct optimization.

---

## Results at a Glance

| Metric | Value |
|---|---|
| Training simulator runs | 30 |
| Surrogate R² on test set | ≥ 0.999 |
| Optimization evaluations | 1,000 (instant) |
| Total simulator calls | ~31 |
| Estimated speedup | ~300× |

---

## Requirements

```bash
pip install scikit-learn scipy numpy matplotlib
```

All dependencies are standard and available on PyPI. No additional solvers or licensed software are required.

| Package | Purpose |
|---|---|
| `scikit-learn` | Gaussian Process Regressor, preprocessing |
| `scipy` | Latin Hypercube Sampling (`scipy.stats.qmc`) |
| `numpy` | Numerical computation |
| `matplotlib` | All plots and visualizations |

Python 3.8 or higher is recommended.

---

## How to Run

1. Install the dependencies listed above.
2. Open `kriging_surrogate_model.ipynb` in Jupyter Lab or Jupyter Notebook.
3. Run all cells from top to bottom (`Kernel → Restart & Run All`).

Each section is self-contained and heavily commented. You can modify the design bounds, number of training points, kernel type, or temperature constraint and re-run to explore the effects.

---

## Extending the Notebook

A few natural next steps if you want to go further:

**Adaptive sampling** — Instead of a fixed training set, add new points iteratively where surrogate uncertainty is highest (Bayesian optimization / active learning).

**Higher dimensions** — The workflow scales to more input variables, though accuracy may degrade above ~15–20 dimensions without more training data.

**Different kernels** — Try `RBF` for very smooth responses or `RationalQuadratic` for multi-scale variation. The kernel choice matters for accuracy.

**Cross-validation** — Replace the hold-out test set with k-fold cross-validation for a more robust accuracy estimate when simulator runs are very limited.

**Multi-output surrogates** — Extend to predict multiple outputs simultaneously (e.g., T_max and thermal stress at the same time).

---

## Key Concepts

**Kriging / Gaussian Process Regression** — A probabilistic surrogate that models the response as a realization of a Gaussian process, providing both a mean prediction and a variance (uncertainty) at every point.

**Matérn 5/2 kernel** — A covariance function that assumes the response is twice-differentiable, striking a good balance between smoothness and flexibility for engineering problems.

**Latin Hypercube Sampling** — A space-filling design strategy that divides each input dimension into equal intervals and ensures each interval is sampled exactly once, giving much better coverage than random sampling for the same number of points.

**Surrogate-based optimization** — An optimization strategy where a cheap surrogate replaces the expensive simulator during the search, with occasional verification calls to the real simulator to confirm promising designs.