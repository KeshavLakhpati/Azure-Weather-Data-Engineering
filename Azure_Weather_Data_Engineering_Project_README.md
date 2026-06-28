# Azure Weather Data Engineering Project

## Project Overview

This project demonstrates an end-to-end Azure Data Engineering solution
that ingests weather data from the AccuWeather API, processes it using
Azure Databricks with a Medallion Architecture (Bronze, Silver, Gold),
orchestrates workflows using Azure Data Factory (ADF), and prepares
trusted datasets for analytics.

------------------------------------------------------------------------

# Architecture

``` text
AccuWeather API
        │
        ▼
Azure Data Factory
        │
        ▼
Databricks Notebooks
        │
        ▼
Bronze (Raw JSON)
        │
        ▼
Silver (Clean Delta)
        │
        ▼
Gold (Business Ready)
        │
        ▼
Validation
        │
        ▼
ADF If Condition
   │             │
SUCCESS       FAILURE
   │             │
Reporting   Fail Activity
```

# Services Used

-   Azure Data Lake Storage Gen2
-   Azure Databricks
-   Azure Data Factory
-   Unity Catalog
-   Delta Lake
-   Azure Managed Identity
-   AccuWeather REST API

# Storage Structure

    bronze/
    silver/
    gold/

-   **Bronze** -- Raw JSON files from API.
-   **Silver** -- Cleaned and transformed Delta tables.
-   **Gold** -- Business-ready reporting tables.

# Project Implementation

## Step 1 -- Azure Resource Setup

Created: - Azure Data Lake Storage Gen2 - Azure Databricks Workspace -
Azure Data Factory - Unity Catalog

------------------------------------------------------------------------

## Step 2 -- Unity Catalog Configuration

Configured:

-   Storage Credential
-   External Location
-   Catalog
-   Schemas

Catalog Structure

    weather_catalog
        ├── bronze
        ├── silver
        └── gold

------------------------------------------------------------------------

## Step 3 -- API Integration

### Location Master API

Fetches:

-   English Name
-   Location Key
-   Latitude
-   Longitude

Cities:

-   Pune
-   Mumbai
-   Delhi

### Forecast API

Using the Location Key, fetches:

-   5-Day Forecast
-   Rain
-   Weather Description
-   Thunderstorm Probability

------------------------------------------------------------------------

## Step 4 -- Databricks ETL

### Location Master

-   01_location_master_ingestion
-   02_location_master_bronze_to_silver
-   03_location_master_silver_to_gold
-   07_validate_location_master

### Forecast

-   04_forecast_ingestion
-   05_forecast_bronze_to_silver
-   06_forecast_silver_to_gold
-   08_validate_forecast

Each notebook performs a single responsibility.

------------------------------------------------------------------------

## Step 5 -- Bronze Layer

-   Calls REST APIs.
-   Stores raw JSON files in ADLS Bronze.

Example

    pune_YYYYMMDD.json
    mumbai_YYYYMMDD.json
    delhi_YYYYMMDD.json

------------------------------------------------------------------------

## Step 6 -- Bronze to Silver

-   Reads JSON.
-   Flattens nested objects.
-   Cleans data.
-   Selects required columns.
-   Writes Delta tables.

------------------------------------------------------------------------

## Step 7 -- Silver to Gold

-   Reads Silver tables.
-   Applies business transformations.
-   Writes reporting-ready Gold tables.

------------------------------------------------------------------------

## Step 8 -- Incremental Processing

Initially Spark read all files from Bronze, causing duplicate records.

### Solution

Implemented `pathGlobFilter` with `process_date` to read only the
current day's files.

Benefits:

-   Faster execution
-   No duplicate processing
-   Supports incremental loading

------------------------------------------------------------------------

## Step 9 -- Azure Data Factory

Separate pipelines for:

-   Location Master
-   Forecast

### Activities Used

-   Set Variable
-   Databricks Notebook
-   If Condition
-   Fail Activity

Pipeline Flow

    Set Process Date
          │
          ▼
    Bronze Notebook
          │
          ▼
    Bronze → Silver
          │
          ▼
    Silver → Gold
          │
          ▼
    Validation
          │
          ▼
    If Condition
     ├── PASS → Reporting
     └── FAIL → Fail Activity

------------------------------------------------------------------------

## Step 10 -- Parameterization

Pipeline parameter:

-   process_date

Passed from ADF to Databricks using notebook widgets.

------------------------------------------------------------------------

## Step 11 -- Data Validation

Location Master

-   Row count
-   Duplicate keys
-   Null values
-   Latitude range
-   Longitude range

Forecast

-   Row count
-   Duplicate forecasts
-   Null values
-   Expected locations
-   Forecast record validation

Validation returns:

-   SUCCESS
-   FAIL

ADF proceeds only when validation succeeds.

------------------------------------------------------------------------

## Step 12 -- Security

Implemented:

-   Azure Managed Identity
-   Unity Catalog
-   Storage Credential
-   External Location

No storage keys are required for Delta access.

------------------------------------------------------------------------

## Step 13 -- Delta Lake

Benefits:

-   ACID Transactions
-   Schema Enforcement
-   Reliable Updates
-   Optimized Reads
-   Time Travel Support

------------------------------------------------------------------------

## Step 14 -- Medallion Architecture

    Bronze
       │
    Silver
       │
    Gold

------------------------------------------------------------------------

# Challenges and Solutions

  -----------------------------------------------------------------------
  Challenge                             Solution
  ------------------------------------- ---------------------------------
  Storage Credential SQL failed         Used Databricks SDK with Azure
                                        Managed Identity

  Catalog creation issue                Created catalog using Managed
                                        Location

  Duplicate records                     Implemented incremental loading
                                        using `pathGlobFilter`

  Validation failures                   Added dedicated validation
                                        notebooks and ADF quality gate

  ADF process_date prompt               Used pipeline parameter and
                                        notebook widgets

  Unity Catalog permissions             Configured Storage Credential and
                                        External Location
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# Best Practices Implemented

-   Medallion Architecture
-   Delta Lake
-   Unity Catalog
-   Managed Identity
-   Parameterized Pipelines
-   Incremental File Processing
-   Modular Notebooks
-   Data Validation
-   Error Handling
-   Enterprise ETL Design

------------------------------------------------------------------------

# Technology Stack

-   Azure Data Factory
-   Azure Databricks
-   Azure Data Lake Storage Gen2
-   Unity Catalog
-   Delta Lake
-   PySpark
-   Python
-   REST APIs
-   JSON
-   SQL

------------------------------------------------------------------------

# Future Enhancements

-   Audit Logging
-   Email Notifications
-   Configuration Tables
-   Power BI Monitoring Dashboard
-   CI/CD using Azure DevOps
-   Databricks Workflows

------------------------------------------------------------------------

# Conclusion

This project demonstrates an end-to-end Azure Data Engineering solution
using modern Azure services and enterprise best practices. It covers
secure ingestion, governed storage, transformation, orchestration,
incremental processing, validation, and reporting-ready datasets
suitable for production-style implementations.
