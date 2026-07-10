# Insight: CMS Part B Compliance Analytics Engine

> **Portfolio Navigation:**  
> * **Portfolio Summary:** [Overview of Portfolio](./README.md)
> * **Moderation & Privacy Infrastructure:** [Moderation & Privacy Infrastructure](mod-privacy.md)  
> * **AI-Driven CSR Platform:** [AI-Driven CSR Platform](ai-csr.md)  
> * **Autonomous Community Operations RPA:** [Autonomous Community Operations RPA](auto-ops-rpa.md)
> * **CMS Data Engineering Pipeline:** [CMS Data Engineering Pipeline](cms-de-pipe.md)
> * **Insight: CMS Part B Compliance Analytics Engine (Here):** [Insight: Compliance Analytics Engine](insight.md)
> * **Portfolio Summary & Operational Scope** [Community Scope & Closure](./scale-and-scope.md)

---

## 📄 Summary

Healthcare organizations generate massive volumes of reimbursement data, yet extracting meaningful operational insight from publicly available CMS datasets remains technically complex. Raw Medicare Part B data spans millions of records across fragmented spreadsheets, inconsistent schemas, and varying provider classifications, making scalable compliance analysis difficult using traditional spreadsheet-based workflows.

To address this challenge, I designed and developed Insight, a healthcare compliance analytics platform that automates the complete analytical lifecycle — from raw CMS data ingestion through dimensional modeling, statistical benchmarking, anomaly detection, and an AI-assisted reporting layer.

Beyond ingestion and analytics, Insight incorporates an integrated diagnostic framework that continuously validates warehouse health, analytical view integrity, and referential consistency after every processing batch. Whether ingesting a single state or multiple states simultaneously, the pipeline verifies that analytical datasets maintain 100% join integrity before reporting layers become available.

Built upon a custom ETL framework, dimensional star schema, analytical SQL views, and configurable benchmarking architecture, Insight transforms millions of fragmented reimbursement records into actionable operational intelligence capable of supporting compliance reviews, reimbursement analysis, and revenue integrity initiatives.

---

## 💭 Engineering at Scale

| **Metric** | **Value** |
|------------|-----------|
| **Raw CMS Archive** | **3.03 GB** (compressed) |
| **Raw Dataset Size** | **4+ GB** extracted |
| **Source Records** | **10,000,000+** provider-service records |
| **States Supported** | **1–2 configurable** per analytical warehouse |
| **Filtered Analytical Dataset** | **576,000+** validated records |
| **Warehouse Architecture** | Dimensional **Star Schema** (Fact + 4 Dimensions + Benchmarks) |
| **Join Integrity** | **100% verified** across every completed ETL batch |
| **Processing Pipeline** | Automated **ETL → Validation → Analytics → AI-Assisted Reporting** |

---

## 📊 Technologies:
* Python
* PySpark
* Pandas
* SQLite
* SQL
* Excel
* Dimensional Star Schema
* ETL Engineering
* Gemini Flash API
* CMS Public-Facing API
* JSON
* FPDF2
* Data Modeling
* Healthcare Analytics

---

## ❗  Problem Statement

Healthcare revenue integrity teams routinely analyze large Medicare reimbursement datasets to identify unusual billing behavior, coding inconsistencies, and reimbursement anomalies.<br>

> 💡 However, publicly available CMS datasets present several engineering challenges:
> * Massive raw datasets exceeding 10 million records and multiple gigabytes in size.
> * Fragmented Excel source files distributed across numerous states and provider groups.
> * Inconsistent schema definitions requiring continual normalization.
> * Manual spreadsheet workflows that become increasingly unreliable as data volume grows.
> * Difficulty identifying statistically meaningful billing outliers across peer organizations.<br>

<br>The result is a highly manual analytical process where considerable effort is spent preparing data rather than interpreting it.

➡️ The objective became clear:

