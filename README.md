
EARTHQUAKE INSURANCE PREMIUM ANALYSIS — README


OVERVIEW
--------
This project analyses how earthquake risk influences insurance premium rates
across US states, using distributed big-data processing on Apache Spark
and Azure cloud services.

Three analyses are performed:
  1. Seismic Hazard (PGA) vs Insurance Premium Correlation
  2. Construction Type & Soil Vulnerability Index (Seismic Vulnerability Index)
  3. Temporal Premium Trend & Pricing Lag Detection

REQUIREMENTS
------------
Python 3.9+
pip install pandas numpy matplotlib seaborn scipy requests

For full Spark deployment (Azure Databricks / HDInsight):
pip install pyspark azure-storage-blob azure-cosmos

DIRECTORY STRUCTURE
-------------------
code/
  00_run_pipeline.py       — Full pipeline orchestrator (run this)
  01_data_ingestion.py     — USGS API fetch + dataset generation
  02_spark_processing.py   — Spark/MapReduce processing (3 analyses)
  03_visualisation.py      — Figures and follow-up analysis
data/
  usgs_earthquakes_2018_2023.csv   — USGS earthquake events (M2.5+)
  insurance_premiums.csv           — Homeowner insurance policies (150k records)
  fema_seismic_hazard_zones.csv    — FEMA seismic hazard by county
output/
  analysis1_seismic_premium_correlation.csv
  analysis2_construction_vulnerability.csv
  analysis3_temporal_trend.csv
  analysis3_pricing_lag.csv
  cosmos_db_documents.json         — Simulated Cosmos DB write
  summary_statistics.json
  figures/
    fig1_pga_premium_scatter.png
    fig2_vulnerability_heatmap.png
    fig3_premium_components.png
    fig4_pricing_lag.png
    fig5_temporal_trend.png

HOW TO RUN
----------
Option A: Full pipeline (includes data ingestion):
    cd code/
    python 00_run_pipeline.py

Option B: Skip ingestion (use existing data files):
    cd code/
    python 00_run_pipeline.py --skip-ingest

Option C: Run steps individually:
    python 01_data_ingestion.py
    python 02_spark_processing.py
    python 03_visualisation.py

AZURE DEPLOYMENT NOTES
----------------------
1. Create Azure Blob Storage container: 'earthquake-insurance-raw'/ LakeHouse in Fabrics
2. Set env var: AZURE_STORAGE_CONN_STR=<your connection string>  /
3. Create Azure Cosmos DB: database='earthquake_insurance'  / Use Connection String
4. Set env var: COSMOS_KEY=<your key>, COSMOS_ENDPOINT=<your endpoint>  
5. Deploy 02_spark_processing.py as Azure Databricks notebook  / Serverless Computer with Spark
   (swap pd.read_csv → spark.read.csv with wasbs:// path)

DATA SOURCES
------------
- USGS Earthquake Catalog API: https://earthquake.usgs.gov/fdsnws/event/1/
- Insurance data modelled after NAIC/Verisk actuarial frameworks
- Seismic hazard data modelled after USGS NSHM 2018 + FEMA HAZUS

RESEARCH QUESTIONS ANSWERED
----------------------------
RQ1: Is there a statistically significant correlation between seismic hazard
     (PGA) and insurance premium rates across US states?
     → YES. Pearson r = 0.9906 (p < 0.001)

RQ2: Which construction type × soil type combinations represent the highest
     under-priced seismic risk segments for insurers?
     → Manufactured housing on fill/liquefiable soil (SVI = 9.36‰)

RQ3: Is there a temporal pricing lag — states where earthquake frequency
     grows faster than premiums are adjusted?
     → YES. 9 of 18 states show significant pricing lag (>5% gap)

================================================================================
