---
tags:
  - gcp
---
## Prerequisites
- Every GCP resource belongs to one of three scopes:
	- Global: Exists across all regions worldwide
	- Regional: Exists inside one region
	- Zonal: Exists inside one zone

```text
Global
  ↓
Region (geographic area)
  ↓
Zone (datacenter inside region)


europe-west1 (Region)
   ├── europe-west1-a (Zone)
   ├── europe-west1-b
   └── europe-west1-c
```

1. **Global Resources**
	- Global resources are not tied to any region or zone - `VPC Network` , `Firewall rules` , `Global Load Balancer` , `Cloud DNS` , `IAM policies` 
	- Failure Behavior: survives regional outage - (because not tied to one region)
	
2. **Regional Resources** 
	- Regional resources exist inside one region, but span multiple zones inside that region - `Subnets` , `Cloud SQL`, `Cloud NAT`, `Static regional IP`
	- Failure Behavior: survives zone failure and fails if region fails
	
3. **Zonal Resources**
	- Zonal resources live inside one single zone - `VM instance`, `Persistent Disk`, `GKE node`
	- Failure Behavior: affected by zone outage 

**VPC networks are GLOBAL but subnets are REGIONAL.**

---

## VPC - Virtual Private Cloud
- It is basically your private network inside Google’s infrastructure - like having your own isolated data-center network, but fully managed by Google.
- Network traffic within a VPC is isolated (not visible) from all other Google Cloud VPCs
- You control all the traffic coming in and going outside a VPC
- (Best Practice) Create all your GCP resources (compute, storage, databases etc) within a VPC
    - Secure resources from unauthorized access
    - Enable secure communication between your cloud resources

- VPC is a **global resource** and contains subnets in **one or more region**
    - (REMEMBER) NOT tied to a region or a zone. VPC resources can be in any region or zone.

---

## Subnets
- A subnet is a range of internal IP addresses inside a VPC where resources live.
- Each subnet belongs to ONE REGION.

```text
VPC (global network)
    ├── Subnet A (10.0.1.0/24) → Backend
    ├── Subnet B (10.0.2.0/24) → Database
    └── Subnet C (10.0.3.0/24) → Internal services
```

**Public vs Private Subnet**
- Resources in a public subnet CAN be accessed from internet 
- Resources in a private subnet CANNOT be accessed from internet
- BUT resources in public subnet can talk to resources in private subnet
- Each Subnet is created in a region

**NOTE**

| Platform | Subnet Scope      |
| -------- | ----------------- |
| GCP      | Regional          |
| AWS      | Availability Zone |

---

## Creating VPCs and Subnets
- VPC types:
	- **OPTION 1**: Default VPC
		- Created automatically once compute engine API enabled in GCP project.
		- Predefined Subnet IPs - Auto created subnet `/20` in every region
		- No IPV6 subnets
		- Default VPC is Auto Mode VPC

	- **OPTION 2**: Auto mode VPC network
		- Subnets are automatically created in each region - One Subnet per region & can be extended
		- When you create a new GCP project, GCP automatically creates a default vpc with Auto Mode.

	- **OPTION 3**: Custom mode VPC network
		- No subnets are automatically created
		- You have complete control over subnets and their IP ranges
		- Recommended for Production

- Default VPC and Auto Mode VPC are used for development and learning purpose only
- You need to design and have complete control over your production networks with custom mode VPCs.
- Options when you create a subnet:
    - Enable **Private Google Access**: Allows VM's to connect to Google API's using private IP's - the communication happens within the goole network. If using a public ip, The communication goes over internet
    - Enable **FlowLogs**: To troubleshoot any VPC related network issues

---

## CIDR (Classless Inter-Domain Routing) Blocks
- CIDR is a way to define a range of IP addresses. 
- CIDR Format: `IP_ADDRESS - Starting IP Addresse / PREFIX_LENGTH - Range`: `10.0.0.0/24`
-  The slash value (e.g. /24) shows how many bits belong to the network prefix; the rest identify individual hosts.
- In every subnet, GCP reserves **4** IP addresses automatically - Google Cloud reserves the first TWO and the last TWO IPv4 addresses in each PRIMARY subnet range.
	- First IP: Network address
	- Second IP: Default gateway
	- Second-to-last: Reserved by Google
	- Last: Broadcast address
	