* Automate ingestion.
* Normalize inconsistent source data.
* Build performant analytical infrastructure.
* Surface meaningful compliance insights.
* Reduce analyst effort by prioritizing only statistically significant exceptions.

---

## 🔌 System Goals

Insight was designed around several core engineering objectives:

1. Automate ingestion of large CMS Medicare Part B datasets.
2. Eliminate manual spreadsheet preparation through repeatable ETL workflows.
3. Normalize fragmented source files into a maintainable dimensional data model.
4. Preserve complete referential integrity across analytical datasets.
5. Generate high-performance SQL views optimized for compliance analysis.
6. Automatically identify reimbursement anomalies using peer-group benchmarking.
7. Produce executive-ready narrative summaries that translate statistical findings into operational insight.
8. Build a modular architecture capable of adapting to future CMS schema revisions without requiring major redevelopment.

---

## 💻  System Architecture

<img width="624" height="936" alt="image" src="https://github.com/user-attachments/assets/9b5d566b-b268-4123-98bd-e7ad822e95de" /><br>

---

# CMS Provider Utilization & Analytics Schema

This repository maintains the SQLite database schema optimized for analyzing CMS (Centers for Medicare & Medicaid Services) provider utilization, pricing variations, and upcoding anomalies. 

> **💡 Architecture Note:** The schema utilizes a classic **Star Schema** design. Dimension tables isolate provider, procedure, and geographical attributes to keep the core analytical fact table narrow, highly indexed, and optimized for aggregation queries.

---

## 📊 Schema Overview

| Table Name | Type | Primary Key | Description |
| :--- | :--- | :--- | :--- |
| `dim_providers` | Dimension | `Rndrg_Npi` | Unique National Provider Identifiers (NPI) and credentials. |
| `dim_procedures` | Dimension | `Hcpcs_Cd` | HCPCS/CPT medical procedure codes and descriptions. |
| `dim_geography` | Dimension | `Rndrg_Prvdr_Zip5` | 5-digit ZIP codes mapped to US State abbreviations. |
| `dim_benchmarks` | Dimension (Derived) | `Benchmark_Id` | Pre-computed statistical averages per specialty and procedure. |
| `fact_provider_services` | Fact | `Fact_Id` | Measurable utilization, submitted charges, and payment metrics. |

---

## 🛠️ Data Definition Language (DDL)

