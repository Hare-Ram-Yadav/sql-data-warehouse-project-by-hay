# SQL Data Warehouse Project

This project is about building a modern data warehouse using SQL Server. It covers the full pipeline — from raw data ingestion to clean, analytics-ready tables using a Medallion Architecture (Bronze → Silver → Gold).

I built this to practice real-world data engineering concepts like ETL development, data modeling with star schemas, and writing quality checks.

---

## Data Architecture

The warehouse follows the **Medallion Architecture** with three layers:

![Data Architecture](docs/data_architecture.png)

- **Bronze Layer** — Raw data loaded directly from CSV source files. No transformations, just a mirror of the source.
- **Silver Layer** — Cleaned and standardized data. Handles things like trimming whitespace, normalizing codes (e.g. 'M' → 'Male'), fixing bad dates, deduplication, etc.
- **Gold Layer** — Business-ready views modeled as a star schema (dimension + fact tables). This is what you'd query for reporting and dashboards.

---

## What This Project Does

1. **Sets up the database** with separate schemas for each layer (bronze, silver, gold)
2. **Loads raw data** from CSV files (CRM and ERP sources) into bronze tables using `BULK INSERT`
3. **Transforms data** in the silver layer — cleaning, standardizing, and enriching
4. **Creates analytical views** in the gold layer with proper dimension and fact tables
5. **Includes quality checks** to validate data at each stage

---

## Data Sources

The project works with two source systems:

- **CRM System** — Customer info, product catalog, and sales transactions
- **ERP System** — Customer demographics, location data, and product categories

All source data is in the `datasets/` folder as CSV files.

---

## Project Structure

```
sql-data-warehouse-project/
│
├── datasets/                          # Source CSV files
│   ├── source_crm/                    # CRM system exports
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   └── source_erp/                    # ERP system exports
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── docs/                              # Documentation and diagrams
│   ├── data_architecture.png
│   ├── data_flow.png
│   ├── data_model.png
│   ├── data_catalog.md                # Column-level docs for gold layer
│   └── naming_conventions.md          # Naming rules used across the project
│
├── scripts/                           # All SQL scripts
│   ├── init_database.sql              # Creates the database and schemas
│   ├── bronze/
│   │   ├── ddl_bronze.sql             # Table definitions for bronze layer
│   │   └── proc_load_bronze.sql       # Stored proc to load CSVs into bronze
│   ├── silver/
│   │   ├── ddl_silver.sql             # Table definitions for silver layer
│   │   └── proc_load_silver.sql       # Stored proc for bronze → silver ETL
│   └── gold/
│       └── ddl_gold.sql               # Views for dimension and fact tables
│
├── tests/                             # Data quality validation scripts
│   ├── quality_checks_silver.sql
│   └── quality_checks_gold.sql
│
├── .gitignore
├── LICENSE
└── README.md
```

---

## How to Run

### Prerequisites
- SQL Server (Express edition works fine)
- SQL Server Management Studio (SSMS)

### Steps

1. **Create the database**
   - Open `scripts/init_database.sql` in SSMS and execute it
   - This creates the `DataWarehouse` database with bronze, silver, and gold schemas

2. **Create bronze tables**
   - Run `scripts/bronze/ddl_bronze.sql`

3. **Load raw data into bronze**
   - Run `scripts/bronze/proc_load_bronze.sql` to create the stored procedure
   - Then execute: `EXEC bronze.load_bronze;`
   - Note: You may need to update the file paths in the procedure to match where you saved the CSV files

4. **Create silver tables**
   - Run `scripts/silver/ddl_silver.sql`

5. **Load cleaned data into silver**
   - Run `scripts/silver/proc_load_silver.sql` to create the procedure
   - Then execute: `EXEC silver.load_silver;`

6. **Create gold layer views**
   - Run `scripts/gold/ddl_gold.sql`

7. **Run quality checks** (optional but recommended)
   - Run `tests/quality_checks_silver.sql` after loading silver
   - Run `tests/quality_checks_gold.sql` after creating gold views

---

## Key Transformations

Some of the notable things happening in the silver layer:

- **Deduplication** — Keeps only the most recent customer record using `ROW_NUMBER()`
- **Code mapping** — Converts abbreviations to full names (e.g. 'S' → 'Single', 'M' → 'Mountain')
- **Date handling** — Converts integer dates (like `20140101`) to proper `DATE` type, filters out invalid ones
- **Data validation** — Recalculates sales amounts when they don't match quantity × price
- **Prefix cleanup** — Strips prefixes like 'NAS' from customer IDs for proper joins

---

## Gold Layer (Star Schema)

The gold layer has three views:

| View | Type | Description |
|------|------|-------------|
| `gold.dim_customers` | Dimension | Customer details with demographics and location |
| `gold.dim_products` | Dimension | Products with categories, costs, and product lines |
| `gold.fact_sales` | Fact | Sales transactions linked to customer and product dimensions |

For detailed column descriptions, see [docs/data_catalog.md](docs/data_catalog.md).

---

## Tools Used

- **SQL Server Express** — Database engine
- **SSMS** — For writing and running SQL
- **Draw.io** — For architecture and data flow diagrams

---

## License

MIT License — see [LICENSE](LICENSE) for details.
