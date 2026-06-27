# Florida Emergency Management — PA Recovery Analysis
## Master Project README — READ THIS BEFORE EVERY ACTION

> **Version 4.0 — First principles approach, from-scratch analysis, human code standards**
> Last updated: June 26, 2026 | Phase 1 COMPLETE | Phase 2 NEXT

---

## ⚠️ COWORK STANDING INSTRUCTIONS — READ FIRST, EVERY SINGLE SESSION

1. Read this entire README before writing a single line of code
2. Every action must serve Q1, Q2, or Q3 — state which one before starting
3. Do not rely on FLPA_Completion_Estimates.docx — treat it as an unverified hypothesis
4. Do not assume data structure, field meaning, or status codes — verify from the data
5. Do not deviate into analysis not tied to the three core questions
6. Build everything from scratch — discover relationships from the data itself
7. After each task, check back against the Phase checklist in this README
8. If uncertain about anything — stop and ask the user instead of assuming
9. Write code like a human would — see coding standards section below
10. If you are about to do something not explicitly in this README — stop and ask first

---

## 📁 Project Folder Structure

```
C:\Users\gopi\Simeon Global Consulting\SGC Project Management - FDEM Data Analytics\
│
├── Recovery Dashboard\
│   └── New Export\                              ← DATA LIVES HERE (read-only, never modify)
│       ├── FLPA Grants\
│       │   └── FLPA_Grant_Export_.csv           ← GRANT EXPORT — Q1+Q2+Q3 anchor
│       ├── FLPA Projects Export\
│       │   ├── FLPA_Projects_Export_.csv        ← PROJECTS EXPORT — Q1+Q2+Q3
│       │   └── nan.csv                          ← duplicate — ignore
│       ├── FLPA Project Version Export\
│       │   └── FLPA_Project_Version_Export_.csv ← PW VERSIONS — Q1
│       ├── FLPA Payables Export\
│       │   ├── FLPA_Payables_Export_.csv        ← PAYABLES — Q2
│       │   └── 20260622-102517643-67221_ListExport.csv  ← verify vs above
│       ├── FLPA Receivables Export\
│       │   └── FLPA_Receivables_Export_.csv     ← RECEIVABLES — Q2
│       ├── FLPA Reimbursement Request\
│       │   └── FLPA_Reimbursement_Request_.csv  ← REIMBURSEMENTS — Q2
│       ├── FLPA Account Closeouts\
│       │   └── FLPA_Account_Closeouts_.csv      ← ACCOUNT CLOSEOUTS — Q3
│       ├── FLPA Large Project Closeout Export\
│       │   └── FLPA_Large_Project_Closeout_Export_.csv  ← LPC — Q3
│       ├── FLPA Appeals Export\
│       │   └── FLPA_Appeals_Export_.csv         ← PROJECT APPEALS — Q3
│       ├── FLPA Applicant Appeals Export\
│       │   └── FLPA_Applicant_Appeals_Export_.csv  ← APPLICANT APPEALS — Q3
│       ├── FLPA F-ROC\
│       │   └── F-ROC_Scores_06262026.csv        ← RISK SCORES — predictor
│       ├── FLPA Quarterly reports\
│       │   └── FLPA Q4 2025 Quarterly Reports.csv  ← PROGRESS REPORTS — Q1/Q2
│       ├── FLPA Completed Qtr Reports by Project\
│       │   └── FLPA Completed Qtr Reports by Project.csv
│       ├── FIDA_44559_PA_EMgrants_FL_ProjVersion_NONPII\
│       │   ├── FIDA_44559_PA_EMgrants_FL_ProjVersion_4337earlier_NONPII.csv
│       │   └── FIDA_44559_PA_EMgrants_FL_ProjVersion_after4337_NONPII.csv
│       ├── FLPA Accounts Export\
│       │   └── FLPA_Accounts_Export_06262026.csv  ← ACCOUNTS — Q3
│       ├── FLPA Applicants Export\
│       │   └── FLPA_Applicants_Export_.csv      ← APPLICANT MASTER — predictor
│       ├── FLPA Project Amendments Export\
│       │   └── FLPA_Project_Amendments_Export_.csv  ← AMENDMENTS
│       └── [other folders]                      ← contacts, logs, sync — not needed
│
└── Analysis_Gopi\                               ← ALL WORK LIVES HERE
    ├── README.md                                ← THIS FILE
    ├── file_paths.py                            ← single source of truth for all paths
    ├── notebooks\
    │   ├── 01_eda_grants.py
    │   ├── 02_eda_projects.py
    │   ├── 03_eda_payments.py
    │   ├── 04_build_relationships.py
    │   ├── 05_q1_obligation_model.py
    │   ├── 06_q2_payment_model.py
    │   └── 07_q3_closeout_model.py
    └── outputs\
        ├── models\
        ├── predictions\
        └── charts\
```

