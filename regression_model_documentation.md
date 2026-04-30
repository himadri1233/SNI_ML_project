

#  SNI Aging Prediction – Regression & Feature Importance

## Overview

This project implements an **end‑to‑end machine learning regression pipeline** on Databricks to predict **`Dm_Aging_Days`** — the aging of **Shipped but Not Invoiced (SNI)** sales‑order line items.

The solution leverages **Spark for large‑scale data processing**, **scikit‑learn pipelines**, and **gradient boosting models** (XGBoost, LightGBM, CatBoost) to deliver both **high prediction accuracy** and **model interpretability** via feature importance.

***

## Business Objective

Late invoicing of shipped orders drives **revenue leakage**, **forecast inaccuracies**, and **manual follow‑ups**.  
This model predicts how long an order line item will remain uninvoiced, enabling:

*   Early identification of **high‑risk aging orders**
*   Proactive operational intervention
*   Improved invoice planning and cash‑flow visibility

***

## Architecture & Environment

*   **Execution Platform:** Databricks (Spark enabled)
*   **Languages:** PySpark + Python
*   **ML Libraries:**
    *   XGBoost
    *   LightGBM
    *   CatBoost
    *   scikit‑learn
*   **Storage:** Hive tables + Delta Lake
*   **Time Zone Handling:** UTC enforced to prevent leakage

***

## Data Source

The dataset is sourced from the enterprise Hive view:

    supplychain.shipped_not_invoiced.fact_enterprise_shipped_not_invoiced_legacy_combined_reporting_view

Each record represents a **Sales Order Line Item** with attributes spanning:

*   Order creation & scheduling dates
*   Logistics & shipping details
*   Customer and product hierarchies
*   Pricing and billing flags
*   Delivery and processing statuses

***

## Data Preparation Pipeline

### 1. Snapshot De‑duplication

Sales orders can appear multiple times due to daily snapshots.  
To avoid duplicates:

*   A **window function** partitions by:
    *   `Sales_Order_Identifier`
    *   `Sales_Order_Line_Item_Identifier`
*   Only the **latest snapshot (`max(Dm_Date)`)** is retained

***

### 2. Date‑Based Filtering

To ensure clean and complete data:

*   Only shipments **on or after 1‑Jan‑2021** are included
*   Rows with snapshot dates **>= yesterday (UTC)** are excluded  
    → prevents partial or in‑flight records from leaking into training

***

### 3. Feature Selection

Over **40 predictor variables** are selected across:

*   Order metadata
*   Logistics & routing
*   Customer segmentation
*   Product hierarchy
*   Financial attributes

**Target variable:**

    Dm_Aging_Days

***

### 4. Missing Value Handling

A rigorous and scalable missing‑data strategy is applied:

#### a. Placeholder Cleanup

*   `"?"` values are treated as **missing**
*   Empty strings are normalized to nulls

#### b. Imputation

*   **Categorical features:** filled with `"Unknown"`
*   **Numerical features:** median imputation using `approxQuantile` (Spark‑safe)

***

### 5. High‑Missing Feature Removal

To reduce noise:

*   Categorical columns with **>70% `"Unknown"` values** are dropped
*   Remaining features provide sufficient signal density

***

### 6. Date Encoding

Date fields are converted into numeric values:

    Days since 2021‑01‑01

This enables compatibility with tree‑based regression models.

***

### 7. Train‑Test Split

*   Data is converted from Spark → Pandas
*   Split:
    *   **80% Training**
    *   **20% Testing**
*   Fixed random seed ensures reproducibility

***

## Feature Engineering & Encoding

### One‑Hot Encoding

*   Categorical variables are encoded using `OneHotEncoder`
*   Numerical variables pass through unchanged
*   Implemented using a **scikit‑learn ColumnTransformer**

***

### Correlation Analysis

*   Correlation is computed on **pre‑encoded features** (to avoid sparse explosion)
*   Highly correlated feature pairs (`|corr| > 0.85`) are identified
*   Selected features are removed to reduce multicollinearity

***

## Model Training

Multiple regression models are trained and evaluated:

### Models Used

*   **XGBoost Regressor**
*   **LightGBM Regressor**
*   **CatBoost Regressor**

### Evaluation Metrics

*   **RMSE** – prediction error magnitude
*   **R²** – variance explained

All models are evaluated on the **same test dataset** for fair comparison.

***

## Model Selection

*   Models are ranked by **RMSE**
*   XGBoost consistently delivers the **best performance**
*   A production‑ready **XGBoost Pipeline** is finalized

***

## Feature Importance & Explainability

### Permutation Importance

*   Uses **Permutation Importance** on the trained pipeline
*   Measures **ΔRMSE impact** when a feature is shuffled
*   Provides **model‑agnostic, reliable feature importance**

### Output

*   Top 20 most influential features are visualized
*   Helps stakeholders understand *why* predictions are high or low

***

## Hyperparameter Tuning

### Approach

*   **RandomizedSearchCV** with K‑Fold cross‑validation
*   Controlled parallelism and resource limits
*   RMSE used as tuning objective

### Advanced Training

*   Model checkpointing
*   Resume‑from‑checkpoint support
*   Early stopping to prevent overfitting

***

## Final Model Evaluation

The tuned final model is evaluated on the test set using:

*   RMSE
*   MAE
*   R²

This confirms generalization performance before deployment.

***

## Output & Consumption

*   Predictions are designed to be merged into a **Delta Lake table**
*   Enables:
    *   Daily batch inference
    *   Upserts via MERGE logic
    *   Downstream analytics & dashboards

***

## Key Takeaways

*   Scalable Spark + pandas hybrid design
*   Strong data quality & leakage prevention
*   Interpretable ML via permutation importance
*   Production‑ready architecture aligned with enterprise standards

***

