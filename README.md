# Subscription Revenue & Customer Lifecycle Intelligence

**Real KKBox subscriber data | segmentation | behavioral trajectories | churn prediction | retention prioritization**

This project demonstrates how customer analytics can move from **describing subscriber behavior** to **prioritizing commercial action**.

Using the real anonymized KKBox churn-competition dataset, the notebook combines membership attributes, transaction history and daily listening behavior to answer five business questions:

1. Which subscriber groups have meaningfully different commercial and engagement profiles?
2. Does recent behavioral trajectory add information beyond static segmentation?
3. How accurately can near-term churn be predicted?
4. Are predicted churn probabilities sufficiently calibrated for economic decision-making?
5. Under limited retention capacity, should the business target the most likely churners or the greatest subscription value at risk?

---

## Headline results

| Metric | Result |
|---|---:|
| Subscribers analyzed | 25,000 |
| Observed churn rate | 9.0% |
| Calibrated LightGBM ROC-AUC | **0.8866** |
| Calibrated LightGBM PR-AUC | **0.6628** |
| Calibrated LightGBM Log Loss | **0.1796** |
| Population contacted at capacity 500 | **2.0%** |
| Churners captured at capacity 500 | **22.0%** |
| Lift vs random at capacity 500 | **11.0×** |
| Churn value captured — probability ranking | **757,844** |
| Churn value captured — value-at-risk ranking | **863,977** |
| Incremental annualized value captured | **106,133** |

At a retention capacity of 500 subscribers, the model contacts only **2% of the population** while identifying about **22% of observed churners**. Ranking by expected annualized subscription value at risk sacrifices very little churn capture while increasing the annualized subscription value represented by selected churners by roughly **106k** in the sampled cohort.

> **Important:** Annualized Subscription Value (ASV) is an analytical run-rate proxy derived from plan duration and amount paid. It is **not** an official KKBox ARR metric.

---

## Business insight

The project separates two complementary views of customer risk:

- **Static segment:** who the subscriber is commercially
- **Engagement trajectory:** how consistently the subscriber has been using the service

The most commercially important group is the **Renewal-Risk** segment.

| Static segment | Engagement state | Subscribers | Churn rate |
|---|---|---:|---:|
| Renewal-Risk Subscribers | Intermittent / Low Engagement | 1,147 | **41.8%** |
| Renewal-Risk Subscribers | Consistent / High Engagement | 1,826 | **32.9%** |
| Stable Loyal Subscribers | Intermittent / Low Engagement | 3,730 | 6.8% |
| Stable Loyal Subscribers | Consistent / High Engagement | 4,149 | 4.4% |
| Low-Usage Auto-Renew Subscribers | Intermittent / Low Engagement | 3,832 | 3.2% |
| Low-Usage Auto-Renew Subscribers | Consistent / High Engagement | 2,221 | 2.5% |

This shows that **engagement state changes risk within the same commercial segment**, while renewal behavior remains the dominant risk signal.

---

## Analytical approach

### 1. Point-in-time customer mart
A feature cutoff of **2017-02-28** is used to construct a reproducible churn cohort from:

- `members_v3.csv`
- `train_v2.csv`
- `transactions.csv`
- `user_logs.csv`

The full label cohort contains almost one million subscribers. A **25,000-subscriber stratified sample** preserves the original churn prevalence while keeping the analysis practical in Google Colab.

### 2. Subscription-value engineering
Plan duration and payment amount are normalized into:

- **Monthly Subscription Value (MSV)**
- **Annualized Subscription Value (ASV)**

This creates a common run-rate proxy across different subscription lengths.

### 3. Static subscriber segmentation
K-means clustering combines:

- subscription value
- tenure
- transaction history
- auto-renew and cancellation behavior
- recent usage and engagement

Cluster count is evaluated with Silhouette, Davies-Bouldin and Calinski-Harabasz metrics.

### 4. Behavioral trajectory analysis
Two approaches are tested:

- **DTW time-series clustering** on raw 30-day listening trajectories
- engineered trajectory features such as activity level, volatility, week-over-week change, slope and inactivity streaks

DTW produced weak cluster separation and was therefore **not forced into the final solution**. A cleaner two-cluster trajectory representation was retained:

- **Intermittent / Low Engagement**
- **Consistent / High Engagement**

### 5. Churn modeling
Two supervised models are compared:

| Model | Log Loss | ROC-AUC | PR-AUC |
|---|---:|---:|---:|
| Logistic Regression | 0.3876 | 0.8734 | 0.5999 |
| LightGBM | 0.2435 | 0.8847 | 0.6606 |
| **Calibrated LightGBM** | **0.1796** | **0.8866** | **0.6628** |

