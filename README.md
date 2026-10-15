# Payment Fraud Detection Project

A data science project that ranks payments by fraud risk, explains every alert in plain English,
checks the model is fair, and tests it the way a bank would before using it.

**Live dashboard:** *(coming soon)*
**Full write-up:** [Insight report](insight_report.md) · [Model card](model_card.md)

---

## Results at a glance
- Caught **99.7% of fraud** on unseen data, with **84% of alerts being real fraud**.
- Needed **97% fewer alerts** than a simple rule (4,757 vs 186,801).
- PR-AUC of **0.981**, compared with **0.564** for a simple baseline model.
- In a simulated trial against the simpler model, it caught **95% of fraud vs 68%**, with
  **73% fewer false alarms**, at the same investigator workload.

## The problem
Fraud investigators can only check a small number of payments each day, so they need to know
which ones to look at first. Fraud is rare (about 3 in every 1,000 transfers), so a model that
always says "not fraud" would look 99.7% accurate while catching nothing. I measured success by
how much fraud is caught and how many alerts investigators have to review.

The data is PaySim, a public synthetic dataset of 6.3 million mobile-money payments.

## What I found

### 1. Checking the data first stopped a misleading result
Some columns recorded balances *after* each payment, which gave the answer away. A model using
them scored more than twice as well, but a bank wouldn't have that information when deciding
whether to stop a payment. I removed them and only used information available beforehand.

![Leakage check](figures/leakage_comparison.png)

### 2. Much more precise than a simple rule
Almost all fraud empties the victim's account. But flagging every payment that does this gives
186,801 alerts, and only 2 in 100 are fraud. The model learns *which* of these payments are
actually suspicious, for example large ones sent to brand-new accounts.

![Recall and precision](figures/threshold_tradeoff.png)

### 3. Four different kinds of fraud
Grouping the fraud cases by behaviour showed four patterns. The biggest, transfers to brand-new
accounts, makes up two-thirds of all fraud. Most of the model's 12 misses were cash-outs through
older, quiet accounts that looked like normal activity, which shows where to improve next.

![Fraud types](figures/cluster_plot.png)

### 4. Every alert comes with reasons
Each flagged payment shows its top three reasons, such as *"Payment empties the account ·
Receiver is brand new"*, so investigators can act quickly and explain decisions to customers.

![What drives the model](figures/shap_bar.png)

### 5. Fairness check
The data has no customer ages, so I created a made-up age group to test the checking method.
Genuine payments were rarely flagged for anyone, but younger customers were flagged about twice
as often as over-65s, through linked spending behaviour rather than age itself. This shows how
this kind of hidden bias can be found.

![Fairness check](figures/fairness_fpr.png)

### 6. Tested like a bank would
I simulated a trial: half the payments scored by the old model and half by the new one, each
flagging the same number of payments. Beforehand I worked out how much data was needed (about
4 days of live traffic). The new model won clearly, and the result held across 50 repeats.

![Trial result](figures/experiment_result.png)

## Limitations
- PaySim is synthetic, and real fraud is messier, so real-world results would be lower.
- The money amounts are unrealistically large, so I don't quote savings in pounds.
- In real life fraud is often confirmed weeks later, and patterns change over time.

## Tools
Python, pandas, scikit-learn, LightGBM, SHAP, Fairlearn, SciPy, statsmodels, Matplotlib, Streamlit

*The code is in a private repository and is available on request.*

---
*Chanakya Shah · BSc Computer Science, Kingston University London*