```sql
PRAGMA foreign_keys = ON;

-- ============================================================================
-- 1. DIMENSION TABLES
-- Dimension tables keep provider, procedure, and geography lookups compact 
-- so the fact table can stay narrow and analytics-friendly.
-- ============================================================================

CREATE TABLE IF NOT EXISTS dim_providers (
    Rndrg_Npi TEXT PRIMARY KEY,
    Rndrg_Prvdr_Last_Org_Name TEXT,
    Rndrg_Prvdr_First_Name TEXT,
    Rndrg_Prvdr_Crdntl TEXT,
    Rndrg_Prvdr_Type TEXT
);

CREATE TABLE IF NOT EXISTS dim_procedures (
    Hcpcs_Cd TEXT PRIMARY KEY,
    Hcpcs_Desc TEXT
);

CREATE TABLE IF NOT EXISTS dim_geography (
    Rndrg_Prvdr_Zip5 TEXT PRIMARY KEY,
    Rndrg_Prvdr_State_Abrvtn TEXT
);

-- ============================================================================
-- 2. FACT TABLE
-- The fact table stores the measurable CMS service metrics and references the
-- dimensions through explicit foreign keys to preserve relational integrity.
-- ============================================================================

CREATE TABLE IF NOT EXISTS fact_provider_services (
    Fact_Id INTEGER PRIMARY KEY AUTOINCREMENT,
    Rndrg_Npi TEXT NOT NULL,
    Hcpcs_Cd TEXT NOT NULL,
    Rndrg_Prvdr_Zip5 TEXT NOT NULL,
    Place_Of_Srvc TEXT,
    Tot_Benes INTEGER,
    Tot_Srvcs REAL,
    Avg_Srvc_Smtd_Chrg NUMERIC,
    Avg_Medcr_Alwd_Amt NUMERIC,
    Avg_Medcr_Pymt_Amt NUMERIC,
    FOREIGN KEY (Rndrg_Npi) REFERENCES dim_providers (Rndrg_Npi)
        ON UPDATE CASCADE
        ON DELETE RESTRICT,
    FOREIGN KEY (Hcpcs_Cd) REFERENCES dim_procedures (Hcpcs_Cd)
        ON UPDATE CASCADE
        ON DELETE RESTRICT,
    FOREIGN KEY (Rndrg_Prvdr_Zip5) REFERENCES dim_geography (Rndrg_Prvdr_Zip5)
        ON UPDATE CASCADE
        ON DELETE RESTRICT,
    CHECK (Tot_Benes IS NULL OR typeof(Tot_Benes) = 'integer'),
    CHECK (Tot_Srvcs IS NULL OR typeof(Tot_Srvcs) IN ('integer', 'real')),
    CHECK (Avg_Srvc_Smtd_Chrg IS NULL OR typeof(Avg_Srvc_Smtd_Chrg) IN ('integer', 'real')),
    CHECK (Avg_Medcr_Alwd_Amt IS NULL OR typeof(Avg_Medcr_Alwd_Amt) IN ('integer', 'real')),
    CHECK (Avg_Medcr_Pymt_Amt IS NULL OR typeof(Avg_Medcr_Pymt_Amt) IN ('integer', 'real'))
);

-- ============================================================================
-- 3. PERFORMANCE INDEXES
-- Explicit indexes on the foreign-key columns keep provider and procedure filters
-- responsive during downstream compliance and anomaly investigations.
-- ============================================================================

CREATE INDEX IF NOT EXISTS idx_fact_provider_services_hcpcs_cd
    ON fact_provider_services (Hcpcs_Cd);

CREATE INDEX IF NOT EXISTS idx_fact_provider_services_rndrg_npi
    ON fact_provider_services (Rndrg_Npi);

CREATE INDEX IF NOT EXISTS idx_fact_provider_services_rndrg_prvdr_zip5
    ON fact_provider_services (Rndrg_Prvdr_Zip5);

-- This multi-column composite index dramatically accelerates peer-group
-- benchmarking and specialty-controlled utilization/upcoding calculations.
CREATE INDEX IF NOT EXISTS idx_fact_provider_services_npi_hcpcs
    ON fact_provider_services (Rndrg_Npi, Hcpcs_Cd);

-- ============================================================================
-- 4. ANALYTICAL OPTIMIZATIONS
-- Pre-computed peer-group benchmark table. Populated by populate_dim_benchmarks.sql.
-- Storing computed stats here decouples analytical views from expensive inline aggregations.
-- ============================================================================

CREATE TABLE IF NOT EXISTS dim_benchmarks (
    Benchmark_Id         INTEGER PRIMARY KEY AUTOINCREMENT,
    Rndrg_Prvdr_Type     TEXT    NOT NULL,
    Hcpcs_Cd             TEXT    NOT NULL,
    Peer_Avg_Submitted_Charge NUMERIC,
    Peer_Avg_Allowed_Amt NUMERIC,
    Peer_Avg_Payment_Amt NUMERIC,
    Peer_Avg_Markup_Ratio NUMERIC,
    Peer_Row_Count       INTEGER,
    Computed_At          TEXT    DEFAULT (datetime('now')),
    UNIQUE (Rndrg_Prvdr_Type, Hcpcs_Cd)
);

CREATE INDEX IF NOT EXISTS idx_dim_benchmarks_type_hcpcs
    ON dim_benchmarks (Rndrg_Prvdr_Type, Hcpcs_Cd);
```

