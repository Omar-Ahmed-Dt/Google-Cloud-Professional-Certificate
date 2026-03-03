---
tags:
  - gcp
---
## BigQuery - Datawarehouse
### Overview
- BigQuery is Google Cloud’s fully managed, serverless data warehouse.
- It can handle exabyte-scale data (extremely large datasets).
- It is designed for:
	- Massive analytics
	- Large-scale SQL queries
	- Business intelligence
	- Real-time + batch analytics

- Relational database (SQL, schema, consistency):
- BigQuery behaves like a relational database: Tables, Columns, Schemas, SQL queries
- You use standard SQL to query data 
- But unlike traditional databases:
	- It separates storage and compute
	- It scales automatically
	- It handles massive datasets

- Query **External Data Source** without storing data in BigQuery
	- Example sources: Cloud Storage, Cloud SQL, Bigtable, Google Drive - This is called **External Tables.**
	- To be able to do that you can use **permanent or temporary** external tables. What we'll be creating is external tables connecting to these storage devices and that would allow us to query from BigQuery.
		- Options: 
			- Permanent external tables
			- Temporary external tables
		- **Example**
			- Query data in Cloud Storage without loading it into BigQuery: Use an **external table**
				- One-time query: Use a **temporary external table**
				- Production reusable source: Use a **permanent external table**
	
- BigQuery offers traditional and modern approaches.
	- **Traditional Data Warehouse** - The traditional approach is to have a lot of storage and compute and use this to run the queries.
		- Storage + Compute
		- Structured schema
		- SQL-based queries
	
	- **Modern Cloud Features** -  offers modern features It's real time and serverless.
		- Serverless (no infrastructure management)
		- Real-time streaming ingestion
		- Autoscaling compute
		- Pay per query

### Importing and Exporting Data
**BigQuery can load data from:**
- Load data from a variety of sources, including streaming data
- Variety of import formats: CSV, JSON, Avro , Datastore backups
- Batch Import (FREE) - Import from Cloud Storage and local files
- Streaming Import (Expensive) - From Cloud Pub/Sub, Streaming Inserts

**BigQuery can export data to:**
- Cloud Storage (for long-term storage)
- Looker Studio (Data Studio) for visualization
- Formats - CSV/JSON (with Gzip compression), Avro (with deflate or snappy compression)

### Automatic Table Expiration
You can configure tables to:
- **Configurable Table Expiration** - Automatically expire after X days
- Reduce storage costs

### BigQuery - Accessing and Querying Data
You can interact with BigQuery using:
- Cloud Console (Web UI)
- `bq` Command Line Tool (NOT gcloud)
- BigQuery REST API
- Client Libraries (Java, Python, .NET) - Use BigQuery SDKs inside applications.

**Important**: BigQuery Queries Can Be Expensive
- BigQuery Queries Can Be Expensive
- BigQuery charges based on: **Amount of data scanned** - If your table is 1 TB and your query scans all of it, you pay for scanning 1 TB (this is not the amount of data returned by the query, This is the amount of data scanned).

**Best Practice**: Estimate Query Cost Before Running
1. Use Dry Run: 
	- `In Console` → `UI - Query Validator shows data scanned`
	- `In CLI` → `--dry_run`
	- This tells you:
		- Estimated bytes scanned
		- Without actually running the query
	
2. Use Pricing Calculator
	- Use Pricing Calculator: Find price for scanning 1 MB data. Calculate cost.

### Partitioning and Clustering BigQuery Tables - Use Case
- Partitioning and Clustering directly affect performance + cost
- BigQuery charges based on: Amount of data scanned
	- If your table has 100 million rows and you run a query, BigQuery may scan the entire table → 💸 expensive.

```text
Questions
---------------------------------
Date        | Question | Category
---------------------------------
2025-10-02  | ...      | AWS
2025-10-02  | ...      | GCP
2025-10-03  | ...      | Azure
```

You want:
- Questions between two dates
- For a specific category

- Without optimization: BigQuery scans everything.
- With optimization: It scans only relevant parts.

**Partitioning** = Divide table into segments.
- Instead of 1 large table `Questions` , You get two tables:

	```text
	First Table: Questions_2025_10_02
	Second Table: Questions_2025_10_03
	```
	
- (DEFAULT) All partitions will share same schema as table
- You can partition by:
	- Ingestion time
	- DATE column
	- TIMESTAMP
	- DATETIME
	- INTEGER column

**Clustering** =  groups rows based on column values.
Example:
Cluster by category, Inside each date partition:
```text
AWS rows together
GCP rows together
Azure rows together
```

**Example:** partitioned + clustered tables formatted
Original Table (No Partitioning)
```text
Questions
-------------------------------------------
Date        | Question           | Category
-------------------------------------------
2025-10-02  | Question Detail... | GCP
2025-10-02  | Question Detail... | AWS
2025-10-03  | Question Detail... | Azure
2025-10-03  | Question Detail... | GCP
2025-10-03  | Question Detail... | Azure
```

```sql
CREATE TABLE `my_data_set.questions_partitioned_and_clustered` # dataset_name.table_name
...
PARTITIONED BY DATE(created_date)
CLUSTER BY category
...
OPTIONS (
  expiration_timestamp=TIMESTAMP "2025-01-01 00:00:00 UTC",
  partition_expiration_days=7
)
```

**Partition1: Questions_2025_10_02)**
```text
| Date       | Question            | Category |
|------------|--------------------|----------|
| 2025-10-02 | Question Detail... | AWS      |
| 2025-10-02 | Question Detail... | GCP      |
| 2025-10-02 | Question Detail... | GCP      |
```

