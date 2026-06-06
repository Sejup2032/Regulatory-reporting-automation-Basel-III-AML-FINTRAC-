🏦 Banking Regulatory Reporting Pipeline
Azure Data Engineering | Basel III Capital Adequacy + AML/FINTRAC Compliance Automation
<p align="left">
  <img src="https://img.shields.io/badge/Azure%20Data%20Factory-0078D4?style=flat&logo=microsoftazure&logoColor=white"/>
  <img src="https://img.shields.io/badge/Azure%20Databricks-FF3621?style=flat&logo=databricks&logoColor=white"/>
  <img src="https://img.shields.io/badge/Delta%20Lake-00ADD8?style=flat&logo=apachespark&logoColor=white"/>
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black"/>
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white"/>
  <img src="https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat&logo=githubactions&logoColor=white"/>
</p>

🔴 The Real Problem This Solves
Every Canadian bank regulated by OSFI (Office of the Superintendent of Financial Institutions) must submit two critical reports on a strict schedule:
ReportRegulatorFrequencyManual Effort (Before)Basel III Capital Adequacy Ratio (CAR)OSFI / BISQuarterly2–3 weeks of analyst timeSuspicious Transaction Reports (STRs)FINTRACOngoing / ad-hocManual review of flagged alerts
What happens today in most mid-size banks:

Risk analysts manually pull data from 5–8 siloed systems (core banking, loan origination, treasury, CRM)
Data is reconciled in Excel spreadsheets — error-prone, not auditable
A single incorrect capital ratio submission can trigger an OSFI regulatory review or financial penalty
FINTRAC requires suspicious transactions to be reported within 30 days of detection — delays are penalised

This project automates that entire workflow end-to-end on Azure — from raw data ingestion to auditable regulatory report delivery — replacing weeks of manual work with a fully orchestrated, testable, and auditable data pipeline.

📐 Architecture
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐  ┌──────────┐  │
│  │ IEEE-CIS     │  │ HMDA 2023    │  │Frankfurter │  │Synthetic │  │
│  │ Transactions │  │ Mortgage     │  │ FX API     │  │Core Bank │  │
│  │ (590K rows)  │  │ (500K loans) │  │ (REST)     │  │  Tables  │  │
│  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘  └────┬─────┘  │
└─────────┼─────────────────┼────────────────┼──────────────┼────────┘
          │         Azure Data Factory (ADF)  │              │
          │    Watermark CDC · Key Vault Auth · Parameterised│
          ▼                  ▼                ▼              ▼
┌─────────────────────────────────────────────────────────────────────┐
│  BRONZE LAYER — ADLS Gen2  (raw, immutable, append-only)            │
│  bronze/transactions/  bronze/loans/  bronze/fx_rates/  bronze/core │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │  Azure Databricks + PySpark
                                  │  Schema validation · DQ Framework
                                  │  SCD Type 2 · Delta MERGE INTO
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  SILVER LAYER — Delta Lake  (cleansed, validated, enriched)         │
│                                                                     │
│  ┌──────────────────────┐    ┌──────────────────────────────────┐   │
│  │  Data Quality Layer  │    │       AML Rule Engine            │   │
│  │  · Schema enforce    │    │  · Txns > CAD 10,000 flagged     │   │
│  │  · Null checks       │    │  · Velocity rules (>3 txns/hr)   │   │
│  │  · Deduplication     │    │  · Structuring pattern detection │   │
│  │  · Bad → quarantine  │    │  · Cross-border flag (non-CAD)   │   │
│  └──────────────────────┘    └──────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │  PySpark · Basel III calc logic
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  GOLD LAYER — Synapse Analytics  (regulatory-ready aggregations)    │
│                                                                     │
│  ┌────────────────────────┐  ┌──────────────────────────────────┐   │
│  │  Basel III CAR Report  │  │  FINTRAC STR Summary Report      │   │
│  │  CAR = (T1+T2) / RWA   │  │  · Flagged txns by branch        │   │
│  │  Per quarter / region  │  │  · Alert age tracking            │   │
│  │  Minimum threshold: 8% │  │  · Analyst assignment queue      │   │
│  └────────────────────────┘  └──────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │  Power BI DirectLake
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│  SERVING LAYER — Power BI  (executive + compliance dashboards)      │
│  · CAR trend vs 8% OSFI threshold  · RLS by compliance officer role │
│  · AML heatmap by province         · Auto PDF report export         │
└─────────────────────────────────────────────────────────────────────┘

