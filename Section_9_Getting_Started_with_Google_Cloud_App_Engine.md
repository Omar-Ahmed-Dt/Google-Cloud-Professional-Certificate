---
tags:
  - gcp
---
# App Engine
- Simplest way to deploy and scale your applications in GCP
	- Provides end-to-end application management
	
- Supports:
	- Go, Java, .NET, Node.js, PHP, Python, Ruby using pre-configured runtimes
	- Custom runtime (write code in any language)
	- Connect to various Google Cloud storage products (Cloud SQL, etc.)
	
- No usage charges - Pay only for **resources provisioned**
- Features:
	- Automatic load balancing & auto scaling
	- Managed platform updates & application health monitoring
	- Application versioning - Roll back to specific versions
	- Traffic splitting

## Compute Engine vs App Engine

| Feature                | Compute Engine                                       | App Engine             |
| ---------------------- | ---------------------------------------------------- | ---------------------- |
| Service Model          | IaaS                                                 | PaaS (Serverless)      |
| Flexibility            | More flexibility                                     | Lower flexibility      |
| Responsibility         | More responsibility                                  | Lesser responsibility  |
| Infrastructure Control | Full control over VM                                 | No server management   |
| OS Management          | You choose & manage OS image                         | Managed by Google      |
| Software Installation  | You install & configure                              | Managed runtime        |
| Hardware Choice        | You choose machine type                              | Abstracted             |
| Access & Security      | Fine-grained control (firewalls, certificates, etc.) | Mostly managed         |
| Availability           | You configure HA & scaling                           | Automatic scaling & HA |

## App Engine Environments
- **Standard**: Applications run in language-specific **sandboxes**
    - Complete isolation from OS / Disk / Other Apps
    - **V1**: Java, Python, PHP, Go (OLD Versions)
        - ONLY for Python and PHP runtimes:
            - Restricted network access and Only white-listed extensions and libraries are allowed
			
        - No restrictions for Java and Go runtimes
		
    - **V2**: Java, Python, PHP, Node.js, Ruby, Go (NEWER Versions)
        - Full network access and no restrictions on language extensions

- **Flexible**: Application instances run within Docker containers
    - Behind the scenes: Google creates Compute Engine VMs, Your app runs inside Docker, You have more control
    - Supports ANY runtime (built-in support for Python, Java, Node.js, Go, Ruby, PHP, .NET)
    - Provides access to background processes and local disks

## App Engine – Application Component Hierarchy
```text
Application
    → Service
        → Version
            → Instance
```

1. **Application (Top Level)**
	- **You can only have one App Engine application per Google Cloud project**
	- Lives in one region (chosen at creation time) - Once you choose a region (e.g., us-central1) for your App Engine app, you cannot change it for that project.
	- The container of everything.

2. **Service (The Microservices)**
	- Inside an application, you can have multiple services.
	- Each service acts as a standalone microservice (e.g., one service for your frontend, one for your api, and one for a background-worker).
	- Each service can have its own settings, scaling rules, and even different environments (Standard vs. Flexible).

3. **Version (The Deployments)**
	- Inside each Service, you can have multiple Versions.
	- Traffic Splitting: This is where the magic happens. You can have v1 and v2 running simultaneously. You can tell App Engine to send 90% of traffic to v1 and 10% to v2 to test a new feature.

4. **Instance (The Workers)**
	- These are the actual "virtual machines" or sandboxes where your code is running.
	- Depending on your traffic, a single Version might have 0 instances (scaled to zero), 1 instance, or 1,000 instances.
	- You don't manage these individually; App Engine spins them up and down based on the load.

```text
                               +-------------+
                               | Application |
                               +-------------+
                                      |
                  -----------------------------------------
                  |                                       |
          +---------------+                       +---------------+
          |    Service    |                       |    Service    |
          +---------------+                       +---------------+
                  |                                       |
          -----------------                       -----------------
          |               |                       |               |
     +----------+     +----------+            +----------+     +----------+
     | Version  |     | Version  |            | Version  |     | Version  |
     +----------+     +----------+            +----------+     +----------+
          |               |                       |               |
     -----------      -----------             -----------      -----------
     |         |      |         |             |         |      |         |
+----------+ +----------+   +----------+ +----------+  +----------+ +----------+
| Instance | | Instance |   | Instance | | Instance |  | Instance | | Instance |
+----------+ +----------+   +----------+ +----------+  +----------+ +----------+
```

## App Engine – Comparison

| Feature | Standard | Flexible |
|----------|----------|----------|
| Pricing Factors | Instance hours | vCPU, Memory & Persistent Disks |
| Scaling | Manual, Basic, Automatic | Manual, Automatic |
| Scaling to zero | Yes | No (Minimum one instance) |
| Instance startup time | Seconds | Minutes |
| Rapid Scaling | Yes | No |
| Max. request timeout | 1 to 10 minutes | 60 minutes |
| Local disk | Mostly (except for Python, PHP). Can write to `/tmp`. | Yes. Ephemeral. New disk on startup. |
| SSH for debugging | No | Yes |

## App Engine – Scaling Instances
- **Automatic** – Automatically scale instances based on the load:
    - Recommended for continuously running workloads
        - Auto scale based on:
            - Target CPU Utilization – Configure a CPU usage threshold
            - Target Throughput Utilization – Configure a throughput threshold
            - Max Concurrent Requests – Configure max concurrent requests an instance can receive
			
        - Configure Max Instances and Min Instances

- **Basic** – Instances are created as and when requests are received:
    - Recommended for ad-hoc workloads
        - Instances are shut down if ZERO requests
            - Tries to keep costs low
            - High latency is possible
			
        - NOT supported by App Engine Flexible environment
        - Configure Max Instances and Idle Timeout

- **Manual** – Configure specific number of instances to run:
    - Adjust number of instances manually over time

## App Engine - From the Command Line
```bash
cd default-service
gcloud app deploy
gcloud app services list
gcloud app versions list
gcloud app instances list
gcloud app deploy --version=v2
gcloud app versions list
gcloud app browse # url for app
gcloud app browse --version 20210215t072907
gcloud app deploy --version=v3 --no-promote  # --no-promote : don't switch the traffic to a new version
gcloud app browse --version v3
gcloud app services set-traffic split=v3=.5,v2=.5
gcloud app services set-traffic splits=v3=.5,v2=.5
watch curl https://melodic-furnace-304906.uc.r.appspot.com/
gcloud app services set-traffic --splits=v3=.5,v2=.5 --split-by=random
 
cd ../my-first-service/
gcloud app deploy
gcloud app browse --service=my-first-service
 
gcloud app services list
gcloud app instances list
gcloud app versions list
```