**Partition2: Questions_2025_10_03**
```text
| Date       | Question            | Category |
|------------|--------------------|----------|
| 2025-10-03 | Question Detail... | Azure    |
| 2025-10-03 | Question Detail... | Azure    |
| 2025-10-03 | Question Detail... | GCP      |
```

---

### BigQuery Storage & Expiration
- we pay for two important things: One is **storage**- how much data you're storing. Number two is **queries**, how much data you are scanning in your queries. You can reduce both of these by expiring data or deleting data in BigQuery.

**BigQuery Hierarchy**: `DataSet > Table > Partitions`
You can configure expiration at each level:
- Configure `table expiration` (default_table_expiration_days) for datasets  
- Configure `expiration time` (expiration_timestamp) for tables  
- Configure `partition expiration` (partition_expiration_days) for partitioned tables  

**Examples:**
1️⃣ Dataset-Level Default Table Expiration - All new tables in this dataset expire after 30 days:
```sql
CREATE SCHEMA mydataset
OPTIONS (
  default_table_expiration_days = 30
);
```

2️⃣ Table-Level Expiration - Expire entire table at a specific timestamp:
```sql
ALTER TABLE my_dataset.my_table
SET OPTIONS (
	partition_expiration_days = 7
);
```

3️⃣ Partition-Level Expiration - Automatically delete partitions after 7 days:
```sql
CREATE TABLE my_dataset.logs
PARTITION BY DATE(event_time)
OPTIONS (
  partition_expiration_days = 7
);
```
This deletes old partitions automatically.

**BigQuery Expiration Hierarchy:**
```text
Dataset (default_table_expiration_days)
    ↓
Table (expiration_timestamp)
    ↓
Partition (partition_expiration_days)
```

### Streaming Data into BigQuery
#### Bulk Load vs Streaming
- Bulk Loading (Batch) - Import from Cloud Storage and local files: `Cloud Storage → BigQuery (Load Job)`
	- FREE
	- Recommended for large historical data

- Streaming Inserts - `App → BigQuery (streaming insert API)`
	- Insert rows in real-time
	- NOT FREE
	- Has quotas and limitations
	- Streaming costs money per amount of data inserted.
	- Streaming Can Create Duplicates so How To Avoid Duplicates: 
		- Use `insertId` - Add `insertld` with each streaming insert
			- Any duplicate rows with the **same insert ID** within **the next one minute** will not be inserted.
			- BigQuery uses it for Best-effort deduplication (within 1 minute window) - Deduplication is not permanent, It only works for ~1 minute. It is best-effort (not guaranteed).
			
		- If you need strict deduplication:
			- Use Datastore / transactional system
			- Or use Dataflow with deduplication logic

#### Streaming Quotas
BigQuery enforces strict limits:

**Case 1: If you DO NOT use** `insertId`
- You are limited by:
	- you can stream up to 1 GB per second per project - (Per project - NOT per table)
	-  this is the maximum amount of data that you can stream in per second, per project.
	
**Case 2: If you USE** `insertId`
- Limits change to:
	- Maximum rows per second per project:
		- US & EU multi-regions → 500,000 rows/sec , Other regions → 100,000 rows/sec
		- Per table: 100,000 rows/sec
	
	- Maximum bytes per second: 100 MB/sec

- If you need Millions of rows per second , BigQuery streaming is not ideal, Better choice is **Bigtable**

### BigQuery Best Practices
- Estimate your queries before running them: Use `bq --dry_run` flag or `dryRun` API parameter
- Use clustering and partitioning for your tables
- Avoid streaming inserts when possible
    - Loading data in bulk is free but streaming data is NOT FREE
    - Offers best-effort de-duplication (when using `insertId`)
    - Remember quota limits

- Expire Data Automatically:
    - Configure default table expiration (`default_table_expiration_days`) for datasets
    - Configure expiration time for tables
    - Configure partition expiration for partitioned tables

- Consider **Long-term storage** option
    - Long-term storage: Table in which data is NOT edited for 90 consecutive days, consider it as **long-term storage**: Lower storage cost (similar to Cloud Storage Nearline)

- BigQuery is fast for complex queries:
    - BUT it is not well optimized for narrow-range queries (prefer Cloud Bigtable)
    - Remember: Too much complexity in setting up a query can hurt performance

- Optimize BigQuery usage using audit logs:
    - Analyze queries/jobs that were run earlier
    - Stream your audit logs (`BigQueryAuditMetadata`) to BigQuery
        - Understand usage patterns (query costs by user)
        - Optimize (visualize using Google Data Studio / Looker Studio)

---

## Cloud Dataproc
- A managed Spark and Hadoop service in Google Cloud.
- Instead of installing Hadoop/Spark clusters yourself, Google manages the infrastructure.
- Dataproc supports: Spark, PySpark, SparkR, SparkSQL, Hive, Pig ,Hadoop MapReduce
- So if your company already uses Hadoop or Spark, you can move that cluster to Google Cloud.
- Dataproc is used for:
	- Complex batch processing
	- Large-scale data transformations
	- Machine learning workloads
	- Custom distributed processing

- It is NOT just SQL.
- You can choose:
	- Single Node
	- Standard Cluster
	- High Availability

- A data analysis platform (cluster-based) - It is NOT serverless like BigQuery or Dataflow. You manage clusters (even though Google manages infra).
- You can export cluster config BUT NOT cluster data automatically.

**BigQuery vs Dataproc**
Choose BigQuery if:
- You only need SQL analytics
- You want serverless
- You want simple analytics on large datasets

Choose Dataproc if:
- You need Spark/Hadoop
- You need complex batch ML pipelines
- You want open-source frameworks
- You need more than SQL - analysis platform

---
