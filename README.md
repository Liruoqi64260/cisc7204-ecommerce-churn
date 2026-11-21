# CISC7204 Final Project — E-Commerce Customer Churn Prediction

**Course:** CISC7204 Data Science and Data Visualization, Fall 2026 (Class B, Team 05)
**University:** University of Macau
**Submission deadline:** 2026-12-05 23:00 (HKT/Macau time)
**Submission channel:** Class HOME UMMoodle forum (NOT the main CISC7204 UMMoodle site). Every team member submits the archive individually.

---

## 1. Project overview

This project builds and compares two supervised binary classifiers that predict whether an e-commerce customer will **churn within 180 days of the first purchase**. Churn is operationalised as *no repeat purchase within the 180-day observation window* (`churn_180d = 1`). The study addresses the question:

> To what extent do first-order delivery experience, review score, voucher discount and monetary value predict early customer churn?

The analytic sample is the full-window cohort (`in_cohort = 1`, n = 62,835) drawn from the 93,350-customer cleaned table. Only features observable at the time of the **first order** are used as predictors; all post-outcome and lifetime descriptive fields are excluded to prevent target leakage. Two models — a class-weighted Logistic Regression and a class-weighted Random Forest — are trained on a stratified 80/20 split and compared with ROC-AUC, balanced accuracy, rare-class PR-AUC, recall and confusion matrices.

Every figure used to communicate a conclusion is accompanied by a Markdown self-assessment written against Alberto Cairo's framework in *How Charts Lie* (design, truthfulness, functionality, beauty, insightfulness).

---

## 2. Package contents

| Path | Description |
| :--- | :--- |
| `README.md` | This file: overview, contents, installation and execution guide. |
| `CISC7204_FinalProject_Churn_Analysis.ipynb` | Executed notebook: import, cleaning, EDA, feature engineering, modelling, evaluation and visualisation. Each figure is followed by a five-point Cairo critique Markdown cell. |
| `CISC7204_FinalProject_Churn_Analysis.html` | Static HTML rendering of the executed notebook for quick browsing. |
| `datasets/cleaned_customer_features.csv.gz` | Gzip-compressed analytic dataset (93,350 rows x 33 columns); the notebook decompresses it automatically on first run. |
| `datasets/DATASET_DESCRIPTION.md` | Data provenance, sample size, missing-value profile, target definition and limitations. |
| `codebook/CODEBOOK.md` | Independent variable handbook: name, type, business meaning, encoding and preprocessing. |
| `figures/*.png` | Eleven exported publication-resolution visualisations (150+ dpi). |
| `GITHUB_REPOSITORY.md` | Public repository setup and directory-structure specification. |
| `requirements.txt` | Pinned minimal Python dependencies for reproduction. |

---

## 3. Environment installation

Python 3.10 or newer is required (developed and verified on Python 3.14.6).

```bash
# 1. Create an isolated environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Register the kernel if you want to use Jupyter interactively
python -m ipykernel install --user --name cisc7204-churn
```

`requirements.txt` contains: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`, `nbformat`, `nbclient`, `ipykernel`, `jupyter`.

---

## 4. Running the analysis

The notebook uses **relative paths** (`datasets/` and `figures/`), so launch it from the package root.

**Option A — Jupyter interactively**

```bash
jupyter notebook CISC7204_FinalProject_Churn_Analysis.ipynb
# Kernel > Restart Kernel and Run All Cells
```

**Option B — command line reproduction**

```bash
jupyter nbconvert --to notebook --execute \
  --ExecutePreprocessor.timeout=1200 \
  --output CISC7204_FinalProject_Churn_Analysis.executed.ipynb \
  CISC7204_FinalProject_Churn_Analysis.ipynb
```

Successful execution regenerates all eleven PNG files inside `figures/`. The random seed is fixed at 42, the train/test split is stratified, and all preprocessing is fitted inside scikit-learn pipelines, so results are deterministic and leakage-free.

---

## 5. Reproducibility notes and caveats

* The raw source is the public **Olist Brazilian E-Commerce dataset** (Kaggle, 9 relational tables). The supplied cleaned customer table is derived from it by the upstream cleaning pipeline; no additional external data are used.
* The 180-day churn label is highly imbalanced: churn is the **majority** class (98.3% in the cohort) and retention is the rare class (1.7%). Accuracy, churn-class recall and churn-class PR-AUC are base-rate artefacts and are **not** used as selection criteria; ROC-AUC, balanced accuracy and rare-class (retained) PR-AUC receive priority.
* The label measures *absence of a repeat purchase*, not a survey-based intention to leave. Interpretation as "customer loss" should be cautious.
* Only first-order predictors are used. Fields prefixed `desc_` and `days_to_next_order` are descriptive/outcome information and are deliberately excluded.

---

## 6. Reference

Olist. (2018). *Brazilian E-Commerce Public Dataset by Olist*. Kaggle. https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Cairo, A. (2019). *How Charts Lie: Getting Smarter about Visual Information*. W. W. Norton & Company.
