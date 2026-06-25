# dfu-multimodal-ml
Development of an Adverse Outcome Prediction Model for Severe Diabetic Foot Ulcers Based on Multiple Databases and Interpretable Machine Learning (SHAP)
Project Introduction
This repository stores the preprocessed de-identified critical care cohorts for diabetic foot (DFU) prognostic modeling. The dataset includes MIMIC-IV training cohort, eICU independent external validation cohort and integrated mixed validation file. Complete data extraction, screening, cleaning and feature engineering pipelines are recorded in this document.
1. Dataset File Description
mimic4_dfu_cohort.csv
Source database: MIMIC-IV v3.1
Total sample number: 9102 patients, including 1091 DFU positive cases
Usage: Original training and internal test cohort, containing unified demographic, laboratory, vital sign, treatment features, all derived scoring indexes and adverse outcome labels.
eicu_dfu_cohort.csv
Source database: eICU-CRD v2.0
Total sample number: 1200 patients, including 250 DFU positive cases
Usage: Independent external validation dataset, feature dimension and variable naming consistent with MIMIC-IV cohort for unified model verification.
external_validation.csv
Source: Combined subset of MIMIC internal test samples and partial eICU samples
Usage: Integrated unified file for cross-database generalization performance testing of prediction models.
2. Data Extraction Pipeline
2.1 Database and Query Tools
Original database: MIMIC-IV v3.1 (PhysioNet restricted database)
Query tools: PostgreSQL 15, DBeaver visualization client, psycopg2 Python batch script
Raw core tables extracted: chartevents, labevents, prescriptions, diagnoses_icd, admissions, patients
2.2 DFU Diagnostic Screening ICD Codes
ICD-9 standard: 250.xx (all diabetes types), 707.1x (lower limb skin ulcer)
ICD-10 standard: E10–E14 (all diabetes subtypes), L97.x (lower limb ulcer), E10.621 / E11.621 / E13.621 (specific diabetic foot ulcer diagnosis codes)
2.3 Inclusion Standards
Patient age ≥ 18 years old
ICU hospitalization duration ≥ 24 hours
Electronic medical records contain DFU-related ICD diagnostic codes
Only retain the first ICU admission record for patients with multiple ICU admissions, eliminate repeated measurement bias caused by multiple hospitalizations of the same individual
2.4 Exclusion Standards
Patients with lower limb amputation history before admission
Cases with key clinical variable missing rate over 30%
Combined severe trauma, active malignant tumor, or pregnant patients
3. Prediction Outcome Definitions
Three primary adverse clinical endpoints used for model construction:
In-hospital all-cause mortality
90-day all-cause mortality after ICU admission
Major lower extremity amputation, judged by ICD-10 Z89.5–Z89.7 and E11.52
4. Feature Extraction and Derived Indexes
Original Raw Variables
Demographic indicators: age, gender, race, BMI
Comorbidity records for calculating Charlson index
First 24h ICU vital signs: heart rate, MAP, SpO2, body temperature, respiratory rate
First 24h laboratory test indicators: creatinine, lactate, white blood cell, platelet, albumin, hemoglobin, blood glucose, HbA1c, BUN, CRP, procalcitonin, INR, fibrinogen, eGFR
ICU treatment characteristics: mechanical ventilation, vasopressor application, insulin use, antibiotic use, wound culture positive result
Scoring systems: SOFA score, APACHE-II score, ICU length of stay
Derived Ratio Indexes
NLR (neutrophil-to-lymphocyte ratio)
PLR (platelet-to-lymphocyte ratio)
LMR (lymphocyte-to-monocyte ratio)
CCI (Charlson Comorbidity Index)
5. Standard Data Preprocessing Workflow
5.1 Abnormal Physiological Value Correction
Formulate normal human physiological value ranges referring to clinical textbooks and guidelines; values exceeding physiological limits are regarded as measurement errors and converted to missing values.
Typical abnormal threshold examples: heart rate <10 beats/min or >250 beats/min, HbA1c >20%.
All continuous variables adopt 1% / 99% quantile Winsorization truncation to reduce the interference of extreme outliers.
Reserved sensitivity analysis scheme: separately test 2.5%-97.5% and 5%-95% truncation thresholds for comparative analysis.
5.2 Missing Value Processing Rules
Variables with missing proportion higher than 30% are directly deleted
Variables with missing proportion ≤30% use MICE multiple imputation method, parameter setting m=5 imputation datasets, maxit=50 iteration times
Imputation effect verification: Kolmogorov-Smirnov test, P>0.05 means the variable distribution has no significant difference before and after imputation, and the imputation result is credible
5.3 Dataset Splitting and Class Imbalance Processing
The complete MIMIC-IV cohort is stratified and divided into training set and internal test set at a ratio of 7:3, random_state fixed to 42 for reproducibility
The training set adopts 5-fold stratified cross-validation for hyperparameter tuning
Solve sample imbalance: only apply SMOTE oversampling on training set, k_neighbors=5
Internal test set and eICU external validation set do not perform any oversampling to retain real-world clinical sample distribution
6. Data Access and Ethics Statement
Original MIMIC-IV and eICU-CRD databases belong to PhysioNet restricted data. Researchers need to complete CITI training and sign the Data Use Agreement to obtain raw data access permission.
All uploaded CSV files have completed de-identification processing, and there is no protected health information (PHI) or personal identifiable content in the data.
The research scheme corresponding to this dataset has passed the review of the institutional ethics committee.
7. Citation Guide
When using the dataset in academic research, three parts need to be cited simultaneously:
Original published literature of MIMIC-IV database
Original published literature of eICU-CRD database
Corresponding academic manuscript of this dataset (supplemented after formal publication)
8. Contact and Version Information
We will upload all the code once our paper is published.
If there is any question, please feel free to contact me (ethan_dha@163.com)
Dataset Version: v1.0
Release Time: 2026-06
