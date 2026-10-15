# Model Card: Payment Fraud Detection Model

## Model details
- **Type:** LightGBM gradient-boosted trees (best of several hand-picked settings, chosen on the
  validation set; 44 trees), with isotonic calibration.
- **Output:** a fraud probability for each payment; payments at or above the chosen threshold
  (0.01) are flagged for review.
- **Author:** Chanakya Shah.

## Intended use
- **For:** ranking payments so fraud investigators review the riskiest first.
- **Not for:** automatically blocking payments or making decisions without human review.

## Data
- PaySim: synthetic mobile-money transactions (6.36M); only TRANSFER and CASH_OUT used (2.77M),
  as fraud occurs only in these.
- Time-based split: train steps ≤ 322 (1.94M rows), validation 323-376 (416k), test > 376 (416k).
- The test period has about 5x the fraud rate of training, which inflates absolute test PR-AUC.

## Features
- 15 features using only information available **before** a payment completes: amount, sender's
  starting balance, time of day, and receiving-account history (money-mule signals).
- After-payment balance columns were excluded after they were shown to leak the label.

## Performance (test set)
- PR-AUC **0.981** (95% CI 0.978-0.984) vs logistic regression baseline 0.564; improvement +0.417
  (paired bootstrap 95% CI +0.402 to +0.433).
- At the chosen threshold: **99.7% of fraud caught (4,008 of 4,020), 84.3% precision, 4,757 alerts.**
- With 100 alerts/day: 38.8% of fraud caught (the maximum possible is about 39%).
- Time-series cross-validation: PR-AUC 0.947 ± 0.021.
- A one-line rule ("flag if the payment empties the account") needs 186,801 alerts at 2.1% precision.
- Without balance features, PR-AUC drops to 0.247 (ablation). Adding an Isolation Forest anomaly
  score made no difference, so it was not used.

## Explainability
- Every alert comes with its top 3 value-aware, plain-English reasons (SHAP).
- Most important features: whether the payment empties the account and the payment as a share of
  balance (about 66% of total impact), then payment size and sender balance.
- Main weakness: cash-outs through established, quiet receiving accounts from smaller accounts,
  which look like normal activity (9 of the 12 missed frauds).

## Fairness
- Tested with a **synthetic** age attribute linked to behaviour (PaySim has no demographics).
- False positive rates: 18-34 0.239%, 35-64 0.184%, 65+ 0.113%; ratio 0.47 (below the 0.8 rule
  of thumb); chi-square p < 0.0001.
- Fraud-catch rates were equal across groups (about 99.7%).

## Limitations
- Synthetic data: PaySim's fraud is simple and dominated by account-draining.
- Money figures are not realistic because PaySim amounts are very large.
- No label delay modelled, and no drift monitoring yet.

## Ethical considerations
- Wrongly flagged payments delay customers' legitimate payments; this harm may fall unequally
  on some groups, including vulnerable customers.
- Human review of every alert, clear reasons and regular fairness checks are required,
  consistent with the FCA's Consumer Duty expectation of good customer outcomes.
- No real customer data was used.