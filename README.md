# 2026 SIOP Machine Learning Competition — 2nd Place Solution

## Overview

Second-place solution for the [2026 SIOP Machine Learning Competition](https://computationaloutreach.com/siopmlcompetition2026/overview.html). The task involved automating the extraction and aggregation of effect sizes from empirical research papers for meta-analytic coding which is a process traditionally done by hand by researchers.

The pipeline reads PDFs of academic articles, identifies relevant statistical relationships between psychological constructs, and produces an aggregate Pearson's *r* for each study.

## Approach

The solution uses a **two-agent architecture** powered by Google's Gemini API:

### Agent 1 — Structured Extraction
- Ingests each study PDF along with construct definitions and a detailed system prompt specifying extraction rules.
- Returns structured JSON (validated with Pydantic) containing every relevant statistic: correlations, regression betas, group comparisons, etc.
- Handles reverse coding flags, sub-study structure, and statistic metadata (sample sizes, reliability, CIs).

### Agent 2 — Aggregation
- Receives the extracted JSON for each study and computes an aggregate Pearson's *r* using Fisher's *z* transformation.
- Prioritizes zero-order correlations over regression coefficients to avoid conflating bivariate and partial associations.
- Uses Gemini's code execution tool for the math rather than relying on the model to compute it in-context.

### Post-Processing
- A final pass uses Gemini's Deep Research agent to review all extracted data holistically across studies, applying the same aggregation logic with access to all construct definitions.
- Missing values are imputed using the sample-size-weighted mean *r* from studies sharing the same construct pair.

## Key Design Decisions

- **Structured output with Pydantic schemas** to enforce consistent extraction across 66 studies.
- **Detailed system prompts** encoding meta-analytic coding rules (statistic priority, reverse coding, temporal matching, deduplication).
- **Separation of extraction and aggregation** into two LLM calls to reduce error propagation.

### Inputs (not included)
- `test_articles.csv` — study metadata and construct pair assignments
- `test_construct_definitions.csv` — detailed inclusion criteria for each construct
- `Test Data/` — PDF articles (study1.pdf … study66.pdf)

### Outputs
- `results.csv` — per-study aggregate effect sizes
- `all_papers_raw_test_agent_2.json` — full structured extraction
- `debug_responses.json` — raw model responses for inspection
