# Risk Register: Customer Churn ML Framing

Scale: Likelihood (L) and Impact (I) rated Low / Med / High. Owners are roles to be assigned.

| # | Risk | L | I | Mitigation | Trigger / early signal | Owner |
|---|---|---|---|---|---|---|
| R1 | **Too little data**: 12 rows, 5 positives; any model is unreliable. | High | High | No production model; use rule baseline; collect several months of timestamped data. | Anyone proposes deploying a model trained on this file. | ML lead |
| R2 | **Label leakage**: `support_tickets` / `last_login_days` recorded after the churn decision. | Med | High | Confirm feature timestamps; rebuild features as of snapshot date; drop any post-outcome field. | Perfect or near-perfect separation (already observed). | Data engineer |
| R3 | **Undefined target**: churn definition and window not documented. | High | High | Write and sign off the definition (cancel vs inactive, 30-day window) before labelling. | Different teams quote different churn rates. | Product owner |
| R4 | **Proxy bias** via plan type and spend (possible income proxy). | Med | Med | Metrics by plan/tenure; scores only add support; no price or service reduction based on score. | Flag-rate or recall gap between plans beyond agreed tolerance. | Responsible-AI reviewer |
| R5 | **Consent / privacy gap**: lawful basis for behavioural data unknown. | Med | High | Legal review under DPDP Act 2023; pseudonymise IDs; restrict access; honour deletion. | Missing or unclear consent record. | Privacy officer |
| R6 | **Misuse beyond retention** (pricing, denial, collections). | Low | High | Written prohibited-use list; access control; change review for new uses. | Requests to use scores in non-retention workflows. | Data owner |
| R7 | **False negatives**: churners not contacted, revenue lost. | Med | Med | Cost-based threshold; rule fallback; spot-check unflagged customers monthly. | Realised churn among unflagged customers rises. | Retention lead |
| R8 | **False positives**: unnecessary offers and customer annoyance. | Med | Low | Human review; outreach cap; per-customer contact frequency limit. | Offer cost per retained customer exceeds benefit. | Retention lead |
| R9 | **Distribution shift / drift** after launch (plans, pricing, seasonality). | Med | Med | Weekly drift and calibration monitoring; scheduled retraining on rolling window. | Drift alarm on key features; calibration error above limit. | ML lead |
| R10 | **Automation bias**: reviewers rubber-stamp scores. | Med | Med | Require override reason; audit override rate; show rule and score side by side. | Override rate near zero. | Retention lead |
| R11 | **Fallback failure**: no tested way back to rule baseline. | Low | High | Keep rule watchlist runnable; rehearse rollback; document rollback criteria (memo section 5). | Rollback not tested before launch. | ML lead |
| R12 | **Identifier leakage**: `customer_id` used as a feature or shared. | Low | Med | Exclude from features; hash or rotate; limit sharing. | ID appears in feature list or exports. | Data engineer |

## Rollback conditions (summary)
Revert to the rule watchlist if recall or calibration breaches agreed limits for two consecutive weeks, drift alarms fire on key features, a pipeline fault is found, or a fairness gap exceeds tolerance.
