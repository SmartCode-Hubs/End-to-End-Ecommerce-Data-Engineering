# Ecommerce Catalog Medallion Architecture

## Project Overview

This project implements a complete end-to-end data pipeline for an ecommerce business using the **Medallion Architecture** (Bronze → Silver → Gold) on Databricks. The pipeline ingests raw CSV data, cleanses and enriches it, and creates business-ready analytical mart tables optimized for Power BI dashboards and reporting.

The project demonstrates modern data engineering best practices including:
* Delta Lake for ACID transactions and time travel
* Modular layered architecture for data quality and governance
* Automated job orchestration with Databricks workflows
* Serverless compute for cost optimization
* Unity Catalog for data governance

## Architecture

![Medallion Architecture Diagram](./architecture_diagram.png)
*High-level view of the ecommerce data pipeline using medallion architecture*

```
┌───────────────────────────────────────────────────────────────────┐
│                          DATA SOURCES                                │
│                                                                       │
│  📄 customers.csv    📄 products.csv    📄 orders.csv    📄 order_items.csv  │
└───────────────┬───────────────────────────────────────────────────┘
                │
                │ CSV Ingestion
                ▼
┌───────────────────────────────────────────────────────────────────┐
│                        🥉 BRONZE LAYER                               │
│                     (Raw Delta Tables)                               │
│                                                                       │
│  • ecommerce_catalog.bronze.customers                                │
│  • ecommerce_catalog.bronze.products                                 │
│  • ecommerce_catalog.bronze.orders                                   │
│  • ecommerce_catalog.bronze.order_items                              │
│                                                                       │
│  Purpose: Minimal transformation, preserve raw data                  │
└───────────────┬───────────────────────────────────────────────────┘
                │
                │ Cleansing, Enrichment, Joins
                ▼
┌───────────────────────────────────────────────────────────────────┐
│                        🥈 SILVER LAYER                               │
│                  (Cleaned & Enriched Tables)                         │
│                                                                       │
│  • ecommerce_catalog.silver.orders_enriched                          │
│  • ecommerce_catalog.silver.order_items_detailed                     │
│  • ecommerce_catalog.silver.customer_summary                         │
│                                                                       │
│  Purpose: Business logic, data quality, analytics-ready              │
└───────────────┬───────────────────────────────────────────────────┘
                │
                │ Aggregation, Business Metrics
                ▼
┌───────────────────────────────────────────────────────────────────┐
│                        🥇 GOLD LAYER                                 │
│                    (Business Mart Tables)                            │
│                                                                       │
│  Time-Based:                                                         │
│    • daily_sales_summary                                             │
│    • monthly_sales_summary                                           │
│    • monthly_sales_mart                                              │
│                                                                       │
│  Product Analytics:                                                  │
│    • product_performance                                             │
│    • top_products_mart                                               │
│                                                                       │
│  Customer Analytics:                                                 │
│    • customer_insights                                               │
│    • customer_lifetime_value_mart                                    │
│                                                                       │
│  Geographic:                                                         │
│    • country_sales                                                   │
│                                                                       │
│  Purpose: Pre-aggregated KPIs for dashboards                         │
└───────────────┬───────────────────────────────────────────────────┘
                │
                │ Direct Query / Refresh
                ▼
┌───────────────────────────────────────────────────────────────────┐
│                    📊 POWER BI DASHBOARD                             │
│                                                                       │
│  • Sales Trends  • Top Products  • Customer LTV  • Geography         │
└───────────────────────────────────────────────────────────────────┘

                        ┌─────────────────┐
                        │ ⚙️  ORCHESTRATION │
                        │                 │
                        │  Databricks Job │
                        │  (Serverless)   │
                        └─────────────────┘
```

### Architecture Components

1. **Bronze Layer**: Raw data ingestion from CSV files into Delta tables with minimal transformations
2. **Silver Layer**: Data cleansing, enrichment, and business logic application with joins and derived columns
3. **Gold Layer**: Aggregated business metrics and KPIs optimized for analytics and reporting
4. **Orchestration**: Databricks Job to automate the full Bronze → Silver → Gold pipeline

## Dataset

The project uses ecommerce transactional data with the following entities:

* **Customers**: Customer demographic and profile information
* **Products**: Product catalog with names, categories, and pricing
* **Orders**: Order transactions with dates, customers, and payment details
* **Order Items**: Line-item details for each order (products, quantities, prices)

