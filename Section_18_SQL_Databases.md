---
tags:
  - gcp
  - database
---
## Overview
### Transaction Log
- A transaction log is a sequential record of every change made to the database.
- Before the database actually changes data on disk, it writes the operation into the transaction log first. ==If crash happens → DB rebuilds data using logs==

### Availability vs Durability
- **Availability** = Can I access my system or data RIGHT NOW? `Yes → High availability , No → Downtime` - How often a service is up and responding correctly.
	- **Increasing Availability:** Have **multiple standbys available** in multiple Zones  and in multiple Regions

- **Durability** = Will my data EVER be lost? - Probability that stored data survives over time.
	- **Increasing Durability:** Multiple **copies of data** (standbys, snapshots, transaction logs and replicas) in multiple Zones and in multiple Regions
	
### Challenge and Solution
1. Challenge 1: Your database will go down if the data center crashes:
	- **You can switch to the standby database**
	
2. Challenge 2: You will lose data if the database crashes: 
	- **You can setup database from latest snapshot and apply transaction logs**

3. Challenge 3: Database will be slow when you take snapshots:
	- **Take snapshots from standby.**
	- **Applications connecting to master will get good performance always.**

### RTO and RPO
- **RTO** (Recovery Time Objective): Maximum acceptable downtime
- **RPO** (Recovery Point Objective): Maximum acceptable period of data loss

### New reporting and analytics applications are being launched using the same database
- These applications will **ONLY read data**
- Within a few days you see that the database performance is impacted
- How can we fix the problem?
    - **Vertically scale the database** - increase CPU and memory
    - **Create a database cluster (Distribute the database)** - the database clusters are expensive to setup
    - **Create read replicas** - run read-only applications against read replicas

### Read Replica
- Connect reporting and analytics applications to read replica
- Reduces load on the master databases
- Upgrade read replica to master database (supported by some databases)
- Create read replicas in multiple regions
- Take snapshots from read replicas

### Consistency
- How do you ensure that data in multiple database instances (standbys and replicas) is updated simultaneously?
- **Strong consistency** - Synchronous replication to all replicas
    - Will be slow if you have multiple replicas or standbys

- **Eventual consistency** - Asynchronous replication  
  A little lag (few seconds) before the change is available in all replicas
    - In the intermediate period, different replicas might return different values
    - Used when scalability is more important than data integrity
    - Examples: Social Media Posts

- **Read-after-Write consistency** - Inserts are immediately available (When you create new data, you can read it immediately after writing)
    - However, updates would have eventual consistency (When you modify existing data, replicas may take time to sync)

### OLTP (Online Transaction Processing)
- A database system optimized for many small real-time transactions.
- A database optimized for real-time operations.
- Handles transactions.
- Fast, small query (INSERT, UPDATE, DELETE)
- GCP OLTP Services: Cloud Sql , Cloud Spanner
- Use Cases: E-commerce, Banking systems, ERP / CRM, Booking systems

### OLAP (Online Analytical Processing)
- A system optimized for analytics and reporting.
- Reads huge datasets to extract insights.
- GCP OLAP service: BigQuery
- User Cases: Sales reports, Data aggregation, Business intelligence dashboards, Machine learning analysis 

### SQL (Relational DB) and NoSQL (Non-Relational DB)
**SQL mindset**: Structure first → then store data, You design schema BEFORE inserting data.
```sql
CREATE TABLE users (
  id INT,
  name VARCHAR(50),
  email VARCHAR(100)
);
```

- Every record must follow this structure. 
- SQL → Vertical Scaling

**NoSQL mindset**: Store data first → structure evolves later, Each record can look different: 
```json
{
  "name": "Omar",
  "email": "omar@test.com"
}
```

```json
{
  "name": "Sara",
  "email": "sara@test.com",
  "avatar": "img.png"
}
```

- NoSQL → Horizontal Scaling
- Designed for distributed systems.

> - SQL = structured data + strong consistency
> - NoSQL = flexible schema + massive scalability

### Database - Summary
| Database Type | GCP Services | Description |
|---|---|---|
| **Relational OLTP databases** | Cloud SQL, Cloud Spanner | Transactional use cases needing **predefined schema** and very **strong transactional capabilities** (Row storage).<br><br>**Cloud SQL:** MySQL, PostgreSQL, SQL Server databases.<br>**Cloud Spanner:** Unlimited scale and 99.999% availability for global applications with horizontal scaling. |
| **Relational OLAP databases** | BigQuery | Columnar storage with predefined schema. Data warehousing and Big Data analytical workloads. |
| **NoSQL Databases** | Cloud Firestore (Datastore), Cloud Bigtable | Apps needing **quickly evolving structure (schema-less)**.<br><br>**Cloud Firestore:** Serverless transactional document database supporting mobile & web apps. Small to medium databases (0 → few TBs).<br><br>**Cloud Bigtable:** Large databases (10 TB → PBs). Used for streaming (IoT), analytical and operational workloads. NOT serverless. |
| **In-memory databases / caches** | Cloud Memorystore | Applications needing **microsecond response times** (caching layer). |

