# Development of an Adverse Outcome Prediction Model for Severe Diabetic Foot Ulcers Based on Multiple Databases and Interpretable Machine Learning (SHAP)

## Project Introduction

This repository stores the preprocessed de-identified critical care cohorts for diabetic foot (DFU) prognostic modeling. The dataset includes:
- **MIMIC-IV training cohort**
- **eICU independent external validation cohort**
- **Integrated mixed validation file**

Complete data extraction, screening, cleaning and feature engineering pipelines are recorded in this document.

---

## 1. Dataset File Description

| File Name | Source Database | Sample Size | DFU Positive Cases | Usage |
|:---|:---|:---|:---|:---|
| `mimic4_dfu_cohort.csv` | MIMIC-IV v3.1 | 9,102 patients | 1,091 | Original training and internal test cohort, containing unified demographic, laboratory, vital sign, treatment features, all derived scoring indexes and adverse outcome labels |
| `eicu_dfu_cohort.csv` | eICU-CRD v2.0 | 1,200 patients | 250 | Independent external validation dataset, feature dimension and variable naming consistent with MIMIC-IV cohort for unified model verification |
| `external_validation.csv` | Combined subset of MIMIC internal test samples and partial eICU samples | — | — | Integrated unified file for cross-database generalization performance testing of prediction models |

---

## 2. Data Extraction Pipeline

### 2.1 Database and Query Tools

- **Original database**: MIMIC-IV v3.1 (PhysioNet restricted database)
- **Query tools**: PostgreSQL 15, DBeaver visualization client, psycopg2 Python batch script
- **Raw core tables extracted**:
  - `chartevents`
  - `labevents`
  - `prescriptions`
  - `diagnoses_icd`
  - `admissions`
  - `patients`

### 2.2 DFU Diagnostic Screening ICD Codes

| Standard | Codes |
|:---|:---|
| **ICD-9** | `250.xx` (all diabetes types), `707.1x` (lower limb skin ulcer) |
| **ICD-10** | `E10–E14` (all diabetes subtypes), `L97.x` (lower limb ulcer), `E10.621` / `E11.621` / `E13.621` (specific diabetic foot ulcer diagnosis codes) |

### 2.3 Inclusion Standards

1. Patient age ≥ 18 years old
2. ICU hospitalization duration ≥ 24 hours
3. Electronic medical records contain DFU-related ICD diagnostic codes
4. Only retain the **first ICU admission record** for patients with multiple ICU admissions, eliminating repeated measurement bias caused by multiple hospitalizations of the same individual

### 2.4 Exclusion Standards

- Patients with lower limb amputation history before admission
- Cases with key clinical variable missing rate over 30%
- Combined severe trauma, active malignant tumor, or pregnant patients

---

## 3. Prediction Outcome Definitions

Three primary adverse clinical endpoints used for model construction:

| No. | Outcome | Judgment Criteria |
|:---|:---|:---|
| 1 | **In-hospital all-cause mortality** | — |
| 2 | **90-day all-cause mortality after ICU admission** | — |
| 3 | **Major lower extremity amputation** | ICD-10 `Z89.5–Z89.7` and `E11.52` |

---

## 4. Feature Extraction and Derived Indexes

### 4.1 Original Raw Variables

#### Demographic Indicators
- Age, gender, race, BMI

#### Comorbidity Records
- For calculating **Charlson Comorbidity Index (CCI)**

#### First 24h ICU Vital Signs
- Heart rate, MAP, SpO₂, body temperature, respiratory rate

#### First 24h Laboratory Test Indicators
- Creatinine, lactate, white blood cell, platelet, albumin, hemoglobin, blood glucose, HbA1c, BUN, CRP, procalcitonin, INR, fibrinogen, eGFR

#### ICU Treatment Characteristics
- Mechanical ventilation, vasopressor application, insulin use, antibiotic use, wound culture positive result

#### Scoring Systems
- SOFA score, APACHE-II score, ICU length of stay

### 4.2 Derived Ratio Indexes

| Abbreviation | Full Name | Description |
|:---|:---|:---|
| **NLR** | Neutrophil-to-Lymphocyte Ratio | Neutrophil-to-lymphocyte ratio |
| **PLR** | Platelet-to-Lymphocyte Ratio | Platelet-to-lymphocyte ratio |
| **LMR** | Lymphocyte-to-Monocyte Ratio | Lymphocyte-to-monocyte ratio |
| **CCI** | Charlson Comorbidity Index | Charlson comorbidity index |

---

## 5. Standard Data Preprocessing Workflow

### 5.1 Abnormal Physiological Value Correction

Formulate normal human physiological value ranges referring to clinical textbooks and guidelines; values exceeding physiological limits are regarded as measurement errors and converted to missing values.

**Typical abnormal threshold examples:**
- Heart rate &lt; 10 beats/min or &gt; 250 beats/min
- HbA1c &gt; 20%

**Continuous variables processing:**
- All continuous variables adopt **1% / 99% quantile Winsorization truncation** to reduce the interference of extreme outliers
- **Reserved sensitivity analysis scheme**: separately test 2.5%-97.5% and 5%-95% truncation thresholds for comparative analysis

### 5.2 Missing Value Processing Rules

| Missing Proportion | Processing Method |
|:---|:---|
| &gt; 30% | Directly deleted |
| ≤ 30% | **MICE multiple imputation method**, parameter setting: `m=5` imputation datasets, `maxit=50` iteration times |

**Imputation effect verification:** Kolmogorov-Smirnov test, *P* &gt; 0.05 means the variable distribution has no significant difference before and after imputation, and the imputation result is credible.

### 5.3 Dataset Splitting and Class Imbalance Processing

- **Splitting ratio**: The complete MIMIC-IV cohort is stratified and divided into training set and internal test set at a ratio of **7:3**, `random_state` fixed to **42** for reproducibility
- **Cross-validation**: The training set adopts **5-fold stratified cross-validation** for hyperparameter tuning
- **Class imbalance handling**:
  - Only apply **SMOTE oversampling** on the **training set**, `k_neighbors=5`
  - The internal test set and eICU external validation set **do not perform any oversampling** to retain real-world clinical sample distribution

---

## 6. Data Access and Ethics Statement

- Original MIMIC-IV and eICU-CRD databases belong to **PhysioNet restricted data**. Researchers need to complete **CITI training** and sign the **Data Use Agreement (DUA)** to obtain raw data access permission.
- All uploaded CSV files have completed **de-identification processing**, and there is **no protected health information (PHI)** or personal identifiable content in the data.

---

## 7. Citation Guide

When using the dataset in academic research, three parts need to be cited simultaneously:

1. Original published literature of MIMIC-IV database
2. Original published literature of eICU-CRD database
3. Corresponding academic manuscript of this dataset (supplemented after formal publication)

---

## 8. Contact and Version Information

- **Code release**: We will upload all the code once our paper is published
- **Contact**: If there is any question, please feel free to contact [ethan_dha@163.com](mailto:ethan_dha@163.com)
- **Dataset Version**: v1.0
- **Release Time**: 2026-06