**RULE: Never write anything into Recovery Dashboard or New Export**
**RULE: Never hardcode file paths in notebooks — always import from file_paths.py**
**RULE: Never copy or duplicate raw data files into Analysis_Gopi**
**RULE: file_paths.py is the only place paths are defined — update it there if anything changes**

---

## 🎯 The Three Core Questions

Every script, model, chart, and decision must map to one of these.
Nothing else gets built.

### Q1 — Expected Obligation Completion
> "For a given disaster, when will all projects be fully obligated?"

- Obligation = federal commitment of funds to a Project Worksheet (PW)
- Done = every PW in the disaster has an Obligated Date
- This is a time-to-event problem: how long from declaration to full obligation?

### Q2 — Expected Payment Completion
> "For a given disaster, when will all obligated funds be fully paid out?"

- Payment = actual disbursement to the subrecipient after obligation
- Done = Total Paid / Total Obligated reaches ~98%+
- Depends on Q1 — cannot be complete before obligation is complete

### Q3 — Expected Disaster Closeout
> "For a given disaster, when will the formal closeout occur?"

- Closeout = FEMA signs off, all accounts reconciled, grant officially closed
- Done = Grant is Closed = True and Closed Date is populated
- Depends on Q1 and Q2 — last step in the chain

### Dependency Chain
```
Disaster Declaration
        ↓
[Q1] Obligation Completion  ──→ feeds into
        ↓
[Q2] Payment Completion     ──→ feeds into
        ↓
[Q3] Disaster Closeout
```

---

## 🔬 First Principles Analytical Approach

### The Existing Document Is NOT the Source of Truth

`FLPA_Completion_Estimates.docx` exists and contains estimates. It should be
treated as an unverified prior — a hypothesis to test, not a baseline to build on.

Reasons to start from scratch:
- The document uses heuristic bracket rules — not statistical models
- We do not know if those brackets reflect actual historical patterns
- File mislabeling in the data folder suggests the export may have errors too
- Payment estimates use a linear model which is likely wrong (payments are S-curves)
- Closeout estimates assume sequential bottlenecks without testing for interaction

**Our job is to let the data tell us what is true.**

### What "From Scratch" Means in Practice

1. Do not assume which fields are populated — check null rates first
2. Do not assume field meanings match their names — verify against actual values
3. Do not assume the column schema matches the Data Dictionary — verify every file
4. Do not assume relationships between files — discover and test join keys
5. Do not assume the existing status categories mean what you think they mean
6. Do not assume linear relationships — plot distributions before modeling
7. Do not assume a model type — let the data shape and distribution guide selection

### Phase 2 Is Entirely About Understanding the Data

Before any model is chosen:
- What does the data actually contain?
- What are the real distributions of key variables?
- How do files actually relate to each other?
- How many closed disasters exist to train on?
- Are the key date fields reliable and consistent?
- Are there outliers, data quality issues, or anomalies to address?

Only after Phase 2 is complete do we move to model selection.

---

## 🗂️ Confirmed File Inventory

### New Export folder has properly named files in subfolders
The mislabeling issue was in the old Recovery Dashboard root folder.
The New Export folder has the correct files with correct names.
Still verify every file by actual columns before trusting it — do not assume.

### Modeling Files — Core (import via file_paths.py)

