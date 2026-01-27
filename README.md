# PBC-Counterfactual-XAI

# Problem Statement
Primary Biliary Cholangitis (PBC) patients can present with a wide range of lab values and clinical signs.  
This project builds a binary classifier to predict:
- Target: `status == 2` vs. others (`status_eq_2`)  
- Goal: Produce useful risk probabilitiesv(not just labels), and generate counterfactual “what-if” explanations showing what feature changes would move a high-risk case to a lower predicted risk in the model.

# Dataset
- Source file: [`pbc_clinical_cohort.csv`](https://www.kaggle.com/datasets/homayoonkhadivi/primary-biliary-cirrhosis-pbc-disease-dataset)
  
# Clinical Significance: Why does this matter for a patient?
In chronic liver disease, risk is rarely yes/no. Clinicians often want:
- Risk stratification to guide follow-up intensity
- Trend sensitivity (e.g., worsening bilirubin or protime)
- Interpretability to support shared decision-making

# Leakage Prevention: How did I ensure the model didn't cheat?

# Split discipline
- I created a strict Train / Val / Test split once.
- Test is touched once at the end for reporting.
  
# Preprocessing without leakage
- Preprocessing stats (sex mode, numeric medians) is fitted only on the fit split.
- During CV: preprocessing is re-fitted inside each fold on the fold’s training portion only.

# Calibration without leakage
- The base model is fit on X_fit
- Calibration is learned on a held-out calibration subset (X_cal)
  
# Approach:

# Models
- Gradient Boosting (uncalibrated)
- Gradient Boosting and sigmoid calibration
- Logistic Regression baseline

# Metrics
- ROC-AUC
- Accuracy at threshold 0.5
- Brier score

# Key results


# Visuals
- Calibration curve (Test)
- Top feature importances (GB)
- Predicted probability change (baseline vs best CF) 
- Distributions for changed features: Distribution protime, Distribution platelet, Distribution bili

# Limitations
- Single dataset and a simple feature set, no external validation.
- Threshold fixed at 0.5 so accuracy depends on threshold choice.
- DiCE counterfactuals depend on constraints, and different constraints can change results.
- The reported bootstrap 2.5–97.5% range is best interpreted as an instability band under retraining and recalibration, not a classical confidence interval for the exact final fitted model.
  
# How to Run
1. Install dependencies
pip install -r requirements.txt
2. Download dataset
Download pbc_clinical_cohort.csv from Kaggle and place it in the project root.
3.Run the notebook
jupyter notebook PBC_Counterfactuals_XAI.ipynb

# Future Work
- External validation: Evaluate on an independent PBC cohort (different site/time period) to test generalization and calibration stability.
- Threshold selection + operating points: Report sensitivity/specificity (and PPV/NPV) across multiple thresholds instead of fixing 0.5; optionally add decision-curve analysis for clinical utility.
- More robust probability calibration: Compare sigmoid vs isotonic calibration using repeated CV or cross-fitting to reduce split sensitivity; consider calibration-in-the-loop evaluation.
- Uncertainty reporting (cleaner than current bootstrap):Bootstrap or cross-fit the *entire* fit + calibration procedure (not just retrain + fixed-val recalibration) so uncertainty bands align better with the reported point estimate.
- Counterfactual realism upgrades: Add a cost function (penalize large/unrealistic lab shifts), clinician-reviewed constraints, and/or pathway constraints (e.g., linked labs) to reduce “implausible but valid” counterfactuals.
- Model comparisons:Try stronger tabular baselines (e.g., XGBoost/LightGBM if allowed, or tuned GB/LR pipelines) and report whether improvements are consistent under repeated splits.
- Fairness and subgroup checks: Audit performance/calibration across clinically relevant subgroups (sex, stage categories) and report any major disparities.
- Feature engineering + missingness: Model missingness explicitly (missingness indicators) and evaluate whether it improves robustness without introducing leakage.



