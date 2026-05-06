# CAPA Dashboard — MedFluss GmbH

A Corrective and Preventive Action (CAPA) dashboard for a fictional ISO 13485-certified medical device company, built on real FDA recall data with a synthetic CAPA layer.

---

## Company Background

**MedFluss GmbH** is a fictional medical device manufacturer headquartered in Germany. The company operates under ISO 13485 and markets infusion pump products in the EU (primary) and the USA through its US subsidiary. The US subsidiary is subject to FDA regulation, making FDA recall data a realistic source for nonconformity data in their CAPA system.

---

## Problem Statement

A functioning QMS requires visibility into CAPA performance across departments and severity levels. This dashboard answers the key questions a Quality Manager or Regulatory Affairs team would ask:

- How many CAPAs are open, closed, and overdue?
- Which departments are carrying the highest CAPA burden?
- What are the most frequent root causes driving nonconformities?
- Are CAPAs being closed effectively?
- Is the volume of new CAPAs increasing over time?

---

## Data Sources

### Real Data Layer — FDA Device Enforcement Records

- **Source:** openFDA Device Enforcement endpoint (`open.fda.gov/apis/device/enforcement`)
- **Filter:** Infusion pump recalls only
- **Raw file:** 1.4 GB JSON → filtered to 666 records × 27 columns
- **After cleaning:** 529 records (137 "Other" root cause records dropped)
- **Fields used:** recall ID, recalling firm, product description, reason for recall, root cause, recall status, dates

### Synthetic CAPA Layer

529 CAPA records were generated on top of the recall data to simulate an internal ISO 13485 CAPA process. Each CAPA record maps to one FDA recall event.

| Field | Description |
|---|---|
| `capa_id` | Unique CAPA identifier (CAPA-MF-1000 onwards) |
| `company` | MedFluss GmbH |
| `department` | Assigned department based on root cause |
| `assigned_owner` | Named owner (2 per department, realistic German/international names) |
| `severity` | Critical / Major / Minor — risk-based weighted probabilities |
| `capa_status` | Open / Closed — derived from FDA recall status |
| `open_date` | Synthetic: Closed CAPAs 2022–2025, Open CAPAs 2025–2026 |
| `close_date` | Synthetic: `open_date` + random days based on severity |
| `days_open` | Calculated from open and close dates |
| `overdue` | Yes/No — thresholds: Critical >30d, Major >90d, Minor >180d |
| `effectiveness_result` | Closed CAPAs only: Effective / Partially Effective / Not Effective |

**Severity distribution:**  
Critical 43 (8.1%) · Major 325 (61.4%) · Minor 161 (30.4%)  
*Rationale: Reflects a functional QMS — too many Critical CAPAs would signal a systemic regulatory problem.*

---

## Root Cause Reclassification

FDA root cause labels are inconsistently applied in the source data. Each record's nonconformity text was reviewed and root causes were reassigned based on domain knowledge. Key corrections:

| FDA Original | Reclassified To | Reason |
|---|---|---|
| Component design/selection | Device Design | Component selection is a design decision (R&D), not a supplier issue |
| Mixed-up of materials/components | Manufacturing | Mix-ups occur on the production floor |
| Labeling mix-ups | Manufacturing | Execution error, not a labeling design problem |
| Use error | Device Design | Use errors are addressed through human factors engineering |
| Software in the Use Environment | Device Design | Software usability is a design/human factors problem |
| Software Manufacturing/Software Deployment | Software Design | Actual nonconformities were software logic failures, not production line issues |
| Process Change Control | Quality Assurance | QA owns change control as a QMS function |

The "Other" category (137 records, 20.6%) was dropped entirely — too vague to classify meaningfully.

---

## Final Data Distribution

**Root cause breakdown (529 records):**

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

**CAPA status:** Closed 417 (78.8%) · Open 112 (21.2%)  
**Overdue (open CAPAs only):** 62 CAPAs · Avg days open: 96 days  
**Effectiveness (closed CAPAs only):** Effective 68.8% · Partially Effective 19.4% · Not Effective 11.8%

---

## Tool Stack

| Tool | Purpose |
|---|---|
| Python (pandas) | Data cleaning, root cause reclassification, synthetic data generation |
| PostgreSQL | Data storage and KPI queries |
| SQLAlchemy + psycopg2 | Python–PostgreSQL connection |
| Tableau Public | Dashboard visualisation |

---

## Dashboard Structure

**Page 1 — Executive Overview**  
Total CAPAs · Open · Closed · Overdue KPIs · Status breakdown (pie) · Severity breakdown (bar)

**Page 2 — Root Cause & Department Analysis**  
Root cause distribution · Department totals · Department breakdown by severity (stacked bar)

**Page 3 — Overdue & Open CAPA Tracker**  
Overdue by department · Overdue by severity · Avg days open KPI

**Page 4 — Effectiveness & Closure Trends**  
Effectiveness breakdown (closed CAPAs) · CAPAs opened per year (2022–2026)

**Live dashboard:** https://public.tableau.com/app/profile/jetal.bhanarkar/viz/CAPA-dashboard/SeverityBreakdown

---

## Repository Structure

```
capa-dashboard/
├── data_cleaning.ipynb              # Data cleaning and root cause reclassification
├── create_synthetic_data.ipynb      # Synthetic CAPA record generation
├── sqlalchemy_file.ipynb            # PostgreSQL connection and KPI queries
├── infusion_pump_recalls.csv        # Raw FDA recall data (666 records)
├── recall_df_clean.csv              # Cleaned FDA data (529 records)
├── capa_medfluss.csv                # Final synthetic CAPA dataset (529 records)
└── requirements.txt                 # Python dependencies
```

---

## How to Run

1. Install dependencies:
   ```
   pip install -r requirements.txt
   ```

2. Set up PostgreSQL and create a database named `medfluss`

3. Run notebooks in order:
   - `data_cleaning.ipynb` — generates `recall_df_clean.csv`
   - `create_synthetic_data.ipynb` — generates `capa_medfluss.csv`
   - `sqlalchemy_file.ipynb` — loads data into PostgreSQL and runs KPI queries

4. Connect Tableau Public to `capa_medfluss.csv` (Text file connection)

---

## Notes

- **2026 data is a partial year** — 50 CAPAs opened Jan–Apr 2026. The upward trend seen from 2022–2025 should not be extrapolated from the 2026 figure.
- The fictional MedFluss GmbH context provides a realistic framing for a German medtech company under ISO 13485, where FDA data is sourced from their US subsidiary.
