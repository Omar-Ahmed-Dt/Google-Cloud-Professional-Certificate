---
tags:
  - gcp
---
## Cloud VPN
- Cloud VPN allows you to securely connect your on-premises network (or another cloud network) to your Google Cloud VPC using encrypted tunnels over the public internet.
- Cloud VPN creates an IPsec VPN tunnel between: `On-Prem Network  ⇄  Encrypted Tunnel (IPsec)  ⇄  GCP VPC Network`

### Types of Cloud VPN
1. **Classic VPN** (Legacy)
	- Older version
	- Supports static routing only
	- Lower availability
	- Not recommended for new setups
	- Needs a Google Compute Engine VPN gateway - Not managed service

		```text
		+------------------------+                     +------------------------+
		|   On-Premise Network   |                     |       VPC Network      |
		|                        |                     |                        |
		|   [On-Prem Router]     |==== Tunnel 1 ======>|  [VPN Gateway]         |
		|                        |                     |        |               |
		|                        |                     |   [VPC Network]        |
		+------------------------+                     +------------------------+
		```

2. **HA VPN** (Recommended ✅)
	High Availability VPN provides:
	- 99.99% SLA
	- Automatic failover - When you create an HA VPN gateway, Google Cloud automatically chooses two external IP addresses, one for each of its interfaces
	-  Supports dynamic routing (BGP) only 
	- Redundant tunnels

		```text
		+-----------------------+                         +-----------------------+
		|   On-Premise Network  |                         |      VPC Network      |
		|                       |                         |                       |
		|     [On-Prem Router]  |==== Tunnel 1 ==========>| Gateway Interface 0  |
		|           ||          |                         |         (VPN)         |
		|           ||          |                         |                       |
		|           ||          |==== Tunnel 2 ==========>| Gateway Interface 1  |
		|                       |                         |         (VPN)         |
		+-----------------------+                         +-----------------------+
		```

---

## Cloud Interconnect
- Cloud Interconnect provides a private, high-bandwidth, low-latency connection between your on-premises data center and Google Cloud.
- Unlike Cloud VPN (which uses the public internet), Interconnect uses a dedicated private connection.
- Data exchange happens through a private network: 
	- Your on-prem servers can talk directly to: `10.x.x.x` , `172.16.x.x` , `192.168.x.x`
	- Reduces egress costs: As public internet is NOT used

### Types of Cloud Interconnect
1️⃣ **Dedicated Interconnect**
- Direct physical fiber connection to Google
- Minimum private connection speed of 10Gbps - (Options: 10 Gbps or 100 Gbps per connection)
- High throughput
- Best for:
	- Large enterprises
	- Massive data transfer
	- Very low latency needs

2️⃣ **Partner Interconnect**
- Uses a Google-approved service provider
- Lower bandwidth options (50 Mbps → 50 Gbps)
- Easier setup
- Best for:
	- Mid-size companies
	- Flexible bandwidth needs
	- Faster deployment

---
