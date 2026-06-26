# 🌍 Earthquake Insurance & Disaster Aid Analysis — Microsoft Fabric Pipeline



An end-to-end **big data pipeline built entirely on Microsoft Fabric**, analysing the relationship between US seismic risk and federal disaster-aid activation patterns. The pipeline ingests real FEMA and USGS data, processes it at scale with distributed Spark, and surfaces the results through a Fabric Data Warehouse and Power BI dashboard.

---

## 📐 Architecture

The project follows a **Medallion Architecture** (Bronze → Silver → Gold) entirely inside a single Fabric Lakehouse, with a Warehouse layer added at the end for SQL/BI consumption.

```
                     ┌─────────────────────────┐
   FEMA OpenFEMA API │                         │
   USGS FDSNWS API   │   Notebook 1: Ingestion │
   Derived SRI data  │                         │
                     └────────────┬────────────┘
                                  │
                     Lakehouse Files/  (Bronze — raw CSV)
                                  │
                     Lakehouse Tables/ (Silver — cleaned Delta)
                                  │
                     ┌────────────▼────────────┐
                     │ Notebook 2: Spark        │
                     │ Processing (3 analyses)  │
                     └────────────┬────────────┘
                                  │
                     Lakehouse Tables/ (Gold — aggregated Delta)
                                  │
                     ┌────────────▼────────────┐
                     │ Notebook 3: Visualisation│
                     └────────────┬────────────┘
                                  │
                     ┌────────────▼────────────┐
                     │ Notebook 4: Data         │
                     │ Warehouse & Power BI     │
                     └────────────┬────────────┘
                                  │
                     Fabric Data Warehouse (T-SQL)
                                  │
                     Power BI Semantic Model → Dashboard
```

| Layer | Storage | Purpose |
|---|---|---|
| **Bronze** | Lakehouse `Files/` | Raw CSVs exactly as received from source APIs |
| **Silver** | Lakehouse `Tables/` | Cleaned, typed Delta tables |
| **Gold** | Lakehouse `Tables/` | Aggregated analysis results |
| **Warehouse** | Fabric Data Warehouse (T-SQL) | SQL endpoint for BI tools and ad-hoc analysis |
| **Reporting** | Power BI semantic model | Dashboards and DAX measures |

---

## 📓 Notebooks

| # | Notebook | Role |
|---|---|---|
| 1 | `Notebook_1_Data_Ingestion.ipynb` | Pulls all source data into the Lakehouse and writes Silver Delta tables |
| 2 | `Notebook_2_Spark_Processing.ipynb` | Runs three distributed Spark analyses and writes Gold Delta tables |
| 3 | `Notebook_3_Visualisation.ipynb` | Reads Gold tables and produces the report's five figures |
| 4 | `Notebook_4_DataWarehouse_PowerBI.ipynb` | Promotes Gold tables to the Fabric Warehouse and documents the Power BI model |

### Notebook 1 — Data Ingestion

Ingests three datasets into the Lakehouse:

| # | Dataset | Source | Format | Records |
|---|---|---|---|---|
| 1 | FEMA Disaster Declarations Summaries | OpenFEMA API | CSV | 69,896 |
| 2 | USGS Earthquake Catalog | USGS FDSNWS Event API | GeoJSON → CSV | ~20,000 |
| 3 | FEMA Seismic Risk Index | Derived from Dataset 1 | CSV | 14 states |

Writes:
- `Tables/silver_fema_all_disasters`
- `Tables/silver_usgs_earthquakes`
- `Tables/silver_seismic_risk_index`

### Notebook 2 — Spark Processing & Distributed Analysis

Three analyses, each answering a research question:

| Analysis | Pattern | Research Question |
|---|---|---|
| **1** | MapReduce aggregation + equi-join | RQ1: Does Seismic Risk Index correlate with Individual Assistance (IA) activation? |
| **2** | Multi-key combinatorial group-by | RQ2: What does the multi-hazard aid portfolio and insurance exposure structure look like? |
| **3** | Spark SQL window functions (`lag`) | RQ3: How have temporal trends and the post-2010 insurance gap evolved? |

Writes the Gold tables:
- `gold_analysis1_eq_aid_correlation`
- `gold_analysis2_multihazard_profile`
- `gold_analysis2_eq_state_profile`
- `gold_analysis3_national_trend`
- `gold_analysis3_annual_by_state`
- `gold_summary_statistics`

