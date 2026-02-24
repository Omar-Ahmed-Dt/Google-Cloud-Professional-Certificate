---
tags:
  - gcp
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
- By default, every project has a default VPC
- You can create YOUR own VPCs:
    - **OPTION 1**: Auto mode VPC network
        - Subnets are automatically created in each region
        - When you create a new GCP project, GCP automatically creates a default vpc with Auto Mode.

    - **OPTION 2**: Custom mode VPC network
        - No subnets are automatically created
        - You have complete control over subnets and their IP ranges
        - Recommended for Production

- Options when you create a subnet:
    - Enable **Private Google Access**: Allows VM's to connect to Google API's using private IP's - the communication happens within the goole network. if using a public ip, The communication goes over internet
    - Enable **FlowLogs**: To troubleshoot any VPC related network issues

---

## CIDR (Classless Inter-Domain Routing) Blocks
- CIDR is a way to define a range of IP addresses. 
- CIDR Format: `IP_ADDRESS - Starting IP Addresse / PREFIX_LENGTH - Range`: `10.0.0.0/24`
-  The slash value (e.g. /24) shows how many bits belong to the network prefix; the rest identify individual hosts.
-  [CIDR Calculator](https://cidr.xyz/)
- Example: **10.88.135.144/28**
	- 32-28 = 4 => 2^4= 16 => 144+15 = 159
	- `128 192 224 240 248 252 254 255` => /28 => 240
		- CIDR Base IP: 10.88.135.144
		- Broadcast IP: 10.88.135.159
		- First Usable IP: 10.88.135.145
		- Last Usable IP: 10.88.135.158
		- Count: 16
		- Netmask: 255.255.255.240

---


