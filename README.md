# Bayesian Hierarchical Modeling of Worker Productivity in the Garment Sector

**Course Context.** Statistical Methods in Data Science II (a.y. 2024–2025), M.Sc. in Data Science, Sapienza University of Rome  

## 1. Purpose and Scope

This repository presents a rigorous Bayesian analysis of worker productivity using the UCI dataset [*Productivity Prediction of Garment Employees*](https://archive.ics.uci.edu/dataset/597/productivity+prediction+of+garment+employees). The work compares three hierarchical specifications one **centered** and two **non-centered**, combining linear terms and **spline-based** effects including interactions to capture complex, nonlinear production dynamics. Model adequacy is assessed via **convergence diagnostics**, **posterior predictive checks (PPC)** and **DIC**.

> **Result in brief.** The **non-centered model with spline interaction `wip × smv` and a smooth for `incentive` (Model 3)** delivers the best trade-off between fit, flexibility and interpretability. It reproduces the empirical density (central mass and tails) and explains group-level heterogeneity better than alternatives.

## 2. Repository Contents (Two Files Only)

- **`garment_productivity_bhm_report.Rmd`**: the complete and annotated R Markdown source including data description, EDA, preprocessing, model specifications (centered/non-centered), inference code for JAGS, diagnostics (PPC, R-hat) and DIC comparison.  
- **`garment_productivity_bhm_report.html`**: the compiled report.

> **Important.** The HTML report is **too large for GitHub online preview**.  
> Please **download** `garment_productivity_bhm_report.html` and open it locally in a browser.

No other files are tracked in this repository. Fitted objects and summaries are generated locally at render time and saved as RDS/TXT.

## 3. Modeling Strategy (Three Hierarchical Specifications)

### **Model 1 — Centered hierarchy (sum-to-zero `u[j]`)**
- **Fixed effects.** Linear: `targeted_productivity`, `incentive`, `idle_time_flag`.  
  Splines: `over_time`, `smv`.  
- **Findings.** Excellent convergence; all linear predictors statistically relevant. PPC reveals **underestimated variability** and **tail misalignment**.  
- **Team heterogeneity.** Largest variance: **sd_team ≈ 0.12**, indicating substantial unexplained between-team variation.

### **Model 2 — Non-centered hierarchy with interaction (`incentive × wip`)**
- **Fixed effects.** Linear: `targeted_productivity`, `idle_time_flag`.  
  Splines: `over_time`, `smv`, and **spline-based interaction `incentive × wip`**.  
- **Findings.** Interaction uncovers **strong positive effects** in specific regimes not captured by Model 1; the effect of `idle_time_flag` weakens and becomes uncertain.  
- **Team heterogeneity & fit.** **sd_team ≈ 0.054**; richer fixed-effect structure explains more between-team variance. PPC **substantially improved**.

### **Model 3 — Non-centered hierarchy with `wip × smv` (preferred)**
- **Fixed effects.** Linear: `targeted_productivity`, `idle_time_flag` (negative and significant).  
  Smooths: **univariate spline for `incentive`**; **bivariate spline interaction `wip × smv`**.  
- **Findings.** Several spline coefficients show large magnitude with tight credible intervals, revealing **nuanced nonlinear trends** tied to production complexity and task duration.  
- **Team heterogeneity & fit.** **sd_team ≈ 0.071**; more meaningful team effects are detected. PPC shows **further improvement**, accurately reproducing both center and tails.  
- **Conclusion.** Best balance of fit, flexibility and interpretability → **selected model.**

## 4. Research Questions — Interpretation Under Model 3

**Why reconstruct smooths?** Posterior summaries of spline **basis coefficients** do not display the shape of the implied functions. We reconstruct smooths by post-multiplying the mgcv spline design matrices with posterior draws, evaluating both on **observed data** and on a **regular grid** for clearer interpretation across the support (especially when predictors are skewed).

- **RQ1 — Is `incentive` nonlinear?**  
  Yes. The smooth is flat/slightly negative at low values and becomes **strongly positive** at higher values, confirming that spline modeling is **crucial** for `incentive`. A practical **threshold** (grid point where the mean effect turns positive) can be reported from the reconstructed curve.

- **RQ2 — Does `over_time` reduce productivity?**  
  The posterior smooth is **mostly negative** with 95% credible intervals below zero in the lower range and only a mild recovery later. A Bayesian check indicates **Pr(mean effect < 0) ≈ 0.81**, i.e., substantial—though not conclusive—evidence of a detrimental average effect.

- **RQ3 — Does the interaction `wip × smv` matter?**  
  Yes, strongly and **nonlinearly**. In high-complexity regimes (`smv` large), the interaction surface remains **negative on average** (posterior probability ≈ **1.0** for a negative mean effect when `smv > 2`). While `wip` can mitigate adverse effects in some regions, it **does not offset** the strong negative impact of high `smv`.

- **RQ4 — Do team effects justify a hierarchical model?**  
  Yes. The posterior for `sd_team` concentrates away from zero and the distribution of team-specific effects \(u_j\) shows clear **between-team heterogeneity** (negative, neutral and positive teams), validating the hierarchical structure for organizational data.
