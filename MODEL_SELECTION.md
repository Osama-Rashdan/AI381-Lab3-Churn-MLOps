# Model selection

## The question

The model predicts whether a telecommunications customer is likely to leave the company (churn). A false negative means the company misses a customer who may leave, while a false positive may lead to unnecessary retention offers and costs.

## Candidates

| Candidate | Why I tried it | AUC | F1 | F1 per slice (lowest) | p95 latency (ms) | Size (KB) | MLflow run ID |
|---|---|---:|---:|---:|---:|---:|---|
| Logistic Regression | Simple, fast baseline | 0.7982 | 0.4948 | 0.2500 (one-year and two-year) | 2.4 | 3.55 | `3e2b8ab111724229bbee5c361feda9c9` |
| Random Forest | To test a tree-based ensemble | 0.7589 | 0.4536 | 0.2222 (one-year) | 9.9 | 3160.13 | `bccd5ef1e4054c498632eaf43581fc9f` |
| Gradient Boosting | To test another boosting-based approach | 0.7476 | 0.4167 | 0.2222 (two-year) | 2.8 | 141.68 | `c3fe4f39e95a4b8ca42b4e71f72e5ecc` |

## The champion

I selected Logistic Regression as the champion. It achieved the highest overall F1 (0.4948) and AUC (0.7982), while also having the smallest model size and low p95 latency (2.4 ms).

I accepted weaker performance on some contract slices: its F1 was 0.5432 for month-to-month customers and 0.2500 for both one-year and two-year customers. Random Forest performed better on the two-year slice, while Gradient Boosting performed better on the one-year slice. However, neither achieved better overall F1.

## My promotion rules

- Latency budget (p95): 25 ms, because the prediction API should respond quickly. All three candidates fit within this budget.
- Minimum gain to replace the champion (F1 points): 0.005, because a new model should provide at least a 0.5 percentage-point absolute F1 improvement before replacing the existing champion.

The promotion script selects the best candidate within the latency budget and blocks promotion if its F1 does not exceed the current champion by the required minimum gain.

## What I would check next

I would investigate the one-year and two-year contract slices further, including their sample sizes and false-negative rates. Their low F1 scores may indicate that the model does not identify churn effectively for these customers. I would also monitor changes in customer behavior over time to detect data drift.