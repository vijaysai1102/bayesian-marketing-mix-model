# Assignment Questions & Answers

### 1. How do you model spend carry-over?
Spend carry-over is modeled using a **Geometric Adstock** transformation. Marketing spend does not impact revenue only in the week it is spent; its effect decays over time. We model this recursively as:
`Adstocked_Spend[t] = Raw_Spend[t] + alpha * Adstocked_Spend[t-1]`
where `alpha` (between 0 and 1) is the retention rate. Instead of hard-coding this decay rate, we define it as a random variable (`alpha_channel`) in our PyMC model and use `pytensor.scan` to dynamically compute the geometric decay within the computational graph. This allows the NUTS sampler to learn the optimal, channel-specific carry-over effect directly from the data.

### 2. Explain your choice of prior inputs to the model?
We used **weakly informative priors** to constrain the model to physically realistic bounds without forcing a specific outcome:
- **`beta_channel` (HalfNormal, sigma=0.3):** Constrained to be strictly positive because we assume marketing spend does not actively destroy revenue. Scaled appropriately for our MaxAbsScaler normalized data.
- **`alpha_channel` (Beta(2, 2)):** Constrained strictly between 0 and 1. A Beta(2,2) prior is symmetric and peaks at 0.5, reflecting a prior belief that some decay exists, but keeping the model open to discovering both very short (near 0) and very long (near 1) carry-over effects.
- **`intercept` (Normal, mu=0.3, sigma=0.3) & `beta_trend/fourier` (Normal, mu=0, sigma=0.2):** Standard uninformative priors allowing the baseline organic revenue and seasonality to shift as needed to fit the time-series.

### 3. How are your model results based on prior sampling vs. posterior sampling?
- **Prior Sampling:** The prior predictive check generates revenue trajectories before the model has "seen" any data, relying purely on our prior distributions. In the notebook, this produces a wide array of potential revenue curves (the grey bands). This confirms our priors are theoretically sound (they don't generate physically impossible values like negative revenue or infinite spikes).
- **Posterior Sampling:** The posterior predictive check shows the model's predictions *after* updating its beliefs using the 104 weeks of observed data. The posterior mean tightly tracks the actual observed revenue, and the 94% High Density Interval (HDI) accurately captures the variance. The contrast proves the model successfully learned from the data.

### 4. How good is your model performing? How you do measure it?
Model performance is measured in two ways:
1. **In-Sample Error Metrics:** We calculated standard regression metrics by comparing the deterministic posterior mean to the observed revenue. The model fits well, capturing both the long-term trend and seasonal spikes (e.g., holiday peaks).
2. **Predictive Performance (LOO-CV):** Standard error metrics can be misleading because adding complexity (like adstock) always improves in-sample fit. To rigorously measure performance, we used Leave-One-Out Cross-Validation (LOO-CV) via Pareto-Smoothed Importance Sampling. We compared our Full Adstock model against a Baseline model (raw spend, no carry-over). The Full model achieved a significantly better Expected Log Pointwise Predictive Density (ELPD), proving that modeling spend carry-over genuinely improves the model's predictive power.

### 5. What are your main insights in terms of channel performance/ effects?
- **Organic vs. Paid:** A significant portion of revenue is driven by the base organic trend and annual seasonality (the Fourier features).
- **Varying Carry-over:** Channels exhibit different adstock decay rates (`alpha`). Some channels drive immediate, short-lived spikes, while others build equity over several weeks.
- **Collinearity & Certainty:** Because the model is Bayesian, the posterior distributions capture our uncertainty. Channels with highly correlated spend patterns or lower signal-to-noise ratios have wider HDIs (fatter violin plots in the contribution chart), indicating we are less certain about their exact true effect.

### 6. Can you derive ROI estimates per channel? What is the best channel in terms of ROI?
Yes. ROI is derived by replaying the adstock transformation over the actual historical spend for every single draw of the posterior. 
`ROI = (Total Channel Contribution) / (Total Raw Spend)`
Because we calculate this for all 8,000 posterior samples, we don't just get a point estimate; we get a full credible interval (HDI) for each channel's ROI.

Based on the posterior distributions:
- **Best ROI:** **Channel 2** generated the highest return per dollar spent.
- **Worst ROI:** **Channel 7** has the lowest ROI (though it commands a massive share of the absolute budget).
- **Actionable Takeaway:** Our budget reallocation simulation proves that shifting just 20% of the budget from Channel 7 to Channel 2 yields a massive expected increase in total 2-year revenue (with the 94% HDI confirming this reallocation is profitable with high probability). *Caveat: This assumes linear scaling and does not account for diminishing marginal returns (saturation), which would occur at high spend levels.*
