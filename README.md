# Telco Customer Churn Analysis  
Preprocessing, Modeling & Business Evaluation

---

## Project Overview & Objective

This project builds an end-to-end customer churn analytics pipeline for a telecommunications business.

The core business question addressed is:

> **Which customers are most likely to churn, and which modeling approach best balances accuracy, interpretability, and deployability for a telco environment?**

Rather than prioritizing model complexity alone, this project focuses on:

- Data integrity and validation  
- Leakage-safe preprocessing  
- Fair model comparison  
- Business-aligned interpretation  

The goal is to produce churn predictions that are **accurate, explainable, and actionable**.

---

## Dataset Overview

The analysis uses the Telco Customer Churn dataset containing customer demographics, service subscriptions, billing information, and churn outcomes.

- **Records:** 7,043 customers  
- **Initial Features:** 21  
- **Target Variable:** `Churn` (Yes / No → 1 / 0)  
- **Class Distribution:** Moderately imbalanced  

---

## Data Inspection & Validation

Before any transformations, the dataset was inspected to ensure structural integrity.

### Validation Steps

- Verified dataset shape and column names  
- Confirmed presence of customer identifiers, services, billing, and churn label  
- Inspected churn class distribution  
- Checked for missing values across all columns  

### Key Findings

- No missing values after ingestion  
- All required columns present  
- Target variable is imbalanced, requiring careful evaluation metrics  

This step establishes a reliable baseline for all downstream analysis.

---

## Data Cleaning & Type Corrections

Targeted cleaning steps were applied to ensure numerical correctness.

### Cleaning Actions

- Converted `TotalCharges` from text to numeric using coercion  
- Replaced invalid numeric values with `0` to preserve all records  
- Encoded churn target:
  - **Yes → 1**
  - **No → 0**

### Outcome

- No rows dropped  
- Corrected data types  
- Dataset remains complete and modeling-ready  

---

## Relational Database Design (SQLite)

To simulate a production analytics workflow, the cleaned dataset was normalized into a relational schema and stored in SQLite.

### Tables Created

- **Customer** — demographics and tenure  
- **Services** — subscribed services and add-ons  
- **Contract** — contract type, billing, payment method  
- **Charges** — monthly and total charges  
- **Churn** — churn outcome  

Each table uses `customer_id` as the primary key and enforces referential integrity via foreign keys.

### Integrity Checks

- Verified all tables via `sqlite_master`  
- Each table contains exactly **7,043 rows**  

---

## SQL Joins & Dataset Reconstruction

All relational tables were joined using inner joins on `customer_id` to create a modeling-ready dataset.

### Resulting Dataset

- 15 analytical features  
- One row per customer  
- No missing values introduced  
- Fully relationally consistent  

This mirrors how churn features are assembled from operational systems in real-world environments.

---

## Feature Profiling & Statistical Summary

Descriptive statistics were generated for numeric features:

- `tenure`  
- `monthly_charges`  
- `total_charges`  
- `senior_citizen`  

### Observations

- Tenure and total charges are right-skewed  
- Billing variance reflects diverse customer lifecycles  
- Short-tenure customers show higher churn rates  

---

## Correlation Analysis

A correlation matrix was computed for numeric features.

### Key Insights

- **Tenure vs Churn:** Moderate negative correlation  
- **Monthly Charges vs Churn:** Weak positive correlation  
- **Tenure vs Total Charges:** Strong positive correlation  

These relationships align with expected telco customer behavior.

---

## Preprocessing Pipeline Design

To prevent data leakage and ensure reproducibility, a pipeline-based preprocessing approach was used.

### Feature Handling

- Dropped `customer_id` (identifier leakage prevention)  
- Separated numeric and categorical features  

### Transformations

- Numeric features → `StandardScaler`  
- Categorical features → `OneHotEncoder(handle_unknown="ignore")`  

All preprocessing was implemented using `ColumnTransformer`.

---

## Modeling Experiments

All models were evaluated using the same preprocessing pipeline and train/test split.

### Models Tested

**Linear Models**
- Logistic Regression  
- Logistic Regression (`class_weight="balanced"`)  
- Ridge Classifier  

**Tree-Based Models**
- Random Forest  
- Extra Trees  

**Boosting Models**
- Gradient Boosting  
- XGBoost  
- LightGBM  

Each model was tested **with and without PCA** where applicable.

---

## Model Performance Comparison (F1 Score)

| Rank | Model | F1 Score |
|-----:|------------------------------|---------:|
| 1 | Logistic Regression | **0.6212** |
| 2 | Logistic (Balanced) + PCA | 0.6167 |
| 3 | Extra Trees + PCA | 0.6112 |
| 4 | Random Forest | 0.6068 |
| 5 | Random Forest + PCA | 0.6062 |
| 6 | Extra Trees | 0.6045 |
| 7 | LightGBM | 0.5859 |
| 8 | Gradient Boosting | 0.5837 |
| 9 | Gradient Boosting + PCA | 0.5792 |
| 10 | XGBoost | 0.5772 |
| 11 | Logistic + PCA | 0.5698 |
| 12 | Ridge + PCA | 0.5655 |
| 13 | XGBoost + PCA | 0.5614 |
| 14 | Ridge | 0.5595 |
| 15 | LightGBM + PCA | **0.5526** |

---

## Business Interpretation of Experiments

### Why Linear Models Performed Best

- Churn drivers are largely additive  
- Contract type, tenure, and services interact linearly  
- Logistic Regression captures these effects clearly  

**Business implication:**  
Interpretable models are easier to deploy, explain, and trust.

---

### Why PCA Underperformed

- One-hot encoded categorical features contain meaningful churn signals  
- PCA compresses and removes useful variance  

**Business implication:**  
Dimensionality reduction should not be assumed beneficial for structured customer data.

---

### Why Complex Models Did Not Dominate

- Boosting models added complexity without consistent gains  
- Increased risk of overfitting marginal churn patterns  

---

## Best & Worst Performing Experiments

### Best Performing Model

**Logistic Regression (No PCA)**

- Highest F1 score  
- Stable and interpretable  
- Production-ready  

**Telco impact:**  
Ideal for identifying actionable churn drivers and guiding retention strategies.

---

### Worst Performing Model

**LightGBM + PCA**

- Lowest F1 score  
- PCA removed important categorical signals  

**Telco impact:**  
Aggressive dimensionality reduction can reduce churn detection accuracy and miss at-risk customers.

---

## Final Conclusion

This project demonstrates that **effective churn prediction depends more on data quality, structure, and disciplined evaluation than on model complexity**.

For a telecommunications provider:

- Retention efforts should prioritize **short-tenure and high-bill customers**  
- Interpretable models support **policy-level decision-making**  
- Logistic Regression offers the strongest balance of **performance, transparency, and deployability**

**Final Recommended Model:**  
**Logistic Regression (without PCA)**

---

