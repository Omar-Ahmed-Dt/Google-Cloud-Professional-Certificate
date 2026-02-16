---
tags:
  - gcp
---
# Instance Groups
**What is an Instance Group?**
- An Instance Group is a group of VM instances (Compute Engine VMs) that are managed together as a single logical unit.
- Instead of managing VMs one by one, you manage them as a group.
- Location can be Zonal (**Runs in one zone, less resilient**) or Regional (**Regional gives a high availability - as it Runs instances across multiple zones - Recommended)**

**Types of Instance Groups**
There are two main types:
1. **Managed Instance Group (MIG)**
	**identical VMs created from an Instance Template**.
	types: stateless (apps don't have any state) or stateful (for Database)
	It’s a blueprint that defines:
	- Machine type (e.g., e2-medium)
	- OS image
	- Startup script
	- Disk configuration
	- Network settings
	- Metadata
	- Every VM in the group is created from this template.
	
	Managed Instance Groups provide automation and high availability features:
	- Maintain a Fixed Number of Instances
	- Self-healing
	- Health Checks
	- Auto Scaling
	- Rolling Updates
	- Canary Deployment - You deploy new version to 1 or 2 instances first, test it, If everything works → roll out to rest, If broken → rollback safely.
	
2. **Unmanaged Instance Group**
	**Different configurations for VMs in same group**:
	- VMs can be different
	- No instance template
	- No auto scaling
	- No self-healing
	- No rolling updates
	
	Basically:  
	You are responsible for everything manually.
	
	When would you use it?
	Rare cases:
	- You need different VM types in same group
	- Custom configurations
	
---

# Steps to Creating Managed Instance Group (MIG)
- **Instance template** is mandatory 
- Configure **auto-scaling** to automatically adjust number of instances based on load:
	- Auto Scaling Mode: **On** (add and remove instances to and from the group) , **Scale out** (only add instance to the group), **Off** (do not auto-scale)
		- Minimum number of instances
		- Maximum number of instances
		- Autoscaling metrics: Like CPU Utilization or any other metric from Stackdriver
			- Cool-down period: How long to wait before looking at auto scaling metrics again?
			- Scale In Controls: Prevent a sudden drop in number of VM instances
				- Example: Don't scale in by more than 10% or 3 instances in 5 minutes
				
- **Initialization Period**: is the time allotted for a new VM instance to start and become fully operational before it starts receiving traffic.
- **Autohealing**: Configure a Health check with 
	- Protocol , Port, Path , Check Interval , Timeout, Healthy Threshold , Unhealthy Threshold
	- Initial Delay - refers to the time duration that a load balancer or health checker waits before starting to perform health checks on a newly registered or newly created instance.

```sh
# test scaling on Ubuntu (cpu utilization)
nohup bash -c 'while true; do wrk -t4 -c200 -d60s http://localhost/; sleep 2; done' \
  > loadtest.log 2>&1 &
```

If you want two VMs to always be running, `set the minimum and maximum number to the same value`.

## Updating a Managed Instance Group (MIG)
### 1. Rolling update
Gradual update of instances in an instance group **to the new instance template**.
- Specify **new template - Optional**: You can use a second instance template for canary deployment and specify the target number (or percentage) of VMs that will run from this template.
- Specify **Edit type**: how you want the update to be done: When should the update happen?
	- **Selective - Opportunistic**: `allows you to update only specific instances or settings within a managed instance group, rather than applying changes to all instances simultaneously - You manually trigger an update for specific instances`.
	- **Automatic - Proactive**: `refers to applying updates or changes automatically to all instances within a managed instance group. This method ensures that all instances are consistently updated without manual intervention - You don't need to trigger updates for individual instances, the MIG controller handles everything`.
		- **Actions allowed to update VMs**:
			- Refresh, restart or replace
				- Refresh:
					- Updates instance metadata and properties without stopping the VM
					- Changes things like instance template metadata, labels, or network tags
					- VM keeps running—minimal disruption
					
				- Restart:
					- Stops and restarts the VM to apply changes
					- Used for updates that require a reboot (like changing machine type or certain disk configurations)
					- Brief downtime during restart
					
				- Replace:
					- Deletes the old instance and creates a new one from the updated template
					- Most disruptive but ensures complete consistency with new template
					- Gets a new name, internal IP (unless using static), and ephemeral disk data is lost
					
			- Restart or replace
			- Refresh or restart
			- Only refresh
			- Only restart
			- Only replace
	
- How should the update happen?
	- **Maximum surge**: How many instances are added before deleting the old instances? 
	- **Maximum unavailable**: How many instances can be offline during the update?

### 2. Rolling Restart / Replace
Gradual restart or replace of all instances in the group.
- **No change in template, but restart or replace existing VMs**
- Configure:
	- **Maximum surge**
	- **Maximum unavailable**
	- Choose action: (**Restart** / **Replace**)

---

# Load Balancing
- Distributes user traffic across instances of an application in a single region or multiple regions  
- Fully distributed, software-defined managed service  
- Important Features:
	- Health check – Routes traffic to healthy instances  
		- Recovers from failures  
	- Auto Scaling  
	- Global load balancing with a single anycast IP `The same IP address exists in multiple Google edge locations around the world.` 
	    - Also supports internal load balancing  

- Enables:
	- High Availability  
	- Auto Scaling  
	- Resiliency  

| Layer | Protocols |
|-------|-----------|
| **Application Layer (Layer 7)** | HTTP, HTTPS, SMTP |
| **Transport Layer (Layer 4)** | TCP, TLS, UDP |
| **Network Layer (Layer 3)** | IP |

## Layer 7 Load Balancer (HTTP/HTTPS)

👉 Works at Application layer
Supports:
- HTTP
- HTTPS
- gRPC

What it can do:
- URL-based routing (/api, /images)
- Host-based routing (api.example.com)
- SSL termination
- Cloud CDN
- Cloud Armor (WAF)
- Header-based routing
- Redirect HTTP → HTTPS

## Layer 4 Load Balancer (TCP/UDP)
👉 Works at Transport layer
Supports:
- TCP
- UDP
- SSL proxy
- Internal TCP/UDP

What it does:
- Forwards packets based on IP and port
- Does NOT understand HTTP paths
- No URL routing
- No header inspection

**When To Use What?**
Use Layer 7 if:
- Web application
- Need path routing
- Need CDN
- Need WAF
- Need host-based routing

Use Layer 4 if:
- Database traffic
- Game servers
- Custom TCP app
- **High performance TCP**
- Non-HTTP protocols

## Load Balancer Configurations
- **Backend** - Group of endpoints that receive traffic from a Google Cloud Load Balancer  (example: instance groups)
- **Frontend** - Specify an IP address, port, and protocol. This IP address is the frontend IP for client requests ( For SSL, a certificate must also be assigned )
- **Host and Path Rules** (For HTTP(S) Load Balancing) – Define rules that redirect traffic to different backends:
	- Based on **path** – `in28minutes.com/a` vs `in28minutes.com/b`
	- Based on **host** – `a.in28minutes.com` vs `b.in28minutes.com`
	- Based on **HTTP headers** (e.g., Authorization header) and methods (POST, GET, etc.)

## Load Balancing – SSL/TLS Termination / Offloading
- **Client to Load Balancer:** Over the internet (HTTPS recommended)
- **Load Balancer to VM instance:** Through Google internal network (HTTP is OK, HTTPS is preferred)
- **SSL/TLS Termination / Offloading**
	- Client → Load Balancer: HTTPS / TLS  
	- Load Balancer → VM instance: HTTP / TCP  

![lb-product-tree](https://docs.cloud.google.com/static/load-balancing/images/lb-product-tree.svg)


## HTTP(S) Load Balancing - 1 - Multi Regional Microservice
- Backend Services
	- One Backend Service for the Microservice
	
- Backends
	- Multiple backends for each microservice MIG in each region
	
- URL Maps
	- URL /service-a maps to the microservice Backend Service
	
- Global routing: Route user to nearest instance (nearest regional MIG - A user connects to ONE global IP, and Google automatically routes them to the nearest healthy backend region.)
	- Needs Networking Premium Tier:
		- **As in STANDARD Tier:**
			- Forwarding rule and its external IP address are regional
			- All backends for a backend service must be in the same region as the forwarding rule

```text
                   +----------------------+
                   |   Cloud Load         |
                   |     Balancing        |
                   +----------+-----------+
                              |
                              v
                      +------------------+
                      |  Backend Service |
                      |    (/service-a)  |
                      +---------+--------+
                                |
        -------------------------------------------------
        |                       |                       |
        v                       v                       v

+-------------------+   +-------------------+   +-------------------+
| (Backend)         |   | (Backend)         |   | (Backend)         |
| Instance Group:   |   | Instance Group:   |   | Instance Group:   |
|                   |   |                   |   |                   |
| Microservice A    |   | Microservice A    |   | Microservice A    |
| Region A          |   | Region B          |   | Region C          |
+-------------------+   +-------------------+   +-------------------+
```

## HTTP(S) Load Balancing - 2 - Multiple Microservices
- **Backend Services**
	- One Backend Service for each Microservice

- **Backends**
	- Each microservice can have multiple backend MIGs in different regions
	- In the example, we see one backend per microservice

- **URL Maps**
	- URL `/service-a` => Microservice A Backend Service
	- URL `/service-b` => Microservice B Backend Service

```text
                    +----------------------+
                    |   Cloud Load         |
                    |     Balancing        |
                    +----------+-----------+
                               |
                               v
                        +--------------+
                        |   URL Maps   |
                        +------+-------+
                               |
               ----------------+----------------
               |                                 |
               v                                 v

      +------------------+              +------------------+
      |  Backend Service |              |  Backend Service |
      |    (/service-a)  |              |    (/service-b)  |
      +---------+--------+              +---------+--------+
                |                                 |
                v                                 v

      +-------------------------+       +-------------------------+
      |      Instance Group     |       |      Instance Group     |
      |     Microservice A      |       |     Microservice B      |
      +-------------------------+       +-------------------------+
```

## HTTP(S) Load Balancing - 3 - Microservice Versions
- **Backend Services**
	- A Backend Service for each Microservice version

- **Backends**
	- Each microservice version can have multiple backend MIGs in different regions
		- In the example, we see one backend per microservice version

- **URL Maps**
	- `/service-a/v1` => Microservice A V1 Backend Service
	- `/service-a/v2` => Microservice A V2 Backend Service
	- `/service-b` => Microservice B Backend Service

- **Flexible Architecture**
	- With HTTP(S) load balancing, route global requests across multiple versions of multiple microservices!

```text
                    +----------------------+
                    |   Cloud Load         |
                    |     Balancing        |
                    +----------+-----------+
                               |
                               v
                        +--------------+
                        |   URL Maps   |
                        +------+-------+
                               |
        ---------------------------------------------------
        |                         |                       |
        v                         v                       v

+------------------+      +------------------+      +------------------+
|  Backend Service |      |  Backend Service |      |  Backend Service |
| (/service-a/v1)  |      | (/service-a/v2)  |      |   (/service-b)   |
+---------+--------+      +---------+--------+      +---------+--------+
          |                         |                         |
          v                         v                         v

+-------------------------+  +-------------------------+  +-------------------------+
|      Instance Group     |  |      Instance Group     |  |      Instance Group     |
|     Microservice A      |  |     Microservice A      |  |     Microservice B      |
|       Version 1         |  |       Version 2         |  |       Version 1         |
+-------------------------+  +-------------------------+  +-------------------------+
```

---