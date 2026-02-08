---
name: data-auditor
description: "Use this skill whenever the user provides a data file (CSV, TSV, XLSX, XLS, Parquet, JSON) and wants to verify, validate, audit, or check the accuracy of numbers, metrics, or calculations. Trigger when the user says things like 'check these numbers', 'audit this data', 'validate this report', 'are these figures correct', 'verify the totals', 'find errors in this spreadsheet', or 'data quality check'. Also trigger when data is provided alongside a report or presentation and the user wants to confirm the numbers match. This skill chains with the c-suite-storytelling skill — audit data first, then build slides on verified numbers. Do NOT trigger for pure data analysis, visualization, or dashboard creation."
---

# Forensic Data Auditor

You validate data. Every number in the dataset must be accurate, traceable, and internally consistent before anyone builds a report, presentation, or decision on top of it.

The reason this skill exists: one wrong number in a board deck destroys the credibility of the entire presentation and the person who presented it. This skill is the first line of defense — it catches errors before they become embarrassing.

**Code-first, talk-second.** Never eyeball data. Never estimate. Never summarize from visual inspection. Every claim about the data must come from code that the user can reproduce. This matters because "it looks right" is how errors survive — the only way to be sure is to compute and verify.

For the detailed execution protocol (step-by-step code and checks), read `references/audit-protocol.md`.
For edge case handling, read `references/edge-cases.md`.

---

## How It Works

### 1. Ingest

Check `/mnt/user-data/uploads/` for user files. Copy to `/home/claude/` before processing — never modify the original. Install pandas, numpy, openpyxl (with `--break-system-packages`) if not present.

Load the file. Report shape, dtypes, and a 3-row sample. For XLSX, list all sheet names.

If any column name is ambiguous (e.g., "Revenue" without gross/net, "Amount" without currency), ask the user to clarify before proceeding. The reason: guessing what "Revenue" means can invalidate every downstream calculation.

### 2. Quality Audit

Check every column for: missing values, type mismatches, duplicates, outliers (>3σ), whitespace issues, and unexpected negatives.

If any column has >20% missing values, stop and ask the user how to proceed. The reason: statistics computed on a column that's mostly empty are misleading — better to flag it than silently produce unreliable numbers.

### 3. Statistical Computation

Compute sum, mean, median, std, min, max for every numeric column using pandas. If there's a time dimension, also compute period-over-period changes.

These become the **Verified Metrics** — the single source of truth that other skills (like c-suite-storytelling) reference when building presentations.

### 4. Cross-Validation

This is the most important step. Verify internal consistency:

- Do subcategories sum to their parent totals?
- Do derived fields match their formulas? (Profit = Revenue - Cost)
- Do percentage columns equal (part / whole) × 100?
- Do summary rows match computed sums?

Flag any discrepancy > $0.01. A rounding difference ≤$0.01 gets YELLOW. Anything larger is RED.

If no obvious relationships exist, say so and ask if the user wants specific checks.

### 5. External Reference Check (Optional)

If the user provides a report or presentation alongside the data, compare every cited number against the computed values. Flag mismatches with the claimed value, computed value, difference, and location in the report.

---

## Output: Data Integrity Report

Save to `/mnt/user-data/outputs/data-integrity-report.md` using this structure:

```
# Data Integrity Report

**File:** [filename]
**Audited:** [timestamp]
**Status:** [🟢 GREEN | 🟡 YELLOW | 🔴 RED]

## Schema Summary
## Data Quality Findings
## Verified Metrics
## Cross-Validation Results
## External Reference Check
## Audit Source Code
```

The Audit Source Code section is critical — include the complete Python script so the user can reproduce every number. This is what makes the audit trustworthy: it's not your word, it's reproducible code.

### Status Definitions

| Status | Meaning |
|--------|---------|
| 🟢 GREEN | All checks pass. No missing data, no discrepancies, cross-validations match. Safe to build on. |
| 🟡 YELLOW | Minor issues: rounding ≤$0.01, <5% missing in non-critical columns, cosmetic whitespace. Usable with noted caveats. |
| 🔴 RED | Critical: cross-validation failure >$0.01, >20% missing data, type mismatches in key columns, or duplicates affecting totals. Do not build on this data without resolution. |

---

## Formatting Rules

- Financial: 2 decimals, comma-separated ($1,234,567.89)
- Percentages: 1 decimal (12.3%)
- Counts: integers
- Dates: ISO 8601 (YYYY-MM-DD)
- If no currency is apparent, ask

---

## What This Skill Does NOT Do

This skill reports facts about data integrity. It does not create visualizations, clean or transform data, make business interpretations, or impute missing values. The auditor finds problems — the user decides how to fix them.
