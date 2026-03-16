# 🏢 ODS Enterprise – Building an Operational Data Store with SSIS

A complete ETL solution for building an Operational Data Store (ODS) using SQL Server Integration Services (SSIS). This project demonstrates how to integrate data from multiple source systems (Sales and HR) into a centralized ODS for operational reporting and analytics.

---

## 📌 Table of Contents

- [🔎 Project Overview](#-project-overview)
- [🏛 ODS Architecture](#-ods-architecture)
- [🚀 Final Goals](#-final-goals)
- [🏁 Competitors](#-competitors)
- [❗Key Technical Challenges & Roadblocks](#key-technical-challenges--roadblocks)
- [💡 Proposed Solutions](#-proposed-solutions)
- [📈 System Architecture](#-system-architecture)
- [🔧 Features](#-features)
- [🧪 ETL Package Phases](#-etl-package-phases)
- [🧬 Data Flow Diagram](#-data-flow-diagram)
- [🗂 Directory Structure](#-directory-structure)
- [📦 Tech Stack](#-tech-stack)
- [🗓 Roadmap](#-roadmap)
- [🧾 License](#-license)
- [👨‍💻 Author](#-author)
- [📬 Future Improvements](#-future-improvements)
- [🙋‍♂️ Contributing](#-contributing)
- [📞 Contact](#-contact)

---

## 🔎 Project Overview

### What is an ODS?
An **Operational Data Store (ODS)** is an integrated database that supports operational reporting and analysis by consolidating data from multiple source systems. It provides:

- 🔄 Near real-time or batch-updated data
- 📊 Subject-oriented organization
- 🧹 Data cleansing and quality enforcement
- 📈 Support for operational decision-making

This project implements a complete ODS solution that:
- 📦 Integrates data from **SourceDB_Sales** (Customers, Products, SalesOrders, OrderDetails)
- 🏢 Integrates data from **SourceDB_HR** (Departments, Employees, Attendance, Payroll)
- 🗄️ Loads into **ODS_Enterprise** database with 3-schema architecture
- ⚙️ Uses SSIS for ETL processing with incremental and full load patterns

---

## 🏛 ODS Architecture

### **Database Structure**
```
📦 ODS_Enterprise
├── 📁 stg (Raw Data)
│   ├── stg.Customers
│   ├── stg.Products
│   ├── stg.SalesOrders
│   ├── stg.OrderDetails
│   ├── stg.Departments
│   ├── stg.Employees
│   ├── stg.Attendance
│   └── stg.Payroll
│
├── 📁 int (Cleaned & Transformed Data)
│   ├── int.Customers
│   ├── int.Products
│   ├── int.SalesOrders
│   ├── int.OrderDetails
│   ├── int.Departments
│   ├── int.Employees
│   ├── int.Attendance
│   └── int.Payroll
│
└── 📁 meta (ETL Control & Logging)
    ├── meta.LoadControl
    ├── meta.ETLLog
    └── meta.ErrorLog
```

### **Source Systems**
| Source Database | Tables |
|-----------------|--------|
| **SourceDB_Sales** | Customers, Products, SalesOrders, OrderDetails |
| **SourceDB_HR** | Departments, Employees, Attendance, Payroll |

---

## 🚀 Final Goals

- ✅ **3-Schema Architecture**: Implement stg, int, and meta schemas in SQL Server
- ✅ **Incremental Dimension Loads**: SCD Type 1 for Customers, Products, Employees
- ✅ **Full Load Dimensions**: Departments (no incremental needed)
- ✅ **Fact Table Loads**: SalesOrders, OrderDetails with lookup transformations
- ✅ **Master Package Orchestration**: Control flow ensuring dimensions before facts
- ✅ **ETL Logging & Monitoring**: Track row counts, load times, errors
- ✅ **Error Handling**: Event handlers and email notifications
- ✅ **Performance Optimization**: Fast Load, batch sizing, indexing
- ✅ **Deployment Ready**: SSIS Catalog deployment with SQL Agent scheduling

---

## 🏁 Competitors

Several approaches and tools can be used to build ODS solutions:

- **SSIS (SQL Server Integration Services)**: Microsoft's enterprise ETL tool (used in this project)
- **Azure Data Factory**: Cloud-based ETL with similar capabilities
- **Talend**: Open-source ETL alternative
- **Informatica PowerCenter**: Enterprise-grade data integration
- **Custom Python ETL**: More flexible but requires more development

This SSIS-based approach offers:
- ✅ Visual development environment
- ✅ Built-in transformations and connectors
- ✅ Native SQL Server integration
- ✅ Enterprise-grade scheduling and monitoring
- ✅ Comprehensive error handling

---

## ❗Key Technical Challenges & Roadblocks

- **Incremental Load Logic**: Identifying and processing only changed records efficiently
- **Lookup Dependencies**: Ensuring dimension keys exist before loading facts
- **Data Quality**: Handling nulls, duplicates, and invalid data from sources
- **Performance**: Optimizing large volume data flows
- **Error Recovery**: Managing failures without data corruption
- **SCD Type 1 Implementation**: Correctly updating existing records
- **Master Package Orchestration**: Proper sequencing of dependent packages
- **Connection Management**: Securing and managing multiple database connections

---

## 💡 Proposed Solutions

- **Variable-Based Incremental Logic**: Use LastLoadDate variables for change detection
- **Dimension-First Loading**: Load all dimensions before facts in Master Package
- **Lookup Transformations**: Cache dimension keys for faster fact loading
- **Data Conversion Components**: Handle data type mismatches between source and destination
- **Event Handlers**: Log errors and send notifications on failure
- **Row Count Tracking**: Monitor extracted and loaded rows for validation
- **Fast Load Mode**: Optimize OLE DB Destination with batch commits
- **Index Strategy**: Create indexes on lookup columns for performance

---

## 📈 System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        SOURCE SYSTEMS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌───────────────────┐              ┌───────────────────┐        │
│  │   SourceDB_Sales  │              │    SourceDB_HR    │        │
│  │                   │              │                   │        │
│  │ • Customers       │              │ • Departments     │        │
│  │ • Products        │              │ • Employees       │        │
│  │ • SalesOrders     │              │ • Attendance      │        │
│  │ • OrderDetails    │              │ • Payroll         │        │
│  └─────────┬─────────┘              └─────────┬─────────┘        │
│            │                                   │                  │
└────────────┼───────────────────────────────────┼──────────────────┘
             │                                   │
             ▼                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                        SSIS ETL LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   MASTER PACKAGE                          │     │
│  │  ┌─────────────────┐    ┌─────────────────┐            │     │
│  │  │   Sequence 1    │───▶│   Sequence 2    │            │     │
│  │  │ Load Dimensions │    │   Load Facts    │            │     │
│  │  └────────┬────────┘    └────────┬────────┘            │     │
│  └───────────┼───────────────────────┼────────────────────┘     │
│              │                       │                            │
│  ┌───────────▼───────────┐  ┌────────▼───────────┐               │
│  │ Dimension Packages    │  │ Fact Packages      │               │
│  │ • Load Customers      │  │ • Load SalesOrders │               │
│  │ • Load Products       │  │ • Load OrderDetails│               │
│  │ • Load Employees      │  │ • Load Attendance  │               │
│  │ • Load Departments    │  │ • Load Payroll     │               │
│  └───────────────────────┘  └────────────────────┘               │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      ODS DATABASE LAYER                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                   ODS_Enterprise                          │     │
│  │                                                           │     │
│  │  ┌──────────────┐    ┌──────────────┐    ┌────────────┐ │     │
│  │  │    stg       │    │     int      │    │    meta    │ │     │
│  │  │ (Raw Data)   │───▶│ (Cleaned)    │    │ (Control)  │ │     │
│  │  └──────────────┘    └──────────────┘    └────────────┘ │     │
│  │                                                           │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Features

- ✅ **3-Schema Architecture**: Separate layers for staging, integration, and metadata
- ✅ **Incremental Dimension Loads**: SCD Type 1 for slowly changing dimensions
- ✅ **Full Load Options**: For small, static dimension tables
- ✅ **Lookup Transformations**: Key resolution for fact tables
- ✅ **Data Conversion**: Handle type mismatches between systems
- ✅ **Row Count Monitoring**: Track ETL performance and data volume
- ✅ **Error Handling**: Event handlers and logging
- ✅ **Master Package Orchestration**: Coordinated execution of all packages
- ✅ **Connection Management**: Centralized connection managers
- ✅ **SSIS Catalog Deployment**: Enterprise-ready deployment
- ✅ **SQL Agent Scheduling**: Automated execution
- ✅ **Performance Optimization**: Fast Load, batch sizing, indexing

---

## 🧪 ETL Package Phases

<details>
<summary>📦 Package 1: Load Customers (Incremental Dimension)</summary>

**Variables:**
| Variable | Type | Purpose |
|----------|------|---------|
| LastLoadDate | DateTime | Previous ETL run timestamp |
| CurrentLoadDate | DateTime | Current ETL run timestamp |
| RowsExtracted | Int | Count of rows from source |
| RowsLoaded | Int | Count of rows loaded to int |

**Control Flow Steps:**
1. **Get Last Load Date** - Execute SQL Task to retrieve from meta.LoadControl
2. **Truncate Staging** - Clear stg.Customers
3. **Extract to Staging** - Data Flow Task (OLE DB Source → Derived Column → Row Count → OLE DB Destination)
4. **Merge to Integration** - Execute SQL Task with MERGE statement (SCD Type 1)
5. **Update Load Control** - Update meta.LoadControl with CurrentLoadDate
6. **Log Execution** - Insert record into meta.ETLLog

**Data Flow Components:**
- **OLE DB Source**: `SELECT * FROM SourceDB_Sales.dbo.Customers WHERE ModifiedDate > ?`
- **Derived Columns**: Add SourceSystem = "SourceDB_Sales", LoadDate = GETDATE()
- **Row Count**: Store count in RowsExtracted variable
- **OLE DB Destination**: Insert into stg.Customers

**Merge Logic (SCD Type 1):**
```sql
MERGE int.Customers AS target
USING stg.Customers AS source
ON target.CustomerID = source.CustomerID
WHEN MATCHED THEN
    UPDATE SET 
        FirstName = source.FirstName,
        LastName = source.LastName,
        Email = source.Email,
        ModifiedDate = source.ModifiedDate
WHEN NOT MATCHED THEN
    INSERT (CustomerID, FirstName, LastName, Email, ModifiedDate, SourceSystem)
    VALUES (source.CustomerID, source.FirstName, source.LastName, 
            source.Email, source.ModifiedDate, source.SourceSystem);
```

</details>

<details>
<summary>📦 Package 2: Load Products (Incremental Dimension)</summary>

Same pattern as Customers but for:
- Source: SourceDB_Sales.dbo.Products
- Staging: stg.Products
- Integration: int.Products

**Key Differences:**
- Product-specific columns (ProductName, Category, UnitPrice)
- Lookups may be added for category dimension (if exists)

</details>

<details>
<summary>📦 Package 3: Load Departments (Full Load)</summary>

**⚠️ No Incremental Logic - Full Refresh Pattern**

**Control Flow Steps:**
1. **Truncate Staging** - Clear stg.Departments
2. **Extract to Staging** - Full extract from source
3. **Truncate Integration** - Clear int.Departments (full refresh)
4. **Load Integration** - Insert all from staging

**Source Query:**
```sql
SELECT DepartmentID, DepartmentName, ManagerID, CreatedDate
FROM SourceDB_HR.dbo.Departments
```

</details>

<details>
<summary>📦 Package 4: Load Employees (Incremental with Lookup)</summary>

**⚠️ Depends on Departments - Must load Departments first**

**Lookup Transformation:**
- Reference Table: int.Departments
- Join Condition: source.DepartmentID = lookup.DepartmentID
- Return: DepartmentKey

**Control Flow Steps:**
1. Get Last Load Date
2. Truncate Staging
3. Extract to Staging (with Lookup)
4. Merge to Integration
5. Update Load Control
6. Log Execution

**Data Flow Components:**
- OLE DB Source: Extract Employees with ModifiedDate filter
- Lookup: Get DepartmentKey from int.Departments
- Derived Columns: Add metadata columns
- Row Count: Track extracted rows
- OLE DB Destination: Load to stg.Employees

</details>

<details>
<summary>📦 Package 5: Load SalesOrders (Fact Table)</summary>

**Lookups Needed:**
- CustomerKey from int.Customers
- (Optional) ProductKey if at header level (typically in OrderDetails)

**Control Flow Steps:**
1. Truncate Staging
2. Extract to Staging (with Lookups)
3. Load to Integration (Insert Only Pattern)

**Data Flow Components:**
- OLE DB Source: Extract from SourceDB_Sales.dbo.SalesOrders
- Lookup (Customer): Get CustomerKey
- Data Conversion: Handle date formats
- Derived Columns: Add metadata
- OLE DB Destination: Load to stg.SalesOrders

**Post-Load:**
```sql
INSERT INTO int.SalesOrders (OrderID, CustomerKey, OrderDate, TotalAmount, SourceSystem)
SELECT OrderID, CustomerKey, OrderDate, TotalAmount, SourceSystem
FROM stg.SalesOrders
WHERE NOT EXISTS (
    SELECT 1 FROM int.SalesOrders 
    WHERE int.SalesOrders.OrderID = stg.SalesOrders.OrderID
);
```

</details>

<details>
<summary>📦 Package 6: Load OrderDetails (Fact Table)</summary>

**Lookups Needed:**
- OrderKey from int.SalesOrders
- ProductKey from int.Products

**Insert Only Pattern:**
```sql
INSERT INTO int.OrderDetails (OrderDetailID, OrderKey, ProductKey, Quantity, UnitPrice, SourceSystem)
SELECT od.OrderDetailID, so.OrderKey, p.ProductKey, od.Quantity, od.UnitPrice, 'SourceDB_Sales'
FROM stg.OrderDetails od
JOIN int.SalesOrders so ON od.OrderID = so.OrderID
JOIN int.Products p ON od.ProductID = p.ProductID
WHERE NOT EXISTS (
    SELECT 1 FROM int.OrderDetails 
    WHERE int.OrderDetails.OrderDetailID = od.OrderDetailID
);
```

</details>

<details>
<summary>📦 Package 7: Master Package (Orchestration)</summary>

**Structure:**
```
┌─────────────────────────────────────────────────────────────┐
│                    MASTER PACKAGE                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Sequence 1: Load Dimensions                         │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Departments │ (Full Load - No Dependency)    │   │
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Customers │ (Incremental)                    │   │
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Products  │ (Incremental)                    │   │
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Employees │ (Depends on Departments)         │   │
│  │  └──────────────┘                                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                              │                              │
│                              ▼                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Sequence 2: Load Facts                              │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load SalesOrders│ (Depends on Customers)          │   │
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load OrderDetails│ (Depends on SalesOrders & Products)│
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Attendance │ (Depends on Employees)          │   │
│  │  └──────────────┘                                    │   │
│  │         ▼                                            │   │
│  │  ┌──────────────┐                                    │   │
│  │  │ Load Payroll   │ (Depends on Employees)           │   │
│  │  └──────────────┘                                    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**⚠️ Key Principle: Dimensions always before Facts**

</details>

<details>
<summary>🧪 Testing & Validation</summary>

**Test 1: Run Individual Package**
```sql
EXEC SSISDB.Catalog.create_execution ... -- Execute specific package
```

**Test 2: Incremental Testing**
```sql
-- Update source record
UPDATE SourceDB_Sales.dbo.Customers 
SET Email = 'new.email@example.com', ModifiedDate = GETDATE()
WHERE CustomerID = 100;

-- Run package again → Verify only changed row updated
```

**Validation Queries:**
```sql
-- Data Counts
SELECT 'Source' as Source, COUNT(*) as RowCount FROM SourceDB_Sales.dbo.Customers
UNION ALL
SELECT 'Staging', COUNT(*) FROM ODS_Enterprise.stg.Customers
UNION ALL
SELECT 'Integration', COUNT(*) FROM ODS_Enterprise.int.Customers;

-- Load Times
SELECT PackageName, StartTime, EndTime, DATEDIFF(second, StartTime, EndTime) as DurationSec
FROM meta.ETLLog
ORDER BY StartTime DESC;

-- Data Quality
SELECT COUNT(*) as NullEmails 
FROM int.Customers 
WHERE Email IS NULL;
```

</details>

---

## 🧬 Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOURCE SYSTEMS                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │  SourceDB_Sales                          SourceDB_HR    │     │
│  │  ┌──────────────┐  ┌──────────────┐     ┌────────────┐ │     │
│  │  │  Customers   │  │   Products   │     │Departments │ │     │
│  │  └──────┬───────┘  └───────┬──────┘     └─────┬──────┘ │     │
│  │  ┌──────▼───────┐  ┌───────▼──────┐     ┌─────▼──────┐ │     │
│  │  │ SalesOrders  │  │ OrderDetails │     │ Employees  │ │     │
│  │  └──────┬───────┘  └───────┬──────┘     └─────┬──────┘ │     │
│  │         │                  │                   │        │     │
│  └─────────┼──────────────────┼───────────────────┼────────┘     │
│            │                  │                   │               │
└────────────┼──────────────────┼───────────────────┼───────────────┘
             │                  │                   │
             ▼                  ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                        STAGING LAYER (stg)                       │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │stg.Customers │  │stg.Products  │  │stg.Departments│           │
│  └──────┬───────┘  └───────┬──────┘  └───────┬──────┘           │
│  ┌──────▼───────┐  ┌───────▼──────┐  ┌───────▼──────┐           │
│  │stg.SalesOrders│ │stg.OrderDetails│ │stg.Employees │           │
│  └──────┬───────┘  └───────┬──────┘  └───────┬──────┘           │
│         │                  │                   │                 │
└─────────┼──────────────────┼───────────────────┼─────────────────┘
          │                  │                   │
          ▼                  ▼                   ▼
┌─────────────────────────────────────────────────────────────────┐
│                  INTEGRATION LAYER (int)                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │int.Customers │  │int.Products  │  │int.Departments│           │
│  └──────┬───────┘  └───────┬──────┘  └───────┬──────┘           │
│  ┌──────▼───────┐  ┌───────▼──────┐  ┌───────▼──────┐           │
│  │int.SalesOrders│ │int.OrderDetails│ │int.Employees │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      METADATA LAYER (meta)                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐                      │
│  │  meta.LoadControl│  │   meta.ETLLog    │                      │
│  │  - LastLoadDate  │  │  - PackageName   │                      │
│  │  - TableName     │  │  - StartTime     │                      │
│  │  - Status        │  │  - EndTime       │                      │
│  └──────────────────┘  │  - RowsExtracted │                      │
│  ┌──────────────────┐  │  - RowsLoaded    │                      │
│  │  meta.ErrorLog   │  │  - Status        │                      │
│  │  - ErrorDateTime │  └──────────────────┘                      │
│  │  - PackageName   │                                            │
│  │  - ErrorMessage  │                                            │
│  └──────────────────┘                                            │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗂 Directory Structure

```
ODS_ETL_Project/
│
├── README.md                          # Project documentation
├── ODS_ETL_Project.sln                 # Visual Studio solution
│
├── Packages/
│   ├── Master Package.dtsx             # Main orchestration package
│   ├── Load Customers.dtsx              # Incremental dimension
│   ├── Load Products.dtsx               # Incremental dimension
│   ├── Load Departments.dtsx            # Full load dimension
│   ├── Load Employees.dtsx              # Incremental with lookup
│   ├── Load SalesOrders.dtsx            # Fact table
│   ├── Load OrderDetails.dtsx           # Fact table with lookups
│   ├── Load Attendance.dtsx             # Fact table
│   └── Load Payroll.dtsx                 # Fact table
│
├── Scripts/
│   ├── Create_SourceDB_Sales.sql        # Source database creation
│   ├── Create_SourceDB_HR.sql           # Source database creation
│   ├── Create_ODS_Enterprise.sql        # ODS database with schemas
│   ├── Create_Staging_Tables.sql        # stg schema tables
│   ├── Create_Integration_Tables.sql    # int schema tables
│   ├── Create_Metadata_Tables.sql       # meta schema tables
│   ├── Sample_Data_Insert.sql           # Test data
│   └── Validation_Queries.sql           # Data validation queries
│
├── Connection Managers/
│   ├── SourceDB_Sales.connmgr            # Sales source connection
│   ├── SourceDB_HR.connmgr               # HR source connection
│   └── ODS_Enterprise.connmgr            # ODS destination connection
│
├── Documentation/
│   ├── architecture_diagram.png
│   ├── data_flow_diagram.png
│   ├── package_dependencies.md
│   └── deployment_guide.md
│
├── Project.params                        # Project parameters
│
└── .gitignore
```

---

## 📦 Tech Stack

| Category             | Tool/Library                | Purpose                               |
|----------------------|----------------------------|---------------------------------------|
| **Database**         | SQL Server 2019/2022        | Source and ODS databases              |
| **ETL Tool**         | SSIS (SQL Server Integration Services) | Data extraction, transformation, loading |
| **Development**      | SSDT / Visual Studio + SSIS Extension | Package development |
| **Management**       | SSMS (SQL Server Management Studio) | Database management and validation |
| **Scheduling**       | SQL Server Agent            | Automated execution                   |
| **Deployment**       | SSIS Catalog (SSISDB)       | Enterprise package deployment         |
| **Version Control**  | Git + GitHub                | Code management                       |
| **Documentation**    | Markdown                    | Project documentation                 |

---

## 🗓 Roadmap

| Phase | Description | Status |
|-------|-------------|--------|
| ✅ 1 | Source Database Creation | ✅ Done |
| ✅ 2 | ODS Database Creation with Schemas | ✅ Done |
| ✅ 3 | Connection Manager Setup | ✅ Done |
| ✅ 4 | Dimension Load Packages (Incremental) | ✅ Done |
| ✅ 5 | Dimension Load Packages (Full Load) | ✅ Done |
| ✅ 6 | Fact Load Packages with Lookups | ✅ Done |
| ✅ 7 | Master Package Orchestration | ✅ Done |
| ✅ 8 | Error Handling & Logging | ✅ Done |
| ✅ 9 | Testing & Validation | ✅ Done |
| ✅ 10 | Documentation | ✅ Done |
| 🔄 11 | Performance Optimization | 📅 Planned |
| 🔄 12 | Real-time Integration | 📅 Planned |

---

## 🧾 License

No license has been selected for this project yet.
All rights reserved — you may not use, copy, modify, or distribute this code without explicit permission from the author.

---

## 👨‍💻 Author

**Manar Altyp**  
*Data Engineer • ETL Specialist • BI Developer*

🌐 [GitHub](https://github.com/manar-Lang) 

*Project completed for: Building an Operational Data Store (ODS) with SSIS*  
*Completion Date: March 2026*

---

## 📬 Future Improvements

- **Real-Time Integration**
  - Implement Change Data Capture (CDC) for near real-time updates
  - Add Kafka or Azure Event Hub integration
  - Reduce latency to minutes instead of hours

- **Performance Optimization**
  - Implement parallel processing for large fact tables
  - Use partitioning in integration tables
  - Add columnstore indexes for analytics

- **Data Quality Framework**
  - Implement comprehensive data quality rules
  - Add data profiling before staging
  - Create data quality dashboards

- **Monitoring & Alerting**
  - Build real-time monitoring dashboard
  - Set up email/Slack alerts for failures
  - Track SLA compliance

- **Cloud Migration**
  - Migrate to Azure SQL Database
  - Use Azure Data Factory for cloud ETL
  - Implement hybrid architecture

- **Advanced Error Handling**
  - Automatic retry logic
  - Dead letter queue for failed records
  - Data reconciliation reports

- **Metadata-Driven ETL**
  - Build framework that reads table configurations
  - Dynamic package generation
  - Reduce maintenance overhead

---

## ✅ Best Practices Implemented

### **Naming Conventions**
- Packages: `Load_[EntityName].dtsx`
- Variables: PascalCase (LastLoadDate, RowsExtracted)
- Connection Managers: Descriptive source names

### **Error Handling**
- Event Handlers for OnError events
- Log errors to meta.ErrorLog
- Email notifications for critical failures

### **Performance Optimization**
- Fast Load mode with batch commits (10K–50K rows)
- Indexes on lookup columns
- TABLOCK hint for large inserts
- Partitioned staging tables

### **Monitoring**
- Row count tracking at each stage
- Execution time logging
- Status tracking in meta.LoadControl

### **Security**
- Windows Authentication
- SSIS Catalog encryption
- Least privilege principle
- Encrypted sensitive data

### **Deployment**
- SSIS Catalog (SSISDB) deployment
- Environment-specific configurations
- SQL Agent jobs for scheduling
- Off-peak hour execution

---

## 🚀 Deployment Checklist

### **Before Deployment**
- [ ] All packages tested individually
- [ ] Master package tested end-to-end
- [ ] Logging verified in meta.ETLLog
- [ ] Error handling implemented
- [ ] Connection managers configured
- [ ] Project parameters set

### **Deployment Steps**
1. Build Solution in Visual Studio
2. Deploy to SSIS Catalog (SSISDB)
3. Create SQL Agent Job
4. Configure schedule (off-peak hours)
5. Set up notifications
6. Test production run

### **After Deployment**
- [ ] Monitor first execution
- [ ] Validate data counts
- [ ] Review performance metrics
- [ ] Check logs for errors
- [ ] Document any issues

---

## 🛠 Troubleshooting Guide

| Issue | Solution |
|-------|----------|
| ❌ Cannot acquire connection | Verify connection managers, Windows authentication |
| ❌ No data in staging | Check LastLoadDate variable, source query filters |
| ❌ Lookup fails | Ensure dimension loaded first, verify join columns |
| ❌ Slow performance | Add indexes, optimize queries, reduce batch size |
| ❌ Duplicate data | Review MERGE logic, check NOT EXISTS conditions |
| ❌ Package fails randomly | Add retry logic, check memory pressure |
| ❌ Date conversion errors | Use Data Conversion component, check source formats |

---

## 📊 Sample Validation Queries

```sql
-- Check row counts across layers
SELECT 'Source_Customers' as TableName, COUNT(*) as RowCount FROM SourceDB_Sales.dbo.Customers
UNION ALL
SELECT 'Stg_Customers', COUNT(*) FROM ODS_Enterprise.stg.Customers
UNION ALL
SELECT 'Int_Customers', COUNT(*) FROM ODS_Enterprise.int.Customers;

-- Check for orphaned facts
SELECT COUNT(*) as OrphanedOrderDetails
FROM int.OrderDetails od
LEFT JOIN int.SalesOrders so ON od.OrderKey = so.OrderKey
WHERE so.OrderKey IS NULL;

-- View recent ETL runs
SELECT 
    PackageName,
    StartTime,
    EndTime,
    DATEDIFF(second, StartTime, EndTime) as DurationSeconds,
    RowsExtracted,
    RowsLoaded,
    Status
FROM meta.ETLLog
ORDER BY StartTime DESC;
```

---

## 🙋‍♂️ Contributing

Contributions are welcome! This is an educational project, and improvements are appreciated. Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

Please ensure your code follows SSIS best practices and includes appropriate documentation.

---

## 📞 Contact

For questions, suggestions, or feedback about this project:

**Manar Altyp**
📧 Email: manaraltyp44444@gmail.com
🐙 GitHub: [Manar Altyp](https://github.com/manar-Lang)


---

## 📋 Project Summary for CV

**ODS Enterprise – Building Operational Data Store with SSIS**

Designed and implemented a complete Operational Data Store (ODS) solution using SQL Server Integration Services (SSIS) to integrate data from multiple source systems (Sales and HR). Built a 3-schema architecture (stg for raw data, int for cleaned data, meta for ETL control) supporting near real-time operational reporting. Developed 7+ SSIS packages for incremental and full loads of dimensions (Customers, Products, Employees, Departments) and facts (SalesOrders, OrderDetails, Attendance, Payroll). Implemented SCD Type 1 logic with lookup transformations, data conversion, and merge patterns. Created a Master Package for orchestration ensuring proper load order. Established ETL logging, row count monitoring, and validation queries for data quality assurance.

**Tools:** SQL Server, SSIS, SSMS, SSDT, T-SQL, ODS Architecture

---

*Last Updated: March 2026*
