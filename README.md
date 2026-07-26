# Bayesian Marketing Mix Model (MMM)

This repository contains a comprehensive **Bayesian Marketing Mix Model (MMM)** implemented in modern PyMC (v6.2+). The project evaluates historical marketing spend across 7 channels to quantify causal impact, calculate Return on Investment (ROI), and optimise future budget allocation.

## 📊 Overview

Marketing Mix Modeling (MMM) is a statistical technique used to estimate the impact of various marketing tactics on sales and then forecast the impact of future sets of tactics. This project takes a rigorous Bayesian approach to naturally handle uncertainty in the data and the modeling process itself.

### Key Objectives
1. Model the delayed effect (carry-over) of marketing spend using a **Geometric Adstock** transformation.
2. Capture underlying organic revenue using an intercept, linear trend, and **Fourier-based seasonality**.
3. Accurately quantify the **contribution and ROI** of each marketing channel, complete with 94% High Density Intervals (HDI).
4. Perform **Model Comparison (LOO-CV)** to validate that incorporating adstock meaningfully improves predictive performance over raw spend models.
5. Simulate **"What-If" budget reallocations** to discover actionable insights and expected revenue uplifts.

---

## 🛠 Methodology

- **Framework:** PyMC (v6.2), PyTensor (v3.2+), ArviZ (v1.2+)
- **Adstock:** Learnable infinite-lag geometric decay (via `pytensor.scan`)
- **Seasonality:** 2nd-order Fourier features modeling an annual periodic signal (validated via ACF).
- **Inference:** No-U-Turn Sampler (NUTS) Hamiltonian Monte Carlo.
- **Diagnostics:** Rigorous evaluation of r-hat, bulk/tail effective sample size (ESS), and divergences.
- **Uncertainty Propagation:** All post-hoc analyses (ROI calculations, budget simulations) explicitly replay the adstock logic across the full trace of posterior samples.

---

## 📁 Repository Structure

```
├── data/
│   └── MMM_test_data.csv    # 104 weeks (2 years) of revenue and channel spend
├── MMM_Analysis.ipynb       # Fully executed end-to-end Jupyter Notebook
├── gemini.md                # Original assignment prompt and requirements constraints
├── .gitignore
└── README.md                # You are here
```

---

## 🚀 Key Insights & Deliverables

1. **Revenue Decomposition:** A canonical stacked area chart breaking down the timeline of revenue into Organic (Base + Trend + Seasonality) versus the incremental contributions of each channel.
2. **Channel ROI:** Comprehensive ROI estimates indicating that while Channel 7 demands the highest budget, it also provides extremely stable ROI.
3. **Budget Reallocation Simulation:** We ran a Bayesian simulation shifting 20% of the lowest-performing channel's budget to the highest-performing channel, demonstrating a statistically significant uplift in expected revenue while exposing the credible interval of that expectation.
4. **Transparent Code:** Extensive markdown blocks outlining the statistical reasoning, API choices, and business interpretations for every code cell.

---

## 💻 How to Run

To execute this notebook locally, you need a Python 3.10+ environment.

1. **Clone the repository:**
   ```bash
   git clone https://github.com/vijaysai1102/bayesian-marketing-mix-model.git
   cd bayesian-marketing-mix-model
   ```

2. **Install requirements:**
   ```bash
   pip install numpy pandas matplotlib seaborn pymc arviz scikit-learn statsmodels jupyter
   ```

3. **Run the Notebook:**
   ```bash
   jupyter notebook MMM_Analysis.ipynb
   ```
   *Note: Bayesian MCMC sampling can be computationally intensive. The notebook typically takes 5–15 minutes to run end-to-end depending on your hardware.*

---

*Author: Vijay*
