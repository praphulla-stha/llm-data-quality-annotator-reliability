# Using LLMs as Data Quality Annotators: A Reliability Study

## Abstract

Large language models (LLMs) are increasingly used to label, flag, and audit data quality issues in production pipelines, yet little is known about how reliable and consistent these judgments are compared to traditional rule-based methods. We evaluate LLM reliability on two common data quality tasks — entity/duplicate matching and brand-mislabeling detection — on e-commerce product data, comparing zero-shot and few-shot LLM prompting against rule-based baselines and human-labeled ground truth. On entity matching (Abt-Buy benchmark, 2,194 labeled pairs), a simple word-overlap rule-based baseline (F1=0.950) performed on par with LLM zero-shot prompting (F1=0.948); a targeted few-shot prompt improvement, validated on a small hand-picked sample (28/67 errors fixed, 1/50 broken), failed to generalize at full scale (F1=0.914), revealing a reliability risk in small-sample prompt validation. On brand-mislabeling detection (500 Amazon product listings with synthetically injected label errors), the LLM substantially outperformed a naive rule-based baseline (F1=0.833 vs. 0.721), driven by background knowledge of brand-product relationships the rule-based method could not access. Repeated-run consistency testing (200 pairs, 5 runs at temperature=0.7) showed high self-agreement (99.7% average agreement, 99% of pairs fully consistent), with majority-vote ensembling yielding only marginal accuracy gains (+0.005 F1) at 5x inference cost. Our results suggest that LLM reliability and relative advantage over traditional methods are task-dependent: LLMs offer limited benefit when strong lexical signals already exist, but substantial benefit when tasks require background knowledge, while remaining highly self-consistent across repeated queries.

---

## 1. Introduction

Research question: For entity/duplicate matching and brand-mislabeling detection on e-commerce product data, how does LLM (GPT-4o-mini) labeling agreement compare to human-labeled ground truth, across zero-shot prompting, few-shot prompting, and a rule-based normalization-matching baseline — and how consistent are the LLM's labels across repeated runs at different temperature settings?

---

## 2. Related Work

*(To draft — needs a literature pass: cite work on (a) entity resolution / record linkage methods, (b) LLMs for data annotation / labeling tasks, (c) LLM consistency/calibration studies. Not started yet — flag as a task for a future day.)*

---

## 3. Method

### 3.1 Datasets

- **Entity matching**: Abt-Buy benchmark — Abt.csv (1,081 products), Buy.csv (1,092 products), with a human-verified perfect mapping file. Built a labeled pairs set of 2,194 rows (positive pairs from the mapping file, negative pairs via random non-matching sampling).
- **Brand mislabeling**: Amazon.csv (from the Amazon-Google Products benchmark), 500-row sample with populated `manufacturer` field. Ground truth constructed synthetically: ~50% of rows had their manufacturer field swapped with a different, real manufacturer from the dataset; the rest left untouched.

### 3.2 Baselines

- **Entity matching**: Jaccard word-overlap similarity on normalized (lowercased, punctuation-stripped) product names, with a similarity threshold (best: 0.2).
- **Brand mislabeling**: naive substring check — manufacturer name presence in the product title.

### 3.3 LLM Setup

- Model: GPT-4o-mini (OpenAI API).
- Zero-shot prompting for both tasks (temperature=0 for main results).
- Few-shot prompting (entity matching only) with explicit guidance to prioritize shared model/SKU codes, plus 2 worked examples.
- Consistency testing: 5 repeated runs at temperature=0.7 on a 200-pair stratified subsample (entity matching task).

### 3.4 Evaluation

Precision, recall, F1, and accuracy against ground truth for all methods. For consistency testing: per-pair agreement rate across repeated runs, and majority-vote accuracy.

---

## 4. Results

### 4.1 Entity Matching (Abt-Buy, 2,194 pairs)

| Method | Precision | Recall | F1 | Accuracy |
|---|---|---|---|---|
| Rule-based (Jaccard, threshold=0.2) | 0.994 | 0.910 | 0.950 | 0.952 |
| LLM zero-shot | 0.999 | 0.902 | 0.948 | 0.951 |
| LLM few-shot + SKU guidance | 0.994 | 0.846 | 0.914 | 0.920 |

