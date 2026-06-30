# Hospital Readmission Prediction (10-Year Diabetes Dataset)

Academic group project — Machine Learning course, Waterloo Data Science Certificate (2023).

**Team (Group 14):** Jamie Bennion, Mohammad Bhatti, Meriem Meghari, Elaheh Nahvi, Jemar Ugale

## Problem

Using 10 years (1999–2008) of clinical data from 130 US hospitals (~100K inpatient encounters,
50+ features), predict whether a diabetic patient will be readmitted to hospital within 30 days of
discharge — a binary classification problem on a meaningfully imbalanced target (88.84% not
readmitted within 30 days).

## Approach

- **Preprocessing:** cleaned `?` placeholder values, dropped identifier columns and high-cardinality
  diagnosis/specialty fields, collapsed the 3-way `readmitted` field into a binary target, dropped
  hospice/death discharge dispositions (can't meaningfully "readmit"), and encoded the remaining
  categorical features (one-hot for low-cardinality, ordinal for age, binary encoding via
  `category_encoders` for high-cardinality admission/discharge/source IDs to control dimensionality).
- **Class imbalance:** balanced the training set by undersampling the majority class (test set left
  untouched, so evaluation still reflects real-world class proportions).
- **Modeling:** compared 8 classifiers (Logistic Regression, LDA, KNN, Random Forest, Naive Bayes,
  AdaBoost, Gradient Boosting, Extra Trees) at default settings, then tuned Random Forest with
  `GridSearchCV` over depth, estimator count, max features, and min samples split.

## Result

**Tuned Random Forest: AUC = 0.61, weighted-average F1 = 0.63.** Hospital readmission is a
genuinely hard prediction problem — this is modestly better than chance, consistent with published
results on this dataset rather than a sign of a modeling mistake.

The team's official project report compared Logistic Regression, Random Forest, and Gradient
Boosting after tuning and selected **Gradient Boosting as the team's official pick** (~58.4% test
accuracy, F1 = 0.614) — close to, though not identical to, the tuned Random Forest result in this
repo. Both are documented in the README and notebook for context.

## Repo contents

- `hospital_readmission_prediction.ipynb` — the merged, cleaned-up notebook: EDA, preprocessing,
  feature encoding, class-imbalance handling, and model training/evaluation.

## A note on this notebook's origin

Two notebooks existed for this project: one with more rigorous, well-documented EDA and feature
encoding, and one with the actual hyperparameter-tuned model that produced the AUC/F1 result above.
This repo merges the best of both into a single notebook — the EDA/preprocessing is the more
thorough version, with the modeling section adapted onto it. The class-imbalance (undersampling)
step is reconstructed to match the documented approach, since the original notebook's resampling
code wasn't preserved (see the comment in that cell). Everything else — the encoding decisions, the
model comparison, the hyperparameter grid — is the team's original work.

## Data

`diabetes.csv` is not included in this repo due to size. Download it from the
[10 Years Diabetes Dataset on Kaggle](https://www.kaggle.com/datasets/jimschacko/10-years-diabetes-dataset)
and place it in the repo root before running the notebook.

## Requirements

```
pandas
numpy
scikit-learn
category_encoders
seaborn
matplotlib
tabulate
```
