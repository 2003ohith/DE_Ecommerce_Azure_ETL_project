# E-Commerce Data Engineering Project

## Overview

This project implements an Azure-based Medallion Architecture for processing and transforming e-commerce data into analytics-ready datasets.

### Technology Stack

- Azure Data Lake Storage Gen2 (ADLS Gen2)
- Azure Databricks
- PySpark
- Delta Lake
- Unity Catalog
- Bronze, Silver, and Gold layers
- BI / Analytics Consumers

---

## Architecture Flow

```text
E-Commerce API / Application Data
              |
              v
      Azure Data Lake
        Storage Gen2
       Landing Zone
              |
              v
         Databricks
     PySpark + Delta Lake
              |
       +------+------+
       |             |
       v             v
    Bronze         Silver
       |             |
       +------>------+
              |
              v
             Gold
              |
              v
      BI / Analytics
         Consumers
```
#1. Source Systems

The project receives e-commerce data from multiple source systems.

**Sources**
- E-commerce API / Application Data
- CSV Files
- Transactional Source Data

**The source data can contain information related to:**

- Customers
- Products
- Categories
- Brands
- Orders
- Sales transactions

#2. Azure Data Lake Storage Gen2

The source data is stored in Azure Data Lake Storage Gen2 (ADLS Gen2).

The ADLS Gen2 Landing Zone acts as the initial storage location for incoming raw data.

Example Structure
```text
ADLS Gen2
│
└── Landing Zone
    │
    ├── customers
    ├── products
    ├── categories
    ├── brands
    └── orders
```
#3. Databricks

Azure Databricks is used as the main data processing platform.

The project uses:

- PySpark for data transformation
- Delta Lake for reliable table storage
- Databricks for data processing and orchestration of transformation jobs

The data is processed through the Medallion Architecture:
```
Raw → Bronze → Silver → Gold
```
#4. Bronze Layer

The Bronze layer contains the raw data converted into Delta tables.

**Responsibilities**

- Read data from the ADLS Gen2 Landing Zone
- Store raw data as Delta tables
- Preserve source-level information
- Add ingestion metadata
- Perform minimal transformations

Typical Metadata

```
Source_file
Ingested_at
```
Flow

```
ADLS Gen2
    |
    v
Raw Data
    |
    v
Bronze Delta Tables
```
The Bronze layer provides the foundation for downstream transformations.

#5. Silver Layer

The Silver layer contains cleansed, standardized, and refined data.

**Responsibilities:**

- Clean invalid data
- Handle null values
- Handle duplicate records
- Standardize data types
- Maintain schema consistency
- Handle schema drift
- Refine raw datasets
- Prepare trusted datasets for business transformations

Flow

```
Bronze
   |
   +-- Data Cleaning
   |
   +-- Data Validation
   |
   +-- Data Standardization
   |
   +-- Schema Handling
   |
   v
Silver
```
The Silver layer contains trusted datasets that can be used to create analytical models.

#6. Gold Layer

The Gold layer contains analytics-ready and business-oriented datasets.

The project follows a star-schema-style approach for analytical consumption.

- Gold Entities
- Dim Customer
- Dim Product
- Dim Category
- Dim Brand
- Fact Orders / Sales

Example Model
```
                    Dim Customer
                         |
                         |
Dim Product ---- Fact Orders / Sales ---- Dim Category
                         |
                         |
                     Dim Brand
```

**Gold Layer Responsibilities:**

- Join Silver datasets
- Apply business transformations
- Create dimensions
- Create fact datasets
- Add business attributes
- Add regional attributes
- Create analytics-ready datasets

#7. Unity Catalog

Unity Catalog is used for data organization and governance.

The architecture is organized into:
```
Unity Catalog
│
├── Catalog
│
├── Raw / External Volume
│
├── Bronze
│
├── Silver
│
└── Gold
```
Unity Catalog provides centralized management for:

- Catalogs
- Schemas
- Tables
- Volumes
- External Locations
- Credentials
- Access permissions
- Data governance

#8. End-to-End Data Flow

The complete data flow is:
```
Source Systems
      |
      v
ADLS Gen2 Landing Zone
      |
      v
Databricks
      |
      v
Bronze Layer
Raw Delta Data
      |
      v
Silver Layer
Cleansed & Standardized Data
      |
      v
Gold Layer
Analytics-Ready Data
      |
      v
BI / Analytics Consumers
```

#9. Data Layer Responsibilities
| Layer   | Purpose                    | Main Operations                          |
| ------- | -------------------------- | ---------------------------------------- |
| Landing | Store incoming source data | Raw file storage                         |
| Bronze  | Store raw processed data   | Delta conversion, metadata               |
| Silver  | Create trusted data        | Cleaning, validation, standardization    |
| Gold    | Create business-ready data | Joins, dimensions, facts, business rules |
| BI      | Consume analytical data    | Reporting and analytics                  |

#10.Technology Stack

| Technology    | Role                             |
| ------------- | -------------------------------- |
| Azure         | Cloud Platform                   |
| ADLS Gen2     | Data Lake Storage                |
| Databricks    | Data Processing                  |
| PySpark       | Distributed Data Processing      |
| Delta Lake    | Reliable Table Storage           |
| Unity Catalog | Governance and Data Organization |
| BI Tools      | Analytics and Reporting          |

#11. Key Design Principles
- Separation of Data Layers
- Raw, cleansed, and analytical data are separated into different layers.
- Data Quality
- Data is cleaned, validated, standardized, and refined before reaching the Gold layer.
- Reusability
- Silver datasets can be reused by multiple Gold analytical models.
- Governance
- Unity Catalog provides centralized organization and governance of data assets.
- Analytics Readiness
- Gold datasets are designed for BI and analytical consumption.

---

## Getting Started

### Prerequisites

- Azure Databricks workspace with Unity Catalog enabled
- Azure Data Lake Storage Gen2 account linked to the workspace
- Appropriate permissions on the Unity Catalog (CREATE CATALOG, CREATE SCHEMA, CREATE TABLE)

### Notebook Execution Order

Run the notebooks in the following order:

1. `Catalog_setup_and_Volume_creation/catalog_setup.ipynb` — Create the Unity Catalog, schemas, and external volumes pointing to ADLS Gen2.
2. `Medallion_Processing_Dim/Raw-to-Bronze-Dim.ipynb` — Ingest raw dimension data into the Bronze layer.
3. `Medallion_Processing_Dim/Bronze-to-Silver-Dim.ipynb` — Cleanse and standardise dimension data into the Silver layer.
4. `Medallion_Processing_Dim/Silver-to-Gold-Dim.ipynb` — Build Gold-layer dimension tables (Dim Customer, Dim Product, Dim Category, Dim Brand).
5. `Medallion_Processing_Fact/Raw-to-Bronze-Fact.ipynb` — Ingest raw fact/order data into the Bronze layer.
6. `Medallion_Processing_Fact/Bronze-to-Silver-Fact.ipynb` — Cleanse and standardise fact data into the Silver layer.
7. `Medallion_Processing_Fact/Silver-to-Gold-Fact.ipynb` — Build the Gold-layer Fact Orders / Sales table.
