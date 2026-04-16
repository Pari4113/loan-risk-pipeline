# 🏦 Loan Default Risk Pipeline

**End-to-end data engineering pipeline for predicting loan default risk using Databricks, PySpark, and Machine Learning.**


## 📊 Project Overview

This project demonstrates core **data engineering competencies**:
- **ETL Pipeline** — Clean, transform, and feature-engineer massive datasets
- **Delta Lake** — ACID transactions, time travel, ZORDER optimization
- **ML Pipeline** — Model training, evaluation, and experiment tracking with MLflow
- **Production Ready** — Handles messy real-world data, scales to millions of rows
- **Business Impact** — Transforms raw data into actionable risk insights

**Key Results:**
- 🎯 **0.70 AUC** — Best model catches 62% of defaulters
- 📈 **1.34M loans** processed and risk-scored
- 📊 **Default risk variance** — Very High risk borrowers default at 3.9x the rate of Low risk

---

## 🏗️ Architecture — Medallion Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                    LOAN RISK PIPELINE                       │
└─────────────────────────────────────────────────────────────┘

🥉 BRONZE LAYER
   ├─ Source: Lending Club CSV (2.26M raw records)
   ├─ Ingestion: Spark read + Delta save
   └─ Purpose: Immutable backup of raw data
      
        ↓ (Notebook_1_Bronze.ipynb)

🥈 SILVER LAYER
   ├─ Input: Bronze table (2.26M rows)
   ├─ Processing:
   │  ├─ Select 15 relevant columns from 151
   │  ├─ Handle missing values (Pandas + median fill)
   │  ├─ Fix dirty data (bad dates, text in numeric fields)
   │  └─ Feature engineering (8 new features):
   │     ├─ Credit utilization (revol_util / 100)
   │     ├─ DTI ratio (annual installment / income)
   │     ├─ Risk score (composite: grade + DTI + interest rate)
   │     ├─ Log transforms (log_annual_inc, log_loan_amnt)
   │     └─ Grade encoding (A=1 ... G=7)
   └─ Output: 1,345,348 clean loans, 23 columns
   
        ↓ (Notebook_2_Silver.ipynb)

🥇 GOLD LAYER
   ├─ Input: Silver table (1.34M rows, 23 columns)
   ├─ Model Training:
   │  ├─ Compare 3 models:
   │  │  ├─ Logistic Regression (baseline)
   │  │  ├─ Logistic Regression + Scaling + Balanced Weights (✅ BEST)
   │  │  └─ Random Forest (balanced)
   │  ├─ Train/Test split: 80/20 stratified
   │  ├─ MLflow experiment tracking (runs logged)
   │  └─ Model selection: Best AUC + Recall balance
   └─ Scoring:
      ├─ Score all 1.34M loans with best model
      ├─ Generate default_probability (0.0 to 1.0)
      └─ Bucket into risk_label: Low / Medium / High / Very High
      
        ↓ (Notebook_3_Gold.ipynb)

---

## 📈 Default Risk Insights

**By Risk Label:**
| Risk Label | Loan Count | Avg Default Prob | Avg Interest Rate | Avg DTI |
|---|---|---|---|---|
| 🔴 Very High | 291K | 71.1% | 19.6% | 22.6 |
| 🟠 High | 516K | 49.2% | 13.8% | 19.1 |
| 🟡 Medium | 507K | 31.0% | 9.4% | 15.5 |
| 🟢 Low | 31K | 18.2% | 6.6% | 8.9 |

**Key Finding:** Very High risk borrowers have **3.9x higher default probability** than Low risk, pay **2.97x higher interest rates**, and carry **2.54x more debt relative to income**.

---

## 🔧 Tech Stack

| Component | Technology | Why |
|---|---|---|
| Platform | Databricks Community Edition | Scalable, cloud-native Spark |
| Storage | Delta Lake | ACID, time travel, optimized queries |
| Processing | PySpark + Pandas | Handles 1.34M rows efficiently |
| ML | scikit-learn + MLflow | Model training & experiment tracking |
| Version Control | Git + GitHub | Track code changes, resume-ready |

---

## 📚 Notebooks

### Notebook 1: Bronze Layer
**File:** `Notebook_1_Bronze.ipynb`  
**What it does:**
- Downloads Lending Club CSV from Kaggle
- Uploads to Databricks DBFS
- Reads with PySpark (2.26M rows, 151 columns)
- Saves raw data as Delta table `bronze_loans`

**Key learnings:** Basic Spark read/write, Delta table creation

---

