# 🦷 SD-ARCH

### Smart Dental Allocation & Research Hub

A unified data platform for Egyptian university dental clinics — fair patient-student allocation, clinical operations analytics, and research-ready datasets.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Data: Synthetic](https://img.shields.io/badge/Data-Synthetic-blue.svg)](#-dataset)
[![Status: Active](https://img.shields.io/badge/Status-Active-success.svg)](.)
[![SQL Server](https://img.shields.io/badge/Database-SQL_Server-red.svg)](.)
[![Power BI](https://img.shields.io/badge/Dashboards-Power_BI-yellow.svg)](.)

---

## 📋 Table of Contents

1. [About the Project](#-about-the-project)
2. [Problem We Solve](#-problem-we-solve)
3. [Dataset](#-dataset)
4. [Data Model](#️-data-model)
5. [Data Pipeline](#-data-pipeline)
6. [Cleaning Methodology](#-cleaning-methodology)
7. [KPIs & Insights](#-kpis--insights)
8. [Findings](#-findings)
9. [Statistical Analysis](#-statistical-analysis)
10. [Predictive Models](#-predictive-models)
11. [Power BI Dashboards](#-power-bi-dashboards)
12. [Known Data Issues & Improvement Backlog](#-known-data-issues--improvement-backlog)
13. [Web Application](#-web-application)
14. [Repository Structure](#-repository-structure)
15. [Team](#-team)
16. [Roadmap](#️-roadmap)
17. [License](#-license)

---

## 🎯 About the Project

**SD-ARCH** is an end-to-end analytics and operations platform for dental colleges. It addresses four interconnected problems:

| Problem | Our Solution |
| --- | --- |
| Unfair case distribution among students | Fair-distribution algorithm — per-year Gini below the 0.20 target |
| Manual allocation by receptionists | Automated 2-phase algorithm with audit trail |
| No leadership visibility | Three role-based Power BI dashboards |
| Data locked in paper/Excel | SQL Server database + research-ready exports |

The project includes:

- ✅ A **synthetic dataset** of 5 academic years (2021–2026)
- ✅ **SQL Server schema + ETL pipeline**
- ✅ **30 KPIs** across operational, student, patient, and strategic pillars
- ✅ **Statistical analyses** — Gini, Kolmogorov–Smirnov, logistic regression, Kaplan–Meier, chi-square, Sankey/Markov flow
- ✅ **3 predictive models** planned (no-show prediction, demand forecast, graduation risk)
- ✅ **Power BI dashboards** (Admin, Clinic Manager, Student)
- ✅ **A web application** (frontend + backend)
- ✅ **A 43-slide project presentation** covering methodology, findings, and recommendations

> **Note:** All data in this repository is **synthetic**. No real patient records are included.

---

## 🚨 Problem We Solve

In Egyptian university dental clinics today:

> *Some students graduate with 40+ cases while others struggle to find 5. Receptionists match patients to students by hand, relying on memory and goodwill. Leadership has no live view of clinic utilization, student progress, or patient flow. Rich clinical data is fragmented across notebooks and Excel sheets — unusable for research, quality improvement, or accreditation.*

SD-ARCH replaces this with a transparent, data-driven system built on five principles:

- **Minimum guarantee** — every student receives at least one patient per required case type before anyone gets a second.
- **Transparent** — every allocation is logged and auditable; fairness is provable via the Gini coefficient.
- **Anonymized for research** — a k-anonymous research dataset is produced automatically, ready for IRB submission.
- **Calendar-aware** — respects the Egyptian academic year (Sun–Thu week, Ramadan hours, exam breaks, public holidays).
- **Bilingual** — Arabic and English throughout.

### End-to-End Workflow

```
1. Patient Books      →  Online slot in the Diagnosis Clinic
2. Intern Diagnoses   →  Determines what treatment is needed
3. Fair Allocation    →  Algorithm assigns the patient to a student
4. Staff Approves     →  Teaching staff confirms the indication
5. Treatment Flow     →  Follow-ups scheduled; case completes
```

---

## 📊 Dataset

### At-a-Glance Statistics

| Metric | Value |
| --- | --- |
| Academic years covered | 5 (2021–22 → 2025–26) |
| Patient records | 134,640 (≈132K patients served) |
| Students | 2,718 across 7 cohorts |
| Teaching staff | 184 (5 academic ranks) |
| Clinical departments | 9 |
| Cases | 209,065 |
| Diagnosis appointments | 126,444 |
| Treatment appointments | 397,691 |
| **Total appointments** | **524,135** |
| Egyptian governorates represented | 27 |
| Total tables | 13 |

**Clinical departments:** Endodontics · Operative · Pedodontics · Periodontics · Oral Surgery · Fixed Prosthodontics · Removable Prosthodontics · Diagnosis · Intern Clinic

### Generation Approach

- **Reproducible:** Fixed random seed (42) — byte-for-byte identical on re-runs
- **Realistic:** Egyptian Sun–Thu work week, Ramadan reduced hours, exam breaks, public holidays
- **Demographically grounded:** Age pyramid, gender, and governorate distribution match real population data
- **Medically grounded:** Diabetes / hypertension / smoking prevalence by age and gender match Egyptian health statistics
- **Bilingual:** Every name has paired Arabic + English forms
- **UTF-8 BOM encoding:** Arabic displays correctly in Excel, Power BI, and SQL Server

📄 **Full specification:** [`DATA_GENERATION_PROMPT.md`](DATA_GENERATION_PROMPT.md) — hand this to any LLM to regenerate the entire dataset.

---

## 🗄️ Data Model

### 13 Interconnected Tables

```
┌──────────────────┐         ┌────────────────────┐
│    REFERENCE     │         │       ENTITY       │
├──────────────────┤         ├────────────────────┤
│ clinics          │         │ patients           │
│ governorates     │         │ students           │
│ case_requirements│         │ student_enrollments│
│ academic_calendar│         │ student_keenness   │
└──────────────────┘         │ staff              │
                             └────────────────────┘
                                       │
                                       ▼
                            ┌───────────────────────┐
                            │    ACTIVITY (FACT)    │
                            ├───────────────────────┤
                            │ appointments_diagnosis│
                            │ cases                 │
                            │ appointments_treatment│
                            │ student_case_progress │
                            └───────────────────────┘
```

### Table Catalog

| # | Table | Type | Rows | Purpose |
| --- | --- | --- | --- | --- |
| 1 | `clinics` | Reference | 9 | Clinical departments + intern clinic |
| 2 | `governorates` | Reference | 27 | Egyptian governorates, EN + AR |
| 3 | `case_requirements` | Reference | 51 | Required cases per year level |
| 4 | `academic_calendar` | Reference | 1,366 | Sun–Thu, Ramadan, exam breaks, holidays |
| 5 | `students` | Entity | 2,718 | All students across 7 cohorts |
| 6 | `student_enrollments` | Entity | 5,759 | Yearly enrollment state |
| 7 | `student_keenness` | Entity | 2,718 | Per-student willingness level |
| 8 | `staff` | Entity | 184 | Teaching staff (5 ranks) |
| 9 | `patients` | Entity | 134,640 | Patient records with demographics + medical history |
| 10 | `appointments_diagnosis` | Fact | 126,444 | First visits to diagnosis clinic |
| 11 | `cases` | Fact | 209,065 | Allocated cases (one row per case) |
| 12 | `appointments_treatment` | Fact | 397,691 | Treatment appointments (1–6 per case) |
| 13 | `student_case_progress` | Fact (snapshot) | 96,786 | Running tallies per student-year-case |

### Star Schema Diagram

```
        ┌────────────────────┐
        │  academic_calendar │ (Date Dim)
        └──────────┬─────────┘
                   │
┌──────────┐       │       ┌──────────┐
│ patients │───┐   │   ┌───│ students │
└──────────┘   │   │   │   └──────────┘
               ▼   ▼   ▼
┌──────────────────────────────┐
│         cases (FACT)         │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│    appointments_treatment    │
│            (FACT)            │
└──────────────────────────────┘
```

---

## 🔄 Data Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Raw CSVs   │ -> │   Cleaning   │ -> │  SQL Server  │ -> │  Analytics   │
│  (13 files)  │    │   Pipeline   │    │   Database   │    │    Layer     │
│              │    │              │    │              │    │ (Views+KPIs) │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                   │
                                         ┌─────────────────────────┼────────────────┐
                                         ▼                         ▼                ▼
                                 ┌──────────────┐        ┌───────────────┐  ┌──────────────┐
                                 │   Power BI   │        │    Web App    │  │  Predictive  │
                                 │  Dashboards  │        │ (3 audiences) │  │    Models    │
                                 └──────────────┘        └───────────────┘  └──────────────┘
```

### Six Analysis Phases

| Phase | Stage | What happens |
| --- | --- | --- |
| P1 | Data Cleaning | ETL pipeline: dedupe patients, normalize values, validate dates |
| P2 | EDA | Distributions, correlations, outliers |
| P3 | KPI Calculation | Define and compute every metric; unit-test each one |
| P4 | Statistical Analysis | Fairness tests, hypothesis testing, pathway analysis |
| P5 | Predictive Modeling | No-show prediction, demand forecasting, graduation risk |
| P6 | Dashboard Build | Power BI dashboards for each audience (Admin, Clinic, Student) |

### Tools Stack

| Stage | Primary Tool | Alternative |
| --- | --- | --- |
| Raw data inspection | Excel | Power Query |
| Cleaning | Python (pandas) | Power Query |
| Storage | Microsoft SQL Server | PostgreSQL |
| KPI computation | SQL + Python | DAX in Power BI |
| Statistical analysis | Python (scipy, lifelines, scikit-learn) | R |
| Predictive modeling | Python (XGBoost, Prophet) | scikit-learn |
| Dashboards | Power BI (incl. Python visuals) | Tableau |
| Web app | Frontend + backend on the same SQL Server database | — |

---

## 🧹 Cleaning Methodology

*Phase 1 — turning raw clinic data into analytics-ready form.*

### What's Broken in the Raw Data

| Scale | Issue |
| --- | --- |
| **6.5%** | of patients have missing phone numbers |
| **2,640** | duplicate patient records with slight name variations |
| **14** | different representations of True/False in medical flags |
| **16** | different variants of appointment status (`Attended` / `attended` / `Present`…) |
| **44** | variants of governorate names (`Alex` / `Alexandria` / `ALEXANDRIA`) |
| **397** | duplicate appointment IDs from join errors |
| **679** | impossible age values (`0`, `-1`, `150`, `999`) |
| **~0.3%** | impossible dates (DOB in the future, appointment before registration) |

### Our Cleaning Pipeline

The same seven steps run against every table, in order:

| # | Step | What it does |
| --- | --- | --- |
| 1 | **Standardize text** | Lowercase, trim whitespace, Unicode normalization |
| 2 | **Fuzzy dedupe patients** | Levenshtein distance + DOB + phone matching |
| 3 | **Normalize categories** | Map all variants to canonical values |
| 4 | **Fix boolean flags** | Convert the 14 variants to `True` / `False` |
| 5 | **Parse dates robustly** | Handle 4 format variants; flag impossibles |
| 6 | **Validate constraints** | Remove records violating business rules |
| 7 | **Create audit log** | Every transformation logged for traceability |

> ⚠️ **Note on step 2:** near-duplicate patients are **not** merged away. The 2,640 records sharing a DOB and a similar name are linked via a `duplicate_of` column and **both rows are kept** for manual review. Only 127 *exact* duplicates are dropped, so the cleaned `patients` table holds 134,513 rows.

### Cleaning Sequence per Table

Every table runs through the same five core steps plus optional extras. This is the structure used in the cleaning log workbook:

| Stage | What it covers |
| --- | --- |
| **TIP** | Header standardization (lowercase, underscores), blank-row removal |
| **STEP 1** — Data Types | Dates → ISO (`dayfirst=True`), integers → `Int64`, IDs kept as trimmed strings |
| **STEP 2** — Errors | Map every categorical variant to its canonical value |
| **STEP 3** — Nulls | Distinguish acceptable nulls from defects; never silently impute |
| **STEP 4** — Duplicates | Drop exact duplicates (keep first); flag near-duplicates |
| **STEP 5** — Primary Key | Verify PK / composite PK uniqueness |
| **EXTRA 1** — Encoding | Save with `encoding=utf-8-sig` so Arabic renders in Excel |
| **EXTRA 2** — Foreign Keys | Validate FKs; save orphans for review |
| **EXTRA 3** — Sanity | Cross-field logic (completion ≥ creation, DOB precedes registration) |
| **EXTRA 4** — Derived | Add analysis columns (`age_group`, `completion_pct`, `duration_days`, …) |
| **EXTRA 7** — Validation | Run the assertion script; all must pass |

**The NULL rule:** when a value can't be mapped to a canonical category, set it to `NULL`. Never default it to `False` or to a "most likely" guess — an unknown must stay visibly unknown.

### Cleaning Status by File

Ten of thirteen files are cleaned. Tracked in [`SD-ARCH_Cleaning_Logs.xlsx`](SD-ARCH_Cleaning_Logs.xlsx) — one sheet per file, with a `Status` column for Done / Missed / Not Applicable.

| # | Source file | Cleaned output | Rows in | Rows out | Owner | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `appointments_treatment.csv` | `appointments_treatment_clean.csv` | 397,691 | 397,294 | Mahmoud | ✅ Done |
| 2 | `cases.csv` | `cases_cleaned.csv` | 209,065 | 208,865 | Mahmoud | ✅ Done |
| 3 | `patients.csv` | `Cleaned- Patients.xlsx` | 134,640 | 134,513 | Mohamed Ali | ✅ Done |
| 4 | `appointments_diagnosis.csv` | `Cleaned- appointments_diagnosis.xlsx` | 126,444 | 126,314 | Mohamed Ali | ✅ Done |
| 5 | `student_case_progress.csv` | `student_case_progress_cleaned.csv` | 96,786 | 96,736 | Loay | ✅ Done |
| 6 | `student_enrollments.csv` | `student_enrollments_cleaned.csv` | 5,759 | 5,754 | Loay | ✅ Done |
| 7 | `students.csv` | `student_cleaned.xlsx` | 2,718 | 2,718 | Aya | ✅ Done |
| 8 | `student_keenness.csv` | `student_keenness_cleaned.xlsx` | 2,718 | 2,718 | Aya | ✅ Done |
| 9 | `staff.csv` | `staff_cleaned.xlsx` | 184 | 184 | Aya | ✅ Done |
| 10 | `clinics.csv` | `clinic_cleaned.xlsx` | 9 | 9 | Aya | ✅ Done |
| 11 | `academic_calendar.csv` | — | 1,366 | — | Kholod | ❌ Not done |
| 12 | `case_requirements.csv` | — | 51 | — | Kholod | ❌ Not done |
| 13 | `governorates.csv` | — | 27 | — | Kholod | ❌ Not done |

> ⚠️ `academic_calendar` is the highest-priority gap: the broken `is_active_clinic_day` flag lives in it, and it is the date dimension for the entire star schema.

### Cleaning Tools

| Tool | Best for | Code? |
| --- | --- | --- |
| **Excel** | Small files, manual checks, lookup sheets | No |
| **Power Query** | Visual cleaning in Excel/Power BI | No |
| **Python (pandas)** | Large files, batch automation, statistics | Yes |
| **Power BI** | Live dashboards on cleaned data | DAX |

### 📚 Documentation Standards

Every cleaning task produces 4 deliverables:

**1. Data Cleaning Log** ✅ — [`SD-ARCH_Cleaning_Logs.xlsx`](SD-ARCH_Cleaning_Logs.xlsx)

One sheet per source file, with the expected actions for that file. Each row records `# · Step · Column · Issue Found · Action Taken · Rows Affected · Tool · Status · Your Note`. Compare your manual log against the sheet, mark each row Done / Missed / N-A, and re-run cleaning for anything missed. If you find an issue the sheet doesn't list, raise it at the next team meeting.

Representative entries:

| Step | Column | Issue found | Action taken | Rows |
| --- | --- | --- | --- | --- |
| STEP 2 (Errors) | `status` | 16 variants (`Attended`, `ATTENDED`, `Showed`, `Came`, `Present`, `NOSHOW`, `no show`, `DNA`, `Missed`…) | Mapped to canonical 3: Attended / No-Show / Cancelled | 68,694 |
| STEP 2 (Errors) | `governorate_en` | ~44 variant spellings (`Alex`, `ALEXANDRIA`, `الاسكندرية`…) | Mapped to the canonical 27 governorate names | 28,000 |
| STEP 2 (Errors) | `staff_approval_status` | 14 boolean variants (`Yes`, `Y`, `1`, `True`, `نعم`…) | Mapped to True / False; unmapped → NULL | 192,540 |
| STEP 2 (Errors) | `phone` | Format variants (`+20`, `00 20`, dashes, spaces) | Stripped to 11-digit Egyptian format; invalid → NULL | 50,000 |
| STEP 2 (Errors) | `age_at_registration` | ~12 rows with age > 120 | Set to NULL | 12 |
| STEP 4 (Duplicates) | `patient_id` | 2,640 near-duplicates (same DOB + similar name) | Linked via `duplicate_of`; **both rows kept** | 2,640 |
| EXTRA 3 (Sanity) | age vs DOB | Calculated age must match recorded age | ~50 mismatches flagged | 50 |

**Derived columns added during cleaning:** `appt_year`, `appt_month`, `appt_weekday`, `appt_hour`, `is_attended`, `is_no_show`, `is_cancelled` (appointments) · `age_group`, `has_any_comorbidity` (patients) · `duration_days` (cases) · `completion_pct` (student case progress).

**2. Data Dictionary** *(TBD)* — Column · Type · Description · Allowed Values · Notes

**3. Before/After Report** — a short PDF with an executive summary (data quality score before vs. after), each issue with screenshots, cleaning methodology, validation results, and recommendations to prevent issues at source.

**4. Code Documentation** — module-level docstrings (purpose, author, date, runtime), function docstrings (purpose, inputs, outputs, examples), and inline comments for non-obvious logic.

### Validation Rule — the "Golden Rule"

Every DAX measure must be reconciled against a Python ground-truth calculation and agree within **±1%**. Where the two disagree, **Python is authoritative** and the DAX is treated as the defect. Analyses that don't compute reliably in DAX are delivered as Python visuals or pre-computed CSVs during model stabilization.

---

## 📈 KPIs & Insights

### 30 KPIs Across 4 Pillars

#### 🟢 Operational KPIs (8) — live clinic operations & throughput

1. Daily appointment volume (booked / attended / no-show)
2. Real-time queue depth
3. No-show rate (by clinic, day of week, appointment number)
4. Clinic utilization (% of daily capacity)
5. Average appointments per case
6. Staff coverage
7. Patient wait time
8. Referral rate

#### 🟡 Student KPIs (8) — progress, fairness & graduation readiness

1. Case completion percentage per student per year
2. At-risk flag (< 50% completion with < 3 months left)
3. Gini coefficient
4. Cases per student distribution
5. Case type breakdown
6. Ranking by completion (with keenness indicator)
7. Dropout / retention
8. Year-over-year cohort trend

#### 🟠 Patient KPIs (8) — care quality, demographics & outcomes

1. Demographic breakdown (age × gender × governorate)
2. Comorbidity prevalence
3. Treatment completion rate
4. Most common diagnoses
5. Patient journey time
6. Return / loyalty rate
7. Referral pathway analysis (Sankey)
8. Geographic reach

#### 🔵 Strategic KPIs (6) — what leadership needs to see every day

| # | KPI | Status |
| --- | --- | --- |
| 1 | Overall system health score | ✅ Built |
| 2 | Supply-demand ratio | 🚧 To be included |
| 3 | Bottleneck detection alerts | 🚧 To be included |
| 4 | Forecasted patient demand (7/30/90-day) | 🚧 To be included |
| 5 | Accreditation readiness | 🚧 To be included |
| 6 | Research output pipeline | 🚧 To be included |

### 6 Future Research Insights

| Insight | Question it answers |
| --- | --- |
| Patient No-Show Profiling | Who cancels? Age, distance, day of week, prior history — target reminders to high-risk patients. |
| Case Complexity vs Student Year | Do advanced cases route correctly to interns? Detect training gaps. |
| Geographic Access Equity | Which governorates are underserved? Should we add a satellite clinic? |
| Seasonal Demand Patterns | Spike after mid-year break? Ramadan dip? Enables capacity planning. |
| Comorbidity–Outcome Links | Do diabetic patients complete treatment at different rates? |
| Keenness Predictors | Which characteristics predict a high-performing student? |

---

## 🔍 Findings

### Answering the Four Research Questions

#### ⚖️ Fairness — *Does every student receive an equitable share of patients?*

- Distribution is **broadly fair** — per-year Gini sits below the 0.20 target.
- Highly overloaded clinics show **lower per-student fairness** — Fixed Prosthodontics & Pedodontics.
- **Verdict:** the allocation algorithm hits its Gini target. Continue current policy.

> ⚠️ Gini is cohort-sensitive: pooling all five academic years inflates the coefficient because it mixes cohorts at different stages. Always report Gini **per academic year**.

#### 🎓 Graduation — *Which students are at risk of not meeting requirements?*

- **37% fully on track**, 32% need attention, **31% at risk**.
- Bottleneck case types: **Complex Extraction, Simple Extraction, Single Crown** — combined shortfall of ≈4,400 cases.

#### ⚡ Efficiency — *Where are the bottlenecks?*

- No-show rate **10.2%** — stable across years. Drivers: distance, weekends, late appointment numbers.
- **All 7 treatment clinics exceed 100% utilization** (Fixed Prosthodontics peaks at 385%). The capacity definition needs review before this is reported externally.
- **Sunday is the system peak** (130–160% utilization).
- February 2026 shows a **10× volume spike** requiring validation.

#### 🩺 Patient Care — *Are we treating patients well?*

- **Comorbidity prevalence 37%** — chi-square shows a small but real link to outcomes.
- **Dropped cases rising +21.5%** over five years (4,284 in 2025–26), concentrated in complex pathways.
- **≈78% of patients complete** their treatment plan; ≈22% drop off.
- Removable Prosthodontics has the longest patient journey at **35+ days**.

### Operational Snapshot (latest reporting period)

| Metric | Value | Change |
| --- | --- | --- |
| Total appointments | 15,012 | ▼ 231 vs prior year (15,243) |
| Daily average appointments | 136 | across all clinics |
| No-show rate | 10.2% | ▲ from 9.64% |
| Referral rate | 11.1% | cross-clinic referrals |

A 10.2% no-show rate costs roughly **1,531 appointment slots a year** — the equivalent of losing one clinic's full weekly output every month.

### No-Show Rate by Clinic

| Clinic | No-Show % |
| --- | --- |
| Oral Surgery | 11.21% |
| Pedodontics | 10.90% |
| Operative | 10.72% |
| Endodontics | 9.66% |
| Fixed Prosthodontics | 9.47% |
| Periodontics | 9.43% |
| Removable Prosthodontics | 8.88% |

Female patients account for **57%** of no-shows, male patients **43%**.

### Treatment Efficiency by Clinic

| Clinic | Avg appointments per case | Avg days between appointments |
| --- | --- | --- |
| Removable Prosthodontics | 4.1 | 8.5 |
| Fixed Prosthodontics | 3.4 | 8.5 |
| Endodontics | 2.7 | 8.6 |
| Periodontics | 2.5 | 8.4 |
| Pedodontics | 1.6 | 4.3 |
| Operative | 1.5 | 4.3 |
| Oral Surgery | 1.2 | 2.3 |

Removable Prosthodontics is the most complex pathway: 4.1 appointments at 8.5-day intervals means each patient invests **35+ days** in treatment.

### Clinic Utilization Gap

| Clinic | Utilization |
| --- | --- |
| Fixed Prosthodontics | 385.58% |
| Periodontics | 118.06% |

A **3× gap** signals workload imbalance. Cross-training or expanded Periodontics hours could redistribute demand.

### Top Case Types by Volume

| Case type | Cases |
| --- | --- |
| Single Crown | 11,100 |
| Class I Restoration | 7,900 |
| Single RCT | 7,400 |
| 3-Unit Bridge | 6,600 |
| Scaling | 5,300 |
| Class II Restoration | 5,200 |
| Pulpectomy | 5,100 |
| Complete Denture | 5,100 |

### Patient Population Profile

| Dimension | Finding |
| --- | --- |
| **Comorbidities** | 37.15% of patients present with comorbidities (diabetes, hypertension, cardiac) — nearly a third require complex, integrated treatment plans. |
| **Diagnoses** | 209K+ diagnoses across 132K patients ≈ 1.6 diagnoses per patient. |
| **Geography** | Beheira, Alexandria and Gharbia form the primary catchment, with Beheira leading at 22K+ patients — evidence of regional-hub drawing power. |
| **Age** | Young Adults (20–39) and Adults (40–59) account for 73K+ patients combined; child and teen segments remain substantial. |
| **Gender** | 54.2% female / 45.8% male — a well-balanced, equitable-access distribution. |
| **Income** | 81K patients fall in the "No Income / Low Income" segment, underlining the mission of subsidized, accessible care. |

### System Health Score

A single composite score (0–100) summarizing allocation, completion, queue, and accreditation signals.

- **Current value: 68**
- **Target threshold: 75**
- Below target indicates the system is underperforming and leadership should drill into the supporting KPIs.

### Two Headlines for Leadership

1. **SD-ARCH works.** The fair-allocation algorithm hits its Gini target, the data model is sound, and five years of trends show steady growth in both patients (+21%) and appointments (+29%).
2. **Capacity is the constraint.** Every clinic operates above 100% utilization and only 37% of students are graduation-ready. Fixing this requires capacity expansion, KPI redefinition, and predictive intervention — not more data.

---

## 📐 Statistical Analysis

| Test | Question answered | Status / result |
| --- | --- | --- |
| **Gini Coefficient** | Is case distribution fair across students? | ✅ Complete — per-year Gini below the 0.20 target. Report per academic year, not pooled. |
| **Kolmogorov–Smirnov** | Are distributions different across years/clinics? | ✅ Complete — statistically significant difference detected. |
| **Logistic Regression** | What predicts a no-show? | ✅ Complete — AUC near chance. Treated as a legitimate negative finding: no-show flags in the synthetic data carry no learnable signal. Recommendation is **universal** SMS reminders rather than targeted ones. |
| **Kaplan–Meier** | How long do cases take to complete? | ✅ Complete — survival curves by clinic and keenness. |
| **Chi-Square** | Are comorbidities linked to outcomes? | ✅ Complete — small but statistically real association. |
| **Sankey / Markov Flow** | What are the patient referral patterns? | ✅ Complete — top pathways and bottlenecks mapped. |

Statistical procedures, thresholds, and cross-validation requirements are governed by the **SD-ARCH Statistical Analysis Guide**.

---

## 🤖 Predictive Models

*Phase to be considered — three models that move the platform from descriptive to predictive.*

| Model | Method | Predicts | Features | Use case | Target |
| --- | --- | --- | --- | --- | --- |
| **No-Show Prediction** | Classification · XGBoost | Will patient X attend appointment Y? | Age, distance from college, prior no-shows, day of week, appointment number, clinic, comorbidities | Targeted reminder SMS; overbook slots with 60%+ no-show probability | AUC ≈ 0.72 |
| **Patient Demand Forecast** | Time series · Prophet / SARIMA | How many patients will each clinic receive next week / month? | Historical daily volumes, day of week, academic phase, Ramadan, holidays | Capacity planning, staffing schedules, resource ordering | MAPE < 15% |
| **Graduation Risk** | Classification · Logistic Regression | Will this student complete all requirements by year-end? | Current completion %, keenness, GPA, year level, month of year | Mid-year intervention for at-risk students; preferential allocation | Precision > 0.80 on the at-risk class |

> ⚠️ On the current synthetic dataset, no-show outcomes are assigned randomly, so no model can beat chance on that target. These targets apply to real-data deployment.

---

## 📊 Power BI Dashboards

### Three Dashboards, Three Audiences

#### 🔷 Admin / Leadership Dashboard

- Daily appointment volume + 30-day trend
- Clinic utilization heatmap
- At-risk student count
- Demand forecast (30 days)
- Overall system health score (gauge, 0–100)
- Recent alerts panel

#### 🔶 Clinic Manager Dashboard

- Today's queue + no-shows
- Case allocation by student (top performers)
- Case completion progress by case type
- Pending staff approvals (action panel)
- Referrals in / out

#### 🔸 Student Dashboard

- My case progress (per-case-type completion)
- Completion percentage hero card
- Upcoming appointments
- Cohort ranking
- My no-show patients

### Implementation Notes

- Star schema over 13 source tables, mixed `.xlsx` and `.csv`.
- Python visuals used where DAX proves unreliable — K-S test, logistic regression, Kaplan–Meier, Sankey, chi-square, patient journey box plots.
- All fields feeding a Python visual must be set to **"Don't summarize"**; a 30,000-row de-duplication cap applies.
- Shared as a `.pbit` template with a parameterized folder path. Scheduled cloud refresh requires an On-premises Data Gateway; manual refresh is the interim.

---

## 🛠 Known Data Issues & Improvement Backlog

Four areas where SD-ARCH should evolve next, ranked by leadership impact.

### Data Model

- [ ] Split pipe-separated `case_type` columns into rows for proper filtering.
- [ ] Fix `is_active_clinic_day` — currently `True` for weekends and holidays. **Until fixed, filter on the calendar `status` column directly.**
- [ ] Add a composite `student_year_key` to enable correct year-aware filtering.
- [ ] Consolidate the three overlapping appointment tables.

### KPI Definitions

- [ ] Redefine `daily_capacity` as realistic throughput, not chair count — this is what drives the >100% utilization figures.
- [ ] Reconcile every DAX measure with Python ground truth within ±1%.
- [ ] Document each KPI with formula, source table, and validator name.
- [ ] Align the Patient Journey Time definition — the guide specifies registration date → final appointment; the current implementation uses first appointment → final appointment.
- [ ] Rename ambiguous measures (e.g. "Completion %", which measures *student case* completion, not *appointment* completion) before other team members build on them.

### New Capabilities

- [ ] Kaplan–Meier survival page for case duration by clinic.
- [ ] Sankey / Markov page for patient flow between clinics.
- [ ] Per-appointment no-show risk score to drive SMS reminders.

### Operational Actions

- [ ] Investigate the Sunday peak — redistribute appointments to Mon–Thu.
- [ ] Validate the February 2026 spike before reporting 2025–26 numbers externally.
- [ ] Expand Pedodontics capacity to match observed pediatric demand.
- [ ] Implement automated SMS/WhatsApp reminders 48h and 2h before appointments, prioritizing Oral Surgery and Pedodontics.
- [ ] Address the 3× utilization gap between Fixed Prosthodontics and Periodontics.
- [ ] Build structured treatment pathways to reduce inter-visit gaps from 8.5 days to under 6.

### Engineering Notes

- `ISBLANK()` fails on CSV-loaded data — empty cells load as empty strings; use `<> ""`.
- `EARLIER` fails inside `ADDCOLUMNS` of a calculated table; use the `VAR ThisCount` pattern.
- Parameterizing the top-level folder source does **not** remove hardcoded paths from individual query steps — those must be corrected in the formula bar.

---

## 🌐 Web Application

> **General overview** — implementation details TBD.

The SD-ARCH platform also includes a **web application** providing role-based access for the same three audiences (Admin, Clinic Manager, Student) plus a **Patient app** for booking appointments online.

### High-Level Capabilities

- 🔐 **Authentication** — separate login for each audience
- 📱 **Patient booking flow** — Arabic-first, mobile-responsive
- 📊 **Live dashboards** — the same KPIs as Power BI, but interactive and drill-downable
- 🤖 **Model integration** — exposes predictive model outputs (no-show probability, demand forecast)
- ✅ **Workflow tools** — staff approval flow for case allocations
- 🌍 **Bilingual** — full Arabic / English support

### Tech Stack

The frontend and backend are built and maintained by the development team. The web app consumes the same SQL Server database that powers the Power BI dashboards.

> 📌 *Detailed technical documentation, API spec, deployment guide, and frontend repository will be added in a future update.*

---

## 📁 Repository Structure

*(will be added in a future update)*

```
sdarch/
│
├── 📊 data/                                  Synthetic dataset (13 CSV files)
│   ├── patients.csv                          134,640 patient records
│   ├── students.csv                          2,718 students
│   ├── staff.csv                             184 teaching staff
│   ├── cases.csv                             209,065 allocated cases
│   ├── appointments_diagnosis.csv            126,444 diagnosis visits
│   ├── appointments_treatment.csv            397,691 treatment visits
│   ├── clinics.csv                           9 clinical departments
│   ├── governorates.csv                      27 Egyptian governorates
│   ├── case_requirements.csv                 51 requirements per year level
│   ├── academic_calendar.csv                 1,366-day 5-year calendar
│   ├── student_enrollments.csv               5,759 yearly enrollment records
│   ├── student_keenness.csv                  2,718 per-student willingness
│   └── student_case_progress.csv             96,786 running completion tallies
│
├── 🗄️ sql_server/                            Microsoft SQL Server setup
│   ├── 01_create_schema.sql                  T-SQL schema + indexes
│   ├── 02_load_data_bulk.sql                 BULK INSERT loader
│   ├── 03_create_views.sql                   Analytics views + functions
│   ├── load_to_sqlserver.py                  Python pyodbc loader
│   └── README.md                             SQL Server setup guide
│
├── 📽️ presentation/
│   ├── SD-ARCH_Project_Presentation.pptx     43-slide project deck
│   └── build_slides.js                       pptxgenjs build script
│
├── 📈 powerbi/
│   ├── SD-ARCH.pbit                          Parameterized Power BI template
│   └── python_visuals/                       Scripts for K-S, K-M, Sankey, journey plots
│
├── 🛠️ scripts/
│   └── add_utf8_bom.py                       Adds UTF-8 BOM to all CSVs
│
├── 📋 DATA_GENERATION_PROMPT.md              Self-contained spec to regenerate dataset
├── 📜 LICENSE                                MIT
└── 📖 README.md                              ← You are here
```

---

## 👥 Team

| Member | Role | Owns |
| --- | --- | --- |
| **Mahmoud** | Team Lead · Analytics | Project operations, Python ground truth for cross-validation · cleans `appointments_treatment`, `cases` |
| **Loay** | Senior Data Engineer / Validator | Power BI validation, escalation resource · cleans `student_case_progress`, `student_enrollments` |
| **Marina** | Data Modeling | Power BI data model |
| **Mohamed Ali** | Full-Stack Developer (Web application) | Schema and model support · cleans `patients`, `appointments_diagnosis` 
| **Aya** | Analyst | Analysis and reporting · cleans `students`, `student_keenness`, `staff`, `clinics` |
| **Kholod** | Analyst | Analysis and reporting · cleans `academic_calendar`, `case_requirements`, `governorates` |


**Working pattern:** Mahmoud produces the Python ground truth → Loay validates it in Power BI. This producer/validator split is formalized in the SD-ARCH Statistical Analysis Guide.

---

## 🗺️ Roadmap

### ✅ Completed

- [x] Synthetic dataset generation (5 years, 13 tables)
- [x] SQL Server schema + ETL pipeline
- [x] Data dictionary (all 13 tables)
- [x] Cleaning methodology documented
- [x] Repository setup with documentation
- [x] 43-slide project presentation
- [x] Star-schema Power BI data model
- [x] Statistical analysis suite (Gini, K-S, logistic regression, Kaplan–Meier, chi-square, Sankey/Markov)
- [x] Executive summary document with embedded results
- [x] All four research questions answered with evidence

### 🚧 In Progress

- [ ] Cleaning pipeline — 10 of 13 files done; `academic_calendar`, `case_requirements`, `governorates` remaining
- [ ] All 30 KPIs computed and validated against Python within ±1%
- [ ] Remaining 5 strategic KPIs (supply-demand, bottleneck alerts, demand forecast, accreditation readiness, research pipeline)
- [ ] 3 role-based Power BI dashboards finalized
- [ ] Predictive models
- [ ] Web app development
- [ ] `.pbit` template distribution + scheduled refresh via data gateway

### 🔮 Future

- [ ] Real-data deployment at partner dental colleges
- [ ] Multi-language support (English / Arabic UI throughout)
- [ ] Mobile native apps (iOS / Android)
- [ ] Integration with Egyptian Ministry of Health systems
- [ ] Public research dataset (k-anonymized)

---

## 📜 License

This project is licensed under the **MIT License** — see [`LICENSE`](LICENSE) for details.

> ⚠️ **All data in this repository is synthetic.** No real patient records are included. Synthetic names, phone numbers, governorates, and medical histories are generated procedurally for testing and analytical exploration purposes only.

For questions or collaboration inquiries:

- Open an [Issue](https://github.com/Ambitious26/UD-CMS-Mega-Pioneer/issues)
- Contact the team lead

---

**Built with care by the SD-ARCH team · 2026**

[⬆ Back to top](#-sd-arch)
