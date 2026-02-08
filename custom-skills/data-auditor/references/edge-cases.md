# Edge Cases

These are the real-world scenarios that break naive data processing. Handle each one as described — the goal is to never silently produce wrong results.

| Scenario | What to do | Why |
|----------|-----------|-----|
| **Empty file (0 rows)** | Report RED. "File contains no data rows." Stop. | Nothing to audit. |
| **Headers only, no data** | Report RED. "File contains headers but no data." Stop. | Same — no data means no audit. |
| **>500,000 rows** | Process in chunks of 100k. Note in report: "Large file — processed in chunks." Verify full-file statistics match chunked computation. | Pandas can handle this in memory on most systems, but chunking prevents timeout issues and makes progress visible. |
| **Duplicate column names** | Rename with suffix (_1, _2). Flag in report. Ask user which is canonical. | Pandas silently allows duplicate column names but operations on them produce unpredictable results. |
| **Mixed encoding (UTF-8/Latin-1)** | Try UTF-8 first → Latin-1 → CP1252. Report which encoding was used. | Silently mangling characters (especially in names, currencies, or special symbols) corrupts string-based analyses. |
| **Multiple sheets (XLSX)** | Audit first sheet by default. List all sheets and ask if others need auditing. | Users often don't realize which sheet contains the data they care about. |
| **Merged cells (XLSX)** | Unmerge and forward-fill. Flag as a quality issue in the report. | Merged cells create NaN values in pandas that aren't truly "missing" — they're formatting artifacts. Forward-filling recovers the intended data. |
| **Dates stored as strings** | Parse with `pd.to_datetime(errors='coerce')`. Report how many failed to parse. | String dates prevent time-series analysis and period-over-period calculations. Coercing means you see exactly how many entries have non-standard date formats. |
| **Numeric columns stored as strings** | Convert with `pd.to_numeric(errors='coerce')`. Report conversion failures. | "$1,234" or "1 234" stored as strings won't participate in sum/mean calculations. Coercing reveals which values have formatting issues. |
| **Password-protected files** | Stop. Report the error. Ask user for an unprotected version. | Cannot be read programmatically without the password. |
| **Corrupt or unreadable files** | Stop. Report the exact error message from pandas. Ask how to proceed. | Better to fail loudly than produce partial or garbled results. |
| **File with only 1 row** | Audit normally but note: "Single-row dataset — statistical measures (mean, std, outliers) are not meaningful." | Standard deviation of 1 value is undefined. The audit still runs, but the user should know the stats are trivial. |
| **Currency mixing** | If multiple currencies appear in the same column, flag as RED. Do not convert or aggregate. | Summing USD and EUR produces a meaningless number. |