### Scenarios

| Scenario | Solution |
|---|---|
| A startup with quickly evolving schema | Cloud Datastore / Firestore |
| Non relational DB with less storage (10 GB) | Cloud Datastore |
| Transactional global database with predefined schema needing to process millions of transactions per second | Cloud Spanner |
| Transactional local database processing thousands of transactions per second | Cloud SQL |
| Cache data (from database) for a web application | Memorystore |
| Database for analytics processing of petabytes of data | BigQuery |
| Database for storing huge volumes stream data from IoT devices | Bigtable |
| Database for storing huge streams of time series data | Bigtable |

### High Availability
**Failover**: If Primary Instance is unresponsive, cloud SQL automatically switches to serving data from the standby one.
**Post Failover**: When the failed instance recovers, it becomes the standby instance. 
**Failback process**: If you want to reroute traffic back to the original instance initiate

---

## Cloud SQL - (Relational Database for Transactional Applications)
- Cloud SQL is a fully managed relational database service for MySQL, PostgreSQL, and SQL Server. This frees you from database administration tasks so that you have more time to manage your data.
- Configure your needs and **do NOT worry about managing the database**
- Supports **MySQL, PostgreSQL, and SQL Server**
- Regional service providing **High Availability (99.95%)**
- Use **SSDs or HDDs** (For best performance: use SSDs)
- Up to **416 GB of RAM** and **30 TB of data storage**
- **Use Cloud SQL for simple relational use cases:**
    - To migrate local **MySQL, PostgreSQL, and SQL Server** databases
    - To reduce your maintenance cost for a simple relational database

- (REMEMBER) Use **Cloud Spanner** (Expensive) instead of Cloud SQL if:
    - You have huge volumes of relational data (TBs) **OR**
    - You need infinite scaling for a growing application (to TBs) **OR**
    - You need a **Global (distributed across multiple regions)** database **OR**
    - You need higher availability (**99.999%**)

- Cloud SQL instances can be accessed using networking models:
	1. **Public IP**:
		- Connectivity happens through the internet.
		- You must authorize specific client IP addresses through Authorized Networks
		- Enforcing SSL/TLS encryption for all public connections - by default the traffic is not encrypted
			- **SSL / TLS Encryption:**
				- end to end encryption for data **in transit**
				- Two modes: **One Way TLS** OR **Mutual TLS**
					1. **One-Way TLS** (also called Server Authentication TLS) means:
						- Only the server proves its identity to the client using a certificate - The client does NOT present a certificate. So **authentication happens in one direction only.**
						- Cloud SQL provides:
							- Server CA certificate
							- You download `server-ca.pem`
							- Client uses it to verify Cloud SQL  - `If valid` → continue then open Encrypted tunnel between client and Cloud SQL ,Then when you login to Cloud SQL, it asks you for a username and password,  `If not` → connection rejected
							
					2. **Mutual TLS** - The server verifies the client and The client verifies the server - Authentication happens **in both directions.**
						- **How Mutual TLS Works (Step-by-Step)**
							- Client starts connection: Client application connects to Cloud SQL - `Client APP → Cloud SQL`
							- Server sends its certificate: `server-ca.pem - Used to verify Cloud SQL` then Client verifies: "This is the real Cloud SQL server."
							- Client sends its certificate: `client-cert.pem - Client identity certificate` AND `client-key.pem - Client private key (secret)`
							- Now the server verifies the client identity.
							- Both sides validate certificates
							- Encrypted tunnel established: TLS handshake finishes - Encrypted Tunnel Established
							- All database traffic is now: Encrypted , Server authenticated , Client authenticated 

	2. **Private IP (via Private Service Access – PSA)**
		- Private Service Access is **not designed only for Cloud SQL**, but is used by many Google managed services (Cloud SQL, Memorystore, Filestore, etc.).
	    - It creates a **private connection between your VPC network and Google-managed services VPCs** using internal IP ranges.
	    - Private Service Access enables private communication between: **Your VPC** and  **Google Managed VPC (service producer network)**
	    - **Networking Flow:**
				1. From your VPC, you **enable Private Service Access**.
				2. You **allocate a private IP range** for Google services `Example: 10.240.0.0/16`
				3. Google creates or uses a **Google-managed VPC (service producer VPC)** in its own project and reserves a **subnet from the allocated range** Example: `Allocated range 10.240.0.0/16 , Service subnet 10.240.0.0/24`
				4. This subnet is used internally by the managed service (e.g., Cloud SQL instance).
				5. Each Google managed service may consume a portion of the allocated PSA range.
				6. Google establishes **VPC Network Peering** between: `Your VPC (consumer network) and Google managed VPC (producer network)`
				7. This peering enables **private connectivity** between resources without using the public internet.

	3. Recommended - **Cloud SQL Auth Proxy**
		-  Cloud SQL Auth Proxy is a secure connection mechanism that allows applications to connect to Cloud SQL **without managing SSL certificates or exposing the database to public networks.**
		- Handles automatically the lifecycle and management of SSL/TLS certificates
		    - Generates and rotates **ephemeral certificates** automatically
		    - No manual certificate download or renewal required
		
		- No need to configure Authorized Networks
		    - Database does not need public IP allowlists
		    - Access is controlled using IAM authentication
			
		- **Connection Flow:**
			1. Client application connects locally to the proxy , App → localhost:3306
			2. Proxy authenticates using Google IAM, Uses service account or user credentials, Requires roles: `roles/cloudsql.client` , `roles/cloudsql.instanceUser`
			3. Proxy requests an OAuth2 token from Google IAM API
			4. Proxy requests an ephemeral SSL certificate from Cloud SQL Admin API, Certificates are short-lived and automatically rotated
			5. Proxy establishes encrypted tunnel to Cloud SQL, Encrypted channel uses port 3307 internally
			6. Proxy forwards traffic securely to Cloud SQL instance, Proxy Server → SQL Instance (port 3306)

			```text
					Client Environment:
					        Client App
					            ↓ (local connection)
					        Cloud SQL Auth Proxy Client
					            ↓ (encrypted TLS tunnel)
				
					Google Infrastructure:
					        Cloud SQL Auth Proxy Server
					            ↓
					        Cloud SQL Instance
		
					All traffic is encrypted and authenticated using IAM identity.
			```
		
