# Results Summary

## Dataset and cohort
- Real anonymized KKBox churn competition data
- 25,000-subscriber stratified portfolio sample
- Observed churn rate: **8.996%**
- Feature cutoff: **2017-02-28**

## Segmentation
The final static segmentation uses five K-means clusters.

A separate behavioral trajectory layer was created after testing raw DTW time-series clustering.

### DTW result
Proper DTW silhouette scores were weak:
- k=3: ~0.104
- k=4: ~0.090
- k=5: ~0.081

The notebook therefore pivots to interpretable trajectory features rather than forcing weak raw time-series clusters.

### Final engagement-state solution
Two robust trajectory groups:
- Intermittent / Low Engagement
- Consistent / High Engagement

## Cross-segment churn
- Renewal-Risk + Intermittent / Low Engagement: **41.8% churn**
- Renewal-Risk + Consistent / High Engagement: **32.9% churn**
- Stable Loyal + Intermittent / Low Engagement: **6.8% churn**
- Stable Loyal + Consistent / High Engagement: **4.4% churn**
- Low-Usage Auto-Renew + Intermittent / Low Engagement: **3.2% churn**
- Low-Usage Auto-Renew + Consistent / High Engagement: **2.5% churn**

## Predictive modeling

| Model | Log Loss | ROC-AUC | PR-AUC |
|---|---:|---:|---:|
| Logistic Regression | 0.38761 | 0.87344 | 0.59986 |
| LightGBM | 0.24353 | 0.88471 | 0.66061 |
| Calibrated LightGBM | **0.17965** | **0.88664** | **0.66281** |

Calibrated LightGBM Brier score: **0.04774**

## Calibration
- Observed churn rate: ~8.99%
- Raw LightGBM mean predicted probability: ~18.3%
- Calibrated mean predicted probability: ~8.7%

## Retention targeting

At capacity 500:
- population contacted: **2.0%**
- churners captured by probability ranking: **22.0%**
- lift vs random: **~11.0x**
- annualized churn value captured by probability ranking: **757,844**
- annualized churn value captured by ASV-at-risk ranking: **863,977**
- incremental annualized value represented: **106,133**

At capacity 1,000:
- churners captured by probability ranking: **43.3%**
- churners captured by ASV-at-risk ranking: **43.0%**
- churn value captured by probability ranking: **39.6%**
- churn value captured by ASV-at-risk ranking: **46.1%**

## Recommended decision rule
- Use calibrated churn probability when the goal is to maximize the number of subscribers identified.
- Use expected annualized subscription value at risk when the goal is to maximize economic value protected.