**Source**: CSV files containing ecommerce transaction data  
**Storage**: Unity Catalog (`ecommerce_catalog` catalog with `bronze`, `silver`, `gold` schemas)

## Tools Used

* **Databricks**: Unified analytics platform for data engineering and analytics
* **Apache Spark (PySpark)**: Distributed data processing engine
* **Databricks SQL**: SQL-based data transformations
* **Delta Lake**: Open-source storage layer providing ACID transactions
* **Unity Catalog**: Unified governance solution for data and AI assets
* **Databricks Workflows**: Job orchestration and scheduling
* **Power BI**: Business intelligence and dashboard visualization

## Bronze Layer

**Purpose**: Ingest raw data from CSV files with minimal transformation

**Notebook**: `01_bronze/01_bronze_layer`

**Tables Created**:
* `ecommerce_catalog.bronze.customers` - Raw customer data
* `ecommerce_catalog.bronze.products` - Raw product catalog
* `ecommerce_catalog.bronze.orders` - Raw order transactions
* `ecommerce_catalog.bronze.order_items` - Raw order line items

**Key Operations**:
* Schema inference from CSV files
* Data type preservation
* Delta table creation with OVERWRITE mode
* No data cleansing or transformation

## Silver Layer

**Purpose**: Cleanse, enrich, and join data to create analytics-ready tables

**Notebook**: `02_silver/02_silver_layer`

**Tables Created**:
* `ecommerce_catalog.silver.orders_enriched` - Orders joined with customer and product details
* `ecommerce_catalog.silver.order_items_detailed` - Order items with product information and calculated line totals
* `ecommerce_catalog.silver.customer_summary` - Customer aggregations with lifetime metrics

**Key Transformations**:
* Join orders with customers to enrich with customer name, country, and demographics
* Join order items with products to add product names and details
* Calculate derived columns (line totals = unit_price × quantity)
* Data type conversions and standardization
* Null handling and data quality checks

## Gold Layer

**Purpose**: Create business-ready aggregated mart tables for dashboards and reporting

**Notebook**: `04_gold/03_Gold_Layer`

**Tables Created**:

### Time-Based Sales
* `ecommerce_catalog.gold.daily_sales_summary` - Daily aggregated sales metrics
* `ecommerce_catalog.gold.monthly_sales_summary` - Monthly sales trends
* `ecommerce_catalog.gold.monthly_sales_mart` - Monthly revenue with average order values

### Product Analytics
* `ecommerce_catalog.gold.product_performance` - Product-level revenue and order counts
* `ecommerce_catalog.gold.top_products_mart` - Top-selling products by revenue and quantity

### Customer Analytics
* `ecommerce_catalog.gold.customer_insights` - Customer spending and order frequency
* `ecommerce_catalog.gold.customer_lifetime_value_mart` - CLV with first/last order dates

### Geographic Analytics
* `ecommerce_catalog.gold.country_sales` - Revenue breakdown by country

**Key Features**:
* Pre-aggregated metrics for fast dashboard queries
* Business KPIs (total orders, revenue, average order value, customer lifetime value)
* Optimized for Power BI Direct Query mode
* Sorted by relevant dimensions for efficient filtering

## Dashboard

**Platform**: Power BI

**Connection**: Direct Query to Databricks SQL Warehouse via Unity Catalog

**Key Visualizations**:
* **Daily Sales Trend**: Line chart showing revenue over time
* **Monthly Revenue**: Bar chart for month-over-month comparison
* **Top 10 Products**: Horizontal bar chart of best-selling products
* **Revenue by Country**: Geographic distribution of sales
* **Customer Lifetime Value**: Bar chart of highest-value customers
* **Customer Segmentation**: Pie chart showing customer categories

**Dashboard Features**:
* Real-time data refresh from Gold layer tables
* Interactive filters and slicers
* Drill-down capabilities from monthly to daily views
* KPI cards for key business metrics

## Project Structure


consolidated_pipeline/
│
├── 01_bronze/
│   └── 01_bronze_layer           # Bronze layer ingestion notebook
│
├── 02_silver/
│   └── 02_silver_layer           # Silver layer transformation notebook
│
├── 04_gold/
│   └── 03_Gold_Layer             # Gold layer aggregation notebook
│
└── README.md                      # Project documentation (this file)


