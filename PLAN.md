# HealthAI Suite: Master Plan

Owner: Loghesh | Mentor: Bala Chandar | Source brief: `docs/HealthAI Suite - Scoped Guideline (v1).docx`

## 1. Goal
Six objectives, scoped by the guideline:

| ID | Objective | Track |
|----|-----------|-------|
| O1 | Predict hospital length of stay (regression) | Classic ML on MIMIC |
| O2 | Classify disease-risk category (classification) | Classic ML on MIMIC |
| O3 | Discover patient subgroups (clustering) | Classic ML on MIMIC |
| O4 | Healthcare chatbot (light RAG over a small FAQ) | Hugging Face |
| O5 | Medical translator (MarianMT, Helsinki-NLP opus-mt) | Hugging Face |
| O6 | Sentiment analysis on patient feedback | Hugging Face |

Data: MIMIC-III Clinical Database Demo v1.4 (100 patients, 129 admissions), ODbL licence.
Tables used: PATIENTS, ADMISSIONS, DIAGNOSES_ICD, LABEVENTS, ICUSTAYS, PRESCRIPTIONS + lookups D_ICD_DIAGNOSES, D_LABITEMS.

## 2. How we work (read this first)
- Loghesh is learning. Explain every step briefly, in plain language. Do ONE phase per session.
- Start every phase with a short plan and wait for approval before writing code.
- **Execution mode:** notebooks run on **Google Colab** (VS Code Colab extension) with the data on Google Drive:
  `/content/drive/MyDrive/HealthAISuite/Datasets/mimic-demo-1.4`
  Claude Code can edit the notebook/.py files but CANNOT run Colab cells. So: write the cells, tell Loghesh to run them, and ask him to paste outputs back before interpreting anything. Never guess outputs.
- Every notebook starts with a `DATA_DIR` cell that works in Colab and falls back to a local `./mimic-demo-1.4`.
- Never invent data, numbers or results. If something is missing, say so.
- End each phase with: (a) a 5-line summary Loghesh can share with Bala, (b) assumptions made, (c) "What I should learn" (2-4 points), (d) a suggested commit message. Loghesh runs the commit.

## 3. Repo layout (target)
```
healthai-suite/
├── README.md              overview, setup, results, limitations, ODbL notice
├── PLAN.md  CLAUDE.md
├── requirements.txt       pinned libs for Colab installs
├── configs/config.yaml    seed, paths, n_splits, label definitions
├── docs/                  Bala's brief, pipeline diagram, user guide, model cards
├── notebooks/             01_eda, 02_features, 03_ml_O1-O3, 04_nlp_O4-O6
├── src/                   data/, features/, models/, nlp/  (reusable code, added after notebooks work)
├── tests/                 small unit tests (leakage checks, split checks)
├── models/                small trained artifacts only (.joblib < 25 MB)
├── reports/               figures, final report, slides
└── .gitignore             data, caches, secrets
```
Never committed: MIMIC data, API keys/tokens, large files (>25 MB), `.venv`, `__pycache__`, `mlruns/`.

## 4. Phases

### Phase 0: Repo hygiene
- Move Bala's two .docx files into `docs/`. Create folders above, `configs/config.yaml` (seed: 42, n_splits: 5), `requirements.txt`, README skeleton with ODbL line:
  "Contains information from MIMIC-III Clinical Database Demo, made available under the Open Database License (ODbL)."
- Done when: structure exists, README renders on GitHub, first clean commit.

### Phase 1: Data quality + EDA (`notebooks/01_eda.ipynb`)
For each of the 8 tables: shape, dtypes, nulls, duplicate rows, key uniqueness (subject_id / hadm_id), date sanity, outliers.
Specific checks: age >89 (shifted to ~300); hospital LOS distribution (skew, negatives, zeros); admissions per patient; ICU coverage (admissions with no ICU stay); lab coverage in first 24h; ICD-9 counts for diabetes (250.xx) and heart disease (410-414, 428) per admission; whether CHARTEVENTS.csv is actually in the Drive folder (guideline says it is not).
- Done when: findings table + class counts for the O2 candidate labels.
- **STOP:** Loghesh shares findings with Bala. Lock the O2 label before Phase 2.

