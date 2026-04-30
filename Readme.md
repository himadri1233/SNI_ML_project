

***

#  Shipped Not Invoiced (SNI) – Machine Learning POC

## Overview

This repository contains a **Proof of Concept (POC)** that applies **Machine Learning (ML)** to improve a key **Supply Chain business KPI: *Shipped Not Invoiced (SNI)***.

**Shipped Not Invoiced (SNI)** refers to sales orders where:

*   the product has already been **physically shipped**
*   but the **invoice has not yet been generated**

Until an invoice is created, **cash flow is blocked**, directly impacting working capital and financial visibility.

This POC demonstrates how ML can be used to:

*   **identify SNI risk early**
*   **predict how long an order may remain uninvoiced**
*   **alert business teams** so corrective action can be taken earlier

***

## Business Problem

In large-scale supply chains:

*   thousands of orders are shipped daily
*   manual monitoring of invoicing delays is inefficient
*   aging SNI orders often surface **only after they become critical**

Key challenges:

*   Limited early visibility into which orders will age
*   Reactive rather than proactive intervention
*   Direct impact on **cash flow and revenue realization**

***

## POC Objective

The goal of this POC is to build an **intelligent, automated mechanism** that:

1.  Identifies **which shipped orders are likely to become aging SNI**
2.  Predicts **how long an order is expected to remain uninvoiced**
3.  Generates **actionable insights** that allow the business to:
    *   prioritize follow‑ups
    *   unblock invoices faster
    *   improve cash‑flow KPIs

***

## Solution Overview

To address the problem, **two complementary ML models** are built:

### 1️⃣ Binary Classification Model – *SNI Aging Risk*

This model answers the question:

> **“Is this sales order likely to become an aging SNI?”**

*   Output:
    *   `0` → Not aging
    *   `1` → Aging / At‑Risk
*   Helps business teams **focus attention only on risky orders**
*   Optimized for **high recall**, so fewer aging cases are missed

***

### 2️⃣ Regression Model – *Expected Aging Duration*

This model answers the question:

> **“If an order ages, how long is it expected to remain uninvoiced?”**

*   Output:
    *   Predicted number of **aging days**
*   Enables:
    *   prioritization of **high‑impact orders**
    *   early escalation for orders predicted to age longer

***

## End‑to‑End Flow

At a high level, the solution works as follows:

1.  **Source Data**
    *   Historical SNI data from enterprise supply chain tables
    *   Includes order, shipment, customer, product, and logistics attributes

2.  **Model Training**
    *   Databricks notebooks train:
        *   Binary classification model
        *   Regression model
    *   Strong data governance and leakage prevention built in

3.  **Daily Automation**
    *   A scheduled Databricks job runs **once per day**
    *   Pulls **new / updated SNI records from the last 24 hours**
    *   Applies trained models to generate predictions

4.  **Prediction Output**
    *   Results are written to a **target table**
    *   Each record includes:
        *   SNI risk flag
        *   Expected aging duration (days)
        *   Order identifiers for business action

5.  **Business Action**
    *   Output can be consumed by:
        *   dashboards
        *   alerts
        *   downstream workflows
    *   Enables **proactive follow‑up before cash flow is impacted**

***

## Repository Structure

    ├── notebooks/
    │   ├── binary_classification/
    │   │   └── SNI_aging_risk_model_training.ipynb
    │   │
    │   ├── regression/
    │   │   └── SNI_aging_days_regression_model.ipynb
    │
    ├── automation/
    │   └── daily_sni_prediction_job.py
    │
    ├── docs/
    │   ├── binary_classification_documentation.docx
    │   ├── regression_model_documentation.docx
    │
    └── README.md

***

## Notebooks

### 🔹 Binary Classification Notebook

*   Trains a model to classify orders as **Aging vs Non‑Aging**
*   Uses:
    *   Spark‑based feature preparation
    *   scikit‑learn pipelines
    *   Ensemble and tree‑based models
*   Includes:
    *   correlation handling
    *   threshold tuning aligned to business recall needs
    *   feature importance for explainability

***

### 🔹 Regression Notebook

*   Predicts **number of days an order is expected to age**
*   Uses:
    *   Gradient boosting models
    *   Robust missing‑value handling
    *   Feature importance analysis
*   Designed to support downstream prioritization logic

***

## Automation & Scheduled Job

An automated scoring pipeline is implemented using **Databricks Jobs**.

### What the automation does:

*   Runs **daily**
*   Pulls SNI orders with:
    *   shipment or snapshot dates from the **last 24 hours**
*   Applies:
    *   binary classification model
    *   regression model
*   Writes predictions to a **target table** for consumption

This ensures the solution is:

*   repeatable
*   scalable
*   production‑aligned (even in a POC setup)

***

## Key Benefits Demonstrated by the POC

*   Early identification of SNI risk
*   Improved cash‑flow visibility
*   Reduced manual monitoring effort
*   Explainable ML outputs for business trust
*   Clear path from POC to productionization

***

## Ownership & Context

*   **Initiative:** Supply Chain ML POC – SNI KPI Improvement
*   **Sponsorship / Context:** Brian Straley
*   **Current Stage:** Proof of Concept
*   **Next Steps:** KPI impact validation and production readiness evaluation

***

