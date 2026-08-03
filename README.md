# 📉 IBM Telco Customer Churn Analytics & Predictive Modeling

**Live Interactive Dashboard:** https://datastudio.google.com/reporting/9ae07afb-ac11-47a2-a8d6-883405650370
---

## 📌 Project Overview

This end-to-end project investigates why a telecommunications company is losing customers and builds an advanced Machine Learning pipeline to identify high-risk customers, using the standard IBM Telco dataset.

More importantly, it translates standard machine learning metrics into direct business value — optimizing the model to maximize expected profit from retention campaigns rather than simply chasing accuracy.

**Tools Used:** Python (Pandas, Scikit-Learn, XGBoost, SHAP), SQL (MySQL), Google Looker Studio.

---

## 📊 Phase 1: KPI Extraction & Business Impact (SQL + Looker)

Using MySQL, I processed over 7,000 customer records to extract core business metrics.

### Business Impact

| Metric | Value |
|---|---|
| Overall Churn Rate | **26.54%** |
| Monthly Revenue at Risk | **₹139,130** |

The interactive Looker Studio dashboard revealed that **Month-to-Month customers on premium Fiber Optic plans account for the vast majority of churn.**

---

## 🤖 Phase 2: Feature Engineering & The "Honest Data Scientist" Pivot

To give the ML algorithms the best possible chance of finding hidden patterns, I engineered new features, including:

- **`Extra_Charges_Fee`:** Calculated as `TotalCharges - (MonthlyCharges * Tenure)`. My initial hypothesis was that hidden fees were driving customers away.

### The SHAP Pivot

After training the model, I used SHAP (SHapley Additive exPlanations) values to look inside the black box. **The SHAP summary plot objectively proved my initial hypothesis wrong** — `Extra_Charges_Fee` barely registered as a churn driver.

Instead of hiding this, I let the data dictate the strategy and dropped the engineered feature. The SHAP plot revealed the true root causes of churn:

- **Fiber Optic:** Having a Fiber Optic plan massively increases churn probability.
- **Monthly Charges:** The base price of the plans is the true friction point, not hidden fees.
- **Tenure:** Customers are highly likely to abandon the service early in their lifecycle.

---

## ⚖️ Phase 3: Model Selection & The SMOTE Ablation

I benchmarked progressively more complex approaches rather than reaching straight for gradient boosting.

| Model | ROC-AUC | PR-AUC |
|---|---|---|
| Random Forest (baseline) | 0.76 | — |
| Logistic Regression | 0.84 | — |
| **XGBoost (no resampling)** | **0.86** | — |
| XGBoost + SMOTE | 0.80 | — |

**The Logistic Regression Baseline:** A plain logistic regression reached ROC-AUC 0.84 — within two points of the tuned ensemble. Establishing this first prevents mistaking model complexity for model value.

**The SMOTE Ablation:** I initially introduced SMOTE to balance the classes, using `imblearn.pipeline` so synthetic samples are generated only inside the training folds during cross-validation, keeping validation folds uncontaminated.

The ablation showed SMOTE **degraded ranking quality: XGBoost fell from 0.86 to 0.80 ROC-AUC.** The likely cause is that standard SMOTE interpolates linearly across one-hot encoded categorical features, manufacturing impossible customer profiles (e.g. `Contract_Month-to-month = 0.37`).

I tried SMOTE, measured it, and removed it. The final champion is a non-resampled XGBoost model at **ROC-AUC 0.86**, with class imbalance handled at the decision threshold instead — see Phase 4.

---

## 🎯 Phase 4: Expected Value & Profit Optimization

A standard model uses a rigid 50% confidence threshold. A deployable solution derives its threshold from business costs.

*(Note: the ₹2,000 LTV is illustrative; this framework accepts real finance numbers directly.)*

### Business Assumptions

| Parameter | Value |
|---|---|
| Margin Saved (successful retention) | ₹2,000 |
| Retention Success Rate | 30% |
| **Expected value per True Positive** | **₹600** |
| Cost of Offer (per customer contacted) | ₹200 |

### The Profit Equation

A false negative is **not** a model cost. If we do nothing, that customer churns and we lose them anyway — that is the baseline, not a penalty the model introduces. Profit must be measured *relative to doing nothing*. We pay ₹200 for everyone we contact, but earn ₹600 only on true positives.

```
Expected Profit = TP × ₹600 − (TP + FP) × ₹200
```

### Deriving the Optimal Threshold

The decision is made per customer, not in aggregate. Contacting a customer with predicted churn probability `p` is profitable when:

```
p × ₹600 − ₹200 > 0    →    p > 1/3
```

**The optimal threshold is therefore 0.333 — it equals the cost ratio ₹200 / ₹600.** No sweep is strictly required; the sweep confirms it.

---

## 🚀 Final Deployment Recommendation

Sweeping thresholds from 0.05 to 0.95 produces a genuine interior maximum at **0.32**, matching the analytically derived 0.333. That agreement is itself a useful result: it indicates the model's predicted probabilities are **well calibrated**, so they can be read as genuine probabilities rather than arbitrary confidence scores.

At this threshold, precision is 46%. Per 1,000 customers flagged:

```
460 true positives  × ₹600  =  ₹276,000
1,000 contacted     × ₹200  =  ₹200,000
                    Net profit  =  ₹76,000
```

The 540 false alarms look wasteful in isolation, but each costs only ₹200 while each catch returns ₹600 — so the wider net is comfortably profitable. **Total expected campaign profit on the held-out test set: ₹[INSERT ACTUAL FIGURE].**

**Sensitivity:** because the threshold equals the cost ratio, it moves predictably with business inputs. A cheaper ₹50 offer pushes the optimum to 0.083 (cast a far wider net); an expensive ₹400 offer pushes it to 0.667 (target only near-certain churners). The framework adapts without retraining.

---

## 🔮 Next Steps: Maturity & Uplift Modeling

This model maximizes profit based on *likelihood of churn*, assuming a flat 30% retention success rate across all customers. The honest next step is **Uplift Modeling**.

Discounting high-risk customers who are determined to leave regardless of any offer is pure financial loss. Future iterations will target **persuadable** customers rather than merely likely churners — modeling the *incremental effect* of the intervention instead of the probability of the outcome.

---

## 🛠️ Reproducibility & Setup

- **Dataset:** IBM Telco Customer Churn (7,043 records)
- **Seed:** `random_state=42` across all splits and algorithms
- **Data Split:** 80/20 stratified train/test
- **Environment:** `pip install -r requirements.txt`

```
project-root/
├── requirements.txt
├── notebooks/
│   ├── 01_eda_and_sql_queries.ipynb
│   └── 02_xgboost_and_expected_value.ipynb
├── sql/
└── images/
```