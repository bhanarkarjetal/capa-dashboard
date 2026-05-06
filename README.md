# CAPA Dashboard — MedFluss GmbH

A CAPA (Corrective and Preventive Action) performance dashboard built for a fictional ISO 13485:2016certified medical device company. The project uses real FDA recall data as the nonconformity source, with a synthetic CAPA layer built on top to simulate an internal quality management process.

---

## Company Background

**MedFluss GmbH** is a fictional medical device manufacturer headquartered in Hamburg, Germany, operating under ISO 13485. The company markets infusion pump products in the EU and the USA through its US subsidiary,making FDA Device Enforcement data a realistic nonconformity source for their CAPA system.

---
## Dashboard Overview

The dashboard is designed to answer the questions a Quality Manager or Regulatory Affairs team would typically track:

- How many CAPAs are open, closed, and overdue?
- Which departments carry the highest CAPA burden?
- What root causes are driving nonconformities?
- Are corrective actions being closed effectively?
- Is the volume of new CAPAs trending upward?

---

## Data Sources

### FDA Device Enforcement Records

- **Source:** openFDA Device Enforcement endpoint (`open.fda.gov/apis/device/enforcement`)
- **Filter:** Infusion pump recalls only
- **Raw file:** 1.4 GB JSON - filtered to 666 records × 27 columns
- **After cleaning:** 529 records (137 "Other" root cause records dropped)
- **Fields used:** recall ID, recalling firm, product description, reason for recall, root cause, recall status, dates

Download `infusion_pump_recalls.csv` from `data\` folder OR run `data_extraction.ipynb` if starting from the raw openFDA JSON.

### Synthetic CAPA Layer

529 CAPA records generated on top of the recall data to simulate an ISO 13485 CAPA process. Each record maps to one FDA recall event.

| Field | Description |
|-------|-------------|
| `capa_id` | Unique CAPA identifier (CAPA-MF-1000 onwards) |
| `company` | MedFluss GmbH |
| `department` | Assigned based on root cause |
| `assigned_owner` | Named owner (2 per department) |
| `severity` | Critical / Major / Minor - risk-based weighted probabilities |
| `capa_status` | Open / Closed - derived from FDA recall status |
| `open_date` | Synthetic: Closed CAPAs 2022–2025, Open CAPAs 2025–2026 |
| `close_date` | Synthetic: open_date + random days based on severity |
| `days_open` | Calculated from open and close dates |
| `overdue` | Yes/No - Critical >30d, Major >90d, Minor >180d |
| `effectiveness_result` | Closed CAPAs only: Effective / Partially Effective / Not Effective |

**Severity distribution:**
Critical 43 (8.1%) | Major 325 (61.4%) | Minor 161 (30.4%)
*Distribution reflects a functional QMS — a high proportion of Critical CAPAs would indicate a systemic regulatory issue.*

---

## Root Cause Reclassification

The FDA root cause labels in the source data are inconsistently applied. Each category was reviewed against the actual nonconformity text and reassigned where the original classification did not accurately reflect the failure:

| FDA Original | Reclassified To | Reason |
|---|---|---|
| Component design/selection | Device Design | Component selection is a design decision, not a supplier issue |
| Mixed-up of materials/components | Manufacturing | Mix-ups occur on the production floor |
| Labeling mix-ups | Manufacturing | Execution error, not a labeling design problem |
| Use error | Device Design | Use errors are addressed through human factors engineering |
| Software in the Use Environment | Device Design | Software usability is a design problem |
| Software Manufacturing/Software Deployment | Software Design | Actual failures were software logic issues, not production line issues |
| Process Change Control | Quality Assurance | QA owns change control as a QMS function |

The "Other" category (137 records, 20.6%) was dropped because they were too vague to reclassify meaningfully.

---

## Final Data Distribution

| Root Cause | Count | % |
|---|---|---|
| Device Design | 217 | 41.0% |
| Process Control | 98 | 18.5% |
| Nonconforming Material | 93 | 17.6% |
| Software Design | 77 | 14.6% |
| Assembly Error | 14 | 2.6% |
| Human Error — Production | 12 | 2.3% |
| Release Without Testing | 6 | 1.1% |
| Labeling Design Error | 4 | 0.8% |
| Documentation Error | 3 | 0.6% |
| Under Investigation | 3 | 0.6% |

**CAPA status:** Closed 417 (78.8%) | Open 112 (21.2%)
**Overdue (open CAPAs):** 62 CAPAs | Avg days open: 96 days
**Effectiveness (closed CAPAs):** Effective 68.8% | Partially Effective 19.4% | Not Effective 11.8%

---


| Tool | Purpose |
|---|---|
| Python (pandas) | Data cleaning, reclassification, synthetic data generation |
| PostgreSQL | Data storage and KPI queries |
| SQLAlchemy + psycopg2 | Python–PostgreSQL connection |
| Tableau Public | Dashboard visualisation |

---

## Dashboard Structure

**Page 1 — Executive Overview**
Total CAPAs, Open, Closed, Overdue KPIs, Status breakdown, Severity breakdown

**Page 2 — Root Cause & Department Analysis**
Root cause distribution, Department totals, Department breakdown by severity

**Page 3 — Overdue & Open CAPA Tracker**
Overdue by department, Overdue by severity, Avg days open

**Page 4 — Effectiveness & Closure Trends**
Effectiveness breakdown, CAPAs opened per year (2022–2026)

**Live dashboard:** https://public.tableau.com/app/profile/jetal.bhanarkar/viz/CAPA-dashboard/SeverityBreakdown

---

## Repository Structure

capa-dashboard/
├── data_extraction.ipynb            # Filter raw openFDA JSON for infusion pump records
├── data_cleaning.ipynb              # Data cleaning and root cause reclassification
├── create_synthetic_data.ipynb      # Synthetic CAPA record generation
├── sqlalchemy_file.ipynb            # PostgreSQL connection and KPI queries
├── data/
│   ├── infusion_pump_recalls.csv    # Filtered FDA data (666 records)
│   ├── recall_df_clean.csv          # Cleaned FDA data (529 records)
│   └── capa_medfluss.csv            # Final synthetic CAPA dataset (529 records)
└── requirements.txt

---

## How to Run

1. Install dependencies: `pip install -r requirements.txt`
2. Set up PostgreSQL and create a database named `medfluss`
3. Update `DB_USER` in `sqlalchemy_file.ipynb` to your PostgreSQL username
4. Run notebooks in order:
   - `data_extraction.ipynb` - optional, only if starting from the raw JSON
   - `data_cleaning.ipynb` -> `create_synthetic_data.ipynb` -> `sqlalchemy_file.ipynb`
5. Connect Tableau Public to `data/capa_medfluss.csv`

---

**Note:** 2026 figures cover January–April only. The upward trend from 2022–2025 should not be extrapolated from the partial 2026 data.