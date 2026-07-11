# Automated Medicare Inpatient Data Extraction Pipeline

> **Portfolio Navigation:**  
> * **Portfolio Summary:** [Overview of Portfolio](./README.md)
> * **Moderation & Privacy Infrastructure:** [Moderation & Privacy Infrastructure](mod-privacy.md)  
> * **AI-Driven CSR Platform:** [AI-Driven CSR Platform](ai-csr.md)  
> * **Autonomous Community Operations RPA:** [Autonomous Community Operations RPA](auto-ops-rpa.md)
> * **CMS Data Engineering Pipeline (Here):** [CMS Data Engineering Pipeline](cms-de-pipe.md)
> * **Insight: CMS Part B Compliance Analytics Engine:** [Insight: Compliance Analytics Engine](insight.md)
> * **Portfolio Summary & Engineering Philosophy:** [Profile Scope & Closure](summary.md)

---

## 📊 Technology:
* Python 
* Pandas
* Excel Automation
* Data Cleaning
* Healthcare Analytics

---

## 📄 Summary

This project automates the extraction and consolidation of Medicare inpatient utilization data from a complex, multi-sheet CMS Excel workbook.

The pipeline dynamically scans each worksheet, identifies the correct header row, extracts selected national metrics, standardizes inconsistent values, calculates additional healthcare performance indicators, and exports a clean analytical dataset for downstream reporting.

Rather than relying on fixed worksheet names, row positions, or manually copied values, the workflow is designed to adapt to semi-structured government spreadsheets whose formatting may vary across sheets.

---

## ❗ Problem Statement

CMS public datasets frequently contain:

* Multiple worksheets with different structures
* Titles and metadata positioned above the actual table headers
* Headers containing line breaks and inconsistent spacing
* Currency symbols, commas, and mixed data types
* Metrics distributed across several worksheets
* Geographic data that must be filtered and consolidated

📢 *Manually locating and combining these values is time-consuming, difficult to reproduce, and vulnerable to transcription errors.*

---

## Engineering Solution

I developed a Python-based extraction pipeline that:

* Validates the source file and resolves its active path
* Inspects every workbook worksheet
* Scans the first 30 rows to dynamically locate table headers
* Normalizes text for whitespace, capitalization, and line-break inconsistencies
* Uses flexible header matching to identify relevant datasets
* Extracts Medicare metrics for the United States and North Carolina
* Converts currency and formatted text into analysis-ready numeric values
* Prevents duplicate metric assignments across worksheets
* Consolidates the results into one structured DataFrame
* Calculates additional utilization and outcome measures
* Exports the final dataset to a reusable Excel file<br>

<img width="730" height="310" alt="image" src="https://github.com/user-attachments/assets/1978f416-917e-4608-8278-38051e40159b" /><br>
<img width="739" height="344" alt="image" src="https://github.com/user-attachments/assets/01741c3c-e826-4c49-8721-daf54b2e5f12" /><br>



## 📈 Metrics Extracted

→ **The pipeline consolidates**:

* Original Medicare Part A enrollees
* Total inpatient discharges
* Discharges beginning with an emergency-room visit
* Discharges per 1,000 Medicare enrollees
* Average inpatient length of stay
* Average Medicare payment per discharge
* Inpatient deaths

→ **It also derives**:

* **Emergency-Room Dependency Rate**
  Percentage of inpatient discharges that began with an emergency-room visit

* **Inpatient Mortality Rate**
  Percentage of recorded discharges resulting in death

## Technical Design

The workflow separates raw CMS column labels from standardized analytical field names through a centralized mapping layer. This produces clear, database-friendly columns such as:

* `Total_Discharges`
* `ER_Discharges`
* `Avg_Length_of_Stay`
* `Avg_Payment_per_Discharge`
* `Mortality_Rate_Pct`

The pipeline also uses defensive validation and type coercion so missing, malformed, or nonnumeric values do not interrupt execution.

<img width="1012" height="717" alt="image" src="https://github.com/user-attachments/assets/ff5b87c6-1d4e-4bfe-a346-9334e83ca967" /><br>

*The above image represents a single tab from the originating .xslx document. The pipeline normalizes the headers, rows, and columns prior to distributing a benchmarked .xslx between defined states.*

## Output

The final process generates, for example:

`NC_vs_US_Healthcare_Final_Consolidated.xlsx`

The exported file contains one standardized record for each comparison region and can be used directly in:

* SQL databases
* Power BI dashboards
* Statistical analysis
* Healthcare utilization reporting
* Medicare cost and outcome comparisons
* Larger ETL and analytics pipelines<br>

<img width="1239" height="187" alt="image" src="https://github.com/user-attachments/assets/4b5abff8-6b29-42fc-b87c-5a01e8ee6b8a" /><br>

```
py python_cleaning_script.py --regions US NC GA SC
```

*Example .xlsx compiled output for US, GA, SC and NC. State filters are easily customizable and ready for Power BI, etc.*

---

## Engineering Value

This project demonstrates the ability to transform inconsistent, human-oriented spreadsheets into reliable machine-readable data.

👉 It highlights practical experience with:

* Semi-structured data ingestion
* Dynamic schema and header detection
* Data normalization
* Defensive pipeline development
* Healthcare metric engineering
* Automated file generation
* Reproducible analytical workflows

The same extraction architecture can be extended to additional states, reporting years, CMS workbooks, or automated ingestion into a relational data warehouse.

<img width="1434" height="802" alt="image" src="https://github.com/user-attachments/assets/8ebfea1a-b8f6-45eb-a6de-b4b3ca37fbad" />

*Example of a completed dashboard rendered directly from the project output: NC_vs_US_Healthcare_Final_Consolidated.xlsx*
