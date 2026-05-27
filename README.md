# AdventureWorksLT-Data-Lakehouse-Architecture-ETE-Pipeline

A modern, scalable Data Lakehouse platform built on the Medallion Architecture, integrating data from an on-premises SQL database using  Azure Data Factory (ADF), Azure Data Lake Storage (ADLS Gen2), Azure Databricks (PySpark/Delta Lake), and Azure Synapse Analytics.

This project orchestrates the ingestion, transformation, and optimization of relational ERP data from an on-premises SQL Server environment into high-performance, analytical Star Schemas ready for business intelligence.

## Prerequisites
- Azure Subscription with an active Resource Group.
- Knowledge of PySpark( Autoloader, Batch streaming, Delta tables

## Architecture
![](Architecture.jpeg)
1. The data platform transitions through an automated pipeline designed for incremental updates, minimal processing costs, and maximum query performance:
Ingestion (Staging): Azure Data Factory securely copies transactional data from on-premises SQL Server instances and drops them as compressed Parquet files into a raw landing zone in ADLS Gen2.
![](ADF_pipeline.jpeg)

3. Bronze Layer (Raw Replication): Databricks Auto Loader monitors cloud folders to discover new batches incrementally, appending raw entries seamlessly into schema-inferred Bronze Delta Tables.

4. Silver Layer (Cleaning & Harmonization): Real-time Delta-to-Delta streaming cleanses corrupt records, resolves structural anomalies (e.g., malformed decimal strings), standardizes data types, and flattens transactional constraints.

5. Gold Layer (Analytical Star Schema): Structured dimension tables (e.g., flattened, multi-address customer profiles) and numerical fact tables are assembled via precise upsert (MERGE) operations.
![](Final_Sales_Pipeline.jpeg)

6. Serving Layer: Azure Synapse Serverless SQL Pools expose the optimized Gold Delta files as relational views, serving sub-second, highly cost-efficient aggregations straight to Power BI.

## Getting Started - Setup

- Provision Azure Resources: Log in to the Azure Portal, create a Resource Group, and provision the following services:
  - Azure Storage Account
  - Azure Data Factory (ADF)
  - Azure Databricks Access Connector
  - Azure Databricks Workspace

- Configure the Data Lake: During the Storage Account setup, enable Hierarchical Namespace (HNS) to upgrade standard Blob storage into a true directory-based Data Lake (ADLS Gen2). Create your storage containers. To allow Unity Catalog to securely access the data lake, go to the Storage Account's Access Control (IAM) and assign the following roles to the Access Connector's Managed Identity:
  - Storage Blob Data Contributor
  - Storage Account Contributor
  - EventGrid EventSubscription Contributor
  - Storage Queue Data Contributor

- Configure Unity Catalog & Storage Credentials: Databricks strictly enforces a limit of one Unity Catalog metastore per region per account; all workspaces in the same region share this central metastore. Log into your Databricks workspace as a Metastore Admin or Account Admin. In the Catalog Explorer, create a new Storage Credential by pasting the Azure Resource ID of your Access Connector. Once connected, set up your external locations, schemas, and external tables pointing to the ADLS Gen2 data lake.

- Initialize Ingestion: Launch Azure Data Factory and configure the integration runtime/linked services to connect to your on-premises database (see resources below).

- Process Data: Once the ADF ingestion pipeline is successfully built and running, launch Azure Databricks to begin data transformation.


## Resources 
- https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/setup-uc
- https://learn.microsoft.com/en-us/azure/databricks/ingestion/cloud-object-storage/auto-loader/unity-catalog
- https://learn.microsoft.com/en-us/azure/databricks/tables/external


