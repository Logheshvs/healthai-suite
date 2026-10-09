# HealthAI Suite

An end-to-end AI/ML project on patient health data: three classic machine-learning models on the MIMIC-III demo database, plus three Hugging Face pipelines for patient-facing language tasks.

> **Status: work in progress.** No results are published yet. This README is updated as each phase finishes.

## Objectives

| ID | Objective | Approach |
|----|-----------|----------|
| O1 | Predict hospital length of stay (regression) | Baseline, Ridge, Random Forest |
| O2 | Classify disease-risk category | Logistic regression, XGBoost, MLP |
| O3 | Discover patient subgroups (clustering) | k-means / HDBSCAN |
| O4 | Healthcare chatbot for patient queries | Pre-trained LLM with a small FAQ file (light RAG) |
| O5 | Medical translator for regional languages | MarianMT (Helsinki-NLP opus-mt) |
| O6 | Sentiment analysis on patient feedback | Pre-trained Hugging Face sentiment pipeline |

## Data and limitations

- **MIMIC-III Clinical Database Demo v1.4** (100 patients, 129 admissions). The data is **not stored in this repository**; it is open access on [PhysioNet](https://www.physionet.org/content/mimiciii-demo/).
- This is a **methodology demo**, not a clinical tool. The demo patients were selected from patients who eventually die, so results do not represent all ICU patients. With so few records, metrics are reported with cross-validation and uncertainty ranges.
- The chatbot output is **not medical advice**.

## Method highlights

- Train/test splits are grouped by patient, so one patient never appears in both sets.
- Leakage columns (discharge time, death time, discharge location, DRG codes) are excluded from features.
- Fixed random seeds and config-driven experiments for reproducibility.

## Repository structure

```
configs/     settings (seed, paths)
docs/        project brief, model cards, pipeline diagram
notebooks/   EDA, ML (O1-O3), NLP (O4-O6)
src/         reusable code (data, features, models, nlp)
tests/       unit tests
models/      small trained model files
reports/     figures, final report, slides
PLAN.md      master plan and phase status
```

## How to run

Notebooks run on Google Colab (through the VS Code Colab extension) with the MIMIC demo files in Google Drive. Setup details will be added as the notebooks are finished.

## Data licence and attribution

Contains information from the MIMIC-III Clinical Database Demo, made available under the Open Database License (ODbL). MIMIC-III raw data is not redistributed here.
