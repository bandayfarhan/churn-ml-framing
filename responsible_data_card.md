# Responsible Data Card: customer-churn-training

## Dataset purpose
**May support:** ranking active customers for a human-reviewed retention outreach list, and exploratory analysis of churn drivers.

**Must not support:** automated pricing, plan changes, service denial, credit or eligibility decisions, penalising individual customers, or any use outside retention outreach. At 12 rows it must not be used to train or validate a production model.

## Provenance and permission
- File: `customer-churn-training.csv`, 12 rows, 7 columns, UTF-8.
- Creation method, source system, extraction date, and observation window: **not documented; owner must supply.**
- Consent / lawful basis: **unknown.** Confirm that customers agreed (or the privacy notice covers) use of usage and support data for retention analytics. Currency is INR, suggesting Indian customers, so check obligations under the Digital Personal Data Protection Act, 2023.
- Licensing and sharing: assumed internal use only. Do not share outside the retention/analytics team without approval.
- Retention: delete or re-pseudonymise `customer_id` once linkage to source systems is no longer needed.

## Population and representation
- 12 customers: Basic 5, Standard 4, Pro 3.
- Tenure 1 to 30 months; monthly spend INR 499 to 1,499; churn rate 5/12 (41.7%).
- Sampling method unknown. The high churn rate and clean separation suggest a curated or synthetic sample rather than a random draw of the customer base.
- Missing groups: no age, region, or segment fields, so representation cannot be assessed. No customers with tenure over 30 months, no Pro-plan churners, and no customers with more than 5 tickets.
- Known imbalance: all churners are Basic-plan; Basic is the only plan with churn events, so plan effects cannot be separated from tenure, spend or tickets.

## Features and target
| Column | Type | Meaning | Concerns |
|---|---|---|---|
| `customer_id` | string | Pseudonymous customer key | Direct identifier; exclude from modelling. |
| `tenure_months` | int | Months since signup | Fine if measured at snapshot date. |
| `support_tickets` | int | Tickets raised (window not stated) | **Leakage risk** if counted up to or after cancellation; may also reflect service quality problems, not customer traits. |
| `monthly_spend_inr` | int | Monthly spend in INR | Tied to plan (Basic 499 to 799, Standard 799 to 999, Pro 1,299 to 1,499); possible income proxy. |
| `last_login_days` | int | Days since last login | **Leakage risk**: inactivity may be a consequence of churn, not a cause. |
| `plan_type` | string | Basic / Standard / Pro | Perfectly predictive in this sample; possible socioeconomic proxy. |
| `churned` | int (0/1) | Target: churned | Definition and window **not documented**. |

## Quality checks
Computed in `baseline_notebook.ipynb`:
- Missingness: 0 missing values.
- Duplicates: 0 duplicate rows, 0 duplicate `customer_id`.
- Outliers: none apparent in a table this small (tickets 0 to 5, login days 1 to 30).
- Class balance: 5 churned / 7 retained.
- Train/test separation: **none exists.** Only 12 rows; leave-one-out is the sole feasible check and is highly unstable. A time-based split is required for any real evaluation.
- Separation: tickets >= 3, last login >= 10 days, and Basic plan each separate the classes perfectly. This is a red flag for leakage or an unrepresentative sample.

## Risks and safeguards
| Risk | Why it matters | Safeguard |
|---|---|---|
| Bias / proxy | Plan and spend may proxy for income; low-spend customers could be targeted or neglected differently. | Report metrics by plan and tenure band; use scores only to add support, never to reduce service. |
| Privacy | Behavioural data tied to an ID. | Pseudonymise, restrict access, log use, honour deletion requests. |
| Misuse | Score repurposed for pricing or denial. | Document prohibited uses; access control; review before any new use. |
| False negatives | Missed churners mean lost revenue. | Rule fallback; threshold chosen by cost; periodic review of unflagged customers. |
| False positives | Unneeded offers waste budget and annoy customers. | Human review; outreach capacity cap; limit offer frequency per customer. |
| Overfitting | n=12, perfect separation. | No production model; gather more data. |

## Intended evaluation
Defined before any training:
- **Baselines:** majority class (0/5 recall), rule `support_tickets >= 3 or last_login_days >= 10`.
- **Performance:** recall, precision, PR-AUC at the cost-optimal threshold, on a time-based holdout.
- **Calibration:** Brier score and reliability curve, compared with a constant-rate predictor.
- **Fairness:** recall, precision and flag rate by plan type and tenure band, with agreed maximum gaps.
- **Error analysis:** review every false negative and false positive; check whether errors cluster by plan, tenure or ticket count.
- **Success criterion:** beat the rule baseline on expected net cost (retained revenue minus outreach cost), not just accuracy.
