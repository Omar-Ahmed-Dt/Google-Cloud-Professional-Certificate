---
tags:
  - gcp
---
## Cloud Key Management Service (KMS) Overview
- KMS centralizes creation, storage, rotation, and access control of encryption keys used to protect sensitive data across your GCP infrastructure and applications.

---

## Data States in Cloud Computing
There are 3 states of data:
1. Data at Rest
	- Data that is stored on a physical or logical storage system: ==Data on a hard disk==
	
2. Data in Motion (Data in Transit)
	- Data that is being transferred across a network: ==Copying data from on-prem to cloud || Uploading file to Cloud Storage==
	
3. Data in Use
	- Data that is actively being processed in memory (RAM) : ==Data loaded into RAM==

---

## KMS
- Create and manage **cryptographic keys** (symmetric and asymmetric)
- Control their use in your applications and GCP services
- Provides an API to encrypt, decrypt, or sign data
- Use existing cryptographic keys created on premises
- Integrates with almost all GCP services that need data encryption:
	- **Google-managed key**: No configuration required  
		- Google Creates the encryption key, Stores it, Rotates it, Protects it, Uses it automatically, You don’t see or manage the key.
		
	- **Customer-managed key (CMEK)**: Use key from Cloud KMS  
		- You Create the key inside Cloud KMS , Control IAM permissions , Control rotation schedule , Can disable or destroy the key , But Google still stores the key in its infrastructure
	
	- **Customer-supplied key (CSEK)**: Provide your own key
		- You Generate the key yourself , Keep it outside Google , Google does NOT store it , Google uses it temporarily, After operation Google forgets the key.

---

## Use Cases
- Data Encryption
	- Encrypt data at rest in Cloud Storage, BigQuery, Compute Engine disks

- Secrets Management
	- Store API keys, passwords, certificates, tokens
	- Often used alongside Secret Manager (which can use KMS for additional encryption)

---
