---
tags:
  - gcp
---
## Cloud Monitoring
Cloud Monitoring is a managed service that lets you:
- Collect metrics (CPU, memory, requests, latency, etc.)
- Create dashboards
- Configure alerts (when metrics are NOT healthy)
- Alerts can be done by defining alerting policies: 
	- Condition -  a specific alert has to be raised
	- Notifications, channels - in which a specific notification has to be sent.
	- Documentation - you can also have some documentation attached, The documentation can describe the specific condition and maybe explain what can be done to resolve the issue.

### Cloud Monitoring - Workspace
- You can use Cloud Monitoring to monitor one or more GCP projects and one or more AWS accounts
- How do you group all the information from multiple GCP projects or AWS Accounts?
- **Create Workspace:**
	- A workspace allows you to see monitoring information from multiple projects
	- Step I: Create workspace in a spedific project - **Host Project**
	- Step II: Add other GCP projects (or AWS accounts) to the workspace - **Monitored Project**

```text
Project A ─┐
Project B ─┼──► Monitoring Workspace
Project C ─┘
```

### Cloud Monitoring - Virtual Machines
- **Default metrics** monitored (For Compute instance) include:
	- CPU utilization
	- Some disk traffic metrics
	- Network traffic, and
	- Uptime information

- Install Cloud Monitoring agent on the VM to get more disk, CPU, network, and process metrics:
	- collected-based daemon
	- Gathers metrics from VM and sends them to Cloud Monitoring

---

## Cloud Logging
- Cloud Logging is Google Cloud’s fully managed logging service that lets you collect, store, search, analyze, and export logs from GCP resources, applications, and hybrid environments.
- Cloud Logging is a real-time log management and analysis service in Google Cloud.
- It centralizes logs coming from: Applications, Google Cloud services, User activities, Audit logs, On-premises systems
- Store, search, analyze, and alert on massive volumes of data
- Exabyte-scale, fully managed service
- No server provisioning or patching required
- Can ingest logs from any source

**All logs from different sources are captured and stored in Cloud Logging.**

### Key Features:
1. **Logs Explorer**
    - Search logs
    - Sort logs
    - Analyze logs using flexible queries

2. **Logs Dashboard**
    - Rich visualization of logs
    - Create visual insights from log data

3. **Logs-Based Metrics**
    - Generate metrics from logs
    - Use queries or matching strings
    - Create dashboards based on derived metrics

4. **Logs Router**
	- Route log entries to following destinations- Send logs to:
        - BigQuery
        - Cloud Storage
        - Pub/Sub
        - Other supported services

### Log Collection in Cloud Logging

**1. GCP Managed Services (Automatic)**
Most Google Cloud managed services automatically send logs to Cloud Logging: GKE, App Engine, Cloud Run, **No configuration required with these serivces**

**2. GCE Virtual Machines**
To ingest logs from Compute Engine VMs:
- Install Logging Agent (based on Fluentd)
- Recommended: Run Logging Agent on all VM instances

The Logging Agent:
- Collects system logs
- Collects application logs
- Sends them to Cloud Logging

**3. On-Premises Log Ingestion**
To send logs from on-prem systems:
- **Recommended Option:** Use BindPlane tool (from Blue Medora)
- Alternative Option:
	- Use Cloud Logging API
	- Call API directly from on-prem machine

```text
Applications / GCP Services / Audit Logs
            ↓
        Cloud Logging
            ↓
    - Search (Logs Explorer)
    - Visualize (Dashboards)
    - Create Metrics
    - Route Logs (Logs Router)
    - Trigger Alerts
```

> [!summary] Cloud Logging:
> - Centralizes logs from multiple sources
> - Provides real-time analysis
> - Enables log-based metrics
> - Supports routing to multiple destinations
> - Is fully managed and scalable

---

## Security & Audit layer inside Cloud Logging.
It highlights two major concepts:
### 1️⃣ Access Transparency Logs
Access Transparency Logs record:
> - Actions performed by Google Cloud personnel on your data.
> - This is NOT about your users.
> - This is about Google engineers accessing your content (for support, troubleshooting, etc.).

**Why does this exist?**
- If you open a support case, sometimes Google engineers may need temporary access to:
- Your VM instance
- Your Cloud Storage bucket
- Your configuration
- Logs for troubleshooting

**Access Transparency tells you:**
- Who from Google accessed
- What they accessed
- When they accessed
- Why they accessed (support case reference)

**Important Notes**
❗ Not supported by all GCP services
❗ Available ONLY for organizations with Gold support plan or higher
❗ Mostly used by enterprises for compliance

