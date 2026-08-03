# 📉 Telco Customer Churn Analytics & Predictive Modeling

**Live Interactive Dashboard:** https://datastudio.google.com/reporting/9ae07afb-ac11-47a2-a8d6-883405650370

---

## 📌 Project Overview

This end-to-end project investigates why a telecommunications company is losing customers (Churn) and builds a predictive Machine Learning pipeline to identify high-risk customers **before** they leave.

**Tools Used:** Python (Pandas, Scikit-Learn, XGBoost), SQL (MySQL), Google Looker Studio, Machine Learning (Random Forest, Gradient Boosting).

---

## 📊 Phase 1: KPI Extraction & Root Cause Analysis (SQL + Looker Studio)

Using MySQL, I processed over **7,000 customer records** to extract core business metrics.

### Business Impact

| Metric | Value |
|---|---|
| Overall Churn Rate | **26.54%** |
| Monthly Revenue at Risk | **₹139,130** |

I connected the aggregated SQL views to Google Looker Studio to build an interactive dashboard and identify the root cause.

### Root Cause Insight

The dashboard revealed that **Month-to-Month customers on premium Fiber Optic plans account for the vast majority of churn.** Because Fiber Optic is a premium service, customers refusing to sign 1-year contracts are highly price-sensitive and likely jumping to competitors after introductory promos expire.

---

## 🤖 Phase 2: Feature Engineering & Data Preprocessing

To give the Machine Learning algorithms the best possible chance of finding hidden patterns, I enriched the raw dataset:

- **Feature Engineering:** Created an `Extra_Charges_Fee` column by calculating `TotalCharges - (MonthlyCharges * Tenure)`. A high discrepancy here flags customers who are being hit with hidden fees — a strong psychological driver for churn.
- **Categorical Encoding:** Applied One-Hot Encoding (`pd.get_dummies()`) to convert text-based categories into machine-readable binary formats.

---
## ⚖️ Phase 3: Advanced Machine Learning & SMOTE

### The Imbalance Problem

Because only 26% of customers churn, standard models get "lazy" and just predict that everyone will stay — resulting in high accuracy but terrible business value (missing the actual churners).

### The Solution

- **SMOTE (Synthetic Minority Over-sampling Technique):** I utilized SMOTE to generate synthetic, mathematically accurate "churners" in the training data, forcing the algorithm to learn on a perfectly balanced 50/50 dataset.
- **XGBoost & GridSearchCV:** I trained an Extreme Gradient Boosting (XGBoost) classifier, explicitly optimizing for the **F1-Score** to find the mathematical middle ground between Precision (not guessing randomly) and Recall (catching the churners).

**Initial Tuned Result:** The model achieved an Overall Accuracy of **78.8%**, but Recall (ability to catch churners) sat at **68.6%**.

---
## 🎯 Phase 4: Business Value via Custom Threshold Tuning

The hardest truth in Machine Learning is the **Precision-Recall Trade-off**: you cannot force both numbers to go up simultaneously.

Instead of retraining the model, I extracted the raw prediction probabilities (`predict_proba`) and tested **Custom Classification Thresholds**. By default, algorithms use a rigid 50% confidence threshold to flag a churner. By manually lowering this threshold, I created a "volume knob" for business stakeholders to choose their strategy based on retention budgets:

| Strategy | Threshold | Overall Accuracy | Recall (Caught Churners) | Precision |
|---|---|---|---|---|
| **Aggressive** | 30% | 69.5% | **86.6%** | 46.0% |
| **Balanced** | 40% | 75.4% | 78.3% | 52.3% |
| **Conservative** | 50% | **78.8%** | 68.6% | **58.5%** |

---


## 🚀 Final Deployment Recommendations

As a Data Scientist, my goal is to give business leaders actionable levers to maximize revenue:

1. **The Aggressive Strategy (Threshold 0.30):** If our retention campaign is very cheap (like an automated email), we lower the threshold to 30%. We catch a massive **86.6%** of all leaving customers. We accept the lower accuracy because accidentally emailing a loyal customer costs us nothing.

2. **The Balanced Strategy (Threshold 0.40):** The sweet spot. If we are offering a moderate $10 discount, we capture an excellent **78.3%** of churners while keeping accuracy highly respectable at 75%.

3. **The Conservative Strategy (Threshold 0.50+):** If we are giving away expensive hardware or huge cash discounts, we only want to target people we are absolutely sure are leaving — protecting our profit margins.