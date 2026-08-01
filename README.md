# Using LLMs as Data Quality Annotators: A Reliability Study

A study evaluating how reliable and consistent large language models (LLMs) are when used to
label or flag data quality issues in e-commerce product data — specifically entity/duplicate
matching and brand-mislabeling detection — compared against rule-based baselines and
human-labeled ground truth.

## Summary of Findings

- **Entity matching** (Abt-Buy benchmark, 2,194 labeled pairs): a rule-based Jaccard
  word-overlap baseline (F1=0.950) performed on par with LLM zero-shot prompting (F1=0.948).
  A targeted few-shot prompt improvement, validated on a small sample, failed to generalize
  at full scale (F1=0.914) — see Discussion in the report for why.
- **Brand mislabeling** (500 Amazon listings, synthetic label corruption): the LLM
  substantially outperformed a naive rule-based baseline (F1=0.833 vs. 0.721), likely due to
  background knowledge of brand-product relationships unavailable to a simple string-matching rule.
- **Consistency**: across 5 repeated runs at temperature=0.7 on a 200-pair sample, the LLM
  showed high self-agreement (99.7% average agreement, 99% of pairs fully consistent across
  all 5 runs). Majority-vote ensembling gave only a marginal accuracy gain (+0.005 F1) at 5x
  the inference cost.

Full write-up: [`report/report.md`](report/report.md)

## Repo Structure

```
.
├── data/
│   ├── raw/                                    # Original Abt-Buy and Amazon-Google Products benchmark files
│   │   ├── Abt-Buy/
│   │   └── Amazon-GoogleProducts/
│   └── processed/                              # Constructed pairs, predictions, and comparison tables
├── notebooks/
│   ├── scoping_and_pairs.ipynb                 # Research scope, Abt-Buy pairs construction
│   ├── rulebased_baseline.ipynb                # Jaccard similarity baseline (entity matching)
│   ├── llm_zeroshot_sample_test.ipynb          # LLM API setup, small-sample validation test
│   ├── llm_zeroshot_fullrun.ipynb              # Full-scale zero-shot entity matching (2,194 pairs)
│   ├── entity_matching_error_analysis.ipynb    # Disagreement analysis: LLM vs. rule-based
│   ├── fewshot_sku_prompt_test.ipynb           # Few-shot/SKU-guidance prompt, small-sample validation
│   ├── llm_fewshot_fullrun.ipynb               # Full-scale few-shot run (2,194 pairs)
│   ├── consistency_analysis_temp07.ipynb       # Repeated-run consistency testing (temperature=0.7)
│   ├── brand_mislabel_dataset_generation.ipynb # Synthetic brand-mislabel dataset construction
│   ├── brand_mislabel_baseline_and_llm.ipynb   # Rule-based baseline + LLM zero-shot (brand mislabeling)
│   └── brand_mislabel_error_analysis.ipynb     # Disagreement analysis: brand mislabeling task
├── report.md                                   # Full written report (abstract, method, results, discussion)
├── requirements.txt
└── README.md
```

## Reproducing the Results

1. Clone the repo and install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Set your OpenAI API key as an environment variable:
   ```bash
   export OPENAI_API_KEY="your-key-here"   # macOS/Linux
   $env:OPENAI_API_KEY = "your-key-here"   # Windows PowerShell
   ```
3. Run the notebooks in `notebooks/` in numerical order. Raw data is included under
   `data/raw/`; running the notebooks will regenerate everything under `data/processed/`.

## Results Tables

### Entity Matching (Abt-Buy, 2,194 pairs)

| Method | Precision | Recall | F1 | Accuracy |
|---|---|---|---|---|
| Rule-based (Jaccard, threshold=0.2) | 0.994 | 0.910 | 0.950 | 0.952 |
| LLM zero-shot | 0.999 | 0.902 | 0.948 | 0.951 |
| LLM few-shot + SKU guidance | 0.994 | 0.846 | 0.914 | 0.920 |

### Brand Mislabeling (Amazon, 500 listings)

| Method | Precision | Recall | F1 | Accuracy |
|---|---|---|---|---|
| Rule-based (manufacturer-in-title) | 0.564 | 1.000 | 0.721 | 0.622 |
| LLM zero-shot | 0.827 | 0.840 | 0.833 | 0.836 |

See [`report/report.md`](report/report.md) for full method details, error analysis, and discussion.

## Author

Praphulla — Data Quality Analyst, [Grepsr](https://grepsr.com)
