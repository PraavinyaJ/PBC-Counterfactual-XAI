# PBC-Counterfactual-XAI

# Problem Statement
Primary Biliary Cholangitis (PBC) patients can present with a wide range of lab values and clinical signs.  
This project builds a binary classifier to predict:
- Target: `status == 2` vs. others (`status_eq_2`)  
- Goal: Produce useful risk probabilities (not just labels), and generate counterfactual “what-if” explanations showing what feature changes would move a high-risk case to a lower predicted risk in the model.

# Dataset
- Source file:(https://www.kaggle.com/datasets/homayoonkhadivi/primary-biliary-cirrhosis-pbc-disease-dataset)
  
# Clinical Significance: 
- Primary Biliary Cholangitis (PBC) is a chronic, progressive liver disease in which risk evolves gradually rather than as a binary outcome. Clinicians rarely make decisions based on a single threshold, instead, they monitor risk trajectories, worsening laboratory values, and overall disease progression to guide follow up intensity and treatment escalation.
- Risk stratification to guide follow-up intensity
- Trend sensitivity (e.g., worsening bilirubin or protime)
- Interpretability to support shared decision-making

# Feature Significance
- Bilirubin (bili) is a marker of hepatic excretory function and cholestasis. Rising bilirubin is a core indicator of disease severity and progression in PBC.
- Prothrombin time (protime) reflects liver synthetic function, as most coagulation factors are produced by the liver. Prolongation of protime indicates impaired hepatic synthesis and worse prognosis.
- Platelet count often decreases in advanced liver disease due to portal hypertension and splenic sequestration, making it a surrogate marker of disease progression.
- Albumin reflects liver synthetic reserve and nutritional status, with lower values associated with more advanced disease

# Counterfactual Interpretation (Clinical Logic)
- Counterfactual explanations in this project are used as model-level “what-if” analyses, not causal recommendations. They highlight which feature changes would move a patient from higher to lower predicted risk according to the trained model, under clinically plausible constraints.
- Indicators of which physiological dimensions (cholestasis, synthetic function, portal hypertension) are most influential for the model’s risk estimate
- Tools for hypothesis generation and shared decision-making
- Not as direct clinical interventions or causal guarantees

# Leakage Prevention:

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

# Test Set
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

2. Download dataset:
 pbc_clinical_cohort.csv from Kaggle and place it in the project root.

3. Run the notebook
jupyter notebook PBC_Counterfactuals_XAI.ipynb

# Future Work
- Validate on a new cohort. I want to test the model on an external PBC dataset (or a later time-split) to see if performance and calibration hold up outside this one sample.  
- Picking thresholds like a clinician would. Report sensitivity/specificity (and PPV/NPV) across a few thresholds instead of hard-coding 0.5, so the model can be tuned for catching more cases vs avoiding false alarms.  
- Make explanations/uncertainty more robust. I want to improve the counterfactual constraints with clinician review (so the what-if's feel realistic) and tighten the resampling uncertainty so it reflects the full fit and calibration procedure more cleanly.  





