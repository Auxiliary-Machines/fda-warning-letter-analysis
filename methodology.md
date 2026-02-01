# Methodology

This document describes the data collection, classification, and analysis methodology used in this study.

## Data Source

**Primary Source:** FDA Warning Letters Database
https://www.fda.gov/inspections-compliance-enforcement-and-criminal-investigations/compliance-actions-and-activities/warning-letters

**Access Date:** January 2025

**Date Range:** July 2022 – January 2025

Warning Letters are official FDA communications notifying regulated entities of significant violations discovered during inspections. They are public records.

## Scope

### Inclusion Criteria

Letters were included if they met ALL of the following:

1. **Issuing Office:** One of the following FDA centers or divisions:
   - Center for Drug Evaluation and Research (CDER)
   - Center for Devices and Radiological Health (CDRH)
   - Center for Biologics Evaluation and Research (CBER)
   - Division of Pharmaceutical Quality Operations (I, II, III, IV)
   - Office of Pharmaceutical Quality
   - Office of Manufacturing Quality
   - Division of Medical Device and Radiological Health Operations
   - Division of Biological Products Operations

2. **Subject Category:** Manufacturing quality violations, including:
   - CGMP/Finished Pharmaceuticals/Adulterated
   - CGMP/Active Pharmaceutical Ingredient (API)/Adulterated
   - CGMP/QSR/Medical Devices/Adulterated
   - CGMP/QSR/Medical Devices/PMA/Adulterated
   - CGMP Deviations
   - CGMP/Deviations/Biologics License Application (BLA)

### Exclusion Criteria

Letters were excluded if the primary subject was:
- Food, dietary supplements, or cosmetics
- Tobacco products
- Veterinary products
- Import/export violations only
- Clinical trial violations (Bioresearch Monitoring)
- Marketing, labeling, or advertising violations only
- Unapproved drug sales

**Final Dataset:** 417 Warning Letters

## RCA Classification

### Definition

A letter was classified as containing an RCA-related citation if FDA explicitly cited deficiencies in root cause investigation, including:

1. Failure to investigate quality failures
2. Inadequate or superficial investigations
3. Failure to identify the root cause
4. Corrective actions that did not address the root cause

### Search Patterns

The following regex patterns were used (case-insensitive):

```
root cause
underlying cause
fail(ed|ure)? to (adequately )?investigate
inadequate investigation
investigation did not
did not (determine|identify) the cause
corrective action(s)? (did not|failed to) address
capa.{0,50}root cause
```

A letter was flagged as RCA-related if ANY pattern matched.

### Category Definitions

Letters with RCA citations were further categorized:

| Category | Patterns | Definition |
|----------|----------|------------|
| No investigation conducted | `no investigation`, `without.{0,20}investigat`, `fail(ed\|ure)? to.{0,10}investigate`, `did not.{0,10}investigate` | FDA cited that no investigation was conducted |
| Investigation was inadequate | `inadequate investigation`, `investigation.{0,20}(inadequate\|insufficient\|incomplete\|limited)`, `fail(ed\|ure)? to adequately investigate` | Investigation occurred but FDA deemed it insufficient |
| Root cause not identified | `did not (determine\|identify).{0,20}(root\|underlying)?cause`, `(root\|underlying) cause.{0,20}not (identified\|determined)`, `without.{0,20}(root\|underlying) cause` | Investigation did not identify the true underlying cause |
| CAPA did not address root cause | `corrective action(s)?.{0,30}(did not\|failed to).{0,20}address`, `capa.{0,30}(did not\|failed to\|inadequate)` | Root cause was identified but corrective action didn't address it |

A single letter may match multiple categories.

## LLM-Assisted Extraction

For letters flagged as RCA-related, we used Gemma 3 (4B parameter model) via Ollama to extract structured information:

- **Product type:** Classification into categories (injectable drugs, oral solid dosage, medical devices, API, biologics, etc.)
- **Defect:** The specific quality problem that triggered the letter
- **Claimed cause:** The root cause the company proposed (if stated)
- **FDA critique:** The specific criticism FDA made about the investigation

### LLM Methodology Notes

- Model: `gemma3:4b` via Ollama (local inference)
- Context window: ~4000 characters centered on the first RCA pattern match
- Each extraction used a separate prompt with examples and constraints

LLM outputs were used for exploratory analysis only. The primary 44% finding is based entirely on deterministic regex pattern matching.

## Validation

### Manual Review

A random sample of 20 letters classified as RCA-related and 10 letters classified as not RCA-related were manually reviewed to validate pattern accuracy.


### Patterns Excluded

Early analysis included these patterns, which were removed due to high false-positive rates:

- `recur(rence|ring)` — matched boilerplate language about "preventing recurrence"
- `820.100` — matched any mention of CAPA regulation, not just criticism
- `211.192` — similar issue with regulation citations

## Limitations

1. **Warning Letters are not all FDA enforcement.** FDA issues many other actions (483 observations, consent decrees, recalls) not included here.

2. **Pattern matching limitations.** Regex may miss RCA criticism expressed in unusual language or match patterns in non-critical contexts.

3. **Category overlap.** A single letter may contain multiple types of RCA deficiencies. Categories are not mutually exclusive.

4. **LLM extraction variability.** Gemma 3 outputs may vary slightly between runs. LLM results are exploratory, not definitive.

5. **Temporal scope.** Analysis covers 2022-2025 only. Historical trends beyond this window are not assessed.

6. **Selection bias.** Warning Letters represent the most serious violations. Companies with minor issues receive 483 observations instead and are not captured here.

## Reproducibility

All code is available in this repository.

To reproduce without re-fetching letters:
- `data/fda_warning_letters.xlsx` — source index
- `data/letters/` — cached letter HTML
- `data/llm_results.jsonl` — LLM extraction outputs

## Contact

Questions: bella@runlattice.com

## Changelog

| Date | Change |
|------|--------|
| January 2025 | Initial analysis published |
