---
tags:
  - gcp
---
## Planning for High Availability

| Service/Feature           | Features / Best Practices                                                                                                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Geographical Distribution | Availability: Global > Multi-Regional > Regional > Zonal                                                                                                                                    |
| Compute Engine            | - Live Migration <br>- Managed Instance Groups with AutoScaling and Health Checks (Auto healing) <br>- Distribute instances across regions/zones and distribute using Global Load Balancing |
| GKE                       | - Multi master, Regional clusters with pod and cluster autoscaling                                                                                                                          |
| Managed Services          | - Use Managed Services like App Engine, Cloud Functions, Cloud Storage, Cloud Filestore, Cloud Datastore, BigQuery                                                                          |
| Persistent Disks          | - Live resizing improves availability <br>- Use Regional Persistent Disks                                                                                                                   |
| Cloud Bigtable            | - Place clusters in different zones or regions (Each cluster belongs to a zone and you can create multiple clusters for high availability in the same instance)                             |

| Service/Feature             | Features / Best Practices                                                                                                                                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cloud Datastore             | Use Multi-region locations                                                                                                                                                                                                       |
| Cloud SQL                   | - Use HA configuration (regional) — Cluster with primary instance and a standby instance is created <br> - Read replicas will NOT be promoted (So Read Replicas do NOT increase availability for Cloud SQL)                      |
| Network Tier                | Prefer Premium Network Tier to Standard Network Tier                                                                                                                                                                             |
| Hybrid Network Connectivity | - Speed and Availability: Dedicated interconnect > Partner interconnect > VPN <br> - Have a backup connection <br> - Example: VPN can be a backup for Dedicated interconnect or have multiple Dedicated interconnect connections |

---

## Implementing DDoS Protection and Mitigation
- (DDoS) attack: Attempting to bring your apps down with large scale attacks
- Shared responsibility between GCP and Customer
	- GCP provides certain features to protect you from DDoS attacks:
		- Anti-spoofing protection for the private network
		- Isolation between virtual networks
		- App Engine (sits behind Google Front End) automatically protects you from Layer 4 and below attacks - Examples: SYN floods, IP fragment floods, port exhaustion, etc.

	- What you can do to protect apps from DDoS attacks?
		- Reduce the attack surface: Make use of subnetworks and networks, firewall rules and IAM
		- Isolate your internal traffic
			- Use Private IP Address (unless you need Public IP Addresses)
			- Use private load balancing for internal traffic

		- Use Proxy-based Load Balancing: Automatically protects you from Layer 4 and below attacks
		- Integrate Load Balancing with Cloud Armor
			- IP-based and geo-based access control
			- Enforce Layer 7 security policies on hybrid and multi-cloud deployments
			- Pre-configured WAF rules (OWASP Top 10)

---
