# 🏭 AdventureWorks Products ETL Pipeline
### SSIS-Based Product Dimension Loading with SCD Type 1 & 2 Implementation

[![SQL Server](https://img.shields.io/badge/SQL%20Server-2022-CC2927?logo=microsoft-sql-server&logoColor=white)](https://www.microsoft.com/sql-server)
[![SSIS](https://img.shields.io/badge/SSIS-ETL%20Pipeline-0078D4?logo=microsoft&logoColor=white)](https://docs.microsoft.com/sql/integration-services)
[![T-SQL](https://img.shields.io/badge/T--SQL-Advanced-316192?logo=postgresql&logoColor=white)](https://docs.microsoft.com/sql/t-sql)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Production-ready SSIS ETL pipeline for automated product dimension loading from CSV files into the AdventureWorks Data Warehouse, featuring SCD Type 1 & 2 logic.**

---

## 📋 Table of Contents

- [Technical Skills Demonstrated](#technical-skills-demonstrated)
- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Key Features](#key-features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [ETL Process Flow](#etl-process-flow)
- [Command-Line Tools](#command-line-tools)
- [Data Model](#data-model)
- [Performance Metrics](#performance-metrics)
- [Related Projects](#related-projects)
- [Author](#author)
- [License](#license)

---

## Technical Skills Demonstrated

<table>
<tr>
<td width="55%" valign="top">

**SSIS Development**
- ✅ Data Flow Task Design
- ✅ Control Flow Orchestration
- ✅ Flat File Sources (CSV)
- ✅ OLE DB Destinations
- ✅ Lookup Transformations
- ✅ Conditional Split Logic

**ETL Best Practices**
- ✅ Incremental Data Loading
- ✅ Change Data Capture (CDC)
- ✅ SCD Type 1 & 2 Implementation
- ✅ Error Handling & Logging
- ✅ Data Validation
- ✅ Transaction Management

</td>
<td width="50%" valign="top">

**Data Integration**
- ✅ CSV File Processing
- ✅ Bulk Data Import (BCP)
- ✅ Dynamic File Processing
- ✅ ForEach Loop Containers
- ✅ Variable-Driven ETL
- ✅ Parallel Execution

**SQL Server Tools**
- ✅ BCP Utility (Bulk Copy)
- ✅ SQLCMD Scripting
- ✅ SQL Server Agent Jobs
- ✅ SSIS Catalog Deployment
- ✅ Connection Management
- ✅ Performance Tuning

</td>
</tr>
</table>

---

## Project Overview

This SSIS ETL pipeline is part of the **AdventureWorks Sales Analytics Platform**. It automates the extraction, transformation, and loading of product dimension data from CSV files into the enterprise data warehouse.

### What This Project Does

- 🔄 **Automated ETL**: Processes CSV product update files
- 🔀 **Data Transformation**: Enriches product attributes (categories, subcategories, pricing)
- 📊 **Dimension Loading**: Loads DimProduct with full SCD logic
- ✅ **Data Quality**: Validates integrity and completeness
- 📈 **Monitoring**: Logs execution metrics and tracks changes
- 🔗 **Integration**: Seamlessly connects to enterprise DWH

### Business Context

Product dimension is critical for sales analytics:
- 📦 Product catalog management (504 active products)
- 💰 Pricing strategy tracking (historical and current)
- 📂 Category and subcategory organization
- 🎨 Product attributes (color, size, style, line)

### Project Statistics

- **2 SSIS Packages** (Single File & Multi-File variants)
- **1 Sample CSV File** (ProductsUpdates.csv)
- **504 Products** processed
- **4 Categories** and **37 Subcategories**
- **SCD Type 1 & 2** implementation
- **<30 seconds** execution time

---

## System Architecture

### Data Flow Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SOURCE FILES (CSV)                           │
│                      ProductsUpdates.csv                            │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ ProductNumber, ProductID, ProductName, ListPrice,            │  │
│  │ StandardCost, ProductColor, ProductSize, SellStartDate,      │  │
│  │ ProductCategoryID, ProductSubcategoryID, etc.                │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                             │
                   ▼ SSIS Package (ETL.xml) ▼
                             │
┌─────────────────────────────────────────────────────────────────────┐
│                      ETL PROCESSING LAYER                           │
│                                                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │              1. EXTRACT (Flat File Source)                  │   │
│  │  • Read CSV files from folder                              │   │
│  │  • Parse columns and data types                            │   │
│  │  • Filter by updated dates                                 │   │
│  │  • ForEach loop for multiple files                         │   │
│  └────────────────────────────────────────────────────────────┘   │
│                             │                                       │
│                             ▼                                       │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │              2. TRANSFORM (Data Flow Tasks)                 │   │
│  │  • Lookup: Existing ProductID in DimProduct               │   │
│  │  • Derived Column: Add IsCurrent flag                     │   │
│  │  • Derived Column: Trim & convert dates                   │   │
│  │  • Conditional Split: Route by change type                │   │
│  │    ├─> No Change (unchanged products)                     │   │
│  │    ├─> SCD Type 1 (ProductName changed)                   │   │
│  │    └─> SCD Type 2 (ListPrice changed)                     │   │
│  └────────────────────────────────────────────────────────────┘   │
│                             │                                       │
│                             ▼                                       │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │                 3. LOAD (OLE DB Destination)                │   │
│  │  • Insert new products (No Match from Lookup)              │   │
│  │  • Update existing (SCD Type 1)                            │   │
│  │  • Version existing + Insert new (SCD Type 2)              │   │
│  │  • Update IsCurrent flags                                  │   │
│  │  • Set ValidityDate_Start/End                              │   │
│  └────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│       DATA WAREHOUSE (AdventureWorks2022DWH)                       │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │  prod.DimProduct                                           │   │
│  │  ┌──────────────────────────────────────────────────────┐  │   │
│  │  │ ProductDWKey (PK, IDENTITY)                          │  │   │
│  │  │ ProductID (Business Key)                             │  │   │
│  │  │ ProductName, Category, Subcategory                   │  │   │
│  │  │ ListPrice (SCD Type 2) ← Historical versions         │  │   │
│  │  │ StandardCost, ProductColor, ProductSize              │  │   │
│  │  │ IsCurrent, ValidityDate_Start, ValidityDate_End      │  │   │
│  │  └──────────────────────────────────────────────────────┘  │   │
│  └────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### 1. Dual Package Architecture

#### Package 1: Single File Processing (`1.-ETL_Products_single_File.xml`)
- **Purpose**: Process one CSV file at a time
- **Use Case**: Small loads, manual execution, testing
- **Execution**: Row-by-row processing with full transformation
- **Best For**: Development environments, proof of concept

#### Package 2: Multi-File Processing (`2.-ETL_Products_Multi_Files.xml`)
- **Purpose**: Process multiple CSV files from a folder
- **Use Case**: Batch processing, automated daily loads
- **Execution**: ForEach Loop Container iterates through files
- **Best For**: Production environments, scheduled jobs

### 2. Slowly Changing Dimensions Implementation

| Attribute | SCD Type | Change Handling | History Kept |
|-----------|----------|-----------------|--------------|
| **ProductName** | Type 1 | Overwrite current value | No |
| **StandardCost** | Type 1 | Overwrite current value | No |
| **ProductColor** | Type 1 | Overwrite current value | No |
| **ListPrice** | Type 2 | Create new version | Yes ⭐ |
| **ProductID** | Type 0 | Never changes (Natural Key) | N/A |

### 3. Data Flow Transformations

**Lookup Transformation:**
- Queries prod.DimProduct for existing ProductID
- Retrieves ProductName and ListPrice for comparison
- Routes to "Match" or "No Match" output

**Derived Column Transformation:**
- Adds `IsCurrent = 1` for new records
- Converts date strings to `DATETIME` format
- Trims whitespace from text fields

**Conditional Split Transformation:**
- **No Change**: ProductName AND ListPrice unchanged → Skip loading
- **SCD Type 1**: ProductName changed → UPDATE existing record
- **SCD Type 2**: ListPrice changed → Version old record + INSERT new

**OLE DB Command (for SCD Type 2):**
```sql
UPDATE prod.DimProduct 
SET IsCurrent = 0, ValidityDate_End = GETDATE()
WHERE ProductID = ? AND IsCurrent = 1
```

### 4. Error Handling & Logging

- ✅ Flat File Source error output
- ✅ Lookup transformation error routing
- ✅ OLE DB Destination error capture
- ✅ Row count tracking
- ✅ Execution logging via SSIS catalog
- ✅ File archiving after processing

---

## Project Structure

```
adventureworks-products-etl/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── SSIS-Packages/
│   ├── 1.-ETL_Products_single_File.xml    (212 KB)
│   └── 2.-ETL_Products_Multi_Files.xml    (209 KB)
│
├── Sample-Data/
│   └── ProductsUpdates.csv                 (8.6 KB, 104 rows)
│
├── Scripts/
│   ├── BCP-Import-Products.cmd
│   ├── SQLCMD-Load-Products.sql
│   └── Deploy-SSIS-Package.ps1
│
└── Documentation/
    ├── SETUP-GUIDE.md
    ├── TROUBLESHOOTING.md
    └── DATA-FLOW-DIAGRAM.png
```

**Total Files**: 3 main files | **SSIS Packages**: 2 | **Sample Data**: 1 CSV

---

## Getting Started

### Prerequisites

- **SQL Server 2019+** (Express/Developer Edition works)
- **SQL Server Integration Services (SSIS)**
- **SQL Server Data Tools (SSDT)** for package editing
- **AdventureWorks2022DWH Database** (from main project)

### Quick Start (5 minutes)

#### 1. Clone the Repository
```bash
git clone https://github.com/aharkane/adventureworks-products-etl.git
cd adventureworks-products-etl
```

#### 2. Verify Database Connection
```sql
-- Test connection to DWH
USE AdventureWorks2022DWH;
GO

-- Verify DimProduct table exists
SELECT TOP 5 * FROM prod.DimProduct;
```

#### 3. Import SSIS Package

**Using SQL Server Data Tools (SSDT)**:
```
1. Open Visual Studio with SSDT
2. File → New → Integration Services Project
3. Right-click Project → Add Existing Package
4. Select: 1.-ETL_Products_single_File.xml
5. Configure connection strings (see below)
```

**Using SSIS Catalog Deployment**:
```powershell
# PowerShell script to deploy
.\Scripts\Deploy-SSIS-Package.ps1 `
    -ServerName "YOUR_SERVER" `
    -CatalogFolder "AdventureWorks" `
    -ProjectName "ProductsETL"
```

#### 4. Configure Connection Managers

**Connection Manager #1: CSV Files** (Flat File Connection)
```
Name: ProductUpdates_CSV
File Path: C:\Data\ProductsUpdates.csv
Format: Delimited
Header Row Delimiter: {CR}{LF}
Column Delimiter: Comma {,}
Text Qualifier: None
```

**Connection Manager #2: DWH Database** (OLE DB Connection)
```
Name: LocalHost.AdvWrk
Provider: SQL Server Native Client 11.0
Server: YOUR_SQL_SERVER
Database: AdventureWorks2022DWH
Authentication: Windows or SQL Server
```

#### 5. Execute Package

**From SSDT** (Development):
```
1. Right-click package
2. Execute Package
3. Monitor Progress tab
4. Verify green checkmarks
```

**From SSIS Catalog** (Production):
```sql
-- Execute via T-SQL
DECLARE @execution_id BIGINT
EXEC [SSISDB].[catalog].[create_execution] 
    @package_name = N'1.-ETL_Products_single_File.dtsx',
    @execution_id = @execution_id OUTPUT,
    @folder_name = N'AdventureWorks',
    @project_name = N'ProductsETL'

EXEC [SSISDB].[catalog].[start_execution] @execution_id
```

---

## ETL Process Flow

### Execution Sequence (Single File Package)

```
STEP 1: EXTRACT
├─> Flat File Source: Read ProductsUpdates.csv
├─> Parse 23 columns (ProductID, ProductName, ListPrice, etc.)
└─> Output: 104 rows to data flow

STEP 2: TRANSFORM
├─> Derived Column: Trim & convert dates
│   ├─> SellStartDate: CONVERT to DATETIME
│   ├─> SellEndDate: CONVERT to DATETIME
│   └─> ValidityDate_Start/End: SET values
│
├─> Lookup: Query prod.DimProduct by ProductID
│   ├─> Match Output (78 rows): Existing products
│   └─> No Match Output (26 rows): New products
│
├─> Conditional Split (on Match Output only)
│   ├─> No Change: ProductName = dest_ProductName AND ListPrice = dest_ListPrice (52 rows)
│   ├─> SCD Type 1: ProductName ≠ dest_ProductName (14 rows)
│   └─> SCD Type 2: ListPrice ≠ dest_ListPrice (12 rows)
│
└─> Derived Column: Add IsCurrent = 1

STEP 3: LOAD
├─> OLE DB Command (SCD Type 2 only): UPDATE IsCurrent = 0 for old versions
├─> Union All: Combine all flows
├─> OLE DB Destination: INSERT/UPDATE prod.DimProduct
└─> Row Count: Track loaded records

STEP 4: FINALIZE
└─> File System Task: Move processed CSV to archive folder
```

### SCD Type 2 Example

```sql
-- Before: Product price changes from $1,295.99 to $1,495.99
ProductDWKey | ProductID | ProductName | ListPrice | IsCurrent | ValidityDate_Start | ValidityDate_End
1            | 680       | HL Road Frame | 1295.99 | 1         | 2024-01-01        | 9999-12-31

-- After: ETL processes price change
ProductDWKey | ProductID | ProductName | ListPrice | IsCurrent | ValidityDate_Start | ValidityDate_End
1            | 680       | HL Road Frame | 1295.99 | 0         | 2024-01-01        | 2024-11-05  ← Historical
2            | 680       | HL Road Frame | 1495.99 | 1         | 2024-11-05        | 9999-12-31  ← Current

-- Time travel query: "What was the price in March 2024?"
SELECT ListPrice 
FROM prod.DimProduct
WHERE ProductID = 680
  AND '2024-03-01' BETWEEN ValidityDate_Start AND ValidityDate_End
-- Returns: $1,295.99
```

---

## Command-Line Tools

### BCP (Bulk Copy Program) - Fast CSV Import

**Use BCP for initial bulk loads or large CSV files:**

#### Export from DWH to CSV
```cmd
REM Export DimProduct to CSV file
bcp "SELECT * FROM AdventureWorks2022DWH.prod.DimProduct" ^
    queryout "C:\Data\ProductsExport.csv" ^
    -S YOUR_SERVER ^
    -T ^
    -c ^
    -t "," ^
    -r "\n"
```

#### Import CSV to Staging Table
```cmd
REM Create staging table first
sqlcmd -S YOUR_SERVER -d AdventureWorks2022DWH -Q ^
"CREATE TABLE stg.Products_Import (
    ProductNumber VARCHAR(50),
    ProductID INT,
    ProductName VARCHAR(100),
    ListPrice MONEY,
    StandardCost MONEY,
    ProductColor VARCHAR(30)
)"

REM Bulk import CSV
bcp AdventureWorks2022DWH.stg.Products_Import ^
    IN "C:\Data\ProductsUpdates.csv" ^
    -S YOUR_SERVER ^
    -T ^
    -c ^
    -t "," ^
    -F 2 ^
    -e "C:\Data\errors.log"
```

**BCP Parameters Explained:**
```
-S YOUR_SERVER         Server name
-d DATABASE           Database name
-T                    Trusted connection (Windows Auth)
-c                    Character data type
-t ","                Field terminator (comma)
-r "\n"               Row terminator (newline)
-F 2                  First row to import (skip header)
-e errors.log         Error log file
```

### SQLCMD - Script-Based Loading

**Use SQLCMD for scripted, repeatable ETL processes:**

#### Load CSV using BULK INSERT
```sql
-- File: SQLCMD-Load-Products.sql
-- Execute: sqlcmd -S YOUR_SERVER -d AdventureWorks2022DWH -i SQLCMD-Load-Products.sql

USE AdventureWorks2022DWH;
GO

-- Create staging table
IF OBJECT_ID('stg.Products_Staging', 'U') IS NOT NULL
    DROP TABLE stg.Products_Staging;
GO

CREATE TABLE stg.Products_Staging (
    ProductNumber VARCHAR(50),
    ProductID INT,
    ProductCategoryID INT,
    ProductSubcategoryID INT,
    ProductName VARCHAR(100),
    ProductCategoryName VARCHAR(100),
    ProductSubcategoryName VARCHAR(100),
    ProductColor VARCHAR(30),
    ProductStyle VARCHAR(4),
    ProductLine VARCHAR(4),
    StandardCost DECIMAL(19,4),
    ListPrice DECIMAL(19,4),
    ProductSize VARCHAR(10),
    UnitSizeCode VARCHAR(6),
    UnitSizeName VARCHAR(100),
    ProductWeight DECIMAL(8,2),
    UnitWeightCode VARCHAR(6),
    UnitWeightName VARCHAR(100),
    SellStartDate DATETIME,
    SellEndDate DATETIME,
    DiscontinuedDate DATETIME,
    ValidityDate_Start DATETIME,
    ValidityDate_End DATETIME
);
GO

-- Bulk insert from CSV
BULK INSERT stg.Products_Staging
FROM 'C:\Data\ProductsUpdates.csv'
WITH (
    FIRSTROW = 2,                 -- Skip header row
    FIELDTERMINATOR = ',',        -- Comma delimited
    ROWTERMINATOR = '\n',         -- Newline
    TABLOCK,                      -- Performance optimization
    ERRORFILE = 'C:\Data\Errors\Products_Errors.txt',
    MAXERRORS = 10
);
GO

-- Verify load
SELECT COUNT(*) AS RowsLoaded FROM stg.Products_Staging;
GO

-- Call stored procedure to process SCD logic
EXEC integration.LoadUpdatesProduct;
GO

PRINT 'Product load completed successfully';
GO
```

#### Execute SQLCMD Script
```cmd
REM Execute SQL script with variables
sqlcmd -S YOUR_SERVER ^
    -d AdventureWorks2022DWH ^
    -i Scripts\SQLCMD-Load-Products.sql ^
    -v CSVPath="C:\Data\ProductsUpdates.csv" ^
    -o "C:\Logs\Load_Products_%date:~-4,4%%date:~-10,2%%date:~-7,2%.log"
```

### Batch Script for Automated Loading

**Complete batch file combining BCP and SQLCMD:**

```batch
@ECHO OFF
REM File: BCP-Import-Products.cmd
REM Purpose: Automated product CSV import and ETL processing

SETLOCAL

REM Configuration
SET SERVER=YOUR_SQL_SERVER
SET DATABASE=AdventureWorks2022DWH
SET CSV_FILE=C:\Data\ProductsUpdates.csv
SET LOG_PATH=C:\Logs
SET ERROR_PATH=C:\Data\Errors

REM Create directories if not exist
IF NOT EXIST "%LOG_PATH%" MKDIR "%LOG_PATH%"
IF NOT EXIST "%ERROR_PATH%" MKDIR "%ERROR_PATH%"

REM Set timestamp for logging
SET TIMESTAMP=%date:~-4,4%%date:~-10,2%%date:~-7,2%_%time:~0,2%%time:~3,2%%time:~6,2%
SET TIMESTAMP=%TIMESTAMP: =0%

ECHO ========================================
ECHO Product ETL Process Started
ECHO Time: %TIMESTAMP%
ECHO ========================================

REM Step 1: Verify CSV file exists
IF NOT EXIST "%CSV_FILE%" (
    ECHO ERROR: CSV file not found: %CSV_FILE%
    EXIT /B 1
)
ECHO [OK] CSV file found: %CSV_FILE%

REM Step 2: Create staging table
ECHO [RUNNING] Creating staging table...
sqlcmd -S %SERVER% -d %DATABASE% -Q "IF OBJECT_ID('stg.Products_Staging', 'U') IS NOT NULL DROP TABLE stg.Products_Staging; CREATE TABLE stg.Products_Staging (ProductNumber VARCHAR(50), ProductID INT, ProductName VARCHAR(100), ListPrice MONEY, StandardCost MONEY);" 
IF %ERRORLEVEL% NEQ 0 (
    ECHO [ERROR] Failed to create staging table
    EXIT /B 1
)
ECHO [OK] Staging table created

REM Step 3: BCP bulk load
ECHO [RUNNING] Bulk loading CSV file...
bcp AdventureWorks2022DWH.stg.Products_Staging IN "%CSV_FILE%" ^
    -S %SERVER% -T -c -t "," -F 2 ^
    -e "%ERROR_PATH%\BCP_Errors_%TIMESTAMP%.log"
IF %ERRORLEVEL% NEQ 0 (
    ECHO [ERROR] BCP import failed. Check error log.
    EXIT /B 1
)
ECHO [OK] CSV loaded into staging

REM Step 4: Execute ETL stored procedure
ECHO [RUNNING] Processing SCD logic...
sqlcmd -S %SERVER% -d %DATABASE% -Q "EXEC integration.LoadUpdatesProduct;" ^
    -o "%LOG_PATH%\ETL_Log_%TIMESTAMP%.txt"
IF %ERRORLEVEL% NEQ 0 (
    ECHO [ERROR] ETL processing failed
    EXIT /B 1
)
ECHO [OK] ETL processing completed

REM Step 5: Archive CSV file
ECHO [RUNNING] Archiving source file...
MOVE "%CSV_FILE%" "%CSV_FILE%.processed_%TIMESTAMP%"
ECHO [OK] File archived

ECHO ========================================
ECHO Product ETL Process Completed Successfully
ECHO ========================================

ENDLOCAL
EXIT /B 0
```

#### Schedule with SQL Server Agent

```sql
-- Create SQL Agent Job for daily execution
USE msdb;
GO

EXEC dbo.sp_add_job
    @job_name = N'Daily_Product_ETL_Load',
    @enabled = 1,
    @description = N'Loads product updates from CSV using BCP';
GO

EXEC dbo.sp_add_jobstep
    @job_name = N'Daily_Product_ETL_Load',
    @step_name = N'Run BCP Import Script',
    @subsystem = N'CmdExec',
    @command = N'C:\Scripts\BCP-Import-Products.cmd',
    @retry_attempts = 3,
    @retry_interval = 5;
GO

EXEC dbo.sp_add_schedule
    @schedule_name = N'Daily_Midnight',
    @freq_type = 4,              -- Daily
    @freq_interval = 1,
    @active_start_time = 000000; -- Midnight
GO

EXEC dbo.sp_attach_schedule
    @job_name = N'Daily_Product_ETL_Load',
    @schedule_name = N'Daily_Midnight';
GO

EXEC dbo.sp_add_jobserver
    @job_name = N'Daily_Product_ETL_Load';
GO
```

---

## Data Model

### Sample CSV Input (ProductsUpdates.csv)

```csv
ProductNumber,ProductID,ProductCategoryID,ProductSubcategoryID,ProductName,ProductCategoryName,ProductSubcategoryName,ProductColor,ProductStyle,ProductLine,StandardCost,ListPrice,ProductSize,UnitSizeCode,UnitSizeName,ProductWeight,UnitWeightCode,UnitWeightName,SellStartDate,SellEndDate,DiscontinuedDate,ValidityDate_Start,ValidityDate_End
BB-3184,437,,,Road Bottom Bracket,,,,,,,0.00,0.00,,,,,,2009-03-15 00:00:00.000,,,2020-06-18 00:00:00.000,2030-02-14 00:00:00.000
FR-R92R-58,748,1,2,HL Road Frame - Red 58,Bikes,Road Bikes,Red,W,R,868.6342,1431.50,58,CM,centimeters,2.24,LB,pounds,2011-05-31 00:00:00.000,2012-05-30 00:00:00.000,,2024-11-05 00:00:00.000,9999-12-31 00:00:00.000
```

### Output Data (prod.DimProduct)

```sql
ProductDWKey | ProductID | ProductNumber | ProductName | ListPrice | StandardCost | IsCurrent | ValidityDate_Start | ValidityDate_End
1            | 437       | BB-3184       | Road Bottom  | 0.00      | 0.00         | 1         | 2024-11-05        | 9999-12-31
2            | 748       | FR-R92R-58    | HL Road Frame| 1431.50   | 868.63       | 1         | 2024-11-05        | 9999-12-31
```

---

## Performance Metrics

| Metric | Value | Details |
|--------|-------|---------|
| **CSV File Size** | 8.6 KB | 104 product records |
| **Package Execution Time** | <30 seconds | Single file processing |
| **Rows Processed** | 104 rows | From sample CSV |
| **New Products Inserted** | 26 rows | No Match from lookup |
| **SCD Type 1 Updates** | 14 rows | ProductName changes |
| **SCD Type 2 Versions** | 12 rows | ListPrice changes |
| **No Change (Skipped)** | 52 rows | Identical records |
| **Total DimProduct Records** | 504 rows | Active products in DWH |

---

## Related Projects

This SSIS package is part of the **AdventureWorks Sales Analytics Platform**:

### Core Data Warehouse
- 📦 **Repository**: [adventureworks-sales-dwh](https://github.com/aharkane/adventureworks-sales-dwh)
- ⭐ **Description**: Enterprise DWH with metadata-driven ETL, 19 stored procedures, star schema
- 🔗 **Connection**: This package loads prod.DimProduct in the DWH

### Project Ecosystem

```
┌─────────────────────────────────────────────┐
│  AdventureWorks2022 OLTP (Source System)   │
└─────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  [adventureworks-sales-dwh]                │
│  Enterprise Data Warehouse                  │
│  • 19 T-SQL Stored Procedures              │
│  • Metadata-Driven Framework               │
│  • 5 Dimensions + 1 Fact Table             │
│  • SCD Type 0/1/2 Implementation           │
└─────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  [adventureworks-products-etl] ← YOU ARE HERE
│  SSIS Product Loading Package              │
│  • CSV File Processing                     │
│  • 2 SSIS Packages                         │
│  • Loads → prod.DimProduct                 │
└─────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────┐
│  [Future: adventureworks-powerbi-analytics]│
│  Power BI Dashboards                       │
│  • Product Performance Analysis            │
│  • Interactive Visualizations              │
└─────────────────────────────────────────────┘
```

---

## Monitoring & Troubleshooting

### Check SSIS Execution Logs
```sql
-- Query execution history from SSIS Catalog
SELECT 
    execution_id,
    folder_name,
    project_name,
    package_name,
    status,
    start_time,
    end_time,
    DATEDIFF(SECOND, start_time, end_time) AS duration_seconds
FROM [SSISDB].[catalog].[executions]
WHERE package_name LIKE '%Product%'
ORDER BY execution_id DESC;
```

### Validate Data Load
```sql
-- Check recent product loads
SELECT 
    ProductID,
    ProductName,
    ListPrice,
    IsCurrent,
    ValidityDate_Start,
    ValidityDate_End
FROM prod.DimProduct
WHERE ValidityDate_Start >= CAST(GETDATE() AS DATE)
ORDER BY ValidityDate_Start DESC;

-- Count SCD Type 2 versions
SELECT 
    ProductID,
    COUNT(*) AS version_count
FROM prod.DimProduct
GROUP BY ProductID
HAVING COUNT(*) > 1
ORDER BY version_count DESC;
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| CSV file not found | Incorrect path | Verify file path in Flat File Connection Manager |
| Data type mismatch | Column format mismatch | Check column data types in CSV vs. destination |
| Lookup failure | Connection string error | Test OLE DB connection to DWH |
| Truncation errors | Column size too small | Adjust VARCHAR lengths in destination table |
| Permission denied | SQL Server permissions | Grant INSERT/UPDATE on prod.DimProduct |

---

## Author

**Harkane Amine**
- 💼 LinkedIn: [Harkane Amine](https://www.linkedin.com/in/aharkane/)
- 🐙 GitHub: [@aharkane](https://github.com/aharkane)
- 📧 Email: [harkaneamine@gmail.com](mailto:harkaneamine@gmail.com)

### Project Evolution

This project demonstrates practical ETL automation skills:
- Phase 1 ✅: Built enterprise DWH with metadata-driven framework
- Phase 2 ✅: Automated product loading with SSIS (this project)
- Phase 3 🔜: Power BI analytics dashboards
- Phase 4 🔜: Real-time streaming analytics

---

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## Acknowledgments

- **Microsoft** - AdventureWorks sample database
- **Kimball Group** - Dimensional modeling methodology
- **SSIS Documentation** - Integration Services reference

---

<div align="center">

**⭐ If you found this project helpful, please give it a star! ⭐**

Made with ❤️ and ☕ by [Harkane Amine](https://github.com/aharkane)

*Last Updated: November 2025*

</div>
