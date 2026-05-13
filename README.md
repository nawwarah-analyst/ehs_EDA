# EHS Source Data — SQL Exploratory Data Analysis (EDA)

> ⚠️ **Important note:** This EDA was done *after* the EHS Safety Intelligence System dashboard was already built — not before, as it should have been. I only understood the full importance of EDA once the project was complete. This repo is my attempt to do it properly, document what I found, and be honest about the impact on the dashboard's accuracy. It's a learning milestone, not a cover-up.

---

## What this repo is

A structured SQL EDA across the three source tables used in my [EHS Safety Intelligence System](https://github.com/nawwarah-analyst/ehs-safety-intelligence) project — `incidents`, `audits`, and `training`.

EDA stands for **Exploratory Data Analysis**. It's the step you do *before* building anything — to understand your data, catch problems early, and make sure your analysis is built on a clean foundation.

I skipped this step. The dashboard got built on raw, unvalidated data. This EDA was done afterward to understand what I missed and how it affects the output.

---

## Tools used

- **BigQuery (SQL)** — all analysis was done in SQL
- **Google Cloud Platform** — where the source tables are hosted

---

## Files in this repo

| File | What it covers |
|---|---|
| `01_incidents.sql` | EDA on the incidents table |
| `02_audit.sql` | EDA on the audits table |
| `03_training.sql` | EDA on the training table |
| `04_cross-table.sql` | Validation across all three tables together |

Each file follows the same structure: row count → nulls → duplicates → categorical profiling → date checks → logic checks → summary.

---

## What I checked (and what each script does)

### 01 — Incidents
The core table. Every safety incident recorded by site, department, shift, and employee.

Checks done:
- Total records and date range
- Null values across all 15 columns
- Duplicate incident IDs and exact duplicate rows
- Categorical profiling — `dept_name`, `site_location`, `shift`, `injury_type`, `severity`, `root_cause`, `near_miss_flag`
- Date logic — reporting lag, impossible dates (reported before it happened)
- Cross-column logic — near-miss flagged but injury also recorded (contradictory), incidents with no supervisor linked

### 02 — Audits
Compliance audit records by site and department with scores and pass/fail status.

Checks done:
- Total records, date range, missing months
- Null audit IDs (known issue in raw data)
- Duplicate audit IDs and same dept/site/date audited more than once
- Categorical profiling — `dept`, `site`, `status`, `followup_required`, `remarks`
- Score profiling — including non-numeric entries (e.g. `"eighty five"` instead of `85`)
- Cross-column logic — score of 92 marked FAIL, score below 60 marked PASS, failed audits with no followup flagged

### 03 — Training
Employee training records including completion status, dates, and expiry.

Checks done:
- Total records, date range for completion and expiry
- Null check across all columns
- Duplicate employee IDs with different names (possible data entry error)
- Same employee, same training type completed more than once (refresher vs duplicate?)
- Categorical profiling — `dept`, `site`, `training_type`, `completed`, `notes`
- Date format audit — mixed formats found: `2024/10/30`, `27-03-24`, `17-08-25`

### 04 — Cross-table validation
The most important script. Checks whether the three tables are actually consistent with each other — because if they're not, any JOIN will silently drop records.

Checks done:
- Sites in incidents but not in audits
- Sites in incidents but not in training
- Departments in incidents but not in audits
- Full site coverage matrix across all three tables
- Employees who had an incident but have zero training records
- High severity incident employees with incomplete or no relevant training

---

## Key issues found

These are the data quality problems the EDA revealed — all of which affect the accuracy of the existing dashboard.

**Naming inconsistencies**
- Department: `"Prod."` and `"Production"` treated as two different departments
- Site: `"Plant A"` and `"Plant-A"` treated as two different sites
- These cause grouping errors in any chart broken down by dept or site

**Mixed value formats**
- `near_miss_flag` has values: `Y`, `N`, `1`, `0` — all meaning the same two things
- `followup_required` same issue: `Y`, `Yes`, `1`, `N`, `No`, `0`
- `completed` in training: `True`, `Yes`, `1` mixed together
- Date formats inconsistent across all three tables

**Logic contradictions**
- An audit scored **92** is marked as `FAIL` — this is either a data entry error or the pass/fail threshold was applied inconsistently
- Near-miss records that also have an `injury_type` recorded — a near-miss by definition means no injury occurred
- Failed audits with `followup_required = N` — a compliance gap that would be invisible in the current dashboard

**Non-numeric data in numeric columns**
- The `score` column in audits contains text entries like `"eighty five"` instead of `85`
- These rows are silently excluded from any `AVG(score)` calculation — meaning the average audit score in the dashboard is calculated on incomplete data

**Cross-table gaps**
- Some employees who had HIGH severity incidents have no training records at all
- This is either a real compliance failure or a data linkage issue — but it's invisible in the current dashboard because it was never validated

---

## Impact on the EHS dashboard

The dashboard was built before these issues were found. As a result:

- Near-miss counts may be overstated
- Training compliance rates may appear higher than they actually are
- The employee risk linkage (incidents → training) is not validated

The dashboard numbers are not necessarily wrong — but they cannot be confirmed as accurate without fixing these issues first.

---

## What I would do differently

1. Run EDA before building anything
2. Create a data cleaning/transformation layer between raw data and the model
3. Standardise all categorical values and date formats at source
4. Validate cross-table relationships before any JOIN
5. Document data quality decisions explicitly so anyone reviewing the work knows what was cleaned and why

---

## Status

The EHS dashboard has not been rebuilt yet. Fixing it properly requires going back to the data cleaning stage — which is the right next step, but not something I am prioritising right now as I focus on building other skills and applying for roles.

This EDA stands as documentation of what I found, what I learned, and how I would approach it differently next time.

---

*Part of my data analyst portfolio — [nawwarah-analyst.github.io](https://nawwarah-analyst.github.io/github.io/)*
