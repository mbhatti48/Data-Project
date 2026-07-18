# Hospital Readmission Prediction (10-Year Diabetes Dataset)

A machine-learning analysis of ~99,000 hospital encounters of ~70,000 diabetic patients from 130 US hospitals (1999–2008). The project asks one question: **can we tell, at the moment of discharge, which patients are most at risk of being back in hospital within 30 days?**

About 11% of discharges lead to a readmission within the 30-day window — the standard hospital-quality benchmark — making this a heavily imbalanced classification problem on real clinical data.

## Key findings

- **Readmission risk is driven by a simple story.** Two very different models — a logistic regression read through odds ratios and a gradient-boosted model read through feature importances — independently agree on the drivers: the patient's hospitalization history (the dominant signal in both — recent visits, prior stays on record, and whether the last stay already ended in a readmission), where they were discharged to, their age, and their primary diagnosis. Agreement between two unrelated models is good evidence the signal is real.
- **Prior hospitalizations dominate.** Encounters with 3+ inpatient stays in the previous year end in readmission 26.4% of the time — three times the rate of first-time patients (8.6%). Discharge destination is the clear second signal: 9.3% for patients sent home vs 16.3% for those sent to another facility.
- **The evaluation is patient-leakage-free by design.** Thousands of patients appear multiple times in the data; the train/test split and every cross-validation fold are grouped by patient, so no patient is ever both trained and tested on — without discarding the repeat-visit patients a readmission model most needs to learn from.
- **Readmission is genuinely hard to predict.** The tuned model reaches a **test AUC of 0.67**, close to the practical ceiling for this feature set and in line with published results on this dataset. Used as a screening tool with a recall-oriented threshold, it catches about 3 in 5 readmissions while flagging about 38% of discharges for follow-up.

## What the project covers

- **Section 2, Cleaning.** Making `?` placeholders visible as real missing data, sizing the repeat-patient leakage trap, removing encounters that can't be readmitted (deaths and hospice discharges), and building the binary 30-day target.
- **Section 3, Exploration.** The class imbalance, and how readmission rates move with age, prior hospitalizations, and discharge destination.
- **Section 4, Feature preparation.** Grouping coded ID fields into named categories, mapping 700+ ICD-9 diagnosis codes to nine readable chapters, compressing 23 per-drug columns into four medication features, engineering patient-history features built strictly from each patient's earlier stays, and a preprocessing pipeline that fits its scaler on training folds only.
- **Section 5, Modeling.** Metric priority stated up front (AUC first, recall at a screening threshold second, accuracy last), a patient-grouped train/test split with patient-grouped CV folds, four candidates compared by cross-validated AUC, the leader tuned by grid search, one evaluation on the held-out test set, and an operating threshold chosen for screening.
- **Section 6, Interpretation.** Odds ratios versus feature importances — what the two models agree on, and how to read direction from one and ranking from the other.

## Data

`diabetes.csv` is not included in this repo due to size. Download it from the
[10 Years Diabetes Dataset on Kaggle](https://www.kaggle.com/datasets/jimschacko/10-years-diabetes-dataset)
and place it next to the notebook. A feature reference is in the notebook's appendix.

## Running it

```
pip install -r requirements.txt
jupyter notebook hospital_readmission_prediction.ipynb
```

The notebook runs top to bottom once the data file is in place (a few minutes; the grid search is the slow cell).

## Tools

Python, with pandas and numpy for the data work, matplotlib for visualisation, and scikit-learn for the modelling.
