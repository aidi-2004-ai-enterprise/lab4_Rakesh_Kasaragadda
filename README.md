# lab4_Rakesh_Kasaragadda


## 1) Choosing the Initial Models
- **Benchmark = Logistic Regression** → simple, auditable baseline to prove lift.
- **Advanced = Random Forest + Gradient Boosting** → capture non-linear tabular patterns.
- **No clustering** → labels exist; supervised beats unsupervised for bankruptcy.
- **Evidence:** GB **PR-AUC 0.4194** > RF 0.4192 > LR 0.2747.

## 2) Data Pre-processing
- **Median imputation** (robust to skew/outliers).
- **Scale only LR** with StandardScaler; trees are scale-invariant.
- Data already numeric → no encoding needed.
- Do all steps **inside pipelines** to avoid leakage.

## 3) Handling Class Imbalance
- Severe imbalance (**~3.23%** bankrupt) → choose imbalance-aware metrics.
- **Stratified split** preserves minority ratio in both sets.
- **class_weight='balanced'** keeps all data; avoids SMOTE artifacts.
- Threshold tuning later if recall must be prioritized.

## 4) Outlier Detection & Treatment
- Outliers may indicate risk → **preserve unless clearly erroneous**.
- Remove only data-entry errors/duplicates.
- Trees tolerate outliers; LR stability handled via scaling.
- Review high-influence rows only if metrics/PSI flag issues.

## 5) Addressing Sampling Bias (Train/Test)
- Ran **PSI** per feature; all **< 0.10** (top ≈ **0.016**) → stable.
- Confirms train/test distributions aligned; low shift risk.
- No corrective re-split or reweighting required.
- Keep PSI as a guardrail for future resamples.

## 6) Data Normalization
- **Normalize LR only**; trees don’t need it.
- Avoid global scaling to keep ratios interpretable.
- Reduces pipeline complexity and distortion risk.
- Improves LR convergence/stability.

## 7) Testing for Normality
- **Not required** for RF/GB; LR tolerates non-normal inputs.
- Apply **log transform** only if LR underperforms due to heavy skew.
- Prefer simplicity unless diagnostics demand change.
- Monitor with residuals/learning curves.

## 8) Dimensionality Reduction (PCA)
- **Pros:** reduces noise/collinearity; speeds training.
- **Cons:** loses interpretability (important for risk).
- **Decision:** **skip PCA now**; revisit only if overfitting appears.
- Use model importances/regularization first.

## 9) Feature Engineering
- Dataset already rich (95+ ratios) → diminishing returns.
- Add **few targeted interactions** only if metrics stall.
- Keep features minimal for explainability.
- Avoid synthetic features that obscure business meaning.

## 10) Testing & Addressing Multicollinearity
- Inspect **correlation heatmap**; compute **VIF** if interpreting LR.
- High collinearity inflates LR variance; trees less affected.
- Drop/merge **redundant (>0.9)** pairs.
- Keep the more predictive feature (via CV/importance).

## 12) Feature Selection Methods
- **Correlation filter → RF/GB importances** to trim low-value vars.
- Optional **L1 (lasso)** for linear sparsity.
- Too many → overfit; too few → underfit → stop when **CV PR-AUC** plateaus.
- Retain interpretable core drivers.

## 13) Hyperparameter Tuning
- **Random Search** first (fast, broad) on depth/estimators/learning rate.
- **Small Grid** around best region from random search.
- **Stratified 5-fold CV** with **PR-AUC** objective.
- Respect compute budget; early stopping where available.

## 14) Cross-Validation Strategy
- **Stratified K-Fold (k=5)** to maintain ratios per fold.
- Keep preprocessing **inside** the CV pipeline.
- Report **mean ± std PR-AUC**.
- Use fixed seeds for reproducibility.

## 15) Evaluation Metrics Selection
- Use **`predict_proba`** → enables threshold tuning (0.5 is arbitrary).
- **Primary:** **PR-AUC** (baseline ≈ **0.0323**) + **F1**; track **Recall**.
- **Secondary:** ROC-AUC / Precision for trade-off visibility.
- Evidence: **GB PR-AUC 0.4194**; **LR Recall 0.7121** at 0.5 (more FPs).

## 16) Evaluating Drift & Model Degradation
- Compute **PSI** train vs test; all **< 0.10** → **no drift now**.
- Monitor PSI over time; **alert > 0.20** → investigate/retrain.
- Track production **PR-AUC/F1/Recall** for degradation.
- Periodic shadow testing with recent data.

## 17) Interpreting Model Results & Explainability
- Use **tree feature importances** for quick global drivers.
- Add **SHAP** on chosen model for regulator-friendly local/global explanations.
- Likely top drivers: **Cash flow rate**, **Fixed Assets/Assets**, **Inventory/Working Capital**.
- Improves trust and actionability for risk teams.

---


- **Class balance:** ~96.8% non-bankrupt / **3.2% bankrupt** (both splits).
- <img width="590" height="390" alt="image" src="https://github.com/user-attachments/assets/816088de-7b7c-4177-9f24-145035de4d8b" />

- **PSI top features:** 0.016 (Cash flow rate), 0.0158 (Fixed Assets/Assets), 0.0150 (Inventory/Working Capital) → **stable**.
- **Model metrics (thr=0.5):** GB **PR-AUC 0.4194**, RF 0.4192, LR 0.2747; LR **Recall 0.7121**.
