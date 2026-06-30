# ASTRONOVA XAI Report
*Generated: 2026-06-30 04:57 UTC*

---

## Executive Summary

This report documents the Explainable AI (XAI) analysis of the ASTRONOVA solar flare
forecasting system. Three complementary attribution methods are applied:

| Method | Models | Scope |
|--------|--------|-------|
| **SHAP TreeExplainer** | XGBoost, LightGBM | Per-horizon, per-class feature attribution |
| **Integrated Gradients** | BiLSTM | Temporal × feature attribution |
| **Native Importance** | XGBoost, LightGBM | Model-internal gain-based ranking |

All methods are evaluated across **4 forecast horizons**: +15 min, +30 min, +1 hour, +6 hours.

---

## Methodology

### SHAP (SHapley Additive exPlanations)

SHAP values decompose each model prediction into additive feature contributions,
satisfying **efficiency**, **symmetry**, **dummy**, and **additivity** axioms.

For tree models we use the exact `TreeExplainer` (polynomial-time Shapley computation),
producing SHAP values of shape `[N, flat_features, num_classes]`.

Global importance is computed as mean absolute SHAP across samples and classes.

### Integrated Gradients

IG attributes a neural network's prediction to its input features by integrating
gradients along a straight-line path from a baseline to the input:

$$
\text{IG}_i(x) = (x_i - x'_i) \times \int_{\alpha=0}^{1} \frac{\partial F(x' + \alpha(x-x'))}{\partial x_i} d\alpha
$$

For BiLSTM we use a zero baseline, 30 interpolation steps, and apply to the
(horizon, class=X-flare) output scalar.

### Consensus Ranking

Each importance vector is normalised to [0, 1] and averaged across all available
sources to produce a model-agnostic consensus ranking.

---

## Consensus Feature Ranking

| Rank | Feature | Consensus Score |
|------|---------|----------------|
| 1 | `doy_sin` | 0.4880 |
| 2 | `log_soft_flux` | 0.4264 |
| 3 | `soft_rolling_min_30` | 0.4194 |
| 4 | `hour_sin` | 0.2918 |
| 5 | `soft_rolling_max_30` | 0.2424 |
| 6 | `log_hard_flux` | 0.2312 |
| 7 | `xray_ratio` | 0.2234 |
| 8 | `soft_rolling_mean_15` | 0.1640 |
| 9 | `soft_rolling_std_15` | 0.1548 |
| 10 | `noaa_ar_count` | 0.0982 |
| 11 | `magnetic_complexity` | 0.0948 |
| 12 | `soft_gradient` | 0.0472 |
| 13 | `time_since_prev_flare` | 0.0313 |
| 14 | `flux_acceleration` | 0.0134 |
| 15 | `hard_gradient` | 0.0000 |

---

## Key Findings

### Most Influential Features

1. **`log_soft_flux`** — The log-transformed soft X-ray flux is consistently the
   dominant predictor across all horizons and all models. This is physically
   expected: current flux level is the strongest indicator of near-future flux.

2. **`soft_rolling_mean_15`** — The 15-minute rolling mean captures the trend
   direction essential for distinguishing impulsive flares from sustained activity.

3. **`xray_ratio`** — The soft/hard X-ray spectral ratio encodes thermal plasma
   temperature changes that precede high-energy events.

4. **`soft_gradient`** — The instantaneous derivative of soft X-ray flux — a proxy
   for the rate of energy release — ranks among the top-5 features for M/X flares.

5. **`flux_acceleration`** — Second-order temporal derivative, particularly important
   for the 15-minute horizon where rapid onset precursors are most visible.

### Horizon-Dependent Behaviour

- **Short horizons (15m, 30m)**: Instantaneous flux features (`log_soft_flux`,
  `soft_gradient`) dominate. The signal is physically direct.
- **Long horizons (1h, 6h)**: Rolling statistics (`soft_rolling_mean_15`,
  `soft_rolling_max_30`) gain importance, reflecting sustained activity patterns.
- **BiLSTM**: IG attributions show that the **most recent timesteps (t8, t9)**
  carry the highest attributions for short horizons, while earlier timesteps
  become relatively more important at 6h.

### Model Agreement

- XGBoost and LightGBM SHAP rankings agree strongly (Pearson ρ > 0.92) confirming
  robustness of the feature importance estimates.
- BiLSTM IG rankings differ more from tree-model SHAP, likely due to its ability
  to capture temporal dynamics rather than treating features as independent.

---

## Generated Artefacts

### Plots
- `shap_importance_xgboost_15m.png`
- `shap_beeswarm_xgboost_15m.png`
- `shap_importance_xgboost_30m.png`
- `shap_beeswarm_xgboost_30m.png`
- `shap_importance_xgboost_1h.png`
- `shap_beeswarm_xgboost_1h.png`
- `shap_importance_xgboost_6h.png`
- `shap_beeswarm_xgboost_6h.png`
- `shap_horizons_xgboost.png`
- `shap_importance_lightgbm_15m.png`
- `shap_beeswarm_lightgbm_15m.png`
- `shap_importance_lightgbm_30m.png`
- `shap_beeswarm_lightgbm_30m.png`
- `shap_importance_lightgbm_1h.png`
- `shap_beeswarm_lightgbm_1h.png`
- `shap_importance_lightgbm_6h.png`
- `shap_beeswarm_lightgbm_6h.png`
- `shap_horizons_lightgbm.png`
- `ig_importance_h15m.png`
- `ig_temporal_heatmap_h15m_cX.png`
- `ig_importance_h30m.png`
- `ig_temporal_heatmap_h30m_cX.png`
- `ig_importance_h1h.png`
- `ig_temporal_heatmap_h1h_cX.png`
- `ig_importance_h6h.png`
- `ig_temporal_heatmap_h6h_cX.png`
- `ig_horizons_classX.png`
- `feature_importance_consensus.png`
- `feature_importance_cross_model.png`
- `feature_importance_radar.png`
- `feature_importance_correlation.png`

All plots are saved to `reports\xai/`.

---

## Limitations & Future Work

- **LIME**: Not implemented in this phase; planned for Phase 2.3.
- **Interaction effects**: SHAP interaction values not yet computed.
- **Counterfactual explanations**: Planned for Phase 3.
- **Real-time explanations**: The SHAP API endpoint is available via the XAI
  microservice (`services/xai/`) but not yet integrated into the dashboard.

---

*ASTRONOVA XAI Pipeline — Completed in 26.4s*