*Rather than storing data in flat relational tables, Insight utilizes a dimensional star schema separating transactional facts from analytical dimensions. This architecture dramatically simplifies analytical querying while improving maintainability and reporting performance.*

---

## ⚙️ Key Features

→ **Automated ETL Pipeline**:

* The platform automates ingestion of publicly available CMS Medicare Part B datasets, transforming fragmented Excel workbooks into clean, normalized analytical tables.
* *A configurable mapping layer resolves schema inconsistencies during ingestion while preserving data quality throughout the pipeline.*<br>

<img width="957" height="169" alt="image" src="https://github.com/user-attachments/assets/5751a22a-aa66-48d9-b6c3-56ffc508d96e" /><br>

→ **Verified Referential Integrity**:

* Every ETL execution concludes with an automated validation phase that verifies row counts, analytical views, benchmark coverage, orphan detection, and referential joins. Analytical reporting is generated only after the warehouse achieves 100% verified join integrity, ensuring downstream compliance analytics are built upon internally consistent datasets.

→ **Dynamic Ingestion Scope**:

* Rather than producing a single static database, Insight provisions state-scoped analytical warehouses dynamically. The ETL can ingest one or two configurable states per execution, automatically creating isolated databases, rebuilding peer benchmarks, and validating the configured state scope before processing begins. This architecture enables flexible state-to-state benchmarking while preserving analytical integrity across independent datasets.

→ **Dimensional Data Modeling**:

* Rather than storing data in flat relational tables, Insight utilizes a dimensional star schema separating transactional facts from analytical dimensions.
* This architecture dramatically simplifies analytical querying while improving maintainability and reporting performance.<br>

<img width="1481" height="466" alt="image" src="https://github.com/user-attachments/assets/53d112e1-78fc-407f-8acd-f7da5ced73c8" /><br>

→ **Analytical SQL Layer**:

* Custom SQL views expose business-friendly analytical datasets optimized for reimbursement analysis, provider benchmarking, and operational reporting.
* Complex joins, aggregations, and transformations are encapsulated within reusable analytical views rather than duplicated throughout reporting logic.

→ **Peer Group Benchmarking**:

* Using SQL and PySpark, Insight compares providers against statistically similar peer groups rather than relying on simple national averages.
* This allows the platform to identify reimbursement patterns that warrant additional investigation while reducing noise from expected regional variation.<br>

<img width="954" height="145" alt="image" src="https://github.com/user-attachments/assets/5747a72a-a1b0-4af7-ab14-f69855775094" /><br>
```
DELETE FROM dim_benchmarks;
INSERT INTO dim_benchmarks (
    Rndrg_Prvdr_Type,
    Hcpcs_Cd,
    Peer_Avg_Submitted_Charge,
    Peer_Avg_Allowed_Amt,
    Peer_Avg_Payment_Amt,
    Peer_Avg_Markup_Ratio,
    Peer_Row_Count
)
SELECT
    p.Rndrg_Prvdr_Type,
    f.Hcpcs_Cd,
    ROUND(AVG(f.Avg_Srvc_Smtd_Chrg), 4),
    ROUND(AVG(f.Avg_Medcr_Alwd_Amt), 4),
    ROUND(AVG(f.Avg_Medcr_Pymt_Amt), 4),
    ROUND(AVG(f.Avg_Srvc_Smtd_Chrg / NULLIF(f.Avg_Medcr_Alwd_Amt, 0)), 6),
    COUNT(*)
FROM fact_provider_services f
JOIN dim_providers p
    ON f.Rndrg_Npi = p.Rndrg_Npi
WHERE
    f.Avg_Srvc_Smtd_Chrg  IS NOT NULL
    AND f.Avg_Medcr_Alwd_Amt IS NOT NULL
    AND f.Avg_Medcr_Alwd_Amt > 0
    AND p.Rndrg_Prvdr_Type  IS NOT NULL
GROUP BY
    p.Rndrg_Prvdr_Type,
    f.Hcpcs_Cd;
```

