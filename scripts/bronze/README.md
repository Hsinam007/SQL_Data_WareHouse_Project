# 🏗️ Bronze Layer: Automated Data Ingestion Pipeline

### 📌 Project Overview
The **Bronze Layer** represents the initial "Landing Zone" of a Modern Data Lakehouse (Medallion Architecture). This module is responsible for extracting raw data from multiple disparate source systems—specifically **CRM** (Customer Relationship Management) and **ERP** (Enterprise Resource Planning)—and loading it into a centralized SQL Server environment.



The primary goal of this layer is to capture source data with **100% fidelity**. By storing the data in its rawest form, we ensure a reliable "Source of Truth" that remains unaffected by downstream transformation logic.

---

## ⚙️ Pipeline Architecture & Flow
The ingestion process is divided into two distinct phases to ensure the system is modular, repeatable, and easy to maintain.



### 1. Schema Definition (DDL)
**File:** `ddl_bronze.sql`  
This script defines the physical "containers" for our raw data within the `bronze` schema.
* **Idempotent Design:** Every table creation is prefixed with an `IF OBJECT_ID ... IS NOT NULL DROP` check. This allows the script to be re-run at any time to reset the environment without manual intervention—a core requirement for **CI/CD pipelines**.
* **Raw Data Mapping:** Columns are mapped to mirror the source CSV files. We intentionally use flexible data types (e.g., `NVARCHAR` for dates or `INT` for raw keys) to ensure **Zero-Loss Ingestion**. This prevents the pipeline from crashing if the source system sends unexpectedly formatted data.

### 2. The Ingestion Engine (Stored Procedure)
**File:** `proc_load_bronze.sql`  
This is the "brain" of the pipeline, orchestrating the movement of data from local CSV files into SQL Server.
* **Automated Truncate-and-Load:** To maintain a fresh "snapshot" of the source and prevent data duplication, the procedure clears the tables before performing a `BULK INSERT`.
* **Operational Telemetry:** The script captures the `@start_time` and `@end_time` for every table. By printing the **Load Duration**, it provides essential performance metadata to help engineers identify bottlenecks as data volume scales.
* **Resilient Error Handling:** Wrapped in `TRY...CATCH` blocks, the procedure is production-ready. If a file is missing or a bulk insert fails, it captures diagnostic details (Error Message, State, and Number) instead of crashing the entire batch.

---

## 🚀 Key Data Engineering Insights

* **Schema Isolation:** By using a dedicated `bronze` schema, we logically separate "raw" data from "cleaned" data. This prevents analysts from accidentally using unprocessed information for business reporting.
* **Optimized Performance:** The pipeline utilizes the `TABLOCK` hint during `BULK INSERT`. This minimizes transaction log overhead, making the migration of large datasets significantly faster and more resource-efficient.
* **Full Transparency:** The inclusion of visual logging (via `PRINT` statements) creates a clear audit trail, showing exactly what was truncated, what was loaded, and exactly how long each step took.

---

## 📖 Deployment Instructions

To set up and run the Bronze Layer ingestion, execute the scripts in the following order:

1.  **Initialize Structure:** Run `ddl_bronze.sql` to create the `bronze` schema and all necessary tables.
2.  **Deploy Logic:** Run `proc_load_bronze.sql` to compile the ingestion stored procedure in your database.
3.  **Execute the Batch:** Run the following command to start the automated ingestion:
    ```sql
    EXEC bronze.load_bronze;
    ```

---

### 🛠️ Technology Stack
* **Database:** SQL Server (T-SQL)
* **Architecture:** Medallion Architecture (Bronze)
* **Ingestion Method:** Bulk Insert / Stored Procedures
* **Environment:** SQL Server Management Studio (SSMS) / Visual Studio