| Key in file_paths.py | Subfolder | Filename | Size | Questions |
|---|---|---|---|---|
| `grants` | FLPA Grants | FLPA_Grant_Export_.csv | 0.1 MB | Q1+Q2+Q3 |
| `projects` | FLPA Projects Export | FLPA_Projects_Export_.csv | 79.6 MB | Q1+Q2+Q3 |
| `pw_versions` | FLPA Project Version Export | FLPA_Project_Version_Export_.csv | 88.9 MB | Q1 |
| `payables` | FLPA Payables Export | FLPA_Payables_Export_.csv | 47.9 MB | Q2 |
| `payables_2` | FLPA Payables Export | 20260622-...csv | 34.8 MB | Q2 verify |
| `receivables` | FLPA Receivables Export | FLPA_Receivables_Export_.csv | 3.0 MB | Q2 |
| `reimbursements` | FLPA Reimbursement Request | FLPA_Reimbursement_Request_.csv | 22.4 MB | Q2 |
| `account_closeouts` | FLPA Account Closeouts | FLPA_Account_Closeouts_.csv | 5.6 MB | Q3 |
| `large_project_closeouts` | FLPA Large Project Closeout Export | FLPA_Large_Project_Closeout_Export_.csv | 11.3 MB | Q3 |
| `appeals_project` | FLPA Appeals Export | FLPA_Appeals_Export_.csv | 3.8 MB | Q3 |
| `appeals_applicant` | FLPA Applicant Appeals Export | FLPA_Applicant_Appeals_Export_.csv | 0.2 MB | Q3 |
| `froc_scores` | FLPA F-ROC | F-ROC_Scores_06262026.csv | 0.6 MB | predictor |
| `quarterly_reports` | FLPA Quarterly reports | FLPA Q4 2025 Quarterly Reports.csv | 3.1 MB | Q1/Q2 |
| `accounts` | FLPA Accounts Export | FLPA_Accounts_Export_06262026.csv | 12.9 MB | Q3 |
| `applicants` | FLPA Applicants Export | FLPA_Applicants_Export_.csv | 6.6 MB | predictor |
| `fida_pre_4337` | FIDA_44559_... | ...4337earlier_NONPII.csv | 10.7 MB | Q1 |
| `fida_post_4337` | FIDA_44559_... | ...after4337_NONPII.csv | 10.3 MB | Q1 |

### Files Not Used for Modeling

| Subfolder | Reason |
|---|---|
| FLPA All Contacts | People records — not predictive |
| FLPA Contacts | People records — not predictive |
| FLPA Internal External Contacts | People records — not predictive |
| FLPA Assignments | Staff role assignments — not predictive |
| FLPA Audit Tracking | Audit events — not predictive |
| FLPA Data Logs | Field-level change log — not predictive |
| FLPA SB4A Account Export | Program subset — skip for now |
| FLPA SB4A Payables | Program subset — skip for now |
| FLPA RPA Export | Application intake — not predictive |
| FLPA RPA Import | Historical import — not predictive |
| FLPA Costline Sync Analysis | Line-item reconciliation — not needed |
| FLPA PW Import Analysis/Archive | Historical snapshots — not needed |
| FLPA Projects Export/nan.csv | Duplicate of projects file — discard |

---

## 🔗 File Relationship Discovery — Phase 4 Task

This is a key step that must be done from the data, not assumed from the schema doc.

### Known Join Keys (to be verified in data)
- `Grant #` — present in all files, top-level identifier
- `Grant # + Proj S#` — identifies a project
- `Grant # + Proj S# + # (version)` — identifies a PW version
- `Grant # + State Applicant Number` — identifies an account

### What to Verify During EDA
- Do join keys actually match across files? (check for nulls, format differences)
- Are there orphaned records that don't join cleanly?
- Are there duplicate rows? Which key identifies a unique record?
- Does `Grant #` in the Projects file match `Grant #` in the Grant file?
- What % of projects have a matching grant record?

### Relationships to Build in Phase 4
```
Grant (1 row per disaster)
  └── Project (many rows per grant)
        └── Project Version / PW (many versions per project)
              └── Payable (many payments per project)
  └── Appeals (many appeals per grant)
```

---

## 📊 Can We Actually Answer the Three Questions With This Data?

This section is the honest feasibility assessment. Read it before touching the data.
It tells us what we can build now, what we need more data for, and what we may
discover during EDA that changes the entire plan.

---

### Q1 — Expected Obligation Completion — Feasibility

**What we need vs what we have:**
- Start date per disaster (Declared Date) ✅ confirmed in grant file
- End date (date last project was obligated) — derivable from max(Project Obligated Date) per grant ✅
- Features that predict how long obligation takes ✅ workflow steps, project count, size mix

