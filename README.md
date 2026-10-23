# 🧬 Virus Subtype Classification Pipeline

![Pipeline version](https://img.shields.io/badge/pipeline-v2.1.0-2E86AB?logo=git&logoColor=white)
![Python](https://img.shields.io/badge/python-3.12.15-3776AB?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.5.2-F7931E?logo=scikitlearn&logoColor=white)
![Status](https://img.shields.io/badge/status-stable_·_reproducible-2EA043)
![Determinism](https://img.shields.io/badge/random__state-42-000000)
![Schema](https://img.shields.io/badge/schema-SCHEMA_B-8E44AD)

> **Calibrated SVM-primary classifier for the four subtypes (Ia · Ib1 · Ib2 · Ic)** on a fingerprint-scored
> reference corpus, with explicit k-mer preprocessing, user-label integration, fold-safe class balancing,
> and calibrated probabilities for unlabeled sequences.

| | |
|---|---|
| 📦 **Pipeline version** | `v2.1.0` |
| 📅 **Released** | 2026-10-23 |
| ✅ **Status** | Stable — verified by full *Restart & Run All* (37/37 cells, 0 errors) |
| 🔒 **Determinism** | `random_state=42` everywhere; bit-reproducible outputs |
| 🧪 **Schema** | SCHEMA B (label-dependent self-score → QC/EDA only, **never** a feature) |
| 📓 **Entry point** | `virus_subtype_classification.ipynb` |

---

## 🗒️ Versioning

This project follows **semantic versioning**: `MAJOR.MINOR.PATCH`
(MAJOR = protocol/data change · MINOR = new artifacts or analyses · PATCH = cosmetics/docs).

### Changelog

| Version | Date | Highlights |
|---|---|---|
| `v1.0.0` | — | Initial notebook: load/QC/SCHEMA detection, exact dedup (88 → 19 unique references), LOOCV protocol, calibrated SVM primary + Tier-2/3/4 benchmarks, figs 1–13, 7 CSV exports. Winner: **Extra Trees** (CV bal-acc 0.817). |
| `v2.0.0` | — | **Preprocessing rebuild (user instructions):** 2️⃣a explicit k-merization of *all* sequences over fixed ACGT^k vocabularies (k ∈ {3,4,5}); 2️⃣b integration of the 22 user-labeled partials (21 × Ia + seq 5 = Ic) → 17 exact duplicates collapsed → **5 unique user rows** → corpus = **24 originals** (Ia 10 · Ib1 2 · Ib2 4 · Ic 8); 2️⃣c class balancing **"pick the highest value"** (random oversampling to the max per-subtype count, fold-safe — validation folds keep originals only). Steps renumbered 1–9; new fig1; fig10–12 resized for the 10 targets; new Step-8b user-label consistency check. Winner: **E1 Soft Voting (lda+svm+extra_trees)** CV bal-acc **0.896** (tie with LDA disclosed). |
| `v2.1.0` | 2026-10-23 | ➕ **fig14** — 2-D decision-boundary comparison of seven model families in the PCA plane (dpi 300) with per-panel held-out accuracy and leakage-safe 2-D LOOCV balanced accuracy, plus its standalone code export (`fig14_model_decision_boundaries_2d_code.txt`). 🧹 Notebook decluttered (all `#` comments and verbose `print` logging removed; key result tables rendered as rich `display()` outputs). 📖 This README. |

### 🧪 Environment (tested versions)

| Component | Version | Component | Version |
|---|---|---|---|
| Python | 3.12.15 | matplotlib | 3.9.2 |
| scikit-learn | 1.5.2 | seaborn | 0.13.2 |
| pandas | 2.2.3 | LightGBM | 4.5.0 |
| numpy | 2.1.3 | XGBoost | 2.1.3 *(GBM fallback backend)* |
| scipy | 1.14.1 | Biopython | 1.86 |
| jupyter / nbconvert | — | nbformat / ipykernel | — |

> 💡 All heavy libraries degrade gracefully: if LightGBM/XGBoost/seaborn are missing the pipeline
> automatically falls back (sklearn GBM backend / pure-matplotlib styling) and discloses it in the output.

---

## 📥 Inputs

| File | Content |
|---|---|
| `reference_sequences.csv` | 88 reference rows: sequence, subtype (Ia/Ib1/Ib2/Ic), SCHEMA-B fingerprint self-score (`n/total` ratio string) |
| `sequences.csv` | 32 numbered partial sequences (`seq_num`, sequence) — 22 carry user labels (21 × **Ia**, seq 5 = **Ic**); the remaining 10 are the final classification targets |

---

## ⚙️ Pipeline architecture (9 steps)

1. 🔍 **Data & QC** — load, column/schema detection, ambiguous-base inventory, per-subtype statistics, exact dedup (88 → 19), near-duplicate check of the 32 partials against references.
2. 🧮 **Preprocessing (before any modeling)** — **2a** explicit k-mer conversion of every sequence (normalized frequencies over fixed ACGT^k vocabularies → `preprocessed_kmer_dataset.csv`); **2b** user-label validation, within-set dedup, reference cross-check, provenance tags; **2c** class balancing to the highest per-subtype count (→ `balancing_report.csv`).
3. 🧭 **Diagnostic marker mining** — subtype-private k-mer sweep + leave-one-row-out validation against the CSV self-scores; validation **failed** → protocol fallback to **sequence-only features** (disclosed).
4. 📐 **Protocol** — stratified 80/20 hold-out on originals + **LOOCV** (min class < 20) with **fold-safe balancing**: synthetic copies exist only inside training folds.
5. 🎯 **Primary model** — `StandardScaler → CalibratedClassifierCV(SVC, sigmoid)`, grid over kernel × C × gamma × class_weight, scoring = balanced accuracy.
6. 🤖 **Benchmarks & baselines** — Tier 2: LogReg, RF, Extra Trees, GBM (LightGBM→XGBoost→sklearn chain), KNN (cosine), GNB, MNB (raw counts), MLP, LDA · Tier 3: B1 Nearest-Reference, B2 Fingerprint Baseline · Tier 4: soft-voting & stacking ensembles.
7. 🖼️ **Figures** — fig1–fig13 (dpi 300) covering data overview, fingerprint EDA, PCA projection, per-model confusion matrices, ROC, per-class F1, model comparison with bootstrap CIs, calibration, permutation importance, stacked probabilities, consensus and probability heatmaps, marker overview.
8. 📤 **Final predictions** — winner + ensembles refit on the full fold-safe balanced corpus → calibrated probabilities for the 10 unlabeled targets (`new_sequence_predictions.csv`), plus the Step-8b user-label LOOCV consistency check.
9. 📝 **Written summary** — dynamically assembled Markdown report of every headline number.

---

## 🏆 Key results (v2.1.0)

- 🥇 **Winner:** E1 Soft Voting (lda + svm + extra_trees) — CV balanced accuracy **0.896** (tie with LDA, disclosed); beats both biology baselines B1 and B2.
- 🧾 **Predictions for the 10 unlabeled targets:** **Ia × 9, Ic × 1** (sequence 19 → Ic); 0 low-confidence calls; 4 cross-model disagreements (12, 19, 31, 32) disclosed per row.
- 🧬 **User-label consistency (fold-safe LOOCV):** 2/3 user rows reproduced (u08 → predicted Ib1 when left out → laboratory re-check list); reference rows 15/17.
- ⚠️ **Honesty notes:** mined-marker validation failed → sequence-only features; fingerprint self-scores are used for QC/EDA/sensitivity only, never as features; small-sample margins (< ~0.05) are flagged as statistically indistinguishable.

---

## 📁 Outputs

```
download/
├── virus_subtype_classification.ipynb     📓 executed pipeline (cleaned, Restart & Run All verified)
├── virus_subtype_classification.html      🌐 static companion of the executed notebook
├── fig1…fig13 (+ 14 fig4 variants)        🖼️ 26 pipeline figures, dpi 300
├── fig14_model_decision_boundaries_2d.png 🖼️ article figure: 7 models, 2-D decision boundaries
├── fig14_model_decision_boundaries_2d_code.txt 📄 standalone code that generates fig14
├── new_sequence_predictions.csv           📊 per-target × per-model probabilities
├── new_sequence_predictions_wide_final_model.csv 📊 final-model wide table
├── model_comparison.csv                   📊 every model, identical metric format
├── per_model_confusion_matrices.csv       📊 raw + row-normalized
├── mined_markers.csv / flagged_references.csv / permutation_importance.csv
├── fingerprint_scores_for_new32.csv       📊 mined digital fingerprints, all 32 partials
├── feature_set_comparison.csv · preprocessed_kmer_dataset.csv · balancing_report.csv
└── reference_sequences.csv · sequences.csv (input copies for self-containment)
```

## 🚀 How to run

```bash
pip install numpy pandas scipy scikit-learn matplotlib seaborn lightgbm xgboost biopython jupyter
jupyter notebook virus_subtype_classification.ipynb   # then: Run All (≈ 3 min)
```

- ♻️ **Restart & Run All** reproduces every number, figure and CSV from the two input files.
- 🔒 Deterministic: `random_state=42` for splitting, balancing, tuning, calibration and all estimators; `n_jobs=1`.
- 🧷 To regenerate the article figure only: run the notebook once, then execute
  `fig14_model_decision_boundaries_2d_code.txt` as a final cell (prerequisites listed in its header).

## 🔒 Data-integrity guarantees

- 🚫 The 10 unlabeled targets never touch training, scaling, tuning or calibration.
- 🚫 SCHEMA-B self-scores are never model features (QC / EDA / sensitivity only); no scores are fabricated.
- 🧷 Fold-safe balancing: validation folds contain original rows exclusively, so duplicated sequences cannot inflate CV metrics.
- 📣 Every protocol deviation (marker-validation fallback, GBM backend, stacking OOF omission, ties) is disclosed inline.

---
*README v1.0 · 2026-10-23 · generated alongside pipeline `v2.1.0`*