Because churn prevalence is around 9%, **PR-AUC** is emphasized alongside ROC-AUC.

### 6. Probability calibration
Class weighting improves minority-class learning but makes raw probabilities too high on average.

Before calibration:

- observed churn rate: ~9.0%
- average raw LightGBM probability: ~18.3%

After calibration:

- average calibrated probability: ~8.7%

This makes the model more suitable for expected-value calculations.

### 7. Value-at-risk prioritization
Expected annualized subscription value at risk is defined as:

```text
Expected ASV at Risk = Calibrated P(Churn) × Annualized Subscription Value
```

Two clean targeting strategies are compared:

- **Highest churn probability** — maximize the number of churners identified
- **Highest annualized value at risk** — maximize economic value represented by selected churners

The notebook also tests a heuristic retention-priority score, but it is not recommended as the headline strategy because it does not outperform the simpler value-at-risk approach.

---

## Capacity-constrained retention results

### Churner capture

| Capacity | Highest churn probability | Highest ASV at risk |
|---:|---:|---:|
| 100 | 4.4% | 4.4% |
| 250 | 11.0% | 11.0% |
| 500 | **22.0%** | 21.7% |
| 1,000 | **43.3%** | 43.0% |

### Churn value captured

| Capacity | Highest churn probability | Highest ASV at risk |
|---:|---:|---:|
| 100 | 3.7% | **4.9%** |
| 250 | 9.8% | **12.1%** |
| 500 | 20.3% | **23.2%** |
| 1,000 | 39.6% | **46.1%** |

At capacity 500, value-based ranking captures roughly **106k more annualized subscription value** than probability-only ranking in the sampled cohort.

---

## Why this project matters

The goal is not simply to produce a churn score.

The project moves through the full commercial decision chain:

```text
Raw customer behavior
        ↓
Feature engineering
        ↓
Customer segmentation
        ↓
Engagement trajectory
        ↓
Churn propensity
        ↓
Probability calibration
        ↓
Value at risk
        ↓
Capacity-constrained retention decision
```

That makes the output directly relevant to **commercial analytics, RevOps, customer success, growth analytics and subscription strategy**.

---

## Repository contents

```text
kkbox-customer-lifecycle-intelligence/
│
├── KKBox_Customer_Lifecycle_Intelligence.ipynb
├── README.md
├── RESULTS.md
├── DATA_ACCESS.md
├── requirements.txt
├── .gitignore
└── LICENSE
```

Raw KKBox data is intentionally **not included**.

---

## How to run

### Recommended environment
Google Colab.

### Steps

1. Join the Kaggle competition:
   **WSDM - KKBox's Churn Prediction Challenge**
2. Accept the competition rules.
3. Generate a Kaggle API access token.
4. Open the notebook in Google Colab.
5. Run cells from top to bottom.
6. When prompted, paste the Kaggle token into the secure `getpass` input.

The notebook downloads only the four required competition files and processes the large tables in chunks.

---

## Data and licensing

The KKBox competition data is **not redistributed in this repository**.

Users must obtain the data directly through Kaggle after accepting the competition rules. See [`DATA_ACCESS.md`](DATA_ACCESS.md).

The MIT license in this repository applies to the original project code and documentation only. It does **not** grant rights to KKBox or Kaggle data.

---

## Methodological limitations

- The analysis uses historical competition data rather than a live production system.
- ASV is an analytical normalization proxy, not an official KKBox accounting metric.
- Model validation uses a stratified holdout from a single churn cohort.
- A stronger production design would use true out-of-time validation.
- Clustering is descriptive, not causal.
- Retention simulations identify who should be prioritized, not who would actually be saved by intervention.
- Expected value at risk represents model-based exposure, not guaranteed recoverable revenue.

---

## Next extensions

- out-of-time validation using an earlier KKBox cohort
- survival analysis for time-to-churn
- uplift modeling for treatment responsiveness
- causal retention-effect estimation
- explicit offer/contact costs and retained lifetime value
- monitoring for feature drift, calibration drift and segment migration

---

## Portfolio positioning

**Suggested project title**

> Subscription Revenue & Customer Lifecycle Intelligence — Segmentation, Churn Prediction and Retention Optimization

**Short description**

> Built an end-to-end customer lifecycle analytics project using real anonymized KKBox subscriber, transaction and listening data. Combined customer segmentation and engagement-state analysis with calibrated LightGBM churn prediction, annualized subscription value-at-risk estimation and capacity-constrained retention prioritization.