**Honest assessment:**
We can build a Q1 model with current data. Training set = grants at 100% obligation.
Count them in EDA first. If fewer than 15, the model will be statistically weak
and we say so clearly in the output rather than pretend otherwise.

**What would make it better:**
- Full workflow step history per PW (how long each step took, not just current)
- Reason codes for On Hold and Returned statuses
- FEMA review processing times as a separate variable

---

### Q2 — Expected Payment Completion — Feasibility

**What we need vs what we have:**
- Total Obligated, Total Paid per grant ✅ confirmed in grant file
- Payment transaction dates to understand the curve shape ✅ in payables file
- Time from obligation to first payment and payment velocity ✅ derivable from payables
- ⚠️ No pre-built monthly snapshots — we reconstruct the curve from transactions

**Honest assessment:**
The grant file gives current payment % but not how payments accumulated over time.
The payables file (47.9 MB of individual transactions) lets us reconstruct the curve.
The existing estimates use a linear projection. We suspect an S-curve. We test it
from the transaction data before choosing any model — do not assume linearity.

**What would make it better:**
- Monthly payment snapshots going back to declaration date (for direct time-series modeling)
- Reason codes for payment delays
- Subrecipient drawdown request dates (when they ask vs when they're paid)

---

### Q3 — Expected Disaster Closeout — Feasibility

**What we need vs what we have:**
- Closed Date per grant (target variable) ✅ confirmed field exists — count non-nulls in EDA
- Open item counts (accounts, appeals, LPCs) as features ✅ in grant, LPC, appeals files
- ⚠️ Unknown how many grants have a confirmed Closed Date — EDA will tell us

**Honest assessment:**
Q3 is the most uncertain. Closeout depends on FEMA decisions, audit findings,
and appeal outcomes — some of which are not in this data. The model produces
a baseline timeline assuming normal processing. It cannot predict FEMA policy
changes or litigation delays. If fewer than 10 grants have a Closed Date, the
Q3 model will be regression only, not survival analysis, and we say so.

**What would make it better:**
- FEMA audit finding dates and resolution dates per grant
- Appeal outcome dates (when resolved, not just when filed)
- Historical data on how long each LPC workflow step takes

---

### Data We Do Not Have But Would Help All Three Questions

| Missing Data | Why It Helps | How to Get It |
|---|---|---|
| Monthly payment snapshots per grant | Shows payment curve shape over time — needed for Prophet/ARIMA | Ask client to pull historical grant financials by month |
| PW workflow step history (not just current step) | Shows how long each step takes, not just where it is now | FEMA EMMIE system export — may require data request |
| Appeal resolution dates and outcomes | Tells us how long appeals actually block closeout | Should be in appeals file — verify during EDA |
| FEMA audit findings per grant | Major closeout blocker not currently captured | FEMA audit system — separate data request |
| Disaster severity/type attributes | Would improve obligation speed predictions | FEMA disaster declarations API — publicly available |

---

### What Happens If the Current Workflow Assumption Is Wrong

The existing estimates document assumes clean sequential stages:
obligation → payment → closeout. Reality may be messier.
During EDA we test these hypotheses and update the plan if any are false.

**Hypothesis 1 — Are payments actually sequential to obligation?**
Some grants may be paying out while still obligating new projects.
If true: Q1 and Q2 overlap — the model structure changes.

**Hypothesis 2 — Does closeout actually require 100% payment?**
Some grants may close with small unpaid balances (retentions, offsets).
If true: the Q2 completion threshold is not 100% — we discover the real number.

**Hypothesis 3 — Do PA, FMAG, and SPA follow different patterns?**
FMAG grants are smaller and simpler. SPA grants are loans not reimbursements.
If true: separate models per program type, not one combined model.

**Hypothesis 4 — Are large projects the primary driver of closeout delay?**
The existing document assumes LPC count is the key bottleneck.
If true: confirm statistically. If false: find the real driver from data.

**If any hypothesis is false, we update this README and the plan changes accordingly.
We do not force the data into a framework that does not fit.**

---

### Full Analysis Scope — Everything We Will Build

Nothing gets skipped because it seems hard.
Nothing gets added that is not listed here without updating this README first.

**Phase 2 — EDA**
- Grant portfolio overview: age, size, program type, status distribution
- Payment rate distribution across all 60 grants
- Obligation rate distribution across all active grants
- Days-to-obligation distribution for completed grants
- Days-to-closeout distribution for closed grants
- Payment curve shape analysis — test linear vs S-curve from transaction data
- Null rate analysis on every date field we depend on
- Outlier identification — which grants behave very differently and why
  (COVID-19 grant 4486, Surfside 3560 are likely outliers — assess separately)
- Cross-file join verification — do relationships actually hold in the data?

**Phase 3 — Feature Engineering**
- Target variable construction: Q1 days-to-full-obligation, Q2 payment %, Q3 days-to-close
- Payment velocity per grant (from payables transaction dates)
- Bottleneck count features: open accounts, open LPCs, open appeals per grant
- Disaster characteristics: program type, project count, size mix, total obligated
- Progress features: obligation %, payment %, % complete across projects
- Risk features: F-ROC scores, pre/post award risk ratings from applicants file
- Grant age: months since declaration date

**Phase 4 — Model Building**
- Simple baseline first: linear regression on log(days) for Q1, Q2, Q3
- Kaplan-Meier survival curves for each question — visual first
- Cox Proportional Hazards with covariates if survival is the right fit
- Compare baseline vs survival — pick the better fit, document why
- Test whether PA/FMAG/SPA need separate models
- Test whether large-project-heavy grants need separate treatment
- Model selection based on data — not decided in advance

**Phase 5 — Validation**
- Hold out a set of closed grants — train on rest, predict on held-out
- Measure accuracy: how many days off on average?
- Measure calibration: do confidence intervals contain the true date?
- Stress test on edge cases: COVID-19, Surfside Building Collapse
- Document explicitly where the model is reliable and where it is not

**Phase 6 — Practical Output**
- A formula or scoring function for each of Q1, Q2, Q3
- A prediction table: for every active grant, estimated completion dates with ranges
- Plain-English explanation of what drives each estimate
- Flagging system: which grants are behind expected pace and by how much

**Phase 7 — Reporting Layer**
- Summary view of all 60 grants with model estimates
- Comparison of model estimates vs existing document estimates
- Export-ready format FDEM can use in reports and briefings
- Designed to update when new data is pulled

---

## 🧮 Modeling — Approach and Rules

### Model Selection Is Phase 4 — Not Yet

Do not select a model until Phase 2 EDA and Phase 3 feature engineering are done.
The data distributions, training set size, and variable behavior determine the model.

### Candidate Approaches (evaluate after EDA)

| Method | Why It Might Fit | Why It Might Not |
|---|---|---|
| **Survival Analysis (Kaplan-Meier)** | Time-to-event is exactly our problem | Needs enough closed grants to be meaningful |
| **Cox Proportional Hazards** | Adds covariates to survival curve | Assumes proportional hazards — must test |
| **Accelerated Failure Time** | Produces interpretable formula | Assumes parametric distribution — must test |
| **Linear Regression on log(days)** | Simple, explainable, good baseline | May miss non-linear patterns |
| **Prophet** | Handles seasonality in payment flows | Only useful if time-series snapshots exist |
| **XGBoost** | Handles complex feature interactions | Hard to explain — use only if simpler fails |

### Rules That Cannot Be Broken

1. Always build the simplest model first — complexity must earn its place
2. Training set = only grants with a confirmed Closed Date (real observed events)
3. Open/active grants are censored observations — handle them correctly
4. Q2 model must receive Q1 output as an input feature
5. Q3 model must receive Q1 and Q2 outputs as input features
6. Every model output must be a date or date range — not a category label
7. Report confidence intervals alongside every prediction
8. Validate on held-out grants before reporting results

---

## 💻 Code Standards — Write Like a Human, Not a Machine

This is non-negotiable. Every script written in this project must follow these rules.

### What Good Code Looks Like Here

```python
import pandas as pd
import os

# We load the grant file first because it's the anchor for everything else.
# One row per disaster — this tells us declared date, closed date, and financials.
# File is mislabeled in the folder; real identity confirmed by column inspection.

DATA_DIR = r"C:\Users\gopi\Simeon Global Consulting\SGC Project Management - FDEM Data Analytics\Recovery Dashboard"
WORK_DIR = r"C:\Users\gopi\Simeon Global Consulting\SGC Project Management - FDEM Data Analytics\Analysis_Gopi"

grant_file = os.path.join(DATA_DIR, "FLPA_Project_Amendments_Export_.csv")

grants = pd.read_csv(grant_file, encoding="latin-1", low_memory=False)

# Quick sanity check — how many rows and what programs do we have?
print(f"Loaded {len(grants)} grants")
print(grants["Program"].value_counts())
```

### Rules for Every Script

- **Explain why, not just what.** Comments should say why a decision was made,
  not just repeat what the code does. Bad: `# load the file`. Good: `# latin-1
  encoding because the portal exports have non-UTF8 characters in address fields`

- **Name variables for what they represent, not what type they are.**
  Bad: `df`, `df2`, `temp`. Good: `grants`, `projects`, `obligated_projects`

- **No magic numbers without explanation.**
  Bad: `df[df["payment_pct"] > 0.98]`. Good:
  ```python
  # 98% is the practical "complete" threshold — the last 2% is typically
  # final audit adjustments and retentions that clear after formal closeout
  PAYMENT_COMPLETE_THRESHOLD = 0.98
  df[df["payment_pct"] > PAYMENT_COMPLETE_THRESHOLD]
  ```

- **Separate concerns into clear sections with headers.**
  Every script should have: Load → Verify → Clean → Analyze → Output

- **Print meaningful diagnostics as you go.**
  Don't run silently. Show row counts, null rates, and value distributions
  at each step so the output tells a story.

- **Handle errors explicitly — don't let silent failures pass.**
  If a join produces fewer rows than expected, say so and investigate.

- **Never hardcode file paths inline — define them at the top.**

- **No one-liner chains that collapse 5 operations into one line.**
  Break it up. Readability beats cleverness every time.

### What to Avoid

```python
# ❌ AI slop — don't write like this
result = df.groupby('Grant #')['Federal Paid'].sum().reset_index().merge(
    df2[['Grant #','Total Obligated']], on='Grant #').assign(
    pct=lambda x: x['Federal Paid']/x['Total Obligated']).query('pct < 1')

# ✅ Human code — write like this instead
# Aggregate payments to grant level
paid_by_grant = projects.groupby("Grant #")["Federal Paid"].sum().reset_index()
paid_by_grant.columns = ["Grant #", "total_federal_paid"]

# Pull in the obligated amount from the grant file so we can compute payment %
grant_financials = grants[["Grant #", "Total Obligated"]].copy()
payment_status = paid_by_grant.merge(grant_financials, on="Grant #", how="left")

# Calculate what % of obligated funds have actually been paid out
# Grants below 100% still have money sitting in the pipeline
payment_status["payment_pct"] = (
    payment_status["total_federal_paid"] / payment_status["Total Obligated"]
)

# Flag anything under 100% as still having unpaid obligations
incomplete = payment_status[payment_status["payment_pct"] < 1.0]
print(f"{len(incomplete)} grants still have unpaid obligations")
```

---

## 🚦 Project Phases

| Phase | Name | Status |
|---|---|---|
| **1** | Document Review & Question Definition | ✅ COMPLETE |
| **2** | Data Loading, Verification & EDA | 🔄 ACTIVE — START HERE |
| **3** | Relationship Building & Feature Engineering | ⏳ Pending Phase 2 |
| **4** | Model Selection & Baseline Building | ⏳ Pending Phase 3 |
| **5** | Model Evaluation & Refinement | ⏳ Pending Phase 4 |
| **6** | Formula Extraction & Practical Output | ⏳ Pending Phase 5 |
| **7** | Dashboard / Reporting Layer | ⏳ Pending Phase 6 |

---

## 📋 Phase 2 Checklist — Work Through in Order

### 2A — Verify every file before using it

For each confirmed modeling file:
- [ ] Load and print actual column names — do they match expectations?
- [ ] Check row count — is it reasonable?
- [ ] Check for duplicate rows on the expected primary key
- [ ] Check null rates on every date field we need
- [ ] Check value distributions on status/flag fields

### 2B — Grant-level EDA (start here)

Load `FLPA_Project_Amendments_Export_.csv` and answer:
- [ ] How many grants are there total?
- [ ] How many have a non-null `Closed Date`? (this is our training set for Q3)
- [ ] What is the distribution of grant age (today minus Declared Date)?
- [ ] What programs are represented (PA, FMAG, SPA) and in what counts?
- [ ] What does the distribution of Total Obligated look like?
- [ ] What is the range of payment % (Total Paid / Total Obligated)?
- [ ] Do any grants have Total Paid > Total Obligated? (flag as anomaly)

### 2C — Project-level EDA

Load `-2190494-82047.csv` and answer:
- [ ] How many projects are there?
- [ ] How many have `Project is Obligated = 1`?
- [ ] How many have `Project is Closed = 1`?
- [ ] Null rate on `Project Obligated Date` and `Project Closed Date`
- [ ] Distribution of `Eligible Amt` — what does the size spread look like?
- [ ] How many are Small vs Large projects?
- [ ] Distribution of `Percent Complete` across open projects

### 2D — Project Version (PW) EDA

Load `-2190493-36576.csv` and answer:
- [ ] How many PW versions exist?
- [ ] Distribution of `Days Since Submitted` for unobligated versions
- [ ] How many versions are `On Hold`?
- [ ] Most common `Workflow Step` values — what does the pipeline look like?
- [ ] Distribution of `% Comp` — is it reliable?

### 2E — Relationship verification

- [ ] Do all projects join cleanly to a grant? What % match?
- [ ] Do all PW versions join cleanly to a project? What % match?
- [ ] Are there grants in the project file not in the grant file?
- [ ] Are join key formats consistent (spaces, types, casing)?

### 2F — Target variable construction

- [ ] Q1 target: days from `Declared Date` → last `Project Obligated Date` per grant
- [ ] Q2 target: `Total Paid` / `Total Obligated` per grant (current rate)
- [ ] Q3 target: days from `Declared Date` → `Closed Date` (closed grants only)
- [ ] Label each grant as closed (observed) or open (censored)
- [ ] How many closed grants do we have? Is it enough to train a model?

### 2G — Before leaving Phase 2, answer:

- [ ] Is there a meaningful difference in timelines between PA, FMAG, and SPA?
      (If yes: may need separate models per program type)
- [ ] Do large projects behave differently from small ones in obligation timing?
- [ ] Are there any disasters that are clear outliers (COVID, Surfside)?
      (May need to handle separately or exclude from training)
- [ ] What is the actual shape of the payment curve — linear or S-curve?

---

## 🛑 What This Project Is NOT

- ❌ Not a replication of the existing estimates document
- ❌ Not a compliance or audit tool
- ❌ Not a budget forecast for future disasters
- ❌ Not a staff performance measurement system
- ❌ Not a real-time dashboard (that is Phase 7, after models are validated)
- ❌ Do not analyze contacts, assignments, or data logs
- ❌ Do not build anything that cannot be traced directly to Q1, Q2, or Q3

---

## 🗂️ Glossary

| Term | Definition |
|---|---|
| **PA** | Public Assistance — FEMA's main disaster recovery grant program |
| **PW** | Project Worksheet — defines scope and cost of one recovery project |
| **Obligation** | Federal commitment of funds to a PW — money is reserved not yet paid |
| **Payment** | Actual disbursement of obligated funds to the subrecipient |
| **Closeout** | Formal end — all PWs reconciled, audits done, FEMA signed off |
| **Subrecipient** | Local government or non-profit receiving PA funds through the state |
| **FDEM** | Florida Division of Emergency Management |
| **LPC** | Large Project Closeout — formal process specific to large projects |
| **F-ROC** | Florida Readiness and Operations Capacity — FDEM risk score per applicant |
| **FMAG** | Fire Management Assistance Grant — for wildfire disasters |
| **SPA** | State Program Assistance — state-run loan and block grant programs |
| **Account** | One applicant's enrollment in one specific grant |
| **PoP** | Period of Performance — deadline for completing all work |
| **Censored** | Survival analysis term — grant still open, event not yet observed |
| **Grant Export** | The file named `FLPA_Project_Amendments_Export_.csv` in Recovery Dashboard |

---

*Phase 1 complete. Phase 2 is next. Start in Cowork with 01_eda_grants.py.*
*The data is in Recovery Dashboard. All work goes in Analysis_Gopi.*
*Build everything from scratch. Test everything. Trust the data, not the documents.*