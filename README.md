# PBC-Counterfactual-XAI

## Summary
- **Goal:** Predict high-risk PBC status (**status == 2**) and generate counterfactual “what-if” explanations.
- **Data:** Kaggle PBC clinical cohort (**n=419**).
- **Best model:** Gradient Boosting (Test **ROC-AUC 0.812**, **Brier 0.176**).
- **Key methods:** leakage-aware Train/Val/Test split, calibration-aware evaluation, DiCE counterfactuals with clinical constraints.
- **Outputs:** calibration curve, feature importance, counterfactual edits + feature distribution plots.

---

## Problem Statement
Primary Biliary Cholangitis (PBC) patients can present with a wide range of lab values and clinical signs.  
This project builds a binary classifier to predict:
- **Target:** `status == 2` vs. others (`status_eq_2`)
- **Goal:** produce useful **risk probabilities** (not just labels) and generate **counterfactual “what-if” explanations** showing what feature changes would reduce predicted risk.

## Dataset
- Source: Kaggle PBC dataset  
  https://www.kaggle.com/datasets/homayoonkhadivi/primary-biliary-cirrhosis-pbc-disease-dataset

---

## Approach (high level)
- Build baseline and non-linear models for **risk stratification** (probabilities).
- Evaluate both **ranking performance** (ROC-AUC) and **probability quality** (Brier + calibration curve).
- Generate **counterfactual explanations** under **clinically plausible constraints**.

### Models
- Gradient Boosting (uncalibrated)
- Gradient Boosting + sigmoid calibration
- Logistic Regression baseline

### Metrics
- ROC-AUC
- Accuracy at threshold 0.5
- Brier score

---

## Key Results

### Test Set (held out; touched once for final reporting)
| Model | ROC-AUC | Accuracy | Brier |
|---|---:|---:|---:|
| GB (uncalibrated) | 0.812 | 0.738 | 0.176 |
| GB (calibrated) | 0.812 | 0.726 | 0.186 |
| Logistic Regression | 0.772 | 0.714 | 0.183 |

### Cross-Validation Summary (5-fold on Train+Val; fold-specific preprocessing + calibration)
| Model | ROC-AUC | Accuracy | Brier |
|---|---:|---:|---:|
| GB (uncalibrated) | 0.840 | 0.776 | 0.155 |
| GB (calibrated) | 0.840 | 0.776 | 0.159 |
| Logistic Regression | 0.823 | 0.752 | 0.168 |

**Interpretation:** Gradient Boosting achieved the strongest ROC-AUC in this setup. In this run, sigmoid calibration did **not** improve Brier (it slightly worsened it), highlighting split sensitivity at this sample size.

---

## Clinical Significance (why probabilities + interpretability)
PBC is chronic and progressive; risk evolves over time rather than as a binary outcome. Clinicians monitor **trends** in labs and clinical signs to guide follow-up intensity and treatment escalation. This motivates:
- **risk stratification** (probabilities, not just labels)
- sensitivity to meaningful lab changes (e.g., bilirubin, prothrombin time)
- interpretability to support shared decision-making

## Feature Significance (short clinical intuition)
- **Bilirubin (bili):** marker of cholestasis and disease severity.
- **Prothrombin time (protime):** reflects liver synthetic function; prolongation suggests worse prognosis.
- **Platelets:** often decrease in advanced disease (portal hypertension / splenic sequestration).
- **Albumin:** synthetic reserve and nutritional status; lower values often indicate progression.

---

## Counterfactual Interpretation (how to read the what-ifs)
Counterfactual explanations here are **model-level “what-if” analyses**, not causal recommendations.
They identify feature changes that would move a patient from higher to lower predicted risk **according to the trained model**, under clinically plausible constraints. They are useful for:
- highlighting which physiological dimensions (cholestasis, synthetic function, portal hypertension) drive the model’s risk estimate
- hypothesis generation and shared decision-making
- **not** direct clinical interventions or causal guarantees

---

## Leakage Prevention
- **Single Train/Val/Test split (60/20/20).** Test set used once for final reporting.
- **No preprocessing leakage:** imputation stats fit on training data only (and re-fit within each CV fold).
- **Calibration without leakage:** base model fit on an inner fit split; sigmoid calibration learned on a held-out calibration split.
- **CV on Train+Val:** fold-specific preprocessing + calibration to avoid optimistic estimates.

---

## Visuals
- Calibration curve (Test)
- Top feature importances (GB)
- Predicted probability change (baseline vs best counterfactual)
- Distributions for changed features (e.g., `protime`, `platelet`, `bili`)

---

## Limitations
- Single dataset and a simple feature set; no external validation.
- Threshold fixed at 0.5, so accuracy depends on threshold choice.
- DiCE counterfactuals depend on constraints; different constraints can change results.
- **Bootstrap note:** the reported 2.5–97.5% range is best interpreted as an **instability band** under repeated **model refits + recalibration** (not a classical CI for one fixed final model).

---

## How to Run
1. Install dependencies:
   ```bash
   pip install -r requirements.txt
2. Download dataset: pbc_clinical_cohort.csv from Kaggle and place it in the project root.
3. Run the notebook: jupyter notebook PBC_Counterfactuals_XAI.ipynb
