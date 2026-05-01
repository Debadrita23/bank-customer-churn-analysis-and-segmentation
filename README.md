# Bank Customer Churn Prediction & Segmentation

A end-to-end analytics project on 10,000 European bank customers — covering data
cleaning, exploratory analysis, churn prediction modelling, and behavioural
segmentation with actionable retention recommendations.

---

## Business Problem

Customer churn costs retail banks 5–7× more per lost customer than retention would
have. This project answers two questions the product and CRM teams actually care about:

1. **Who is most likely to leave?** — a scored, rankable list of at-risk customers
2. **What kind of customer are they?** — segments that inform tailored interventions,
   not generic campaigns

---

## Key Results

| What | Result |
|------|--------|
| Best model AUC | **0.86** (Tuned Random Forest) |
| Recall improvement via threshold shift | **21.5% → 41.8%** at precision ≥ 52% |
| Churn gap across segments | **12.2% to 26.8%** — 11pp spread |
| Engineered feature rank | `income_v_product` ranked **3rd** in feature importance, above raw CreditScore |
| Highest-risk segment | 4,521 customers · £79k avg balance · **0% active members** |

---

## Project Walkthrough

### 1 · Data Cleaning
The raw Excel workbook arrived with duplicate rows, currency symbols embedded in
numeric fields, a `-999,999` sentinel value, missing ages, and inconsistent geography
labels (`"FRA"`, `"French"`, `"France"`). All resolved in a single reproducible
pipeline before any analysis.

### 2 · Exploratory Data Analysis
- **20.4% churn rate** — 3.9:1 class imbalance makes accuracy a misleading metric
- **German customers** churn at ~32% despite holding the highest average balances
  (£119,730 vs ~£62,000 for France/Spain)
- **Inactive members** churn at more than double the rate of active members — an
  observable, real-time signal for automated re-engagement triggers
- No multicollinearity — no feature pair exceeded 0.7 Pearson correlation

### 3 · Feature Engineering
Two domain-driven features created before modelling:

| Feature | Formula | Outcome |
|---------|---------|---------|
| `balance_to_income` | Balance ÷ EstimatedSalary | Relative wealth proxy |
| `income_v_product` | EstimatedSalary ÷ NumOfProducts | Underserved customer signal — ranked **3rd** in RF importance |

### 4 · Churn Prediction
Two models trained and evaluated:

**Logistic Regression (baseline)**
- AUC: 0.768 · Recall: 21.5% · F1: 0.318
- Interpretable coefficients confirm EDA: age and German geography increase churn
  risk; active membership and product count reduce it
- Threshold shifted from 0.5 → 0.34 to improve recall to 41.8% while keeping
  precision above 50% — nearly doubling churners caught

**Random Forest (tuned)**
- AUC: 0.86 · Accuracy: 85.5% · Precision: 80.4% · F1: 0.517
- Tuned via RandomizedSearchCV (100 iterations) + GridSearchCV refinement
- Top features: Age (0.270), NumOfProducts (0.157), `income_v_product` (0.088)
- Recommended for production deployment

### 5 · Customer Segmentation (K-Means)
Two rounds of clustering:
- **Round 1 (k=5, with geography):** clusters dominated by national boundaries,
  not behaviour — limited product utility
- **Round 2 (k=4, no geography):** surfaced four actionable behavioural segments

| Cluster | Profile | Size | Churn Rate |
|---------|---------|------|------------|
| 1 | High balance · zero engagement | 4,521 | **26.8%** |
| 0 | New multi-product customers | 709 | 21.0% |
| 3 | High balance · fully engaged | 2,827 | 15.5% |
| 2 | Active · no credit card | 1,943 | **12.2%** |

**Core finding:** Cluster 3 holds the *highest* average balance (£122,226) yet churns
at 15.5% — vs Cluster 1's £79,011 and 26.8% churn. Product engagement, not wealth,
drives loyalty.

---

## Recommendations

- **Deploy RF at threshold 0.34** for monthly churn scoring — catches ~94 extra
  churners per 1,000 customers scored vs. the default threshold
- **Priority outreach to Cluster 1** — 4,521 high-balance customers with no active
  membership or credit card; offer wealth management or investment products
- **Entry-level credit card for Cluster 2** — 1,943 engaged customers with no card;
  moving from 1 to 2 products is historically associated with lower churn
- **Tenure milestone rewards for Cluster 0** — 709 new multi-product customers
  showing early drop-off risk; reward at 6 and 12-month marks

---

## Tech Stack

- **Python** — Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn
- **Models** — Logistic Regression, Random Forest, K-Means Clustering
- **Tuning** — RandomizedSearchCV, GridSearchCV
- **Evaluation** — AUC-ROC, Precision-Recall curves, Silhouette scoring

---

## Dataset

`Bank_Churn_Messy.xlsx` — 10,000 customers across France, Germany, and Spain.
Includes demographics (age, gender, geography), account behaviour (balance, tenure,
products), and a binary churn label (`Exited`). Intentionally messy to reflect
real-world data quality challenges.

---

*This project is part of my data analytics portfolio. I'm currently open to analyst
roles in Fintech/BFSI — feel free to connect on 
https://www.linkedin.com/in/debadrita-deb-021772195/.*