### 2️⃣ Cloud Audit Logs
- Cloud Audit Logs record Who did what, where, and when in your Google Cloud project.
- This tracks your users and service accounts — not Google staff.

#### 4 Types of Cloud Audit Logs
- **Admin Activity Logs**
	- Record API calls and actions that modify resource configuration or metadata (e.g., creating a VM, changing IAM policies).
	- **Default enabled**, no charge, retained for 400 days.
	- Examples:
		- Creating a VM
		- Deleting a bucket
		- Changing IAM permissions
		- Updating firewall rules
		
- **Data Access Logs**
	- Reading or interacting with resource data.
	- **must enable manually**, Because High volume, Can generate massive logs, Chargeable
	- Examples:
		- Reading a bucket object
		- Listing VMs
		- Querying BigQuery data
		- Reading secrets

- **System Event Logs**
	- Actions performed automatically by Google Cloud - Not user-triggered.
	- **Default Enabled**
	- Examples:
		- Host maintenance
		- Instance preemption
		- Automatic restart
		- Auto-scaling events 

- **Policy Denied Logs**
	- When a user or service account is denied access.
	- **Default Enabled**
	- Examples:
		- Permission denied
		- IAM policy violation
		- Security policy block

---

## Cloud Logging - Controlling & Routing
When logs are generated (from VMs, GKE, Cloud Run, Audit Logs, on-prem, etc.):
```text
Logs → Cloud Logging API → Log Router → Buckets / Destinations
```

**Log Router (Core Component)**
- The Log Router is the central traffic controller.
- It evaluates logs against configured rules.
	- It decides:
		- ✅ What logs to ingest
		- ❌ What logs to discard
		- 📤 Where logs should be routed

**Two Types of Log Buckets**
- Cloud Logging stores logs in buckets.
- There are two built-in types:
	1. **`_Required Bucket`**:
		- This bucket contains: Admin Activity Logs, System Event Logs, Access Transparency Logs
		- Characteristics:
			- Retention: **400 days, Zero charge, Cannot delete, Cannot change retention period**
			- This ensures: Security & compliance logs are always preserved.

	2. **`_Default Bucket`**:
		- Contains: All other logs, Application logs, Data Access logs (if enabled), VM logs
		- Characteristics:
			- Default retention: 30 days
			- **Billed** according to logging pricing
			- Can change retention (1 to 3650 days = up to 10 years)
			- **Cannot delete the bucket itself**
				- But you Can disable the `_Default` log sink route to disable ingestion. 
				
**Important Concept: Log Sink**
A Log Sink defines:
- Which logs to match (filter)
- Where to send them

Example:
 - Send ERROR logs to BigQuery
 - Send Audit logs to Pub/Sub
 - Archive all logs to Cloud Storage

### Cloud Logging — Export (Concept)
Cloud Logging keeps logs only for a limited time:
- `_Default` bucket → 30 days (default)
- `_Required` bucket → 400 days

Cloud Logging is designed for:
- ✅ Short-to-medium log retention
- ❌ Not ideal for long-term storage or analytics

#### Where Can Logs Be Exported?
1. Cloud Storage (Archival)
	- Best for: Long-term retention, Compliance, Cheap storage, Backup
	- Example structure: `bucket/syslog/2025/05/05`

2. BigQuery (Analytics)
	- Best for: SQL analysis, Reporting, Security investigations, Trend analysis
	- Can query on the logs
	- Example table: 
		```text
				syslog_20250505 (Table name)
				columns:
				    timestamp
				    log
		```

3. Pub/Sub (Real-Time Streaming)
	- Logs are sent as base64 encoded messages.
	- Best for: Event-driven processing, SIEM systems, Automation pipelines, Security monitoring
	- Typical flow: `Cloud Logging → Pub/Sub → Cloud Run / SIEM / Functions`

---

## Cloud Logging – Export – Use Cases
- Use Case 1: **Troubleshoot using VM Logs**
	- Install Cloud Logging Agent in all VMs and send logs to Cloud Logging
	- Search for logs in Cloud Logging
	
- Use Case 2: **Export VM Logs to BigQuery for SQL-like Querying**
    - Install Cloud Logging Agent in all VMs and send logs to Cloud Logging
    - Create a BigQuery dataset for storing the logs
    - Create an export sink in Cloud Logging with BigQuery dataset as sink destination