📊 Datasets Used
DatasetSourceSizeLayerPurposeIEEE-CIS Fraud DetectionKaggle590K transactionsBronze → SilverAML suspicious transaction detectionHMDA 2023 Mortgage DataCFPB (US Gov)~500K loans (NY)Bronze → GoldRisk-Weighted Asset (RWA) calculation for Basel IIIFrankfurter FX APIOpen APIDaily ratesBronzeCurrency normalisation to CAD for FINTRAC reportingSynthetic Core BankingFaker / SDV (generated)10K accounts, 50 branchesBronzedim_account, dim_branch, dim_capital_tier, dim_date

Why these datasets? The IEEE-CIS dataset contains real transaction patterns including device info, email domains, card types and velocity features — the exact signals an AML rule engine uses. HMDA is actual government-reported mortgage data with loan amounts, property values and risk indicators needed for Basel III RWA classification.


🛠️ Tech Stack
CategoryTechnologyUsage in this projectOrchestrationAzure Data FactoryWatermark CDC pipelines, parameterised ingestion, master orchestratorComputeAzure Databricks (PySpark)Bronze→Silver→Gold transformations, AML rule engine, Basel III calculationsStorageADLS Gen2 + Delta LakeMedallion architecture, ACID transactions, time travel audit trailServingAzure Synapse AnalyticsServerless SQL pool, external tables for BI consumptionBIPower BI (DirectLake mode)Compliance dashboards, RLS, automated PDF exportSecurityAzure Key VaultAll credentials and secrets — zero hardcoded keysIaCTerraformFull infrastructure provisioned as codeCI/CDGitHub ActionsDev → Test → Prod pipeline promotion, zero-touch deploymentMonitoringAzure Monitor + Log AnalyticsPipeline alerting, MTTR tracking

📁 Repository Structure
banking-regulatory-pipeline/
│
├── infra/
│   └── terraform/                  # All Azure resources provisioned as code
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── pipelines/
│   └── adf/                        # ADF pipeline JSON definitions
│       ├── PL_Bronze_Transactions.json
│       ├── PL_Bronze_Loans.json
│       ├── PL_Bronze_FX.json
│       └── PL_Master_Bronze.json
│
├── notebooks/
│   ├── bronze/
│   │   └── 00_bronze_verification.py
│   ├── silver/
│   │   ├── 01_transactions_dq_aml.py   # DQ framework + AML rule engine
│   │   ├── 02_loans_scd2.py            # SCD Type 2 on loan/account dims
│   │   └── 03_silver_verification.py
│   └── gold/
│       ├── 04_basel_car_calculation.py  # Basel III CAR logic
│       ├── 05_fintrac_str_report.py     # FINTRAC suspicious txn report
│       └── 06_gold_verification.py
│
├── data_generation/
│   ├── generate_core_banking.py        # Faker synthetic table generator
│   └── prepare_datasets.py             # IEEE-CIS + HMDA preprocessing
│
├── tests/
│   ├── test_dq_framework.py
│   ├── test_aml_rules.py
│   └── test_basel_calculations.py
│
├── docs/
│   ├── architecture_diagram.png
│   ├── data_dictionary.md
│   ├── basel_iii_calc_logic.md         # Documents the regulatory formula
│   └── aml_rule_definitions.md         # Documents each AML rule
│
└── README.md

⚙️ Pipeline Flow — Step by Step
Bronze Layer (Raw Ingestion)

