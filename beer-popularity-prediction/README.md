# Beer Trendline & Popularity Prediction (1996–2018)

Academic group project — Big Data Tools & Management course, Waterloo Data Science Certificate (2022).

**Team (Group 14):** Mohammad Ali Bhatti, Arun A., Darwin A., Supratim D., Mary X.

## Problem

Given ~9 million beer reviews spanning 1996–2018 across three linked datasets (beers, breweries,
reviews), can we predict a beer's popularity score from its attributes (style, ABV, brewery
location)? The goal was to clean and join the three datasets, engineer a single popularity score per
beer, and train a regression model on top of it.

## Approach

- **Data cleaning & integration** (pandas): joined beers, breweries, and reviews into a single table,
  checked for orphaned/mismatched records across the joins, and resolved data quality issues (e.g. a
  mislabeled country/state pair).
- **Feature engineering:** built a `custom_score` target by averaging the five sub-ratings on each
  review (look, smell, taste, feel, overall), falling back to the dataset's own `score` field where
  sub-ratings were missing. Encoded the 112 beer styles as a numeric feature.
- **Modeling** (PySpark / Spark MLlib on Databricks): assembled features with `VectorAssembler` and
  trained/compared a Linear Regression model against a Random Forest, evaluated on a held-out test
  split.

## Result

**Linear Regression, R² = 0.70** — the best-performing model on the test set, per the team's final
report. Random Forest with additional hyperparameter tuning showed only marginal improvement over
Linear Regression.

## Repo contents

- `beer_popularity_prediction.ipynb` — the cleaned-up notebook: EDA, joins, feature engineering, and
  the model training/evaluation section.
- `Beer Popularity Prediction - Project Report.pdf` — the team's original written report (methodology + results write-up).

## A note on the modeling section

The data cleaning, EDA, and feature engineering in this notebook are the team's original work,
restored from our project files and tidied up for this repo. The original model-training notebook was
run directly in the team's Databricks workspace and was never exported back down as code — only the
final report and results survived. The modeling section here reconstructs that step to match the
documented methodology and result.

## Data

`beers.csv`, `breweries.csv`, and `reviews.csv` are not included in this repo due to size. Download
them from the [Brewery Dataset on Kaggle](https://www.kaggle.com/datasets/ankurnapa/brewery-dataset)
and place them in the repo root before running the notebook.

## Requirements

```
pandas
numpy
pyspark
```
