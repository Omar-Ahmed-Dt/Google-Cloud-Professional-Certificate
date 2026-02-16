---
tags:
  - gcp
---
# High Availability for Compute Engine & Load Balancing
- **Highly Available Architecture**
	- Multiple Regional Instance Groups for each Microservice
	- Distribute load using a Global HTTPS Load Balancing
	- Configure Health Checks for Instance Group and Load Balancing
	- Enable Live Migration for VM instances

- **Advantages**
	- Instances distributed across regions, Even if a region is down, your app is available
	- Global Load Balancing is highly available
	- Health checks ensure auto healing

---

# What is Scalability
- Ability to adapt to changes in demand (users, data)
- Deploy to a bigger instance with bigger CPU and more memory
- Increase the number of application instances and setup a load balancer

## Vertical Scaling?
Small  →  XLarge
- Deploying application/database to a **bigger instance**:
	- A larger hard drive
	- A faster CPU
	- More RAM, CPU, I/O, or networking capabilities

- Example: Increasing VM machine size: `e2-standard-2 to e2-standard-4` || `e2-standard-16 to e2-standard-32`
- you need to stop the VM then change the Vm type

### There are limits to vertical scaling
- Physical Hardware Limits
- Manually resize - **Not flexible for traffic spikes.**
- Downtime During Upgrade
- Database Locking & Concurrency Limits

## Horizontal Scaling?
- Deploying multiple instances of application/database
- (Typically but not always) Horizontal Scaling is preferred to Vertical Scaling:
	- Vertical scaling has limits
	- Vertical scaling can be expensive
	- Horizontal scaling increases availability

- (BUT) Horizontal Scaling needs additional infrastructure:
	- Load Balancers, etc.

- Example: 
	- Distribute VM instances: `In a single zone` || `In multiple zones within a single region` || `In multiple zones across multiple regions`
	- Auto scale: Managed Instance Group(s)
	- Distribute load: Load Balancing

---

# Live Migration & Availability Policy
Your VM runs on a physical host machine inside Google’s data center, Sometimes Google must:
- Update host OS
- Patch security
- Replace hardware
- Fix hardware failure

So the question is What happens to your VM when the physical host needs maintenance - infrastructure maintenance ? 
## Live Migration
- Google moves your running VM to another physical host in the same zone
	- WITHOUT shutting down
	- Does NOT change any attributes or properties of the VM: Same IP, disk, memory, config
	- SUPPORTED for instances with local SSDs
	- NOT SUPPORTED for GPUs and preemptible instances

**Important Configuration - Availability Policy:** `Console: VM instance => Create instance => Machine configuration => VM Provisioning model advanced settings => On host maintenance => Migrate Vm instance || Terminate VM instance ,, Automatic restart: On || Off`
1. On Host Maintenance: What should happen during infrastructure maintenance?
	- **Migrate (Default)**: Google live migrates your VM to another host → No downtime.
	- **Terminate**: Google stops the VM during maintenance → VM shuts down.

2. Automatic Restart: Restart VM instances if they are terminated due to non-user-initiated reasons (maintenance event, hardware failure etc ) - What if VM stops due to Hardware failure , Maintenance event, Host crash
	- If **Automatic Restart = ON**: Google restarts your VM automatically.
	- If **OFF**: VM stays stopped.

---

# Compute Engine Features: GPUs
- How do you accelerate math-intensive and graphics-intensive workloads for AI/ML, etc.?
- Add a GPU to your virtual machine:
	- High performance for math-intensive and graphics-intensive workloads
	- Higher cost
	- (REMEMBER) Use images with GPU libraries (Deep Learning) installed , Otherwise, the GPU will not be used
	- GPU restrictions:
		- NOT supported on all machine types  (e.g., not supported on shared-core or memory-optimized machine types)
		- On host maintenance can only have the value **Terminate VM instance**
		
- Recommended Availability Policy for GPUs:
	- Automatic restart: ON

---

# Resiliency for Compute Engine & Load Balancing
Ability of a system to provide acceptable behavior even when one or more parts of the system fail
- Build Resilient Architectures:
	- Run VMs in a Managed Instance Group (MIG) behind global load balancing
	- Enable Live Migration and Automatic restart when available
	- Configure the right health checks
	- (Disaster recovery) Up-to-date image copied to multiple regions

---

# Preemptible VM - Spot VM
- **Short-lived cheaper** (up to 60 : 91% discount compared to on-demand VMs) compute instances, **ideal for fault tolerant workloads**
	- Can be stopped by GCP anytime (preempted) within 24 hours
	- Instances get a 30-second warning (to save anything they want to save)
	- NOT always available
	
- **Use Preemptible VMs if:**
	- Your applications are fault tolerant
	- You are very cost sensitive
	- Your workload is NOT immediate
	- Example: Non-immediate batch processing jobs
	
- Restrictions
	- NOT always available
	- NO SLA and CANNOT be migrated to regular VMs
	- NO Automatic Restarts
	- Free Tier credits not applicable

---

# Google Compute Engine – Billing
- You are billed by the second (after a minimum of 1 minute) - if it runs for 10 seconds, you are charged for at least 60 seconds, So the first minute is mandatory. After the first minute, Google charges you per second.
- You are NOT billed for compute when a compute instance is stopped
	- However, you will be billed for any storage attached to it
	
- (RECOMMENDATION) Always create Budget alerts and make use of Budget exports to stay on top of billing
- What are the ways you can save money?
	- Choose the right machine type and image for your workload
	- Be aware of the discounts available

---

# Compute Engine & Load Balancing - Cost Efficiency
- Use Auto Scaling
	- Have optimal number and type of VM instances running
	
- Understand Sustained Use Discounts `Discounts are automatic discounts you get when you run a VM for a large portion of the month, The longer Vm runs → the bigger the discount.`
- Make use of Committed Use Discounts `You promise Google that you will use a certain amount of compute for 1 or 3 years, you get a big discount` For predictable long-term workloads
- Use Preemptible VMs For non-critical or fault-tolerant workloads

---