### Notebook 2: Silver Layer
**File:** `Notebook_2_Silver.ipynb`  
**What it does:**
- Loads Bronze table
- Selects 15 relevant columns
- Fixes missing values using median fill
- Handles dirty data (bad dates, text in numeric fields)
- Engineers 8 new features (DTI ratio, credit utilization, risk score)
- Saves cleaned, feature-rich data as Delta table `silver_loans`

**Key learnings:** Data cleaning, feature engineering, Pandas + PySpark integration

---

### Notebook 3: Gold Layer
**File:** `Notebook_3_Gold.ipynb`  
**What it does:**
- Loads Silver table (1.34M loans)
- Encodes categorical columns for ML
- Splits data: 80% train, 20% test
- Trains 3 models with MLflow tracking:
  - Logistic Regression
  - Logistic Regression + Scaling + Balanced Weights (✅ Winner)
  - Random Forest
- Scores all loans with best model (AUC 0.7019)
- Generates risk_label buckets
- Saves scored loans as Delta table `gold_loans_scored`

**Key learnings:** ML training, MLflow experiment tracking, model evaluation (AUC, ROC, confusion matrix)

---

## 🚀 How to Run

### Prerequisites
- Databricks Community Edition account (free at community.cloud.databricks.com)
- Kaggle account (free)
- Lending Club dataset (download from Kaggle)

### Setup

**1. Clone repo and create Databricks Git folder:**
```bash
git clone https://github.com/Pari4113/loan-risk-pipeline
cd loan-risk-pipeline
```

**2. In Databricks, create Git folder:**
- Click **Workspace → Create → Git Folder**
- Paste: `https://github.com/Pari4113/loan-risk-pipeline`

**3. Download Lending Club dataset:**
- Go to: https://www.kaggle.com/datasets/wordsforthewise/lending-club
- Download `accepted_2007_to_2018Q4.csv`

**4. Upload to Databricks:**
- In Databricks: **Catalog → Create Volume**
- Upload CSV to `/Volumes/main/default/loan_risk/`

**5. Run notebooks in order:**
```
Notebook_1_Bronze   → Creates bronze_loans table
Notebook_2_Silver   → Creates silver_loans table
Notebook_3_Gold     → Creates gold_loans_scored table
```

---

## 📊 Model Performance

| Model | AUC | Default Recall | Notes |
|---|---|---|---|
| Logistic Regression | 0.6993 | 8% | Baseline |
| Random Forest | 0.6840 | 9% | Overfits to majority class |
| **LR + Scaling + Balanced Weights** | **0.7019** | **62%** | ✅ BEST — catches defaulters |

**Why LR won:** Loan default is largely a linear problem (higher grade → lower default, higher interest rate → higher default). Logistic Regression captures this better than Random Forest on this dataset.

**Why balanced weights matter:** Default rate is 20% but model was predicting 0% defaults because it's safer. Balanced weights force it to pay attention to the minority class (defaulters).

---

## 🎯 Key Competencies Demonstrated

### Data Engineering
✅ ETL pipeline design (Bronze → Silver → Gold)  
✅ Delta Lake (ACID, time travel, ZORDER optimization)  
✅ PySpark — DataFrames, SQL, large-scale transformations  
✅ Data quality — cleaning, validation, null handling  

### Python & Data Manipulation
✅ Pandas for data cleaning and feature engineering  
✅ NumPy for numerical operations  
✅ Spark SQL for complex queries  

### Machine Learning & MLflow
✅ Model training (Logistic Regression, Random Forest)  
✅ Model evaluation (AUC, ROC, confusion matrix)  
✅ Experiment tracking with MLflow  
✅ Feature importance analysis  

### Production Skills
✅ Handles real messy data (dirty values, missing columns)  
✅ Scales to 1.34M+ rows  
✅ Version control with Git  
✅ Databricks workflows & Delta tables  

---

## 💡 Business Impact

1. **Score every loan applicant** for default risk before approval
2. **Adjust interest rates** based on risk tier (Very High → 19.6% vs Low → 6.6%)
3. **Reduce losses** by catching 62% of potential defaults
4. **Query risk data instantly** using plain English ("which loans defaulted in Texas?")
5. **Track decisions** with full audit trail (Delta Lake time travel)

**Estimated impact:** If this model prevents 1% of default-related losses across a $1B loan portfolio, that's **$10M saved annually**.

---

## 🔗 Links

- **GitHub Repo:** https://github.com/Pari4113/loan-risk-pipeline
- **Databricks Cert Study Guide:** https://www.databricks.com/learn/certification/data-engineer-associate
- **Delta Lake Docs:** https://docs.delta.io
- **MLflow Docs:** https://mlflow.org/docs/latest/

---

**Status:** ✅ Complete & Production Ready
