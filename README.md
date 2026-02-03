# LH Sales - Data Pipeline (Microsoft Fabric)

A PySpark-based ETL pipeline that implements a medallion architecture (Bronze → Silver → Gold) for processing sales order data in Microsoft Fabric.

## Overview

This project transforms raw CSV sales data into a dimensional model suitable for analytics and reporting. The pipeline follows data lakehouse best practices with incremental loading, data quality checks, and performance optimization.

## Architecture

### Medallion Layers

- **Bronze**: Raw CSV files ingested from `Files/bronze/*.csv`
- **Silver**: Cleaned and validated data with audit columns
- **Gold**: Dimensional model optimized for analytics

### Data Model

**Dimension Tables:**
- `dimdate_gold` - Date dimension with day, month, year attributes
- `dimcustomer_gold` - Customer dimension with parsed name fields
- `dimproduct_gold` - Product dimension with item details

**Fact Table:**
- `factsales_gold` - Sales transactions with calculated total amount

## Features

- **Incremental Loading**: Delta Lake MERGE operations for upserts
- **Data Quality**: 
  - Flags records with dates before 2019-08-01
  - Handles null customer names (defaults to "Unknown")
  - Filters out records with missing dimension keys
- **Audit Trail**: Tracks batch ID, source system, and timestamps
- **Performance**: Table optimization using Delta Lake OPTIMIZE command
- **Surrogate Keys**: SHA-256 hashing for customer and product IDs

## Data Sources

**Bronze Schema:**
- SalesOrderNumber
- SalesOrderLineNumber
- OrderDate
- CustomerName
- Email
- Item
- Quantity
- UnitPrice
- Tax

## Pipeline Steps

1. **Define Parameters** - Configure table names and batch metadata
2. **Bronze → Silver** - Load CSVs, add audit columns, merge into silver table
3. **Silver → Date Dimension** - Extract distinct dates with attributes
4. **Silver → Customer Dimension** - Parse names, generate customer IDs
5. **Silver → Product Dimension** - Split item info, generate product IDs
6. **Create Fact Table** - Join dimensions and calculate total amount
7. **Optimize Tables** - Compact files for query performance
8. **Visualization** - Generate sales trend charts

## Requirements

- Microsoft Fabric workspace
- PySpark environment
- Delta Lake support
- Python libraries:
  - `pyspark`
  - `delta-spark`
  - visualization

## Usage

1. Place raw CSV files in `Files/bronze/` directory - subfolder in MS Fabric
2. Run all cells in the notebook sequentially
3. Verify row counts and validation messages
4. Query gold tables for analytics

## Table Naming Convention

All tables use the schema: `lh_sales.dbo.<table_name>`

- Silver: `sales_silver`
- Gold dimensions: `dimdate_gold`, `dimcustomer_gold`, `dimproduct_gold`
- Gold fact: `factsales_gold`

## Data Quality Checks

- **IsFlagged**: Boolean flag for pre-2019-08-01 orders
- **Missing Keys**: Excluded from fact table with count reported
- **Duplicate Handling**: Merge logic prevents duplicates

## Performance Optimization

- Delta Lake table format for ACID transactions
- OPTIMIZE command for file compaction
- Selective column projections in transformations

## Output

The pipeline produces:
- Clean dimensional model in gold layer
- Row count validations after each step
- Yearly sales visualization chart

## Notes

- The pipeline is idempotent - can be run multiple times safely
- Batch ID uses Unix timestamp for unique identification
- Customer and product IDs are deterministic based on source attributes