- Use Case 3: **Retain Audit Logs for External Auditors at Minimum Cost**
    - Create an export sink in Cloud Logging with Cloud Storage bucket as sink destination
    - Provide auditors with Storage Object Viewer role on the bucket
    - Use Google Data Studio (Looker Studio) for visualization

---

## Cloud Trace
- Cloud Trace is a distributed **tracing system**
	- Supported Google Cloud Services: Compute Engine, GKE, App Engine (Flexible/Standard) etc  or Instrumented applications (using tracing libraries) using **Cloud Trace API**
	
- Helps you:
	- Measure request latency
	- Track request flow across microservices
	- Identify performance bottlenecks
	- Debug slow API calls

It answers:
- ❓ Why is this request slow?
- ❓ Which service caused the delay?
- ❓ Where is latency happening?

### Key Concepts
1️⃣ **Trace**
- Complete lifecycle of a request across services.

2️⃣ **Span**
- A single unit of work within a trace.
- Example:
	- API call
	- DB query
	- External HTTP request

3️⃣ Latency
- Time taken for request or span to complete.

---

## Cloud Profiler
Cloud Profiler is a continuous production profiling tool that helps you analyze:
- CPU usage
- Memory allocation
- Heap usage
- Thread contention

It answers:
- ❓ Why is my service using too much CPU?
- ❓ Where is memory leaking?
- ❓ Which function consumes the most resources?

**Unlike logging or tracing, Profiler focuses on resource consumption inside your code.**

### Two Major Components of Cloud Profiler
1️⃣ **Profiling Agent**
- (Data Collection Component)
- This runs inside your application.
- What it does:
	- Collects CPU usage data
	- Collects memory allocation data
	- Collects heap information
	- Collects thread contention data
	- Samples performance continuously
	- Sends profiling data to Cloud Profiler backend

- Where it runs:
	- Inside your VM
	- Inside GKE pod
	- Inside Cloud Run container
	- Inside App Engine

**Important:**
It runs with low overhead (~5%), so it is safe for production.

2️⃣ **Profiler Interface**
- (Visualization Component)
- This is the Google Cloud Console UI.
- What it provides:
	- Flame graph visualization
	- CPU usage breakdown
	- Memory usage breakdown
	- Function-level performance analysis
	- Time-range filtering
	- Version comparison

- It helps you visually identify:
	- Which function consumes most CPU
	- Where memory is leaking
	- Which part of code blocks threads

---

## Error Reporting
Error Reporting is a GCP service that automatically:
- Centralized Error Management console: Identify & manage top errors or recent errors
- Firebase Crash Reporting Integration - Use **Firebase Crash Reporting** for errors from Android & iOS client applications
- Detects application errors and Groups similar errors together
- Shows stack traces
- Highlights top production problems
- Helps you fix issues faster

It answers:
- ❓ What errors are happening in production right now?
- ❓ Which error is affecting users the most?

Aggregates and displays errors reported from cloud services (Using stacktraces)

Instead of seeing:
```text
Error: DB connection failed
Error: DB connection failed
Error: DB connection failed
```

You see:
```text
DB connection failed
Occurrences: 12,543
First seen: 10:01 AM
Last seen: 10:10 AM
```

Errors can be reported by:
- Sending them to Cloud Logging OR
- By calling Error Reporting API

---

## Exam Tip 
In exam , you would see still some of services referred to by the  old name:
**Stackdriver (Old Names) → Cloud Operations (New Names)**

| Stackdriver Service              | New Service Name     |
|----------------------------------|----------------------|
| Stackdriver Monitoring           | Cloud Monitoring     |
| Stackdriver Logging              | Cloud Logging        |
| Stackdriver Error Reporting      | Error Reporting      |
| Stackdriver Trace                | Cloud Trace          |
| Stackdriver Profiler             | Cloud Profiler       |

---

## Cloud Operations Scenarios
> [!info]
> - **Google Cloud Debugger** was a debugging tool that allowed developers to inspect the state of their running cloud applications, at any code location, without stopping or slowing them down. **It was deprecated** on May 16, 2022 and shut down on May 31, 2023.
> - **NOTE:** From the perspective of the exam, it is important to remember that there was once a service called Google Cloud Debugger. 

| Scenario | Solution |
|----------|----------|
| You would like to record all operations/requests on all objects in a bucket (for auditing) | Turn on Data Access Audit Logging for the bucket |
| You want to trace a request across multiple microservices | Cloud Trace |
| You want to identify prominent exceptions (or errors) for a specific microservice | Error Reporting |
| You want to debug a problem in production by executing step by step | Cloud Debugger |
| You want to look at the logs for a specific request | Cloud Logging |

---