### Notebook 3 — Visualisation & Follow-up Analysis

Reads the Gold tables and produces five publication-quality figures:

| Figure | Content | Answers |
|---|---|---|
| Fig 1 | SRI vs IA activation rate scatter | RQ1 correlation |
| Fig 2 | Multi-hazard portfolio + IA vs PA bar chart | RQ2 portfolio structure |
| Fig 3 | Declaration breadth vs IA activations bubble chart | RQ2 insurance gap |
| Fig 4 | Temporal trend — county rows & aid composition | RQ3 post-2010 shift |
| Fig 5 | IA rate heatmap by incident type × state | RQ2/RQ3 cross-hazard view |

### Notebook 4 — Data Warehouse & Power BI Reporting

1. Promotes the six Gold Delta tables into the **Fabric Data Warehouse** via Spark JDBC (T-SQL endpoint).
2. Runs analytical SQL queries directly against the Warehouse (state risk ranking, insurance-gap analysis, multi-hazard comparisons).
3. Documents the Power BI semantic model and DAX measures (e.g. *Total EQ County Declarations*, *National IA Rate*, *Insurance Gap Score*, *Peak Declaration Year*).

**Why a separate Warehouse?** The Lakehouse Delta tables are optimised for Spark batch reads. The Warehouse adds a T-SQL endpoint enabling direct SQL access without Spark, Power BI DirectQuery (live, uncached) connections, row-level security for multi-user dashboards, and standard ANSI SQL compatibility.

---

## 🗂️ Repository Structure

```
notebooks/
  Notebook_1_Data_Ingestion.ipynb
  Notebook_2_Spark_Processing.ipynb
  Notebook_3_Visualisation.ipynb
  Notebook_4_DataWarehouse_PowerBI.ipynb
README.md
```

> Notebooks are designed to run in order inside a Microsoft Fabric workspace, each one reading the tables written by the previous one.

---

## ▶️ How to Run

1. **Create a Fabric workspace** and a **Lakehouse** inside it (e.g. `earthquake-insurance`).
2. **Attach the Lakehouse** to each notebook (Notebook → Lakehouse explorer → Add).
3. Run the notebooks **in order**:
   1. `Notebook_1_Data_Ingestion.ipynb` — populates Bronze/Silver layers
   2. `Notebook_2_Spark_Processing.ipynb` — produces Gold layer
   3. `Notebook_3_Visualisation.ipynb` — generates figures
   4. `Notebook_4_DataWarehouse_PowerBI.ipynb` — promotes to Warehouse + documents Power BI model
4. **Create a Fabric Data Warehouse** in the same workspace, and update `WORKSPACE_ID` / `WAREHOUSE_ID` in Notebook 4 with your own IDs (Fabric Portal → Warehouse → Settings → SQL connection string).
5. **Set Service Principal credentials** as environment variables before running Notebook 4:
   ```
   FABRIC_SP_CLIENT_ID=<client_id>
   FABRIC_SP_SECRET=<client_secret>
   ```
6. **Connect Power BI Desktop**: *Get Data → Microsoft Fabric Data Warehouse* → select the Warehouse and the `dbo.gold_*` tables → paste in the documented DAX measures.

---

## 🧰 Tech Stack

- **Microsoft Fabric** — Lakehouse, Notebooks (Spark), Data Warehouse, OneLake
- **Apache Spark (PySpark)** — distributed processing, window functions, Delta Lake I/O
- **Delta Lake** — Silver/Gold table format
- **T-SQL** — Fabric Warehouse queries
- **Power BI** — semantic model and dashboard
- **Python** — `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `requests`

---

## 📡 Data Sources

- **FEMA OpenFEMA API** — Disaster Declarations Summaries
- **USGS FDSNWS Event API** — Earthquake catalog (M2.5+, 2000–2024): `https://earthquake.usgs.gov/fdsnws/event/1/`
- **FEMA Seismic Risk Index** — derived from the FEMA disaster declarations dataset

---

## 🔬 Research Questions

- **RQ1** — Does a state's Seismic Risk Index correlate with its rate of Individual Assistance (IA) activation following earthquake disasters?
- **RQ2** — How does earthquake aid activation compare structurally to other hazard types, and where do the largest insurance/aid gaps appear?
- **RQ3** — How have national and state-level disaster aid patterns trended over time, particularly the shift in Individual Assistance share since 2010?

---

