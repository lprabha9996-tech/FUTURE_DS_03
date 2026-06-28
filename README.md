# Bank Marketing Campaign: Funnel & Conversion Performance Analysis

## 📌 Project Overview
This project focuses on analyzing marketing campaign data from a retail banking institution to evaluate how targeted prospects progress down a multi-stage acquisition funnel. In direct telemarketing environments, optimizing conversion rates, identifying demographic affinities, and pinpointing funnel bottlenecks directly influence institutional acquisition cost-efficiency and overall revenue velocity.

The objective of this analysis is to evaluate stage-by-stage pipeline drop-offs, isolate high-yielding customer segments, identify campaign outreach limits, and provide structured, data-driven recommendations implemented via a dynamic **Power BI Dashboard**.

---

## 📊 Dataset Architecture & Journey Mapping
The analysis utilizes the benchmark **Bank Marketing Campaign Dataset** (`bank-full.csv`), consisting of **45,211 rows** and 17 comprehensive demographic and financial attributes.

The direct-marketing sales lifecycle was modeled as a distinct 3-stage progression funnel within Power BI:
1. **Top-of-Funnel (Total Contacted):** Baseline audience targeted during the telemarketing campaign (Total count of client rows).
2. **Mid-Funnel (Engaged Leads):** Prospects showing behavioral intent and product interest, defined explicitly as phone connections lasting **longer than 120 seconds (`duration > 120`)**.
3. **Bottom-of-Funnel (Subscribed / Converted):** Prospects who completed the conversion milestone by subscribing to the term deposit product (`y = "yes"`).

---

## 🛠️ Data Engineering & ETL Pipeline (Power Query)
To ensure structural integrity and analytical reporting accuracy, the raw dataset underwent the following extraction, transformation, and loading (ETL) steps in **Power Query**:
* **Delimiter Configuration:** Configured parsing engines to explicitly process semicolon separations (`;`) rather than standard commas.
* **Text Normalization:** Extracted embedded text qualifiers (`""`) to establish clean categorical attributes across text dimensions (`job`, `education`, `marital`, etc.).
* **Null Value Handling:** Isolated missing values masked under `"unknown"` labels across high-leverage fields (e.g., `education`) and sanitized them to `"Not Disclosed"` to prevent chart skewing.
* **Primary Key Creation:** Generated a sequential surrogate index row starting from index 1 (`Client_ID`) to ensure accurate record-level aggregation and metric validation.

---

## 📐 Data Modeling & DAX Metric Engine
To support seamless chronological funnel calculations without generating native categoric visual failures, a **Disconnected Helper Table Engine** was constructed containing explicit sequence sorting metrics:

| Stage_ID | Stage_Name |
| :---: | :--- |
| 1 | 1-Total Contacted |
| 2 | 2-Engaged (>2 Mins) |
| 3 | 3-Subscribed (Converted) |

*The `Stage_Name` column is explicitly sorted by `Stage_ID` via Column Tools to preserve architectural ordering.*

### Core Analytical Measures (DAX)

```dax
// Cumulative Funnel Volume Processor
Funnel Volume = 
VAR CurrentStage = SELECTEDVALUE('Funnel Stages'[Stage_Name])
RETURN
SWITCH(
    CurrentStage,
    "1-Total Contacted", COUNTROWS('bank-full'),
    "2-Engaged (>2 Mins)", CALCULATE(COUNTROWS('bank-full'), 'bank-full'[duration] > 120),
    "3-Subscribed (Converted)", CALCULATE(COUNTROWS('bank-full'), 'bank-full'[y] = "yes"),
    BLANK()
)

// Individual Baseline Volume Counters
Total Contacted = COUNTROWS('bank-full')
Engaged Leads = CALCULATE(COUNTROWS('bank-full'), 'bank-full'[duration] > 120)
Total Subscriptions = CALCULATE(COUNTROWS('bank-full'), 'bank-full'[y] = "yes")

// Growth Performance KPI Metrics
Overall Conversion Rate = DIVIDE([Total Subscriptions], [Total Contacted], 0)
Engagement Rate = DIVIDE([Engaged Leads], [Total Contacted], 0)