*Note on the SQL above: Earlier SQL benchmark implementation retained to demonstrate the analytical logic later migrated into the Spark-backed ETL layer.*

→ **Deterministic Compliance Reporting (PDF)**:

Insight's core reporting engine automatically generates one PDF for every high-confidence anomaly exceeding configurable analytical thresholds. Built entirely on deterministic business logic, each report includes statistical evidence, peer benchmark comparisons, supporting metrics, and investigation guidance—producing reproducible compliance documentation without requiring AI.
This allows analysts to receive consistent, reproducible compliance reports that can be reviewed independently or supplemented later by the AI-assisted narrative layer.

<img width="950" height="490" alt="image" src="https://github.com/user-attachments/assets/bcfa5452-e2a5-4f2e-abcb-67763e197963" /><br>
<img width="950" height="490" alt="image" src="https://github.com/user-attachments/assets/0ac49600-e9f0-40be-97c9-94b6cfc891f1" /><br>
<img width="950" height="490" alt="image" src="https://github.com/user-attachments/assets/8c272c89-6a95-4446-ae74-42904fbef330" /><br>

***Note**: The AI portion of this layer has not been released on GitHub and is in active development.*

→ **Modular Analytics Framework**:

* Every component — from ETL configuration through benchmarking thresholds — was designed for portability and reuse.
* State-specific (or multi-state) datasets, evolving CMS releases, and future analytical modules can be incorporated through configuration rather than extensive code modification.

```
_guard_states(connection, states, db_path)
```

```
elif row[0] != encoded:
        raise RuntimeError(
            f"Database '{db_path}' was built for state set '{row[0]}' but "
            f"states are currently '{encoded}'. "
            "Point to a new DB path or re-run with the original state set."
        )
```
---

## 👉 Engineering Decisions

One of the most significant challenges involved designing an ETL pipeline capable of handling more than ten million reimbursement records originating from inconsistent CMS source files.
Rather than tightly coupling ingestion logic to a specific release, I developed a configurable normalization layer capable of adapting to evolving source schemas while preserving downstream analytical consistency.

Designing the dimensional star schema presented another important architectural decision. Separating transactional facts from descriptive dimensions significantly improved analytical query performance while simplifying long-term maintenance as new analytical requirements emerged.

Finally, translating numerical outlier detection into meaningful operational insight required balancing automation with interpretability. Instead of simply reporting statistical anomalies, the narrative generation layer produces structured summaries that provide analysts with context, potential causes, and recommended areas for investigation.

---

## 📈 Business Outcome

Insight transforms fragmented public healthcare reimbursement data into a scalable compliance analytics platform capable of supporting operational decision-making.

By automating ingestion, normalization, benchmarking, and reporting, the platform enables analysts to transition from manually preparing data to investigating meaningful exceptions. The result is a repeatable analytics framework that improves data integrity, accelerates compliance review, and provides healthcare organizations with clearer visibility into reimbursement trends and potential audit risk.

## 🧬 Future Roadmap

Insight continues to evolve as additional analytical capabilities are introduced.

**Planned enhancements include**:

* Expanded reimbursement benchmarking models
* Additional provider peer-group analytics
* Enhanced AI-assisted compliance narratives
* Interactive Power BI dashboards
* Automated anomaly scoring
* Expanded reporting templates
* Additional CMS datasets beyond Medicare Part B
* Advanced statistical trend analysis

---

## 📢 Closing Statement

Insight was never intended to be another reporting dashboard.

Its purpose is to reduce the engineering effort required to transform fragmented healthcare reimbursement data into trustworthy operational intelligence. Every architectural decision—from the normalization layer to the dimensional schema and modular ETL framework—was designed to maximize scalability, maintainability, and analyst efficiency.

Rather than replacing human expertise, Insight amplifies it by allowing analysts to spend less time preparing data and more time investigating meaningful operational questions.

