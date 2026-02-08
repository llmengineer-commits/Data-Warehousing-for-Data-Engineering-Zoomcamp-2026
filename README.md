# Data-Warehousing-for-Data-Engineering-Zoomcamp-2026
# Data Engineering Zoomcamp: Module 3 - Data Warehousing

## Project Overview
This repository contains the solution for Module 3 of the Data Engineering Zoomcamp (2026). The focus of this module is Data Warehousing, BigQuery strategies, and optimization techniques like partitioning and clustering.

**Note on Architecture:**
Due to regional billing verification constraints on Google Cloud Platform (GCP), this project implements a **Local Data Warehousing** strategy using **DuckDB**. This approach allows for the execution of identical SQL logic and data analysis required by the assignment, simulating the behavior of a cloud-based columnar data warehouse locally.

## Technologies Used
* **Python 3.x**: For data orchestration and scripting.
* **DuckDB**: An in-process SQL OLAP database used as a local alternative to BigQuery.
* **Pandas / PyArrow**: For Parquet file handling.
* **Jupyter Notebook**: For interactive development and documentation.

## Data Ingestion
The project analyzes the **New York City Yellow Taxi Trip Records** for the first half of 2024 (January – June).
* **Source:** NYC TLC Data
* **Format:** Parquet
* **Volume:** ~20 million records

The ingestion script (`Assignment3DataWarehousing.ipynb`) handles:
1.  **Sequential Downloading:** Retreiving 6 monthly Parquet files (~40-50MB each).
2.  **Integrity Checks:** verifying file sizes to detect and repair corrupted downloads.
3.  **Local Storage:** Organizing raw data into a local `taxi_data/` directory.

## Homework Solutions & Methodology

### Question 1: Counting Records
**Task:** Determine the exact count of records for the 2024 Yellow Taxi Data (Jan-Jun).
* **Method:** Loaded all 6 Parquet files into a single DuckDB table view.
* **SQL:** `SELECT COUNT(*) FROM yellow_taxi_2024;`
* **Answer:** `20,332,093`

### Question 2: Data Read Estimation
**Task:** Estimate data read for counting distinct `PULocationID` on External vs. Materialized Tables.
* **Analysis:**
    * **External Table:** Points to GCS files. BigQuery cannot cache metadata for external files, resulting in an estimate of **0 MB**.
    * **Materialized Table:** Native BigQuery storage. The engine knows the exact byte size of the `PULocationID` column.
* **Answer:** `0 MB for the External Table and 155.12 MB for the Materialized Table`

### Question 3: Columnar Storage Principles
**Task:** Explain why retrieving two columns costs more than retrieving one.
* **Analysis:** BigQuery and DuckDB are columnar stores. Data is stored by column, not by row.
* **Logic:** Selecting `PULocationID` requires opening one column file. Selecting `PULocationID` AND `DOLocationID` requires opening two. More columns = more data scanned.
* **Answer:** `BigQuery is a columnar database... Querying two columns requires reading more data than querying one...`

### Question 4: Zero Fare Trips
**Task:** Count records where `fare_amount` is 0.
* **SQL:** `SELECT COUNT(*) FROM yellow_taxi_2024 WHERE fare_amount = 0;`
* **Answer:** `8,333`

### Question 5: Table Optimization Strategy
**Task:** Optimize a table for filtering by `tpep_dropoff_datetime` and ordering by `VendorID`.
* **Strategy:**
    1.  **Partitioning:** Breaks the table into physical blocks based on date. This optimizes the `WHERE` clause (filtering).
    2.  **Clustering:** Sorts the data within those blocks. This optimizes the `ORDER BY` clause.
* **Answer:** `Partition by tpep_dropoff_datetime and Cluster on VendorID`

### Question 6: Partition Pruning Benefits
**Task:** Compare bytes processed for a query filtering March 1-15 on non-partitioned vs. partitioned tables.
* **Method:** A query filtering for 15 days of data (March 1-15) allows the engine to ignore data from Jan, Feb, Apr, May, and Jun.
* **Performance:**
    * **Non-Partitioned:** Scans `tpep_dropoff_datetime` for the entire 6-month dataset (~310 MB).
    * **Partitioned:** Scans only the specific partition for March (~26 MB).
* **Answer:** `310.24 MB for non-partitioned table and 26.84 MB for the partitioned table`

### Question 7: External Table Storage
**Task:** Identify where External Table data resides.
* **Concept:** External tables are definitions/pointers only. The actual bytes remain in the source system.
* **Answer:** `GCP Bucket`

### Question 8: Clustering Best Practices
**Task:** Is it best practice to always cluster?
* **Analysis:** Clustering adds metadata overhead. For small tables (<1 GB), this overhead outweighs the performance gains.
* **Answer:** `False`

### Question 9: Metadata Operations
**Task:** Estimate bytes for `SELECT count(*)` on a materialized table.
* **Concept:** Total row counts are stored in table metadata (statistics). The engine reads this single value without scanning any row data.
* **Answer:** `0 Bytes`
