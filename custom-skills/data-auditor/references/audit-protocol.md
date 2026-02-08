# Audit Protocol: Step-by-Step Execution

This reference contains the detailed code and logic for each audit step. Follow these in order — do not skip steps, and do not proceed past a gate if the gate condition is met.

---

## Environment Setup

Run this before any audit work:

```python
import subprocess, sys

required = ["pandas", "numpy", "openpyxl"]
for pkg in required:
    try:
        __import__(pkg)
    except ImportError:
        subprocess.check_call([sys.executable, "-m", "pip", "install", pkg, "--break-system-packages", "-q"])
```

If the file isn't Excel, you can skip openpyxl — but pandas and numpy are always required.

---

## Step 1: Ingest & Schema Check

```python
import pandas as pd

# Adjust reader based on file type
df = pd.read_csv(filepath)       # CSV/TSV
# df = pd.read_excel(filepath)   # XLSX/XLS
# df = pd.read_parquet(filepath) # Parquet
# df = pd.read_json(filepath)    # JSON

print(f"Shape: {df.shape}")
print(f"\nDtypes:\n{df.dtypes}")
print(f"\nSample:\n{df.head(3)}")
```

For XLSX files, also list sheets:
```python
xls = pd.ExcelFile(filepath)
print(f"Sheets: {xls.sheet_names}")
```

**Ambiguity Gate:** If any column name could mean multiple things (Revenue → gross or net? Amount → what currency? Date → what timezone?), list the ambiguous columns and ask the user. The reason you stop here: if you guess wrong about what "Revenue" means and compute everything against the wrong interpretation, the entire audit is invalid. It's cheaper to ask one question now than redo the whole thing later.

---

## Step 2: Data Quality Audit

For every column, compute:

```python
import numpy as np

print("=== DATA QUALITY ===")
for col in df.columns:
    missing = df[col].isna().sum()
    pct = missing / len(df) * 100
    print(f"{col}: {missing} missing ({pct:.1f}%)")

# Duplicates
dupes = df.duplicated().sum()
print(f"\nDuplicate rows: {dupes}")
if dupes > 0:
    print(df[df.duplicated(keep=False)])

# Outliers (numeric columns)
for col in df.select_dtypes(include=["number"]).columns:
    mean = df[col].mean()
    std = df[col].std()
    outliers = df[(df[col] - mean).abs() > 3 * std]
    if len(outliers) > 0:
        print(f"\n{col}: {len(outliers)} outliers beyond 3σ")
        print(outliers[[col]])

# Whitespace in string columns
for col in df.select_dtypes(include=["object"]).columns:
    ws = df[col].str.strip() != df[col]
    ws_count = ws.sum() if ws.any() else 0
    if ws_count > 0:
        print(f"\n{col}: {ws_count} values with leading/trailing whitespace")

# Unexpected negatives
suspect_cols = [c for c in df.select_dtypes(include=["number"]).columns
                if any(word in c.lower() for word in ["revenue", "quantity", "count", "amount", "price", "cost"])]
for col in suspect_cols:
    neg = (df[col] < 0).sum()
    if neg > 0:
        print(f"\n{col}: {neg} unexpected negative values")
```

**Quality Gate:** If any column exceeds 20% missing, stop the audit and report. Ask the user:

> "Column [X] has [Y]% missing values. Should I continue excluding this column, audit with available data, or stop here?"

The reason for this gate: computing statistics on mostly-empty columns produces numbers that look precise but are unreliable. It's better to surface this to the user than silently produce misleading results.

---

## Step 3: Statistical Computation

```python
print("=== VERIFIED METRICS ===")
for col in df.select_dtypes(include=["number"]).columns:
    print(f"\n--- {col} ---")
    print(f"  Sum:    {df[col].sum():,.2f}")
    print(f"  Mean:   {df[col].mean():,.2f}")
    print(f"  Median: {df[col].median():,.2f}")
    print(f"  Std:    {df[col].std():,.2f}")
    print(f"  Min:    {df[col].min():,.2f}")
    print(f"  Max:    {df[col].max():,.2f}")
    print(f"  Count:  {df[col].count()}")
```

