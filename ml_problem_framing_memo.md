# ML Problem-Framing Memo: Customer Churn

Date: 2026-10-12 · Data reviewed: `customer-churn-training.csv` (12 rows, 7 columns)

## 1. Recommendation

**Do not train a production ML model yet.** Use a transparent rule-based watchlist as the baseline and fallback. Re-evaluate ML only after the gates in section 7 are met.

Why: on the 12-row sample, the rule `support_tickets >= 3` flags all 5 churners and no non-churners (TP=5, FP=0, FN=0, TN=7). A leave-one-out logistic regression matches the recall but adds one false positive (TP=5, FP=1, FN=0, TN=6). The model adds complexity without measurable benefit, and 12 rows cannot support any reliable estimate of generalisation. Perfect separation on a sample this small is a warning sign, not a success.

## 2. Decision being supported

| Item | Definition |
|---|---|
| Decision | Which active customers get a proactive retention outreach (call, offer or support follow-up) this cycle. |
| Decision owner | Customer retention / success team (to be confirmed). |
| Not allowed to decide | Pricing, plan downgrades, service denial, credit or eligibility decisions, or any customer-specific penalty. The score only prioritises *help*. |
| Prediction target | `churned` = 1 if the customer cancels or goes inactive within the action window (exact definition **not documented in the file; must be confirmed**). |
| Unit of observation | One customer at one snapshot date (one row per customer in the sample). |
| Action window | Proposed: snapshot at start of month, features measured on or before that date, churn observed over the next 30 days. This gives the team time to act. |
| Non-ML baseline | Rule watchlist: flag if `support_tickets >= 3` or `last_login_days >= 10`; reviewed by a human before outreach. |

## 3. What the data shows (and does not)

- 12 customers, 5 churned (41.7%), no missing values, no duplicate rows or IDs.
- All 5 Basic-plan customers churned; none of the 3 Pro or 4 Standard customers did.
- Churners: tenure 1 to 8 months, 3 to 5 tickets, spend INR 499 to 799, last login 12 to 30 days ago. Non-churners: tenure 7 to 30 months, 0 to 2 tickets, spend INR 799 to 1,499, last login 1 to 9 days ago.
- Classes are separable on several features at once, which at n=12 usually means the sample is too small or too clean, or that features were captured after churn began.

Not known: how the sample was drawn, the observation dates, the churn definition, and whether features were measured before or after the churn event.

## 4. Metrics and error costs

Costs below are **placeholders to be replaced with finance/retention figures**; the structure is what matters.

| Outcome | Meaning | Assumed cost driver |
|---|---|---|
| False negative | A customer who will churn is not contacted. | Lost revenue: monthly spend x expected remaining months (INR 499 to 1,499 per month in the sample). |
| False positive | A stable customer is contacted or offered a discount. | Outreach labour cost plus unnecessary discount and possible annoyance. |

Decision rule: choose the threshold that minimises expected cost, `FN_cost x FN + FP_cost x FP`, on a time-based holdout, subject to a capacity cap on outreach volume.

- **Business metrics:** retained revenue (INR), outreach cost, net saving versus no-action and versus the rule baseline.
- **Model metrics:** recall and precision at the operating threshold, PR-AUC, Brier score and calibration curve (class rate is 41.7% in the sample and will likely be lower in production), plus metrics by plan type and tenure band.
- **Reference points from the sample:** the majority-class baseline gets 0 of 5 churners. Constant-rate Brier is 0.243; the LOO logistic model gets 0.059 (descriptive only, n=12).

## 5. Abstention, human review and monitoring

- **Abstain** (no automated score, route to human) when: any feature is missing or out of the training range, tenure is under the minimum observed window, plan type is unseen, or the model and the rule baseline disagree strongly.
- **Human review** for every outreach decision in the first release; reviewers can override and must record a reason. No fully automated action that affects price or service.
- **Monitor** weekly: input drift (tickets, login recency, spend, plan mix), score distribution, outreach volume, realised churn among flagged vs unflagged customers, calibration, and override rate.
- **Rollback to the rule baseline** if: calibration error or recall falls below pre-agreed limits for two consecutive weeks, input drift alarms fire on a key feature, a data-pipeline fault is found, or a fairness gap exceeds the agreed tolerance.

## 6. Leakage and fairness concerns to resolve first

1. `support_tickets` and `last_login_days` may be recorded *after* a customer decides to leave (cancellation tickets, no logins after cancelling). If so, they are leakage and the rule is useless as an early warning.
2. `plan_type` perfectly predicts churn in this sample; treat it as a possible proxy for income or segment and confirm it is not a stand-in for a protected group.
3. `customer_id` is a direct identifier and must never be a feature.

## 7. Gates before training an ML model

- At least several hundred churn events across multiple months, with timestamps, so a time-based split is possible.
- Confirmed churn definition and confirmed feature timing (no post-outcome fields).
- Documented consent or lawful basis for this use (see data card).
- ML must beat the rule baseline on expected net cost, not just accuracy, on a time-based holdout.

## 8. Deliverables in this package

- `responsible_data_card.md`: filled from the supplied template.
- `baseline_notebook.ipynb`: executed baseline, rules and LOO comparison.
- `risk_register.md`: risks, owners, mitigations and triggers.
