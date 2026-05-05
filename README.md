# Is Educational Inequality a Stronger Predictor of National Happiness Than Income Inequality?

**Author:** Mobin Seyedmohammadi
**Institution:** Sapienza University of Rome — Data Management and Analysis

---

## The Short Answer

**Yes — and it's not close.**

Educational inequality (EII) explains **37% of cross-national variance in happiness** (R²=0.369, p<0.001). The Gini index explains only **6%** (R²=0.062). When both are included in a joint model, Gini loses statistical significance entirely (p=0.647) and its coefficient even flips sign — suggesting no independent negative effect on happiness beyond what it shares with educational inequality.

This finding holds across three independent robustness checks, a bootstrapped confidence interval analysis, and a non-linear Random Forest validation.

---

## Results at a Glance

| Model | Predictor(s) | R² | Adj. R² | p-value |
|-------|-------------|:--:|:-------:|:-------:|
| M1 | Education Inequality (EII) | 0.369 | 0.364 | < 0.001 ✅ |
| M2 | Income Inequality (Gini) | 0.062 | 0.053 | 0.007 ✅ |
| M3 | EII + Gini (joint) | 0.371 | 0.360 | EII: <0.001 ✅ · Gini: 0.647 ❌ |

**Bootstrap 95% CI (n=2000 resamples):**
- EII: R² = 0.374, CI [0.239, 0.502]
- Gini: R² = 0.069, CI [0.006, 0.171]
- CIs do not overlap → difference is statistically meaningful

**Random Forest validation (5-fold CV):** R² = 0.357 ± 0.040 — near-identical to OLS, confirming the relationship is linear. EII permutation importance is **9.5× larger** than Gini (0.758 vs 0.080).

---

## Methodology

### 1. Education Inequality Index (EII)

Built from six pairwise relative gap indicators drawn from the UNESCO WIDE dataset across three disparity dimensions — sex, wealth quintile, and location (rural/urban):

| Metric | Sex Gap | Wealth Gap | Location Gap |
|--------|:-------:|:----------:|:------------:|
| Upper secondary completion rate | ✅ | ✅ | ✅ |
| Minimum math proficiency (Level 1) | ✅ | ✅ | ✅ |

**Why only these three dimensions?** Formal coverage analysis showed Ethnicity (37%), Religion (46%), and Disability (24%) had insufficient country coverage — including them would require imputing >40% of values, introducing more noise than signal.

**Gap formula** — scale-independent relative gap between boundary groups A and B:

$$\text{gap}_{m,d} = \frac{|\bar{m}_A - \bar{m}_B|}{(\bar{m}_A + \bar{m}_B)/2}$$

**Index construction — three steps:**

$$z_{ij} = \frac{x_{ij} - \bar{x}_j}{\sigma_j} \quad \text{(standardise)}$$

$$\text{EII}^*_i = \sum_{j=1}^{6} w_j \cdot z_{ij} \quad \text{(PCA-weighted combination)}$$

$$\text{EII}_i = \frac{\text{EII}^*_i - \min}{\max - \min} \quad \text{(min-max scale to [0,1])}$$

Weights $w_j$ are absolute PC1 loadings from PCA, normalised to sum to 1. PC1 explains 49.2% of variance — more than double any other component. Standardisation happens **before** PCA to avoid scale dominance.

**PCA-derived weights:**

| Indicator | Weight | vs Equal (1/6=0.167) |
|-----------|:------:|:--------------------:|
| upsec_location_gap | 0.203 | ↑ above |
| math_wealth_gap | 0.185 | ↑ above |
| upsec_wealth_gap | 0.177 | ↑ above |
| upsec_sex_gap | 0.157 | ↓ below |
| math_location_gap | 0.156 | ↓ below |
| math_sex_gap | 0.123 | ↓ below |

Location and wealth gaps dominate. Sex gaps carry the least weight — gender parity has improved more than wealth and geographic parity globally.

### 2. Income Inequality — Gini Index

Source: World Bank (2000–2024). **Data quality assessment revealed serious coverage issues** — 42% of countries have fewer than 5 years of observations, and the median country has only 3 data points.

**Methodology — combined approach:**
- **Primary estimate:** most recent available Gini value per country
- **Reliability flag:** ≥5 years = reliable (n=98 in merged sample); <5 years = flagged
- **Correlation between mean and most recent Gini:** r = 0.920 — near-identical rankings, but most recent is more methodologically defensible

### 3. National Happiness

Cantril Ladder scores from the **World Happiness Report 2025** (2024 Gallup World Poll wave). Range: 1.36 (Afghanistan) – 7.74 (Finland).

### 4. Statistical Analysis

- Three OLS models with **HC3 heteroskedasticity-robust standard errors**
- **Breusch-Pagan test** for heteroskedasticity (p=0.180 — not detected; HC3 conservative but valid)
- **VIF check** — both predictors VIF=1.28, no multicollinearity
- **2000-iteration bootstrap** for R² confidence intervals
- **Cook's distance** influence diagnostics
- **Constrained Random Forest** (max_depth=3, min_samples_leaf=10) with permutation importance
- **Bonferroni correction** applied to all multiple comparison tests
- **Pearson vs Spearman** robustness check (all key correlations robust, Δ<0.05)

---

## Robustness Checks

Three independent sensitivity analyses all strengthen the main finding:

| Check | n | EII R² | Gini p in M3 | Conclusion |
|-------|---|:------:|:------------:|-----------|
| Full sample | 113 | 0.369 | 0.647 | EII dominates |
| Excl. 4 influential obs. | 109 | 0.431 | 0.939 | EII stronger |
| Reliable Gini only (≥5 yrs) | 82 | 0.390 | 0.018* | EII stronger |

*p=0.018 does not survive Bonferroni correction (threshold=0.017) and coefficient is positive (+0.033), suggesting suppression not genuine negative effect.

**Influential observations identified (Cook's D > 4/n = 0.035):**

| Country | Cook's D | Pattern |
|---------|:--------:|---------|
| Nicaragua | 0.148 | High EII, unexpectedly happy (Latin American paradox) |
| Guatemala | 0.090 | High EII, unexpectedly happy (Latin American paradox) |
| Sierra Leone | 0.046 | High EII, very unhappy |
| Ukraine | 0.043 | Low EII, unhappy (geopolitical conflict) |

---

## Key Visualisations

The analysis produces 13 publication-quality figures:

| Figure | Description |
|--------|-------------|
| `fig_gap_distributions` | Distribution of sex/wealth/location gaps |
| `fig_missingness_by_income` | Systematic missingness bias by income group |
| `fig_indicator_correlation` | Correlation heatmap between 6 EII indicators |
| `fig_pca_weights` | PCA-derived weights vs equal weighting |
| `fig_pca_regions` | PCA scatter with Mahalanobis ellipses & outlier labels |
| `fig_scree_plot` | Variance explained per component |
| `fig_eii_ranking` | Top/bottom 10 countries by EII |
| `fig_eii_by_region` | EII strip plot by world region |
| `fig_gini_data_quality` | Gini coverage distribution & mean vs recent scatter |
| `fig_correlation_matrix` | Bonferroni-corrected correlation matrix |
| `fig_scatter_regression` | Scatter plots with bootstrap 95% CI bands |
| `fig_influence_diagnostics` | Cook's distance & leverage vs residual |
| `fig_r2_comparison` | Bootstrap R² distributions |
| `fig_rf_validation` | Random Forest CV R² & permutation importance |
| `fig_regional_correlations` | EII vs Gini correlations by region |
| `fig_summary_dashboard` | 4-panel summary figure |

---

## Repository Structure

```
inequality-happiness/
│
├── data/
│   ├── WIDE.csv                  # UNESCO WIDE educational indicators
│   ├── GINI.csv                  # World Bank Gini index (2000–2024)
│   └── happiness2025.xlsx        # World Happiness Report 2025
│
├── outputs/
│   ├── final_dataset.csv         # 113-country merged analysis dataset
│   └── fig_*.png                 # All 16 output figures
│
├── inequality_happiness_analysis.ipynb   # Full analysis notebook
├── requirements.txt
└── README.md
```

---

## Getting Started

### Prerequisites

```bash
pip install -r requirements.txt
```

### Data Sources

| Dataset | Source | Access |
|---------|--------|--------|
| UNESCO WIDE | [education-inequalities.org](https://www.education-inequalities.org/) | Free download |
| World Bank Gini | [data.worldbank.org](https://data.worldbank.org/indicator/SI.POV.GINI) | Free download |
| World Happiness Report | [worldhappiness.report](https://worldhappiness.report/) | Free download |

Place files in `data/` as named above, then run all cells top-to-bottom.

```bash
jupyter notebook inequality_happiness_analysis.ipynb
```

---

## Why This Matters

The standard policy narrative focuses on income redistribution — reduce the Gini coefficient, improve well-being. Our analysis suggests this framing is incomplete.

**Educational inequality operates through different mechanisms:**

1. **Access to opportunity** — wealth and location gaps in schooling determine life trajectories from childhood, constraining social mobility in ways that are viscerally felt
2. **Horizontal inequality** — gaps between demographic groups signal fairness failures and erode social trust more than vertical income distribution
3. **Policy legacy** — former Soviet states dominate the lowest-EII group, showing that historical policy choices have measurable decades-long effects on educational equality and potentially on happiness

**The implication:** Policies targeting equitable access to education — particularly across wealth and geographic lines — may improve national well-being more effectively than income redistribution alone.

**Important caveat:** This is a cross-sectional observational study. No causal claim is made. GDP confounding (wealthier nations have both lower EII and higher happiness) is the most important unresolved threat to validity.

---

## Limitations

- **No causation** — cross-sectional design; reverse causality cannot be ruled out
- **Systematic missingness** — low-income countries disproportionately missing math data; EII likely underestimates inequality for poorest nations
- **Sample loss** — 113 of 177 countries in final sample; lost countries have higher mean EII (0.396 vs 0.359), introducing conservative bias
- **GDP confounding** — not controlled for; most important next step
- **Country name matching** — string-based merge loses some countries (e.g. Côte d'Ivoire encoding issue); ISO 3166-1 codes recommended for future work
- **PCA limitation** — PC1 captures 49.2% of variance; 3 components needed for 80%
- **Happiness subjectivity** — Cantril Ladder has known cultural response-style biases

---

## Citation

```bibtex
@misc{seyedmohammadi2025inequality,
  author    = {Seyedmohammadi, Mobin},
  title     = {Is Educational Inequality a Stronger Predictor of 
               National Happiness Than Income Inequality?},
  year      = {2025},
  publisher = {GitHub},
  url       = {https://github.com/Mobin-Seyedmohammadi/inequality-happiness}
}
```

---

## License

MIT License. Data used subject to terms of respective sources (UNESCO, World Bank, World Happiness Report).
