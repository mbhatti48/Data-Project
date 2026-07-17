# Data-Project

## Purpose

I'm a data and analytics professional with 4+ years across the public and private sectors, turning operational data into reporting that drives decisions. At the Government of Canada I worked hands-on with data, running queries against internal case-management databases, extracting and auditing records in Cognos, and producing reports that fed directly into operational decisions. I now handle data and analytics for a growing e-commerce business. I also hold a Data Science Certificate from the University of Waterloo, covering Python, statistics, machine learning, and big data, and I'm applying to data analyst, data scientist, and data engineer roles.

This repo holds my data projects, both academic coursework and self-taught builds. The work spans data cleaning and analysis, statistical modeling and machine learning, and pipeline and engineering work in Python, SQL, and Spark.

Each project has its own subfolder with its own README covering the problem, approach, and result.

## Projects

- [`financial-vulnerability-canada/`](financial-vulnerability-canada/) (2026) — What separates financially vulnerable Canadian families (those who skipped a payment due to hardship, ~4%) in Statistics Canada's 2023 Survey of Financial Security, and can they be predicted? Hypothesis testing (Welch t-test, chi-square) plus class-weighted Logistic Regression and Random Forest, tuned with GridSearchCV and threshold-adjusted for the rare class. The tuned forest catches 4 of 5 vulnerable families, AUC = 0.83, with a Bank of Canada debt-benchmark epilogue. Self-taught build.
- [`beer-popularity-prediction/`](beer-popularity-prediction/) (2023) — Predicting beer popularity from ~9M reviews (1996–2018) using pandas for data cleaning/feature engineering and PySpark/Spark MLlib for modeling. Linear Regression, R² = 0.70. Academic group project, Waterloo Data Science Certificate.
- [`hospital-readmission-prediction/`](hospital-readmission-prediction/) (2023) — Predicting 30-day hospital readmission for diabetic patients from a 10-year, 130-hospital clinical dataset using scikit-learn. Compared 8 classifiers, tuned Random Forest with GridSearchCV. AUC = 0.61, F1 = 0.63. Academic group project, Waterloo Data Science Certificate.
