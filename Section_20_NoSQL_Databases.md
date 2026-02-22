---
tags:
  - gcp
  - database
---
## Datastore - Highly Scalable NoSQL Document Database
- Cloud Datastore is a fully managed, highly scalable NoSQL **document database** designed for applications that need:
- Automatically scales and partitions data as it grows
- Recommended for up to a few TBs of data, For bigger volumes, **Bigtable** is recommended
- Supports Transactions, Indexes and SQL-like queries (**GQL**) , Does **NOT** support Joins or Aggregate operations (SUM or COUNT)
- For use cases needing flexible schema with transactions, Examples: User Profiles and Product Catalogs
- You can export data ONLY from `gcloud` (NOT from Cloud Console), Export contains a metadata file and a folder with the data
- Structure: **Kind (like a table) , Entity (like a row), Properties (like columns), Key (Unique identifier of an entity.)** , Use namespaces to group entities - Used to logically separate data.

```text
1️⃣ Kind (like a table)
Equivalent to a table in SQL: 
Kind: User
Kind: Product
Kind: Order

2️⃣ Entity (like a row)
Each record stored in Datastore:
User {
  id: 101,
  name: "Omar",
  email: "omar@test.com"
}

3️⃣ Properties (like columns)
Fields inside an entity: 
name → string
age → number
isActive → boolean

4️⃣ Key
Unique identifier of an entity: 
Key = Kind + ID
User(101)

5️⃣ Namespace (multi-tenant feature)
Used to logically separate data: 
tenant_A → namespace A
tenant_B → namespace B
```
- **Prefer batch operations** (to single read, write or delete operations): More efficient as multiple operations are performed with same overhead as one operation

### Firestore = Datastore++ (Optimized for Multi-Device Access)
- Offline mode and data synchronization across multiple devices: Mobile, IoT, and distributed clients
- Provides client-side libraries: Web, iOS, Android and more
- Offers two modes:
    - Datastore Mode
    - Native Mode

---

## Cloud Bigtable
- Cloud Bigtable is a fully managed, petabyte-scale NoSQL wide-column database designed for extremely large datasets and very high read/write throughput with low latency.
- Designed for huge volumes of analytical and operational data  
	- IoT streams  
	- Analytics workloads  
	- Time-series data

- Handles millions of read/write TPS with very low latency
- Supports **single-row transactions**: Multi-row transactions are **NOT supported**
- NOT Serverless: You must create a **server instance (cluster)**, Uses SSD or HDD storage
- Scales horizontally using multiple nodes, No downtime required during cluster resizing
- Cannot export data using **Cloud Console** or **gcloud**, Use a Java application  `java -jar` export/import tools OR use HBase commands
- Use **cbt** command-line tool to work with Bigtable, NOT `gcloud`

---
## Summary
- **Cloud SQL** → relational business logic
- **Datastore** → scalable app objects
- **Bigtable** → massive structured data
- **BigQuery** → analytics
- **Firestore** → realtime apps

---