If the data has a time dimension (year, month, quarter, date column), also compute:

```python
# Sort by time column first
df = df.sort_values("date_column")

for col in numeric_cols:
    df[f"{col}_change"] = df[col].diff()
    df[f"{col}_pct_change"] = df[col].pct_change() * 100
    print(f"\n{col} period-over-period:")
    print(df[["date_column", col, f"{col}_change", f"{col}_pct_change"]])
```

These computed values become the **Verified Metrics table** — the single source of truth. Any downstream skill (like c-suite-storytelling) that references a number from this data must trace it back to this table.

---

## Step 4: Cross-Validation

This is the step that catches real errors. Verify internal consistency with these checks:

### Additive checks
Do subcategory values sum to their parent total?

```python
# Example: regions summing to total
computed_total = df[["North", "South", "East", "West"]].sum(axis=1)
discrepancies = df[abs(df["Total"] - computed_total) > 0.01]
if len(discrepancies) > 0:
    print(f"FAIL: {len(discrepancies)} rows where Total != sum of regions")
    print(discrepancies)
else:
    print("PASS: All totals match sum of components")
```

### Derived field checks
Do calculated fields match their formula?

```python
# Example: Margin = EBITDA / Revenue * 100
computed_margin = (df["EBITDA"] / df["Revenue"] * 100).round(1)
for _, row in df.iterrows():
    diff = abs(row["EBITDA_Margin"] - computed_margin[row.name])
    status = "PASS" if diff < 0.1 else "FAIL"
    print(f"{row['Region']}: Stated {row['EBITDA_Margin']}% vs Computed {computed_margin[row.name]}% → {status}")
```

### Percentage checks
Do percentage columns equal (part / whole) × 100?

```python
# Example: revenue share
total_rev = df["Revenue"].sum()
for _, row in df.iterrows():
    computed_share = row["Revenue"] / total_rev * 100
    print(f"{row['Region']}: {computed_share:.1f}%")
```

### Row/column total checks
If summary rows or columns exist, verify them:

```python
# Check if last row is a "Total" row
if "total" in str(df.iloc[-1, 0]).lower():
    for col in df.select_dtypes(include=["number"]).columns:
        computed = df.iloc[:-1][col].sum()
        stated = df.iloc[-1][col]
        diff = abs(stated - computed)
        status = "PASS" if diff <= 0.01 else "FAIL"
        print(f"{col}: Stated total {stated:,.2f} vs Computed {computed:,.2f} → {status}")
```

**Tolerance:** $0.01 (or 0.01 in native units). Rounding differences at exactly $0.01 get YELLOW. Anything larger gets RED.

If no obvious parent-child or derived relationships exist in the data, say so explicitly: "No cross-validation relationships detected. If you have expected relationships (e.g., column A should equal column B + C), tell me and I'll verify them."

---

## Step 5: External Reference Check

Only applies when the user provides a report, presentation, or dashboard alongside the data.

For every number cited in the external document:
1. Identify which metric it claims to represent
2. Look up the computed value from Step 3
3. Compare

```python
# Example comparison log
claims = [
    {"source": "Slide 3", "metric": "North Revenue", "claimed": 5000000, "computed": df[df["Region"]=="North"]["Revenue"].values[0]},
    {"source": "Slide 3", "metric": "North EBITDA Margin", "claimed": 24.0, "computed": (df[df["Region"]=="North"]["EBITDA"].values[0] / df[df["Region"]=="North"]["Revenue"].values[0] * 100)},
]

for c in claims:
    diff = abs(c["claimed"] - c["computed"])
    status = "MATCH" if diff <= 0.01 else f"MISMATCH (diff: {diff:,.2f})"
    print(f"{c['source']} | {c['metric']}: Claimed {c['claimed']:,.2f} vs Computed {c['computed']:,.2f} → {status}")
```

Report every mismatch with: the claimed value, the computed value, the difference, and where in the document the claim appears.