## How to Run

### Prerequisites
* Databricks workspace with Unity Catalog enabled
* Serverless compute enabled (or provision a cluster)
* CSV source files accessible in the workspace
* Unity Catalog with `ecommerce_catalog` catalog created

### Manual Execution

1. **Create the catalog and schemas**:
   sql
   CREATE CATALOG IF NOT EXISTS ecommerce_catalog;
   CREATE SCHEMA IF NOT EXISTS ecommerce_catalog.bronze;
   CREATE SCHEMA IF NOT EXISTS ecommerce_catalog.silver;
   CREATE SCHEMA IF NOT EXISTS ecommerce_catalog.gold;
   

2. **Run Bronze Layer**:
   * Open notebook: `01_bronze/01_bronze_layer`
   * Execute all cells to ingest CSV files into bronze tables
   * Verify tables created in `ecommerce_catalog.bronze`

3. **Run Silver Layer**:
   * Open notebook: `02_silver/02_silver_layer`
   * Execute all cells to create enriched tables
   * Verify tables created in `ecommerce_catalog.silver`

4. **Run Gold Layer**:
   * Open notebook: `04_gold/03_Gold_Layer`
   * Execute all cells to create aggregated mart tables
   * Verify tables created in `ecommerce_catalog.gold`

### Automated Execution (Databricks Job)

The project includes a Databricks Job definition for automated pipeline execution:

**Job Name**: `Ecommerce ETL Pipeline`

**Tasks**:
1. **bronze** - Ingests raw CSV data
2. **silver** - Cleanses and enriches data (depends on bronze)
3. **gold** - Creates business marts (depends on silver)

**To create the job**:
* Execute Cell 16 in `04_gold/03_Gold_Layer` notebook
* This creates the job using the Databricks SDK
* The job uses serverless compute (no cluster configuration needed)

**To schedule the job**:
1. Navigate to the Databricks Jobs UI
2. Find "Ecommerce ETL Pipeline" job
3. Click "Add trigger" to set up scheduling (e.g., daily at 2 AM)

## Results

### Data Quality
* ✅ Zero data loss during ingestion
* ✅ Referential integrity maintained through joins
* ✅ Calculated fields (line totals) validated
* ✅ Delta Lake ensures ACID compliance

### Performance
* ⚡ Bronze layer: Raw ingestion optimized with Delta
* ⚡ Silver layer: Efficient joins using broadcast hints where applicable
* ⚡ Gold layer: Pre-aggregated for sub-second dashboard queries
* ⚡ Serverless compute auto-scales based on workload

### Business Value
* 📊 Real-time visibility into sales trends
* 🎯 Product performance insights for inventory planning
* 👥 Customer segmentation for targeted marketing
* 🌍 Geographic revenue analysis for regional strategies
* 💰 Customer lifetime value tracking for retention programs

## Future Improvements

### Data Engineering
* [ ] Implement incremental loading using Delta Change Data Feed
* [ ] Add data quality validation checks with expectations
* [ ] Implement slowly changing dimensions (SCD Type 2) for customer history
* [ ] Add data lineage tracking and metadata management
* [ ] Implement automated testing for data transformations
* [ ] Add monitoring and alerting for pipeline failures
* [ ] Optimize performance with Z-ordering and liquid clustering

### Analytics & ML
* [ ] Build predictive models for customer churn
* [ ] Implement product recommendation engine
* [ ] Add forecasting for demand planning
* [ ] Create cohort analysis for customer behavior
* [ ] Implement real-time streaming for live dashboard updates

### Infrastructure
* [ ] Implement CI/CD pipeline using Databricks Asset Bundles
* [ ] Add automated integration tests
* [ ] Set up multi-environment deployment (dev/staging/prod)
* [ ] Implement cost optimization with spot instances for batch jobs
* [ ] Add comprehensive logging and observability

### Dashboard & Reporting
* [ ] Add real-time KPI alerts and notifications
* [ ] Create executive summary dashboard
* [ ] Implement drill-through reports for detailed analysis
* [ ] Add mobile-responsive dashboard views
* [ ] Create automated email reports with key insights

---

**Project Status**: ✅ Production Ready  
**Last Updated**: August 2026 
**Maintained By**: Data Engineering Team