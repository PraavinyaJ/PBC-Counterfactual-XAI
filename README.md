# PBC-Counterfactual-XAI

# Problem Statement
Primary Biliary Cholangitis (PBC) patients can present with a wide range of lab values and clinical signs.  
This project builds a binary classifier to predict:
- Target: `status == 2` vs. others (`status_eq_2`)  
- Goal: Produce useful risk probabilities (not just labels), and generate counterfactual “what-if” explanations showing what feature changes would move a high-risk case to a lower predicted risk in the model.

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

# (Test Set)
| Model | ROC-AUC | Accuracy | Brier |
|---|---:|---:|---:|
| GB (uncalibrated) | 0.812 | 0.738 | 0.176 |
| GB (calibrated) | 0.812 | 0.726 | 0.186 |
| Logistic Regression | 0.772 | 0.714 | 0.183 |

# Cross-Validation Summary 
| Model | ROC-AUC | Accuracy | Brier |
|---|---:|---:|---:|
| GB (uncalibrated) | 0.840 | 0.776 | 0.155 |
| GB (calibrated) | 0.840 | 0.776 | 0.159 |
| Logistic Regression | 0.823 | 0.752 | 0.168 |

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

3. Run the notebook
jupyter notebook PBC_Counterfactuals_XAI.ipynb

# Future Work
- Validate on a new cohort. I want to test the model on an external PBC dataset (or a later time-split) to see if performance and calibration hold up outside this one sample.  
- Picking thresholds like a clinician would. Report sensitivity/specificity (and PPV/NPV) across a few thresholds instead of hard-coding 0.5, so the model can be tuned for catching more cases vs avoiding false alarms.  
- Make explanations/uncertainty more robust. I want to improve the counterfactual constraints with clinician review (so the what-if's feel realistic) and tighten the resampling uncertainty so it reflects the full fit and calibration procedure more cleanly.  





