---
tags:
  - gcp
---
# Cloud Scheduler
- Cloud Scheduler is GCP's fully managed cron service.
- It lets you run jobs automatically at specific times or intervals - Uses Unix cron format - Think of it like Linux cron, but in Google Cloud.
- Examples:
	- Run every day at midnight
	- Trigger every 5 minutes
	- Execute every Monday at 9 AM
- Integrates with App Engine, Cloud Pub/Sub, Cloud Logging 200 any HTTP endpoint ٠
- Manage all your automation tasks from one place
- Provides automated retries

---

# Cloud Emulators
- Cloud Emulators are local versions of Google Cloud services that run on your machine for development and testing.
- Instead of using the real cloud service (which costs money and requires internet), you can run a local emulator.

```text
Your Application
      │
      ▼
Local Emulator
      │
      ▼
Behaves like the real GCP service
```

- Supports emulation of:
	- Cloud Bigtable
	- Cloud Datastore
	- Cloud Firestore
	- Cloud Pub Sub
	- Cloud Spanner

---

# Cloud DNS
- It translates domain names into IP addresses.`api.example.com  →  34.120.10.5`
- A DNS zone is a container of DNS records for the same DNS name suffix. In Cloud DNS, all records in a managed zone are hosted on the same set of Google-operated authoritative name servers.

**Public vs Private Zones**

| Type         | Purpose                                                            |
| ------------ | ------------------------------------------------------------------ |
| Public Zone  | Internet-facing domains `api.example.com → External Load Balancer` |
| Private Zone | Internal DNS inside a VPC `db.internal.local → Internal VM IP`     |

| Record | Purpose                   |
| ------ | ------------------------- |
| A      | Domain → IPv4 address     |
| AAAA   | Domain → IPv6 address     |
| CNAME  | Alias to another domain   |
| MX     | Mail server               |
| TXT    | Verification / SPF / DKIM |
| NS     | Name servers              |

**Example:**
```text
api.example.com    A      34.120.10.5
www.example.com    CNAME  example.com
```

**Example Flow**
```text
User types: app.example.com
        │
        ▼
Cloud DNS resolves to Load Balancer IP
        │
        ▼
Traffic sent to Cloud Run or VM
```

---

# Pricing Calculator
- The Google Cloud Pricing Calculator is a web tool that estimates the monthly cost of your planned GCP resources before you deploy them.

```text
2 × e2-medium VMs
+ 100 GB SSD
+ 1 TB Cloud Storage
+ 100 GB network egress
= Estimated monthly cost
```

---

# Anthos
- Google Anthos is GCP's platform for managing Kubernetes workloads across multiple environments:
	- On-premises datacenters
	- Google Cloud
	- Other clouds such as Amazon Web Services and Microsoft Azure

- It gives you one control plane to manage all your Kubernetes clusters.

**Without Anthos:**
```text
GKE cluster → managed separately
On-prem cluster → managed separately
AWS cluster → managed separately
```

**With Anthos:**
```text
Anthos
   │
   ├── GKE clusters
   ├── On-prem Kubernetes clusters
   ├── AWS / Azure clusters
   └── Managed from one place

                +---------+
                | Anthos  |
                +---------+
                  /   |   \
                 /    |    \
                ▼     ▼     ▼
          GKE Cluster AWS Cluster On-Prem Cluster
```

- Centralized config management (Git repo)
	- Logically group and normalize clusters. Define policies - Kubernetes API, Access control
		- Deploy to clusters on new commits: Use Namespaces, labels, and annotations to decide which clusters to apply changes on 

	- Provides Service Mesh (based on Istio)
		- Sidecar to implement common microservice features
			- Authentication & authorization (service accounts)
			- Distributed Tracing, Automatic metrics, logs & dashboards
			- A/B testing, canary rollouts
			- Cloud Logging & Cloud Monitoring Support

---

# Apigee APl Management
- It helps you publish, secure, monitor, and manage APIs.
- Think of it as an advanced API gateway plus API lifecycle management.
- Design, secure, publish, analyze, monitor, monetize and scale APIs anywhere - Onpremises, Google Cloud, or hybrid
- Apigee sits in front of your APIs and provides:
	- Authentication
	- Rate limiting
	- Monitoring
	- Analytics
	- API keys
	- Quotas
	- Versioning

- Enable Caching with Cloud CDN

---

# Identity Platform
- Identity Platform is Google Cloud's managed authentication and customer identity service.
- It helps you add user sign-in and authentication to your applications.
- Typical sign-in methods:
	- Email/password
	- Google login
	- Facebook login
	- Apple login
	- Phone/SMS login
	- SAML / OIDC providers

**What It Does**
- Instead of building authentication yourself, Identity Platform handles:
	- User registration
	- Login
	- Password reset
	- Social login
	- Multi-factor authentication
	- User session management

```text
User
   │
   ▼
Login Screen
   │
   ▼
Identity Platform
   │
   ▼
Authenticated App
```

**Identity Platform vs IAM**
- Identity Platform → authenticate your app users
- IAM → control who can access a VM or BigQuery dataset

| Scenario | Solution |
|---|---|
| An application on a GCE VM needs access to Cloud Storage | Cloud IAM – Service Account |
| An enterprise user needs access to upload objects to a Cloud Storage bucket | Cloud IAM |
| I want to manage end users for my application | Identity Platform |
| I want to enable "Login using Facebook/Twitter" for my application | Identity Platform |
| I want to create user sign-up and sign-in workflows for my application | Identity Platform |

---

# Eventarc
- Eventarc is Google Cloud's event routing service.
- It lets you connect events from Google Cloud services to event consumers such as:
	- Cloud Run
	- Cloud Functions
	- GKE

 - Instead of microservices calling each other directly: `Order Service → Billing Service → Warehouse Service → Shipping Service`
 - which creates tight coupling, all services communicate through events:
 ```text
 Producer Services
       │
       ▼
  Event Manager
       │
       ▼
Consumer Services
 ```

- The importance of using an event-driven architecture with an event manager like Pub/Sub or Eventarc is that it makes your system much more scalable, flexible, and reliable.
- In an event-driven architecture:
	- Event Provider = the service that produces or emits an event
	- Event Destination = the service that receives and handles the event
	
- Example:
```
Cloud Storage uploads file
        │
        ▼
      Eventarc
        │
        ▼
Cloud Run service creates thumbnail
```
- In this example:
	- Event Provider = Cloud Storage
	- Event Destination = Cloud Run

---

# Service Directory
- Service Directory is GCP's service discovery service.
- It allows applications to find other services dynamically instead of hardcoding IP addresses or hostnames.

**Why It Is Needed**
- Without Service Directory: `Order Service calls: http://10.1.2.15:8080` , If the IP changes, the app breaks.
- With Service Directory: `Order Service calls: billing-service`, And Service Directory resolves it to the current address.

**Main Idea:** It works like an internal DNS for your microservices.
```text
Service Name
      │
      ▼
Service Directory
      │
      ▼
Current Endpoint / IP / Port

##

Order Service
      │
      ▼
Service Directory
      │
      ▼
Billing Service endpoint
```

---
