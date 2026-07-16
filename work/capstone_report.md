# Capstone Report — Lane 2: Content Opportunity Scoring

- **Author:** Abdul Raffay Qureshi
- **Lane:** Lane 2 — Refresh / Content Optimization
- **Repo:** FlyRank-ML-Intern
- **Date:** July 2026

## 0. Abstract

This project presents an algorithmic decision-support framework designed to help editorial teams systematically prioritize decaying web content for optimization. Traditional standalone keyword volume metrics are highly deceptive indicators of content health, showing a remarkably weak correlation coefficient of 0.001 with actual page impressions. Furthermore, search traffic decay behaves non-linearly, making rigid, rule-based scheduling highly inefficient.

To resolve this, we formulate a multi-dimensional Content Opportunity Score using historical features (`impressions_last_30d`, `clicks_last_30d`, `avg_position`, and `word_count`). Strict data safety, privacy, and leakage guardrails are programmatically enforced under the ML-04 Data Contract by isolating the target proxy from the feature space and completely excluding trend-derived metrics. This decision-support system outputs a list-wise ranking queue, allowing content editors to allocate copywriting resources efficiently, supporting organic traffic recovery while minimizing manual guesswork and budgetary waste.

## 1. Problem framing

This project delivers an algorithmic decision-support framework to help FlyRank editorial teams systematically prioritize decaying or stale web content for manual updates.

- **Unit of Analysis:** One unique web page URL tracked within a specific domain context over a fixed historical aggregation window.
- **Model Output:** A continuous, multi-dimensional Opportunity Score used to generate a structured, list-wise ranking queue.
- **Downstream Human Action:** Content editors review this prioritized dashboard queue to assign copywriters to update specific high-opportunity URLs, maximizing organic traffic recovery while minimizing manual guesswork.
- **Cost of a Wrong Call:**
  - _False Positive (Type I Error):_ The system flags a completely dead-end or unrecoverable page for a rewrite. **Cost:** Wasted copywriting budgets, lost editorial resource hours, and missed opportunity costs.
  - _False Negative (Type II Error):_ The model overlooks a decaying page that possessed massive traffic recovery potential. **Cost:** Permanently losing organic ranking visibility to market competitors and losing downstream customer conversions.
- **Why ML/Data is Required:** Analysis of our initial dataset across **30,000 baseline tracks** revealed that the correlation coefficient between keyword search volume and true page impressions is remarkably weak at **0.001**. This mathematical divergence proves that traditional standalone keyword metrics are highly deceptive indicators of actual content health.

Furthermore, search traffic decay behaves non-linearly; dropping ranking positions at the top of page one (e.g., position 1 to 4) triggers exponential click-through rate (CTR) degradation compared to lower-tier fluctuations. Because rigid static rules (such as `days_since_refresh > 365`) cannot dynamically balance these multi-variable trade-offs, a machine learning scoring approach is required.

## 2. Data safety

To guarantee pipeline runtime safety, maintain strict client privacy, and eliminate data leakage risk, our schema metrics are isolated into distinct engineering buckets:

- **Ingested Feature Space:** The model is restricted to safe historical indicators including `impressions_last_30d`, `clicks_last_30d`, `avg_position`, `days_since_last_update`, and `word_count`. These continuous and discrete values provide historical context without leaking future performance signals.
- **Target Isolation:** The target proxy ($Clicks_{prev} - Clicks_{last}$) is calculated and stored independently. It is utilized purely during the evaluation and training loss calculation steps and is strictly isolated from the feature matrix fed to the model.
- **Deliberately Excluded Fields:**
  - `trend_direction` and `trend_pct` — These fields were explicitly excluded because they are directly derived from the target itself. Ingesting them would cause immediate, catastrophic data leakage—allowing the model to trivially calculate future outcomes during training, resulting in perfect validation scores that completely fail when deployed on new data.
  - `client_id` and `content_id` — These pseudonymous identifiers are strictly restricted to downstream database joins, listwise sorting, and human display layout configurations. They are completely barred from serving as modeling features to prevent the algorithm from memorizing client-specific baseline shifts.
- **Privacy and Anonymization Confirmation:** No raw client domain URLs, private text queries, or corporate identity names are present within the `work/` folder or the underlying `content_refresh_anonymized.csv` file. All tracking observations are entirely anonymized, ensuring total data compliance.

## 3. Baseline

To establish a clear performance floor for our machine learning models, we developed a transparent, rule-based heuristic baseline. This baseline was designed and validated using the exact data contract and feature schemas finalized during our Week 3 pipeline audit (`w03_data_contract.ipynb` and `w03_feature_leakage_check.ipynb`).

### The Heuristic Rule: Exposure × Staleness Score

Our baseline assumes that a page represents a high-refresh opportunity if it has historically generated high organic visibility (exposure) but has not been updated recently (staleness). 

Using only our safe, non-leaking features, we define the **Baseline Opportunity Score** mathematically as:

$$Baseline\_Score = \log(1 + feat\_impressions\_90d) \times \left(1 + \frac{feat\_days\_since\_last\_update}{365}\right)$$

To evaluate this as a binary classifier (predicting whether a page is decaying, where $\text{is\_declining\_label} = 1$), we apply a simple threshold:
*   **Predict Decaying (1):** If the page's historical impressions are above the median (top 50%) **AND** it has not been updated in over 180 days (`feat_days_since_last_update > 180`).
*   **Predict Stable (0):** Otherwise.

### Why This is a Fair & Safe Comparison
1. **Zero Data Leakage:** In accordance with our Week 3 feature audit, this heuristic relies *solely* on features knowable at the decision moment. It completely avoids post-decision signals like `trend_pct` or `trend_direction`.
2. **Operational Simplicity:** This represents the exact sort-and-filter approach an SEO team would manually implement in Google Sheets or Excel before deploying an ML pipeline.

### The Leakage Hunt Experiment
During our pipeline validation, we intentionally ran a target-leakage attack by injecting the post-decision variable `trend_pct` into our model. 
*   **With Leakage (Cheating):** Precision jumped to an artificial **99.8%**, as the model trivially reverse-engineered the target.
*   **Without Leakage (Honest Baseline):** Removing the leaked column dropped performance back to reality, establishing our true, honest baseline.

### Baseline Performance Metrics
We evaluated this baseline on our mid-panel training month (March 2026):

| Metric | Baseline Performance | Operational Meaning for Lane 2 |
| :--- | :--- | :--- |
| **Precision** | **52.41%** | Out of all pages flagged as "high opportunity," roughly 52% actually showed a true traffic decline. |
| **Recall** | **48.15%** | The rule caught roughly 48% of the truly decaying pages in our slice. |
| **F1-Score** | **50.19%** | Our balanced harmonic mean. Any future ML model must comfortably beat this 50.19% baseline. |

## 4. Model / analysis

Your method and why it fits the lane. The exact feature list (and what you left out on
purpose). The target or proxy definition, in one sentence.

## 5. Evaluation

Your split (grouped by client? time-aware?) and why. Metrics, model vs baseline **on the same
split**. What the errors look like — a short error analysis beats a big metric table.

## 6. Interpretation

What the model/clusters actually found. Feature importances or cluster profiles in plain
words. Surprises and negative results — a well-understood "no effect" is a valid result.

## 7. Recommendation

The ranked actions or decisions your output supports, and how a FlyRank editor would use them
tomorrow. State your confidence and the limits explicitly.

## 8. Reproducibility

The exact commands to re-run everything from a fresh clone, your random seeds, and your
environment (`pip freeze` highlights or `requirements.txt` deltas).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
