

# SNI Aging Risk Prediction — Binary Classification

## Overview

This project implements an **end‑to‑end binary classification pipeline** on Databricks to predict whether a **Shipped‑Not‑Invoiced (SNI)** sales order line item is **at risk of aging**.

The notebook combines **Spark‑based feature engineering** with **scikit‑learn pipelines** and multiple classification models to produce:

*   a **binary risk flag (`sni_label`)**
*   a **tunable decision threshold** aligned to business recall requirements
*   **explainable feature importance** using permutation methods

***

## Business Objective

Shipped orders that remain uninvoiced for extended periods create:

*   revenue uncertainty
*   operational follow‑ups
*   reporting inaccuracies

This model predicts **SNI aging risk** early by classifying each order line item into:

*   **0 → Non‑Aging**
*   **1 → Aging / At‑Risk**

The output enables proactive intervention, monitoring, and prioritization.

***

## Execution Environment

*   **Platform:** Databricks (Apache Spark)
*   **Languages:** PySpark, Python
*   **ML Framework:** scikit‑learn
*   **Models:**  
    XGBoost, LightGBM, CatBoost, Logistic Regression, Random Forest, Linear SVM, Histogram Gradient Boosting
*   **Storage:** Hive tables + Delta Lake
*   **Timezone:** UTC enforced throughout the pipeline

***

## Data Source

The model is trained on the enterprise SNI fact view:

    supplychain.shipped_not_invoiced.fact_enterprise_shipped_not_invoiced_legacy_combined_reporting_view

Each row represents a **sales order line snapshot** with:

*   order metadata
*   shipment and delivery events
*   customer and product attributes
*   commercial and financial fields

***

## Snapshot Handling & Filtering

### Latest Snapshot Selection

Because the source table contains historical snapshots:

*   the notebook selects the **maximum available `Dm_Date`**
*   deduplicates by:
    *   `Sales_Order_Identifier`
    *   `Sales_Order_Line_Item_Identifier`

This ensures one authoritative record per order line.

### Date Filters

*   Shipments are restricted to **on or after 1‑Jan‑2021**
*   Prevents training on obsolete process behavior

***

## Target Engineering

The binary target column `sni_label` is created as follows:

*   **0 (Non‑Aging)**  
    Values mapped from:
    *   `NON-AGING`
    *   `CONSOLIDATED INVOICE - F`

*   **1 (Aging / At‑Risk)**  
    All remaining aging indicators

This mapping ensures clean supervision and consistent downstream evaluation.

***

## Feature Governance & Leakage Prevention

Explicit **keep / drop lists** are defined to enforce feature hygiene:

### Dropped Categories

*   ID‑like fields and free‑text columns
*   system timestamps and load metadata
*   known leakage columns (e.g. aging days, SNI status, invoice flags, quantities revealing the label)

### Retained Categories

*   **Categorical features:** logistics, routing, customer segmentation, product hierarchy, fulfillment context
*   **Numeric features:** quantities, values, SLA metrics

This design prevents the model from indirectly learning the target.

***

## Feature Engineering

### Time‑Delta Features

Raw dates are temporarily retained to engineer meaningful signals such as:

*   `days_order_to_shipment`
*   `days_shipment_to_pod`
*   `days_planned_vs_actual_shipment`
*   `days_order_to_planned_invoice`
*   `days_crdd_minus_tdd`

After feature creation, **raw date columns are dropped**.

### Delivered Ratio

A bounded ratio is engineered:

    delivered_ratio = Actual_Quantity_Delivered / Shipment_Quantity

*   handles division‑by‑zero
*   clamped to reasonable bounds to reduce outlier impact

***

## Data Cleaning & Type Enforcement

*   Placeholder values (`?`, `NA`, empty strings, etc.) are converted to nulls
*   Numeric columns are explicitly cast to `double`
*   Categorical columns are trimmed and normalized
*   Ensures consistency before exporting to Pandas

***

## Export to Pandas

After final feature selection:

*   the dataset is collected into Pandas
*   optional down‑sampling is supported for memory control
*   features and labels are separated into `X` and `y`

This enables flexible modeling using scikit‑learn.

***

## Modeling Pipelines

Reusable **scikit‑learn Pipelines** are constructed with:

### Preprocessing

*   **Categorical:**  
    `SimpleImputer → OneHotEncoder(handle_unknown='ignore')`
*   **Numerical:**  
    `SimpleImputer(strategy='median')`
*   Version‑safe handling of sparse outputs

### Models Trained

*   XGBoost Classifier (primary)
*   LightGBM Classifier
*   CatBoost Classifier
*   Logistic Regression
*   Random Forest
*   Linear SVM (with calibration)
*   Histogram Gradient Boosting

Class imbalance is handled using:

*   `scale_pos_weight` (tree models)
*   `class_weight='balanced'` (linear / RF)

***

## Training, Evaluation & Threshold Tuning

### Metrics

Models are evaluated using:

*   **ROC‑AUC**
*   **PR‑AUC (Average Precision)**

### Threshold Optimization

Instead of default 0.5:

*   probability thresholds are swept (0.3 → 0.7)
*   thresholds are selected to meet a **target recall (e.g. 80%)**
*   tie‑breaking logic favors recall, then precision

This aligns model behavior with business risk tolerance.

***

## Hyperparameter Tuning

RandomizedSearchCV is applied to:

*   **LightGBM**
*   **XGBoost**

Key characteristics:

*   PR‑AUC as the tuning objective
*   constrained search space for stability
*   reduced iteration counts for performance
*   stratified cross‑validation

***

## Correlation Analysis & Feature Reduction

Correlation is computed on **pre‑encoded features** to avoid sparse explosion.

Highly correlated features (`|corr| > 0.85`) are pruned, for example:

*   overlapping quantity metrics
*   redundant value columns
*   near‑duplicate routing and channel signals

A reduced feature set is retrained and re‑tuned to validate performance stability.

***

## Feature Importance

### Permutation Importance

Using the **tuned XGBoost (reduced)** model:

*   features are permuted one at a time
*   impact on **PR‑AUC** is measured
*   provides model‑agnostic interpretability

Top drivers of SNI aging risk can be inspected and visualized.

***

## Model Persistence

Final artifacts include:

*   trained scikit‑learn pipeline (`model.joblib`)
*   metadata JSON:
    *   library versions
    *   platform info
    *   creation timestamp
    *   selected threshold

This ensures reproducibility and traceability.

***

## Production Scoring

*   The persisted model is loaded back into Spark
*   Feature engineering is applied consistently
*   Predictions are generated via vectorized Pandas UDFs
*   Results are written to Delta tables containing:
    *   order identifiers
    *   shipment date
    *   predicted `sni_label`

***

## Key Design Principles

*   Strict data‑leakage prevention
*   Scalable Spark → sklearn bridge
*   Thresholds aligned to business recall
*   Explainable outputs for stakeholders
*   Production‑ready artifact design

***