-  [CIDR Calculator](https://cidr.xyz/)
- Example: **10.88.135.144/28**
	- 32-28 = 4 , 2^4= 16 , 144+15 = 159 , `10.88.135.144 → 10.88.135.159`
	- `128 192 224 240 248 252 254 255` ,  /28 = 240 
		- **Unusable Addresses**:
			- Network address: `10.88.135.144`
			- Default gateway: `10.88.135.145`
			- Second-to-last - Reserved by Google: `10.88.135.158` 
			- Last - Broadcast address: `10.88.135.159`
			
		- **Usable IPs**: Start from: `10.88.135.146`, End at: `10.88.135.157`
		- **Count**: 16
		- **Netmask**: 255.255.255.240

### Internal IP address types
1️⃣ **Ephemeral (Automatic) Internal IP**
What it means
- Google automatically assigns an available IP from subnet
- You don’t choose the IP
- It may change if VM is deleted and recreated

2️⃣ **Ephemeral (Custom) Internal IP**
What it means
- You manually choose the IP
- It must be within subnet range
- It is NOT reserved permanently
- If VM deleted → IP can be reused by someone else

3️⃣ **Static Reserved Internal IP**
What it means
- You reserve an IP first
- It belongs to your project
- It will NOT change
- It stays reserved even if VM deleted

### External IP - Public IP address types
**Regional External IPv4 Address - Regional External IPv6 Address**
- An external IPv4/IPv6 address that belongs to one specific region.
- Each region has its own set of external IP addresses
- For use by zonal or regional resources or passthrough NLBs.
- Used By
	- VM instances
	- Regional load balancers
	- Passthrough Network Load Balancer (L4)

```text
Region: us-central1
External IP: 34.x.x.x
```

**Global External IPv4 Address - Global External IPv6 Address**
- An IP reachable from anywhere using Anycast.
- Google advertises the same IP globally.
- Same IP exists in multiple locations - User automatically connects to nearest Google edge.
- Used By
	- Global HTTP(S) Load Balancer
	- Application Load Balancer
	- Proxy-based L7 load balancing

**All IPs Can be ephemeral or static reserved.**

---

## Firewall Rules
- Configure Firewall Rules to control traffic going in or out of the network:
    - **Stateful**, If traffic is allowed in one direction, the firewall automatically allows the response traffic back - even if no rule explicitly allows it.
    - Each firewall rule has priority (0–65535) assigned to it , the default value is 1000
    - 0 has **highest priority**, 65535 has **least priority**
    - Default Rules: **implied rules** with lowest priority (65535)
        - **Allow all egress** (Direction: `Egress`,  Target: `All Instances` , IP Filter : `0.0.0.0/0` , Protocol &Ports: `All` , Action: `Allow` , Priority: `65535`), **Deny all ingress** (Direction: `Ingress`,  Target: `All Instances` , IP Filter : `0.0.0.0/0` , Protocol &Ports: `All` , Action: `Deny` , Priority: `65535`)
        - Default rules can't be deleted, You can override default rules by defining new rules with priority 0–65534

    - Default VPC has **4 additional rules with priority 65534**
        - Allow incoming traffic from VM instances in same network (**default-allow-internal**)
        - Allow incoming TCP traffic on port 22 (SSH) **default-allow-ssh**
        - Allow incoming TCP traffic on port 3389 (RDP) **default-allow-rdp** , `Remote Desktop (RDP) access to virtual machines.`
        - Allow incoming ICMP from any source on the network **default-allow-icmp**, ping protocol

	- Firewall Rules are defined and configured on a VPC level But They are applied on an Instance level

---

## Firewall Rules — Ingress and Egress Rules
- **Ingress Rules**: Incoming traffic from outside to GCP targets
    - Target (defines the destination): All instances or instances with TAG/SA -  A network tag is a label you attach to a VM, Firewall rules can then target All VMs that have this tag.
    - Source (defines where the traffic is coming from): CIDR, instances with TAG/SA

- **Egress Rules**: Outgoing traffic to destination from GCP targets
    - Target (defines the source): All instances or instances with TAG/SA
    - Destination: CIDR Block

- Along with each rule, you can also define:
    - Priority - Lower the number, higher the priority
    - Action on match - Allow or Deny traffic
    - Protocol - ex. TCP or UDP or ICMP
    - Port - Which port?
    - Enforcement status - Enable or Disable the rule

---

## Firewall Rules - Best Practices
- Use network tags and control allowed traffic into a VM using firewall rules
- Ensure that firewall rule allow the right kind of traffic:
    - Only allow traffic from load balancing into VM instances - `User → VM directly ❌ (not secure)`,, `User → Load Balancer → VM ✅ (correct)` 
    - `0.0.0.0/0` = ALL IP ADDRESSES ON THE INTERNET, Remove 0.0.0.0/0 from Source IP ranges - Do NOT allow traffic from the entire internet.
        - Example: Add 130.211.0.0/22 and 35.191.0.0/16 
        - Allows health checks from load balancing to VM instances

- (REMEMBER) **All egress from an VM instance is allowed by default**:
    - To allow Specific EGRESS ONLY
        - 1: Create an egress rule with **low priority** to **deny all traffic**
        - 2: Create egress rule with **high priority** to **allow traffic on specific port**

---

## Shared VPC
- Scenario: Your organization has multiple projects. You want resources in different projects to talk to each other?
- How to allow resources in different projects to talk with internal IPs securely and efficiently?
- One central VPC network shared across multiple GCP projects - Allows VPC network to be shared between projects in same organization
- Created at organization or shared folder level **(Access Needed: Shared VPC Admin)** - You need to have this role to create a shared VPC.
- Allows VPC network to be shared between projects in same organization
- Shared VPC contains one host project and multiple service projects: 
	- **Host Project** - Contains shared VPC network
		- Owns: Subnets, Routes, Firewall rules, Cloud NAT, VPN / Interconnect
		- Managed by network administrators
		
	- **Service Projects** - Attached to host projects
		- Do NOT contain a VPC
		- Attach to Host project
		- Can: Create VMs, Create GKE clusters, Deploy Cloud Run
		- Use subnets from Host project
		- Managed by application / resource teams
	
- Helps you achieve separation of concerns:
	- Network administrators responsible for Host projects and Resource users use Service Project

 ---

## VPC Peering
- **Scenario:** How to connect VPC networks across different organizations?
- Enter **VPC Peering**
    - Networks in same project, different projects and across projects in different organizations can be peered
    - All communication happens using internal IP addresses
        - Highly efficient because all communication happens inside Google network
        - Highly secure because not accessible from Internet
        - No data transfer charges for data transfer between services

- **(REMEMBER)** Network administration is NOT changed:
    - Admin of one VPC do not get the role automatically in a peered network

---

## Identity-Aware Proxy (IAP)
- Identity-Aware Proxy (IAP) is a Google Cloud service that lets you access applications and VMs securely using IAM-based identity instead of exposing them to the internet.
- Access based on who you are, Not based on public IP access
- [docs-concepts-overview](https://docs.cloud.google.com/iap/docs/concepts-overview)

**Traditional VM access:**
```text
Internet → External IP → VM
```

**IAP Secure Model:**
```text
User
  ↓ (authenticated with Google)
Google Identity
  ↓
IAP Tunnel
  ↓
Private VM (NO external IP)
```

### How IAP Works for VM Access - Flow
- [img - How IAP works](https://docs.cloud.google.com/iap/docs/concepts-overview#compute-engine)

**Step 1 - User Runs Command**
```bash
gcloud compute ssh VM_NAME --tunnel-through-iap
```
This tells gcloud to:
- The client establishes a HTTPS connection to IAP.
- SSH traffic is encapsulated inside HTTPS - **SSH over HTTPS** 

So: `Client → HTTPS (TLS) → IAP` , No direct connection to VM.

**Step 2 - Authentication (Who Are You?)**
The user must authenticate with Google os IAP uses:
- Google Identity (Google Account / Cloud Identity / Workspace)
- OAuth2 authentication

**Step 3 - Authorization (Are You Allowed?)**
- IAP then checks IAM policies.
- Required role: `roles/iap.tunnelResourceAccessor` , This role allows to establish TCP tunnel via **IAP to a specific VM**. `Without this role → tunnel denied.`

**Step 4 - OS Login / Compute Permission Check**

```text
User Identity
      ↓
IAP Tunnel
      ↓
Compute Engine API
      ↓
VM Service Account (IMPORTANT)
      ↓
OS Login / SSH session
```
- GCP needs to impersonate the target VM's service account to establish the tunnel. Without `roles/serviceAccountUser` on that service account, you'll get a permission denied error.
- IAP controls network tunnel, but SSH login requires compute permissions.
	- `roles/compute.instanceAdmin.v1` (or) `roles/compute.osLogin` (preferred secure model)

If OS Login is enabled (recommended):
- SSH access is controlled via IAM
- No metadata SSH keys required. If OS Login is not enabled, SSH key must be added to instance metadata

**Step 5 - IAP Forwards Traffic to VM**
- IAP forwards traffic from its internal IP range: `35.235.240.0/20`, So your firewall must allow: `Ingress , TCP port 22, Source: 35.235.240.0/20` , Only IAP can reach it. 

**Step 6 - VM Does NOT Need External IP**
- VM must **NOT require external IP**
- IAP works with **internal IP only**
- Traffic stays inside Google network

**Summary - Authentication & Authorization Chain:**
1. User runs gcloud ssh
2. User authenticates via Google OAuth
3. IAP checks IAM (iap.tunnelResourceAccessor)
4. Compute checks SSH permission (osLogin or instanceAdmin)
5. Firewall allows traffic from 35.235.240.0/20
6. SSH session established

---