### Phase 2: Feature table (`02_features.ipynb`)
One row per `hadm_id`. Features: age (capped), gender, admission_type, insurance, ICU flag/first_careunit, first-24h lab aggregates (mean/min/max for a short list of common labs), simple prescription counts within first 24h.
Imputation + scaling + encoding live inside an sklearn `Pipeline`/`ColumnTransformer` (fit on train only).
Leakage list (never features): dischtime, deathtime, hospital_expire_flag, discharge_location, DRGCODES, ADMISSIONS.diagnosis text, ICUSTAYS.los, anything after the first 24h. For O2: all DIAGNOSES_ICD codes, comorbidity flags, and drugs that define the label (e.g. insulin for diabetes).
Split: `GroupKFold` on `subject_id`. A unit test must confirm no subject appears in both train and test.
- Done when: feature table saved to Drive (not Git), leakage test passes.

### Phase 3a, O1: length of stay (regression)
Target: hospital LOS in days = dischtime - admittime. Baselines: mean and median. Models: Ridge, Random Forest. Metrics: MAE, RMSE, R2, cross-validated mean +/- spread. Consider log-transform if skewed. Success = MAE below baseline.

### Phase 3b, O2: disease risk (classification)
Label per Bala's decision (default: ICD-9 disease flag). Models: logistic regression (baseline), XGBoost, MLP. Metrics: accuracy, F1, ROC-AUC with confidence intervals (repeated grouped CV). SHAP for the best model. Fairness check across gender and age group. If positives are too few, say so plainly; do not chase 80% F1.

### Phase 3c, O3: clustering
Cluster on scaled features only (never the O2 label). k-means (choose k with silhouette) and optionally HDBSCAN. Metrics: silhouette, Calinski-Harabasz. Interpret clusters afterwards by profiling them against diagnoses and demographics. Call results exploratory.

### Phase 4: Hugging Face (Colab, small datasets)
- O6 sentiment: needs a labelled review dataset (to source). Pre-trained pipeline, evaluate precision/recall/MCC. Fine-tuning optional.
- O5 translator: MarianMT, 20-50 test sentences with reference translations, BLEU (+ COMET if feasible). Languages pending Bala. Check each language pair exists on the Hugging Face hub first.
- O4 chatbot: small FAQ/guideline .txt, light retrieval + pre-trained LLM, "not medical advice" disclaimer, fixed test questions judged by hand for hallucinated/unsafe answers.

### Phase 5: Packaging
One model card per objective (data, metrics, limitations, intended use), README results, final report, slides, demo video (chatbot, translator, sentiment). Optional pending Bala: FastAPI, Streamlit, Docker.

## 5. Open decisions (waiting on Bala; use defaults meanwhile)
| Question | Default until answered |
|----------|-----------------------|
| O2 label | ICD-9 diabetes / heart disease flag |
| 80% F1 target binding? | Treat as aspirational; report CIs |
| FastAPI / Streamlit / Docker | Skip; notebooks + report |
| O4 appointment scheduling | Drop |
| O5 languages | Propose Tamil + Hindi, verify models |
| Four removed modules out of scope | Assume yes |

## 6. Honest framing (README + LinkedIn)
Methodology demo on the open MIMIC-III demo. The demo's 100 patients were drawn from patients who eventually die, so it is not representative of all ICU patients. No clinical claims.

## 7. Status log
| Phase | Status | Bala feedback |
|-------|--------|---------------|
| 0 Repo hygiene | in progress (repo created, 1 commit) | |
| 1 EDA | not started | |
| 2 Features | not started | |
| 3a O1 | not started | |
| 3b O2 | not started | |
| 3c O3 | not started | |
| 4 Hugging Face | not started | |
| 5 Packaging | not started | |
