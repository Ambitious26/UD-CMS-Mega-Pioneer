<div align="center">

# 🦷 SD-ARCH

### Smart Dental Allocation & Research Hub

A unified data platform for Egyptian university dental clinics — fair patient-student allocation, clinical operations analytics, and research-ready datasets.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Data: Synthetic](https://img.shields.io/badge/Data-Synthetic-blue.svg)](#dataset)
[![Status: Active](https://img.shields.io/badge/Status-Active-success.svg)]()
[![SQL Server](https://img.shields.io/badge/Database-SQL_Server-red.svg)]()
[![Power BI](https://img.shields.io/badge/Dashboards-Power_BI-yellow.svg)]()

</div>

---

## 📋 Table of Contents

1. [About the Project](#-about-the-project)
2. [Problem We Solve](#-problem-we-solve)
3. [Dataset](#-dataset)
4. [Data Model](#-data-model)
5. [Data Pipeline](#-data-pipeline)
6. [Cleaning Methodology](#-cleaning-methodology)
7. [KPIs & Insights](#-kpis--insights)
8. [Statistical Analysis](#-statistical-analysis)
9. [Predictive Models](#-predictive-models)
10. [Power BI Dashboards](#-power-bi-dashboards)
11. [Web Application](#-web-application)
12. [Repository Structure](#-repository-structure)
13. [Quick Start](#-quick-start)
14. [Team](#-team)
15. [Roadmap](#-roadmap)
16. [License](#-license)

---

## 🎯 About the Project

**SD-ARCH** is an end-to-end analytics and operations platform for dental colleges. It addresses four interconnected problems:

| Problem | Our Solution |
|---|---|
| Unfair case distribution among students | Fair-distribution algorithm (Gini = 0.18) |
| Manual allocation by receptionists | Automated 2-phase algorithm with audit trail |
| No leadership visibility | Three role-based Power BI dashboards |
| Data locked in paper/Excel | SQL Server database + research-ready exports |

The project includes:
- ✅ A **synthetic dataset** of 5 academic years (2021–2026)
- ✅ **SQL Server schema + ETL pipeline**
- ✅ **24 KPIs** across operational, student, patient, and strategic dimensions
- ✅ **Statistical analyses** (fairness, no-show drivers, completion survival)
- ✅ **3 predictive models** (no-show prediction, demand forecast, graduation risk)
- ✅ **3 Power BI dashboards** (Admin, Clinic Manager, Student)
- ✅ **A web application** (frontend + backend) .

> **Note:** All data in this repository is **synthetic**. No real patient records are included.

---

## 🚨 Problem We Solve

In Egyptian university dental clinics today:

> *Some students graduate with 40+ cases while others struggle to find 5. Receptionists match patients to students by hand, relying on memory and goodwill. Leadership has no live view of clinic utilization, student progress, or patient flow. Rich clinical data is fragmented across notebooks and Excel sheets — unusable for research, quality improvement, or accreditation.*

SD-ARCH replaces this with a transparent, data-driven system.

---

## 📊 Dataset

### At-a-Glance Statistics

| Metric | Value |
TBD |

### Generation Approach

- **Reproducible:** Fixed random seed (42) — byte-for-byte identical on re-runs
- **Realistic:** Egyptian Sun–Thu work week, Ramadan reduced hours, exam breaks, public holidays
- **Demographically grounded:** Age pyramid, gender, governorate distribution match real population data
- **Medically grounded:** Diabetes/hypertension/smoking prevalence by age and gender match Egyptian health statistics
- **Bilingual:** Every name has paired Arabic + English forms
- **UTF-8 BOM encoding:** Arabic displays correctly in Excel, Power BI, and SQL Server

📄 **Full specification:** [`DATA_GENERATION_PROMPT.md`](DATA_GENERATION_PROMPT.md) — hand this to any LLM to regenerate the entire dataset.

---

## 🗄️ Data Model

### 13 Interconnected Tables

```
┌─────────────────┐         ┌──────────────────┐
│   REFERENCE     │         │    ENTITY        │
├─────────────────┤         ├──────────────────┤
│ clinics         │         │ patients         │
│ governorates    │         │ students         │
│ case_requirements│        │ student_enrollments│
│ academic_calendar│        │ student_keenness │
└─────────────────┘         │ staff            │
                            └──────────────────┘
                                     │
                                     ▼
                            ┌──────────────────────┐
                            │   ACTIVITY (FACT)    │
                            ├──────────────────────┤
                            │ appointments_diagnosis│
                            │ cases                │
                            │ appointments_treatment│
                            │ student_case_progress│
                            └──────────────────────┘
```

### Table Catalog

| # | Table | Type | Rows | Purpose |
|---|---|---|---|---|
| 1 | `clinics` | Reference | 9 | 9 clinical departments + intern clinic |
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
        │      cases (FACT)            │
        └──────────────┬───────────────┘
                       │
                       ▼
        ┌──────────────────────────────┐
        │  appointments_treatment       │
        │           (FACT)              │
        └──────────────────────────────┘
```

---

## 🔄 Data Pipeline

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Raw CSVs   │ -> │  Cleaning    │ -> │  SQL Server  │ -> │  Analytics   │
│  (13 files)  │    │   Pipeline   │    │   Database   │    │  Layer       │
│              │    │              │    │              │    │ (Views+KPIs) │
└──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                    │
                                          ┌─────────────────────────┼────────────────┐
                                          ▼                         ▼                ▼
                                  ┌──────────────┐        ┌──────────────┐  ┌──────────────┐
                                  │   Power BI   │        │   Web App    │  │  Predictive  │
                                  │  Dashboards  │        │  (3 audiences)│  │   Models     │
                                  └──────────────┘        └──────────────┘  └──────────────┘
```

### Tools Stack

| Stage | Primary Tool | Alternative |
|---|---|---|
| Raw data inspection | Excel | Power Query |
| Cleaning | Python (pandas) | Power Query |
| Storage | Microsoft SQL Server | PostgreSQL |
| KPI computation | SQL + Python | DAX in Power BI |
| Statistical analysis | Python (scipy, lifelines) | R |
| Predictive modeling | Python (XGBoost, Prophet) | scikit-learn |
| Dashboards | Power BI | Tableau |
| Web app | 

---

## 🧹 Cleaning Methodology

We will follow the cleaning steps for every table

### Cleaning Tools 

| Tool | Best for | Code? |
|---|---|---|
| **Excel** | Small files, manual checks, lookup sheets 
| **Power Query** | Visual cleaning in Excel/Power BI 
| **Python (pandas)** | Large files, batch automation, statistics 
| **Power BI** | Live dashboards on cleaned data 
---

## 📚 Documentation Standards

Every cleaning task produces 4 deliverables :

### 1. Data Cleaning Log (TBD)

| # | Step | Column | Issue Found | Action Taken | Rows Affected | Tool |
|---|---|---|---|---|---|---|
| 1 | Missing |
| 2 | Inconsistent | 
| 3 | Outliers |
### 2. Data Dictionary (TBD)

| Column | Type | Description | Allowed Values | Notes |


### 3. Before/After Report

A short PDF with:
- Executive summary (data quality score before vs. after)
- Each issue found with screenshots
- Cleaning methodology
- Validation results
- Recommendations to prevent issues at source

### 4. Code Documentation

All Python scripts use:
- Module-level docstring (purpose, author, date, runtime)
- Function docstrings (purpose, inputs, outputs, examples)
- Inline comments for non-obvious logic

---

## 📈 KPIs & Insights

### 24 KPIs Across 4 Pillars

#### 🟢 Operational KPIs (8)
1. Daily appointment volume (booked / attended / no-show)
2. Real-time queue depth
3. No-show rate (by clinic, day of week, appointment number)
4. Clinic utilization (% of daily capacity)
5. Average appointments per case
6. Staff coverage
7. Patient wait time
8. Referral rate

#### 🟡 Student KPIs (8)
1. Completion percentage per student per year
2. At-risk flag (< 50% with < 3 months left)
3. Gini coefficient (display)
4. Cases per student histogram
5. Case type breakdown
6. Ranking by completion
7. Dropout / retention
8. Year-over-year trend

#### 🟠 Patient KPIs (8)
1. Demographic breakdown (age × gender × governorate)
2. Comorbidity prevalence
3. Treatment completion rate
4. Most common diagnoses
5. Patient journey time
6. Return / loyalty rate
7. Referral pathway analysis
8. Geographic reach

#### 🔵 Strategic KPIs (6)
1. Overall system health score
2. Supply-demand ratio
3. Bottleneck detection alerts
4. Forecasted patient demand (7/30/90-day)
5. Accreditation readiness
6. Research output pipeline

### 6 Research Insights
- Patient No-Show Profiling
- Case Complexity vs Student Year
- Geographic Access Equity
- Seasonal Demand Patterns
- Comorbidity-Outcome Links
- Keenness Predictors

---

## 📐 Statistical Analysis

| Test | Question Answered | Expected Result |
|---|---|---|
| **Gini Coefficient** | Is case distribution fair across students? | 0 = perfect fairness, 1 = max inequality. **Our: 0.18** ✅ |
| **Kolmogorov–Smirnov** | Are distributions different across years/clinics? | p > 0.05 = no significant difference |
| **Logistic Regression** | What predicts a no-show? | AUC > 0.70 |
| **Kaplan–Meier** | How long do cases take to complete? | Survival curves by clinic, keenness |
| **Chi-Square** | Are comorbidities linked to outcomes? | Detects statistical associations |
| **Sankey/Markov Flow** | What are the patient referral patterns? | Top pathways + bottlenecks |

---

## 🤖 Predictive Models



**Performance targets:**
- No-show: AUC > 0.70
- Demand: MAPE < 15%
- Graduation: Precision > 0.80 on at-risk class

---

## 📊 Power BI Dashboards

### Three Dashboards, Three Audiences

#### 🔷 Admin / Leadership Dashboard
- Daily appointment volume + 30-day trend
- Clinic utilization heatmap
- At-risk student count
- Demand forecast (30 days)
- Overall system health score
- Recent alerts panel

#### 🔶 Clinic Manager Dashboard
- Today's queue + no-shows
- Case allocation by student (top performers)
- Case completion progress by case type
- Pending staff approvals (action panel)
- Referrals in/out

#### 🔸 Student Dashboard
- My case progress (per-case-type completion)
- Completion percentage hero card
- Upcoming appointments
- Cohort ranking
- My no-show patients


---

## 🌐 Web Application

> **General overview** — implementation details TBD.

The SD-ARCH platform also includes a **web application** providing role-based access for the same three audiences (Admin, Clinic Manager, Student) plus a **Patient app** for booking appointments online.

### High-Level Capabilities

- 🔐 **Authentication** — separate login for each audience
- 📱 **Patient booking flow** — Arabic-first, mobile-responsive (similar to existing Egyptian university hospital apps)
- 📊 **Live dashboards** — same KPIs as Power BI, but interactive and drill-downable
- 🤖 **Model integration** — exposes predictive model outputs (no-show probability, demand forecast)
- ✅ **Workflow tools** — staff approval flow for case allocations

### Tech Stack

The frontend and backend are built and maintained by the development team. The web app consumes the same SQL Server database that powers the Power BI dashboards.

> 📌 *Detailed technical documentation, API spec, deployment guide, and frontend repository will be added in a future update.*

---

## 📁 Repository Structure(will be added in a future update)

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
│   ├── case_requirements.csv                 Requirements per year level
│   ├── academic_calendar.csv                 5-year calendar
│   ├── student_enrollments.csv               Yearly enrollment state
│   ├── student_keenness.csv                  Per-student willingness
│   └── student_case_progress.csv             Running completion tallies
│
├── 🗄️ sql_server/                            Microsoft SQL Server setup
│   ├── 01_create_schema.sql                  T-SQL schema + indexes
│   ├── 02_load_data_bulk.sql                 BULK INSERT loader
│   ├── 03_create_views.sql                   Analytics views + functions
│   ├── load_to_sqlserver.py                  Python pyodbc loader
│   └── README.md                             SQL Server setup guide
│
├── 📽️ presentation/
│   ├── SD-ARCH_Project_Presentation.pptx     23-slide project deck
│   └── build_slides.js                       pptxgenjs build script
│
├── 🛠️ scripts/
│   └── add_utf8_bom.py                       Adds UTF-8 BOM to all CSVs
│
├── 📋 DATA_GENERATION_PROMPT.md              Self-contained spec to regenerate dataset
├── 📜 LICENSE                                MIT
└── 📖 README.md                              ← You are here
```


## 👥 Team

| Member | Role | Owns |
|---|---|---|
| **Mahmoud** | Team Lead 
| **Loay** |
| **Mohamed Ali** |
| **Aya** | 
| **Kholod** | 
| **Marina** | 


---

## 🗺️ Roadmap

### ✅ Completed
- [x] Synthetic dataset generation (5 years, 13 tables)
- [x] SQL Server schema + ETL pipeline
- [x] Data dictionary (all 13 tables)
- [x] 23-slide project presentation
- [x] Cleaning methodology documented
- [x] Repository setup with documentation

### 🚧 In Progress
- [ ] Cleaning pipeline implementation (parallel team work)
- [ ] 24 KPIs computed and validated
- [ ] Statistical analysis report (Gini, K-S, K-M, chi-square)
- [ ] 3 trained predictive models
- [ ] 3 Power BI dashboards
- [ ] Web app development

### 🔮 Future
- [ ] Real-data deployment at partner dental colleges
- [ ] Multi-language support (English / Arabic UI throughout)
- [ ] Mobile native apps (iOS / Android)
- [ ] Integration with Egyptian Ministry of Health systems
- [ ] Public research dataset (k-anonymized) IF THERE IS TIME TO DO IT

---

## 📜 License

This project is licensed under the **MIT License** — see [`LICENSE`](LICENSE) for details.

> ⚠️ **All data in this repository is synthetic.** No real patient records are included. Synthetic names, phone numbers, governorates, and medical histories are generated procedurally for testing and analytical exploration purposes only.


For questions or collaboration inquiries:
- Open an [Issue](https://github.com/[your-username]/sdarch/issues)
- Contact the team lead


---

<div align="center">

**Built with care by the SD-ARCH team · 2026**

[⬆ Back to top](#-sd-arch)

</div>
