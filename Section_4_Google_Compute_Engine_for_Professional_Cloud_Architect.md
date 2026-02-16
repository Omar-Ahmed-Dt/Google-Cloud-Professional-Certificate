---
tags:
  - gcp
---

# Compute Engine

> **Key Takeaways**
> - IaaS offering for running VMs on GCP
> - Choose from different machine families based on workload
> - Static IPs remain attached even when instance is stopped
> - Use Instance Templates for repeatable VM configurations

## 1. Family
**What type of hardware do you want to run your workloads on?**
Different Machine Families for Different Workloads:
- General Purpose (E2, N2, N2D, N1): Best price-performance ratio, Web and application servers, Small-medium databases, Dev environments
- Memory Optimized (M2, M1): high memory workloads, Large in-memory databases and In-memory analytics
- Compute Optimized (C2): Compute intensive workloads, Gaming applications
- Storage Optimized (Z3): storage-intensive workloads, like horizontal, scale-out databases. 

## 2. Types
**How much CPU, memory or disk do you want?**
- Variety of machine types are available for each machine family

| Machine name   | vCPUs | Memory (GB) | Max number of persistent disks (PDs) | Max total PD size (TB) | Local SSD | Maximum egress bandwidth (Gbps) |
| -------------- | ----- | ----------- | ------------------------------------ | ---------------------- | --------- | ------------------------------- |
| e2-standard-2  | 2     | 8           | 128                                  | 257                    | No        | 4                               |
| e2-standard-4  | 4     | 16          | 128                                  | 257                    | No        | 8                               |
| e2-standard-8  | 8     | 32          | 128                                  | 257                    | No        | 16                              |
| e2-standard-16 | 16    | 64          | 128                                  | 257                    | No        | 16                              |
| e2-standard-32 | 32    | 128         | 128                                  | 257                    | No        | 16                              |


> **Example** e2-standard-2
> - e2 - Machine Type Family
> - standard - Type of workload
> - 2 - Number of CPUs

Memory, disk and networking capabilities increase along with vCPUs.
## 3. Image
**What operating system and what software do you want on the instance?**
Type of Images:
- Public Images: Provided & maintained by Google or Open source communities or third party vendors
- Custom Images: Created by you for your projects

---

# Internal and External IP Addresses

> **Important**
> - When you stop a VM instance, the External IP address is lost unless it's a Static IP.

- **External** (Public) IP addresses are **Internet addressable**.
- **Internal** (Private) IP addresses are **internal** to a corporate network
- You **CANNOT** have two resources with same public (External) IP address.
  - HOWEVER, two different corporate networks CAN have resources with same Internal (private) IP address
- All **VM instances** are assigned at least one Internal IP address
- Creation of External IP addresses can be enabled for VM instances
  - (Remember) When you stop an VM instance, External IP address is lost

---

# Static IP Addresses
- A Static IP address is a permanent, fixed IP address that remains constant and doesn't change over time.
- Static IP **remains attached** even if you stop the instance. You have to manually detach it.
- Select the same region as that of your VM (Static address should be in the same region as your VM)
- Static IP **can be switched** to another VM instance in same project

> **Cost Alert**
> - You are **billed for** a Static IP when **you are NOT using it!**
> - Make sure to explicitly release Static IPs when not in use to avoid unnecessary charges.

---

# Simplify VM Setup
- How do we **reduce** the **number of steps** in creating an VM instance and setting up a HTTP Server?
- Let's explore a few options:

1. **Startup script**
	- **Bootstrapping:** Install OS patches or software when an VM instance is launched.
	- In VM, you can configure **Startup script** to bootstrap
	
2. **Instance Template**
	- Need to specify all the VM instance details (Image, instance type etc) **every time** you launch an instance - How about creating a Instance template?
	- Define **machine type, image, labels, startup script** and other properties
	- Used to create **VM instances** and **managed instance groups**
	- Provides a way to create similar instances
	- **CANNOT** be updated: Once you create an instance template, you cannot change it.
	- To make a change, copy an existing template and modify it
	- **To avoid specifying all the VM instance details every time you create a VM**
	
3. **Custom Image**
	- Creating a custom image with OS patches and software **pre-installed**
	- Can be created from an instance, a persistent disk, a snapshot, another image, or a file in Cloud Storage
	- Can be shared across projects
	- (Recommendation) **Hardening an Image** - Customize images to your corporate security standards
	- **Preferred option to reduce the launch time of a VM instance**

---

# Compute Engine Scenarios

| Scenario | Solution |
|----------|----------|
| What are the prerequisites to be able to create a VM instance? | 1. Project<br>2. Billing Account<br>3. Compute Engine APIs should be enabled |
| You want dedicated hardware for your compliance, licensing, and management needs | Sole-tenant nodes |
| I have 1000s of VMs and I want to automate OS patch management, OS inventory management and OS configuration management (manage software installed) | Use "VM Manager" |
| You want to login to your VM instance to install software | You can SSH into it |
| You do not want to expose a VM to internet | Do NOT assign an external IP Address |
| You want to allow HTTP traffic to your VM | Configure Firewall Rules |


> Note
> - [Sole-tenant nodes](https://scribehow.com/viewer/Creating_VMs_on_Dedicated_Host_with_Sole_Tenant_Nodes__DY_BNQJmRo6Pq4JwcoDSRA)
> - **VM Manager**: `From Console => Compute Engine => Vm Manager => Patch`

---
