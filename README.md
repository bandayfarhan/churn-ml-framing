# Customer Churn: ML Problem Framing

Decision: which active customers get a human-reviewed retention outreach (30-day window).

**Finding:** on a 12-row sample, the rule `support_tickets >= 3` catches all 5 churners with 0 false positives, and a leave-one-out logistic regression adds nothing. ML is not justified yet.

## Contents
- [ml_problem_framing_memo.md](ml_problem_framing_memo.md)
- [responsible_data_card.md](responsible_data_card.md)
- [baseline_notebook.ipynb](baseline_notebook.ipynb)
- [risk_register.md](risk_register.md)

Raw sample data is withheld. The notebook expects a `churn.csv` in the same folder.
