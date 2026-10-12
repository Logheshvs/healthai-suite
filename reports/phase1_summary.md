# HealthAI Suite: Phase 1 summary (O1-O3)

Notebook: `notebooks/01_phase1_o1_o3.ipynb`

## What was done

Three classic ML objectives were run end to end on the MIMIC-III Clinical Database Demo v1.4:

| ID | Objective | Task |
|----|-----------|------|
| O1 | Length of stay | Regression on `dischtime - admittime` |
| O2 | Disease risk | Two binary classifiers: diabetes and heart disease (ICD-9 labels) |
| O3 | Patient subgroups | k-means clustering |

O4-O6 (chatbot, translator, sentiment) are not started.

## Data

- 8 of the 26 MIMIC tables: PATIENTS, ADMISSIONS, ICUSTAYS, DIAGNOSES_ICD, D_ICD_DIAGNOSES, LABEVENTS, D_LABITEMS, PRESCRIPTIONS.
- 129 admissions from 100 patients, built into one master table (one row per admission, 29 model features).
- Labels: diabetes 48 admissions (37%), heart disease 52 (40%), both 23, neither 52.
- Demo patients were selected from people who eventually died, so the sample is not representative of ICU patients.

## Method

- **Data quality:** shapes, nulls, duplicates, key uniqueness, lookup coverage, lab unit mix, date windows.
- **Features:** age (capped at 90 with a flag, because MIMIC shifts ages over 89), gender, admission type and location, insurance, ICU by hour 24, first care unit, 20 lab means from -6 h to 24 h, number of unique drugs on the admission day.
- **Leakage controls:** discharge time, death time, discharge location, DRG codes, ICU length of stay and every diagnosis code, flag or count are excluded from features. Diagnosis codes are used for labels only.
- **Preprocessing:** median imputation, scaling and one-hot encoding sit inside an sklearn `Pipeline`, fitted on training folds only.
- **Validation:** 5-fold `StratifiedGroupKFold` on `subject_id` (seed 42), so a patient is never in both train and test. Scores are reported as mean +/- SD across folds, plain and patient-weighted (one patient counts once).
- **Baselines:** training-fold median (O1), majority class and an age-only logistic model (O2).

## Results

All figures are cross-validated; with 129 admissions the spread is wide.

### O1: length of stay (MAE in days, lower is better)

| Model | MAE, mean +/- SD over 5 folds | Pooled MAE | Pooled R2 |
|-------|-------------------------------|------------|-----------|
| Median baseline | 6.05 +/- 2.42 | 6.03 | -0.058 |
| Ridge (tuned, nested CV) | 6.01 +/- 2.37 | 6.00 | -0.061 |
| Random Forest | 6.00 +/- 2.29 | 6.04 | -0.049 |

No model improves on the baseline in any meaningful way. One stay of about 124 days dominates RMSE (13.0 falls to 7.9 without it), and the conclusion is the same with it removed.

### O2: disease classification (ROC-AUC, mean +/- SD)

| Label | Majority | Age-only | Logistic (C = 0.03) | XGBoost (depth 2) |
|-------|----------|----------|---------------------|-------------------|
| Diabetes (48/129) | 0.50 | 0.57 +/- 0.12 | 0.66 +/- 0.11 | 0.59 +/- 0.17 |
| Heart disease (52/129) | 0.50 | 0.77 +/- 0.03 | 0.69 +/- 0.07 | 0.67 +/- 0.13 |

- F1 for the all-feature logistic model is 0.37 +/- 0.14 (diabetes) and 0.51 +/- 0.15 (heart disease), against a target of 80%.
- For heart disease, age alone beats every model, so the other features add nothing.
- The regularisation strength C was chosen on the same folds, so the logistic scores are slightly optimistic.
- Glucose is close to defining the diabetes label.

### O3: clustering (silhouette, higher is better)

| k | All 129 admissions | Without the 15-admission patient |
|---|--------------------|----------------------------------|
| 2 | 0.142 | 0.078 |
| 3 | 0.101 | 0.083 |
| 4 | 0.105 | 0.067 |
| 5 | 0.079 | 0.060 |
| 6 | 0.070 | 0.069 |

No cluster structure (all values are below 0.25). The only apparent split at k = 2 was one patient's 15 repeat admissions plus two others.

## Conclusion

None of the three success targets was met. The pipeline is sound (grouped split, no leakage, baselines, uncertainty), and the negative result is a property of the data: 129 admissions, a population selected from patients who died, and no vitals in the feature set. Any difference between models is smaller than the fold-to-fold noise.

## Findings that change the plan

- **Vitals are available.** The scoped guideline states that `CHARTEVENTS` was not uploaded, but the local demo folder contains `CHARTEVENTS.csv` (758,355 rows) and `D_ITEMS.csv`. First-24 h vitals could therefore be added even on the demo.
- **Sample size is the binding limit.** Folds contain about 5-25 positive cases, which is why tuning and model choice change results by less than the spread.

## Not done

- MLP (O2), HDBSCAN and Calinski-Harabasz (O3), SHAP and a formal fairness table.
- Confidence intervals (only SD across folds is reported).
- O4-O6, model artifacts, tests, experiment tracking, API and dashboard.

## Proposed next step

Repeat O1-O3 on a dataset with enough rows to support conclusions:

| Dataset | Size | Access |
|---------|------|--------|
| UCI Diabetes 130-US Hospitals (1999-2008) | 101,766 encounters | Open, CC BY 4.0 |
| MIMIC-IV | 546,028 hospitalizations, 94,458 ICU stays | Credentialed (CITI training and data use agreement) |
| MIMIC-III (full) | 40,000+ patients | Credentialed |
| PhysioNet/CinC Challenge 2012 | 12,000 ICU stays | Open |

The original brief lists "MIMIC-III or MIMIC-IV" as the data source, so a full MIMIC release matches it; the demo was used to build and validate the pipeline. A non-MIMIC dataset would change the feature set and some label definitions, and the decision should be agreed before work starts.

Open questions:

1. Which dataset should be the main one for O1-O3?
2. Which O2 label: a disease flag from ICD-9 codes, in-hospital mortality, or readmission?
3. Is the 80% F1 target binding, or is a well-validated baseline acceptable?
4. Are FastAPI, Streamlit and Docker required, or are notebooks plus a report enough?

---

Contains information from MIMIC-III Clinical Database Demo, made available under the Open Database License (ODbL). This is a methodology demo and makes no clinical claims.
