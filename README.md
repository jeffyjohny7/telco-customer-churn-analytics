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

## 🤖 Phase 2: Predictive Modeling (Machine Learning)

Instead of offering blanket discounts to all Month-to-Month customers (which would severely hurt revenue), I developed a Machine Learning model to identify the specific customers with the highest probability of leaving.

### Data Preprocessing

- Handled hidden missing values in `TotalCharges`.
- Applied One-Hot Encoding to categorical variables using `pd.get_dummies()`.
- Mapped target variable (`Churn`) to binary integer format.

### Model Selection & Hyperparameter Tuning

I trained and evaluated two advanced tree-based algorithms, utilizing `GridSearchCV` to specifically optimize for **Recall**. In a churn scenario, False Negatives (missing a churner) cost the business significantly more than False Positives (offering a small discount to a loyal customer).

**1. Tuned Random Forest Classifier**

- Hyperparameters: `class_weight='balanced'`, `max_depth=10`
- **Recall (Churners): 81%**
- Overall Accuracy: 77%

**2. Tuned XGBoost (Extreme Gradient Boosting)**

- Hyperparameters: `learning_rate=0.01`, `max_depth=3`, `scale_pos_weight=2.77`
- **Recall (Churners): 87%**
- Overall Accuracy: 74%

| Model | Recall (Churners) | Accuracy |
|---|---|---|
| Tuned Random Forest | 81% | 77% |
| Tuned XGBoost | **87%** | 74% |

---

## 💡 Business Recommendation & Deployment Strategy

The XGBoost model achieved **87% recall**, successfully identifying the vast majority of flight-risk customers.

### Actionable Steps

1. **Low-Cost Interventions:** For low-cost retention strategies (e.g., automated check-in emails, feature awareness campaigns), deploy the **XGBoost model** to cast the widest net possible.
2. **High-Cost Interventions:** If the retention strategy involves high-value financial discounts, fall back to the **Random Forest model**, which balances a highly respectable 81% recall with slightly better precision, minimizing unnecessary revenue loss on False Positives.