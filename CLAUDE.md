# HealthAI Suite: instructions for Claude Code

Read `PLAN.md` first. It is the master plan; this file holds the standing rules.

## Who I'm working with
Loghesh is a beginner-to-intermediate learner doing this project for his mentor, Bala Chandar. Explain the "why" of each step briefly and in plain language. He must be able to defend every line to Bala. Do ONE phase per session, and show a short plan before writing code.

## Hard rules
- Never invent data, numbers or results. If something is missing, say so. Never guess the output of code that has not been run.
- Notebooks run on Google Colab (data on Google Drive). You cannot run Colab cells. Write the cells, ask Loghesh to run them and paste the output back.
- Do not commit or push. Suggest a commit message at the end of each phase; Loghesh runs git himself.
- Never read, copy or commit MIMIC data into the repo. `mimic-demo-1.4/` and any `data/` folder stay git-ignored.
- Never put API keys, tokens or passwords in any file.

## ML rules
- State problem type, metric, baseline and split BEFORE tuning.
- Seed 42 from `configs/config.yaml` everywhere.
- Fit imputers, scalers and encoders inside an sklearn `Pipeline`, on training folds only.
- Split with `GroupKFold` on `subject_id`; a patient must never appear in both train and test.
- Hospital LOS = `dischtime - admittime` (NOT `ICUSTAYS.los`). Labs: first 24 hours of admission only.
- Leakage: never use dischtime, deathtime, hospital_expire_flag, discharge_location, DRGCODES, the ADMISSIONS `diagnosis` text or ICUSTAYS.los as features. For O2 the ICD-9 codes define the label, so no DIAGNOSES_ICD codes, comorbidity flags or label-defining drugs as features.
- MIMIC shifts ages over 89 to about 300; cap or flag them.
- Report cross-validated mean with spread or confidence intervals, never a single score. 129 admissions is tiny, so say so in every result.
- Data quality first: nulls, duplicates, dtypes, outliers, key uniqueness. State assumptions.

## Style
- Clean PEP8 Python, comments on key lines, small functions.
- Concise, no filler. Tables or bullets where they help.
- Point out weak spots in the approach honestly.
- End each phase with: a 5-line summary for Bala, assumptions, next steps, and a "What I should learn" note (2-4 points).

## Licence and honesty
- README and reports must include: "Contains information from MIMIC-III Clinical Database Demo, made available under the Open Database License (ODbL)."
- Frame this as a methodology demo. The demo patients were selected from patients who eventually die, so results are not representative of all ICU patients. No clinical claims.
