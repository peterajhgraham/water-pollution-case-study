# Applied Statistics Case Study: Classical Inference on Multivariate Environmental Data

This repository walks through the full classical statistical inference pipeline - model
specification, fitting, assumption checking, diagnostics, dimensionality reduction, and
robust re-estimation - applied end-to-end to a real multivariate dataset. The
environmental subject matter (state-level water pollution and landmass) is the vehicle;
the focus is the statistical methodology and what each diagnostic tells you when its
assumptions hold or break.

The central question is methodological: *given a small-n, heavy-tailed, multicollinear
design matrix, which conclusions from a standard OLS workflow survive scrutiny, and
which collapse under the LINE diagnostics?*

## Statistical Methods

Each method below is a general-purpose tool. The same pipeline is routinely applied to
neural population recordings, fMRI BOLD time series, single-cell expression matrices,
and any other multivariate dataset where you want to relate a response to a structured
set of predictors and trust the resulting inference.

- **Ordinary Least Squares (OLS) regression.** Closed-form linear estimator minimizing
  squared residuals. Returns coefficient estimates, standard errors, t-statistics, and
  an F-test for overall model significance. Inference is conditional on the LINE
  assumptions below.
- **LINE assumption diagnostics.** The four conditions OLS inference rests on:
  **L**inearity of the conditional mean, **I**ndependence of residuals, **N**ormality of
  residuals, and **E**qual variance (homoskedasticity).
- **Anderson–Darling test.** Tests the normality assumption on residuals. More sensitive
  to tail deviations than Kolmogorov–Smirnov, which makes it a strong default when heavy
  tails would invalidate t- and F-based inference.
- **Durbin–Watson test.** Detects first-order autocorrelation in the residuals.
  Statistics near 2 indicate independence; values toward 0 or 4 indicate positive or
  negative serial correlation, respectively. Standard pre-flight for any analysis where
  ordering carries information (time series, spatial transects, sequential trials).
- **Breusch–Pagan test.** Tests for heteroskedasticity by regressing squared residuals
  on the predictors. A small p-value indicates the error variance depends on the
  predictors, which inflates or deflates OLS standard errors.
- **Singular Value Decomposition (SVD).** Factorization X = UΣVᵀ that underlies PCA,
  pseudoinverses, and low-rank approximation. Used here as the numerically stable route
  to principal components when the design matrix is rank-deficient or near-collinear.
- **Principal Component Analysis (PCA).** Orthogonal rotation of the predictor space
  into directions of maximum variance, obtained via the SVD of the centered data
  matrix. Used both to compress correlated predictors into a smaller orthogonal basis
  and to refit OLS on the principal components when collinearity destabilizes the
  original coefficient estimates.
- **Robust regression (Huber M-estimator).** Re-fits the linear model with a loss
  function that down-weights extreme residuals. Used as a sanity check after the
  normality assumption fails - if the OLS and robust coefficients disagree, the OLS fit
  was being driven by a few high-leverage points.

This same diagnostic stack - fit a linear model, check LINE, decompose the predictor
covariance, refit robustly - is what most applied work in neural data analysis (firing
rate regressions, GLMs on spike counts, fMRI general linear models) and multivariate
time series analysis actually runs under the hood.

## Data Pipeline

- **Sources.** Per-state EPA ICIS-NPDES water pollution data (permits, pollutant
  loadings, total pollutant pounds, toxic-weighted pounds) joined with U.S. Census
  landmass data (total, land, inland water, and coastal water area).
- **Cleaning.** Dropped non-state rows (territories, DC). Reset the index, stripped
  thousands separators from numeric columns, and cast string-encoded numerics to
  floats.
- **Merging.** Concatenated the pollution and landmass tables on aligned state order
  and validated the alignment row-by-row before dropping the validation column.
- **Feature engineering.** Aggregated `Majors` and `Non-Majors` counts and pollutant
  loadings into `Total #` and `Total Pollutant Pounds` columns to give a single
  response variable per state.
- **Final design.** N = 51 (50 states + DC), response = `Total Pollutant Pounds
  (lb/yr)`, predictors = the five landmass area variables.

## Key Results

- **OLS fit is not significant.** R² = 0.013, adjusted R² = −0.096, F(5, 45) = 0.12,
  p = 0.987. None of the five landmass coefficients reach significance at α = 0.05.
  The naive "bigger states pollute more" hypothesis is not supported by a linear model
  on these features alone.
- **Normality fails hard.** Anderson–Darling A² = 12.55, p ≈ 1.8 × 10⁻³⁰. The residuals
  are extremely heavy-tailed (skew ≈ 6.4, kurtosis ≈ 43.5), driven by a small number of
  high-pollution outlier states. Any t- or F-based p-value from the OLS fit should be
  treated as unreliable.
- **Independence and equal variance hold.** Durbin–Watson = 2.14 (essentially no serial
  correlation in the residuals) and Breusch–Pagan p = 0.996 (no evidence of
  heteroskedasticity). Of the four LINE conditions, only normality is violated.
- **Dimensionality reduction does not rescue the fit.** Refitting OLS on the top SVD
  components yields R² = 0.012, p = 0.753 - the predictors are highly collinear (the
  five area variables are near-linearly dependent), but no low-dimensional projection
  of them explains the response either. A Huber robust re-fit confirms the same
  conclusion with a significant intercept and no significant landmass slopes,
  indicating the original OLS was not merely being thrown off by outliers.

The methodological takeaway is that the LINE diagnostics did exactly their job: they
flagged that the seemingly headline result ("states with more area pollute more") was
both statistically insignificant and resting on a violated normality assumption, and
the SVD/robust follow-ups confirmed there was no salvageable linear signal in this
feature set.

## Visualizations

![Tableau visualizations of state-level pollution and landmass](Tableau_Visuals.png)

Choropleths produced in Tableau from `land_and_pollution.csv` (exported from the
notebook) show pollution per unit area across states - Missouri and Idaho stand out as
high pollution-per-area outliers, which is consistent with the heavy-tailed residual
distribution flagged by the Anderson–Darling test.

## Files

- `Water_Pollution_Project.ipynb` - full analysis notebook (cleaning, EDA, OLS, LINE
  diagnostics, SVD, robust regression).
- `State_Statistics_Data.csv` - raw per-state pollution data.
- `Tableau_Visuals.png` - exported choropleth dashboard.
- `White_Paper.pdf` - written summary of the project and findings.

---

*Semester-long project completed through the Brown University Data Science Institute.*