**Error analysis** (disagreement categories, zero-shot vs. rule-based):
- Both correct: 2,022
- Rule-based right, LLM wrong: 67
- LLM right, rule-based wrong: 64
- Both wrong: 41

LLM wins occurred mainly on pairs with low lexical overlap but clear semantic equivalence (e.g., abbreviated retail strings, reformatted SKU codes). Rule-based wins occurred mainly on pairs sharing an exact alphanumeric model/SKU code but differing descriptive text — cases where the LLM under-weighted a strong exact-match signal.

**Few-shot generalization failure**: A prompt revision explicitly instructing the model to prioritize SKU/code matches was validated on a small sample (67 previously-wrong cases + 50 control cases): 28/67 fixed, only 1/50 broken. At full scale (2,194 pairs), this same prompt *reduced* F1 from 0.948 to 0.914 (recall dropped from 0.902 to 0.846) — the intervention over-generalized, causing the model to become more code-dependent across the broader dataset than the small validation sample suggested.

### 4.2 Brand Mislabeling (Amazon, 500 listings)

| Method | Precision | Recall | F1 | Accuracy |
|---|---|---|---|---|
| Rule-based (manufacturer-in-title) | 0.564 | 1.000 | 0.721 | 0.622 |
| LLM zero-shot | 0.827 | 0.840 | 0.833 | 0.836 |

**Error analysis** (disagreement categories):
- Both correct: 272
- LLM right, rule-based wrong: 146
- Both wrong: 43
- Rule-based right, LLM wrong: 39

The rule-based method's high recall but low precision stems from flagging any product whose manufacturer name doesn't literally appear in the title — even when correctly labeled (e.g., "ZoneAlarm Anti-Spyware" manufactured by "Zone Labs," or "SYMC Backup Exec" manufactured by "Symantec," where SYMC is a stock ticker abbreviation). The LLM correctly resolves these via background knowledge of brand relationships. Conversely, the LLM's errors mostly involved accepting unfamiliar or niche publisher names as plausible without verification, missing genuine mislabels that the stricter rule-based check caught.

### 4.3 Consistency / Reliability (200-pair sample, 5 runs, temperature=0.7)

- Average per-pair agreement rate: 0.997
- Fraction of pairs with identical answers across all 5 runs: 0.990
- Majority-vote (5 runs) F1: 0.974 vs. single-run temperature=0 F1: 0.969 (on the same 200-pair sample)

The LLM showed high self-consistency even under non-zero temperature. Majority-vote ensembling across 5 runs yielded a small F1 improvement (+0.005) at 5x the inference cost, suggesting limited practical benefit from ensembling for this task given the already-high single-run consistency.

---

## 5. Discussion / Limitations

*(To draft — key points to cover, all supported by your own results above:)*
- LLM relative advantage over rule-based methods is **task-dependent**: negligible for entity matching (near-tie), substantial for brand mislabeling (LLM wins by a wide margin) — likely because the latter benefits more from background/world knowledge that a lexical rule cannot access.
- Small-sample prompt validation can be misleading — the Day 7/8 few-shot finding is a concrete cautionary example worth its own subsection.
- Abt-Buy is a relatively "easy" entity-matching benchmark (structurally similar names across catalogs); results may not generalize to noisier real-world catalogs.
- Brand-mislabeling ground truth was synthetically constructed (random manufacturer swaps), not human-annotated — swaps were mostly "obvious" mismatches; a harder version (same-category swaps) is a natural extension.
- Single LLM (GPT-4o-mini) tested; findings may not generalize to larger or different model families.
- Consistency testing was limited to entity matching and a single temperature setting (0.7); brand mislabeling consistency untested.

---

## 6. Conclusion

*(To draft — 1 paragraph restating the core finding: task-dependent LLM value, high self-consistency, and the small-sample-validation caution, plus a sentence on future work.)*

---

## References

*(To fill in during Related Work pass)*