**Gcloud Commands with Cloud SQL**
```bash
# Cloud SQL
gcloud sql connect my-first-cloud-sql-instance --user=root --quiet
gcloud config set project glowing-furnace-304608
gcloud sql connect my-first-cloud-sql-instance --user=root --quiet
use todos
create table user (id integer, username varchar(30) );
describe user;
insert into user values (1, 'Ranga');
select * from user;
```

---

### Understanding Cloud SQL Best Practices
#### Use Cloud SQL Proxy
- Securely connect to Cloud SQL from your apps:
    - App Engine (GAE)
    - Cloud Functions
    - Cloud Run
    - GKE
    - Compute Engine VMs
- Provides:
    - Automatic IAM authentication
    - Encrypted connections (TLS)
    - No need for public database exposure

#### Understand Scalability
##### Enable High Availability (HA)
- Primary instance and standby instance created in the same region: `Low latency replication and Protection from zone failure`
- Automatic failover if primary fails

```text
Region: europe-west1

Zone A                    Zone B
┌──────────────┐          ┌──────────────┐
│ Primary DB   │  ⇄ Sync  │ Standby DB   │
│ (Active)     │          │ (Passive)    │
└──────────────┘          └──────────────┘

```

- Cloud SQL cannot scale horizontally for writes `Cloud SQL has ONE primary writer`
- Prefer multiple smaller instances instead of one large instance: Instead of one overloaded DB, Split workload (Database Sharding / Service Isolation)
	```text
		Users DB        → Cloud SQL A
		Orders DB       → Cloud SQL B
		Analytics DB    → Cloud SQL C
		Payments DB     → Cloud SQL D
	```
- Now writes are distributed.

#### Use Read Replicas
- Offload read workloads: Reporting, Analytics
- Read replicas **do NOT increase availability**, Used only for read operations
- Read Replicas can be promoted (Manual promotion procees, then update the connection string in the app to connect the new DB instance) to be a primary instance for read and writes (Regional Disaster Recovery )

#### Backup and Export
**Backup:** 
- A backup is a system snapshot of your entire Cloud SQL instance created by Google.
- Deleted when instance is deleted, Backups deleted too
- Cannot backup single DB or table

**Exports (Flexible Data Copy):**
- More flexible
- You can export: entire instance, single database, single table
- Independent from instance, Export file stored in Cloud Storage so if DB deleted → export still exists.
- Slower than backups because it reads actual data, converts into SQL/CSV format
- **Serverless Export (CLI flag: -OFFLOAD):** to reduce impact
	- Cloud SQL creates Temporary hidden instance , Export happens there , Production DB unaffected 
	
- Import/Export in multiple small batches instead of large batches - Instead of Export 1TB at once , Export 50GB chunks.

---

## Cloud Spanner - (Relational Database for Transactional Applications)
- **Fully managed, mission critical, relational (SQL), globally distributed database**
    - Provides **VERY high availability (99.999%)**
    - Strong transactional consistency at **global scale**
    - Scales to **petabytes (PBs) of data** using automatic sharding

- **Cloud Spanner scales horizontally for reads and writes**
    - Scaling is done by configuring the **number of nodes**
    - In comparison, **Cloud SQL** provides read replicas only: Cloud SQL **cannot horizontally scale write operations**

- **Regional and Multi-Regional configurations**: Allows deployment across regions for higher availability and disaster recovery
- **Expensive (compared to Cloud SQL)**: Pricing is based on **nodes + storage usage**
- **Data Export**: Use **Cloud Console** to export data
	- Other option is to use Data flow to automate export
	- No `gcloud` export option available

---

