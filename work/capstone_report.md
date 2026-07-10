# Capstone Report — Lane 2: Content Opportunity Scoring

- **Author:** Abdul Raffay Qureshi
- **Lane:** Lane 2 — Refresh / Content Optimization
- **Repo:** FlyRank-ML-Intern
- **Date:** July 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. The eight
> sections mirror the Pass / Needs-Work rubric axes, so nothing here is optional.

## 1. Problem framing

What decision does this support? Name the unit of analysis (page, client, day…), the output
(score, rank, cluster, report), the action a human takes from it, and the cost of a wrong
call. Why does data/ML help here at all?

This project delivers an algorithmic decision-support framework to help FlyRank editorial teams systematically prioritize decaying or stale web content for manual updates.

- **Unit of Analysis:** One unique web page URL tracked within a specific domain context over a fixed historical aggregation window.
- **Model Output:** A continuous, multi-dimensional Opportunity Score used to generate a structured, listwise ranking queue.
- **Downstream Human Action:** Content editors review this prioritized dashboard queue to assign copywriters to update specific high-opportunity URLs, maximizing organic traffic recovery while minimizing manual guesswork.
- **Cost of a Wrong Call:**
  - _False Positive (Type I Error):_ The system flags a completely dead-end or unrecoverable page for a rewrite. **Cost:** Wasted copywriting budgets, lost editorial resource hours, and missed opportunity costs.
  - _False Negative (Type II Error):_ The model overlooks a decaying page that possessed massive traffic recovery potential. **Cost:** Permanently losing organic ranking visibility to market competitors and losing downstream customer conversions.
- **Why ML/Data is Required:** Analysis of our initial dataset across **30,000 baseline tracks** revealed that the correlation coefficient between keyword search volume and true page impressions is remarkably weak at **0.001**. This mathematical divergence proves that traditional standalone keyword metrics are highly deceptive indicators of actual content health.

Furthermore, search traffic decay behaves non-linearly; dropping ranking positions at the top of page one (e.g., position 1 to 4) triggers exponential click-through rate (CTR) degradation compared to lower-tier fluctuations. Because rigid static rules (such as `days_since_refresh > 365`) cannot dynamically balance these multi-variable trade-offs, a machine learning scoring approach is required.

## 2. Data safety

Which data you used and which columns you deliberately excluded (and why). Leakage risks you
considered — especially label-derived fields (`trend_direction`, `trend_pct`) and pseudonymous
IDs (grouping only, never features). Confirm nothing client-identifying appears anywhere in
`work/`.

## 3. Baseline

The transparent rule or score you built first. Why it's a fair comparison, and its numbers on
the same data and metric as your model.

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