ADF pulls IEEE-CIS transactions and HMDA loans into ADLS Gen2 as Parquet
Frankfurter FX rates ingested daily via ADF HTTP connector
Watermark-based CDC ensures only new records are processed on each run
All credentials routed through Azure Key Vault — zero hardcoded secrets

Silver Layer (Cleansing + AML Detection)

PySpark Data Quality Framework enforces schema, removes duplicates, quarantines bad records
SCD Type 2 applied on dim_account — full account history preserved in Delta Lake
Delta MERGE INTO for incremental loads — no full table rewrites
AML Rule Engine applies 4 rule categories to flag suspicious transactions:

Rule 1: Single transaction ≥ CAD 10,000 (FINTRAC mandatory reporting threshold)
Rule 2: Velocity — more than 3 transactions to the same beneficiary within 1 hour
Rule 3: Structuring — multiple transactions just below CAD 10,000 within 24 hours
Rule 4: Cross-border transactions in non-CAD currency above risk threshold



Gold Layer (Regulatory Calculations)

Basel III CAR: CAR = (Tier 1 Capital + Tier 2 Capital) / Risk-Weighted Assets

HMDA loans classified by BCBS risk weights (residential mortgage = 35%, commercial = 100%)
Calculated per quarter, per reporting period, vs 8% OSFI minimum threshold


FINTRAC STR Report: Aggregated suspicious transaction summary by branch, province, alert age
Star schema in Synapse: Fact_Transactions, Dim_Account, Dim_Branch, Dim_Date, Dim_Capital

Serving Layer (Power BI)

DirectLake mode — reads directly from Delta Parquet, no import lag
Row-Level Security (RLS) — compliance officers see only their region's data
Automated PDF report export via Logic Apps on quarter-end trigger


📈 Key Outcomes & Metrics
MetricBefore (Manual)After (This Pipeline)Quarterly CAR report prep time2–3 weeksFully automated — runs overnightFINTRAC STR detection latencyDays (manual review)< 1 hour (rule engine)Data audit trailExcel files, no lineage100% — Delta Lake time travelInfrastructure provisioningManual portal clicks< 5 minutes via TerraformDeployment errorsManual, error-proneZero — GitHub Actions CI/CDRecords processed per runN/A590K transactions + 500K loans

🚀 How to Run This Project
Prerequisites

Azure subscription (free tier works)
Azure CLI installed and logged in (az login)
Terraform 1.6+
Python 3.10+ with faker, sdv, pandas, pyarrow

1. Provision infrastructure
bashcd infra/terraform
terraform init
terraform plan
terraform apply
2. Generate synthetic data
bashcd data_generation
python generate_core_banking.py
python prepare_datasets.py
3. Upload to ADLS bronze layer
bash# Upload via Azure CLI
az storage blob upload-batch \
  --account-name <your_storage_account> \
  --destination bronze \
  --source ./data/raw
4. Run ADF master pipeline

Open ADF Studio → Pipelines → PL_Master_Bronze → Debug

5. Run Databricks notebooks in order
00_bronze_verification → 01_transactions_dq_aml → 02_loans_scd2 →
03_silver_verification → 04_basel_car_calculation → 05_fintrac_str_report

🏛️ Regulatory Context
Basel III is a global banking regulation framework set by the Bank for International Settlements (BIS). In Canada, it is enforced by OSFI. The Capital Adequacy Ratio (CAR) measures a bank's capital relative to its risk-weighted assets. A CAR below 8% triggers regulatory intervention.
FINTRAC (Financial Transactions and Reports Analysis Centre of Canada) requires all reporting entities to file Suspicious Transaction Reports (STRs) within 30 days of detecting a suspicious transaction. Non-compliance carries fines up to CAD 2 million per violation.
This project simulates the data infrastructure a compliance or risk data engineering team would build to automate these obligations.

👩‍💻 Author
Sejal Patil — Azure Data Engineer
Mississauga, ON | GitHub | sejalmpatil1@gmail.com
Microsoft Certified: Fabric Data Engineer (DP-700) · Databricks Certified: Data Engineer Associate
