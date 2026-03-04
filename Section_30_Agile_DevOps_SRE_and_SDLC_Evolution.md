---
tags:
  - gcp
---
## Google Cloud Deployment Manager
- Google Cloud Deployment Manager is an Infrastructure as Code (IaC) service in Google Cloud Platform (GCP) used to automatically create, configure, and manage cloud resources using declarative configuration files.
- Instead of manually creating resources (VMs, networks, databases) from the console, you define them in code, and Deployment Manager provisions them for you.
- Define infrastructure using templates and deploy automatically in GCP
- You describe infrastructure in a YAML or Python template.
- Free to use - Pay only for the resources provisioned

**With Deployment Manage**
```text
Write one config file
Run deployment command
All resources created automatically
```

```text
Configuration File (YAML / Python)
           │
           ▼
   Deployment Manager
           │
           ▼
   Creates GCP Resources
           │
   ┌───────────────┬───────────────┬───────────────┐
   ▼               ▼               ▼
Compute Engine   Cloud Storage   Cloud SQL
```

**Example YAML Configuration**
```yaml
resources:
- name: my-vm
  type: compute.v1.instance
  properties:
    zone: us-central1-a
    machineType: zones/us-central1-a/machineTypes/e2-medium

    disks:
    - deviceName: boot
      type: PERSISTENT
      boot: true
      autoDelete: true
      initializeParams:
        sourceImage: projects/debian-cloud/global/images/family/debian-11

    networkInterfaces:
    - network: global/networks/default
      accessConfigs:
      - name: External NAT
        type: ONE_TO_ONE_NAT
```

---

## Site Reliability Engineering (SRE) – Key Metrics
- **Service Level Indicator (SLI)**: Quantitative measure of an aspect of a service.
    - Categories: availability, latency, throughput, durability, correctness (error rate)
    - Typically aggregated – "Over 1 minute"

- **Service Level Objective (SLO)** – SLI + target
    - 99.99% availability, 99.999999999% durability
    - Response time: 99th percentile – 1 second
    - Choosing an appropriate SLO is complex

- **Service Level Agreement (SLA)** – SLO + consequences (contract)
    - What is the consequence of **NOT** meeting an SLO? (Defined in a contract)
    - Have stricter internal SLOs than external SLAs

- **Error budgets** – (100% − SLO)
    - How well is a team meeting their reliability objectives?
    - Used to manage development velocity

---
