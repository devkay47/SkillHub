---
name: csv-data-analyst
description: Analyses CSV files with automatic type detection, missing value handling, outlier identification, and statistical summaries. Use when the user asks to analyse a CSV, explore a dataset, summarise data, find patterns in a spreadsheet, or asks "what does this data show."
---

## Overview

This skill approaches CSV analysis defensively — it assumes the data is dirty until proven otherwise, validates types before aggregating, and surfaces data quality issues alongside the analytical findings. This prevents the common failure mode of producing confidently wrong statistics from malformed data.

## Instructions

When this skill activates, follow this analysis pipeline in order.

**Phase 1 — Load and inspect.** Load the CSV using pandas with `dtype=str` initially to prevent automatic type coercion from silently converting malformed values. Print the shape (rows × columns), the first five rows, and the column names.

**Phase 2 — Data quality assessment.** For every column, compute and report: the count of missing values and their percentage of total rows, the count of duplicate rows in the entire dataset, whether any column that should be numeric contains non-numeric strings, and whether any date column contains multiple inconsistent formats.

Do not proceed to statistical analysis until the data quality report is complete. Present it to the user and ask if they want to proceed with the identified issues present, or if they want to apply cleaning steps first.

**Phase 3 — Type conversion.** Once the user approves, convert columns to their correct types. For numeric columns, coerce errors to NaN rather than raising exceptions (`pd.to_numeric(col, errors='coerce')`). For date columns, parse with `pd.to_datetime(col, infer_datetime_format=True, errors='coerce')`. Report how many values were lost to coercion failures in each column.

**Phase 4 — Statistical summary.** For numeric columns: mean, median, standard deviation, min, max, and the 25th and 75th percentiles. For categorical columns: value counts for the top 10 most frequent values and the count of unique values. For date columns: the date range and any obvious gaps.

**Phase 5 — Outlier detection.** For each numeric column, identify outliers using the IQR method (values below Q1 - 1.5×IQR or above Q3 + 1.5×IQR). Report the count and sample values of outliers. Do not remove them automatically — report them and let the user decide.

**Phase 6 — Key findings.** Summarise in plain language the three to five most interesting or actionable observations from the data. These should be specific observations about the actual values, not generic statements like "the data has some missing values."

## Example Key Findings Output
