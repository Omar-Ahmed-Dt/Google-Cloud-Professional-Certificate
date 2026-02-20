---
tags:
  - gcp
  - storage
---
# Cloud Storage
- Cloud Storage is a service for storing your objects in Google Cloud. An object is an immutable piece of data consisting of a file of any format. You store objects in containers called buckets, and you can optionally organize objects stored in buckets by using folders.
- All buckets are associated with a project, and you can group your projects under an organization. Each project, bucket, managed folder, and object in Google Cloud is a resource in Google Cloud, as are things such as Compute Engine instances.
- After you create a project, you can create Cloud Storage buckets, upload objects to your buckets, and download objects from your buckets. You can also grant permissions to make your data accessible to principals you specify or accessible to everyone on the public internet. 
- The bucket name should be unique globally.
- Most popular, very flexible & inexpensive storage service - **Serverless:** Autoscaling and infinite scale (No Configs needed) - **You don’t create servers or disks — you just upload objects.**
- Store large objects using a key-value approach - KEY   → "photos/user1/image.jpg" , VALUE → actual image data (binary bytes)
- Unlimited Object in the bucket , each bucket is associated with a project , Max object size is 5 TB, you can store unlimited number of such objects
- Treats entire object as a unit (Partial updates not allowed) - If you change anything, You upload a new version of the whole object.
    - Recommended when you operate on entire object most of the time
    - Access Control at Object level
	
 - Provides REST API to access and modify objects
    - Also provides CLI **gsutil** not gcloud cli

**Location type**
- **Region:** Lowest latency within a single region 
- **Dual-region:** High availability and low latency across 2 regions  - A dual-region is a specific pair of regions, such as Tokyo and Osaka: Dual-region pairings can be predefined or configurable.
- **Multi-region:**  Highest availability across largest area - A multi-region is a large geographic area that contains two or more geographic regions, such as the United States.

---

## Securing your data
Once you upload your objects to Cloud Storage, you have fine-grained control over how you secure and share your data. Here are some ways to secure the data you upload to Cloud Storage:
- **Identity and Access Management:** Use IAM to control who has access to the resources in your Google Cloud project. Resources include Cloud Storage buckets and objects, as well as other Google Cloud entities such as Compute Engine instances. You can grant principals certain types of access to buckets and objects, such as update, create, or delete.
- **Data encryption:** Cloud Storage uses server-side encryption to encrypt your data by default. You can also use supplemental data encryption options such as customer-managed encryption keys and customer-supplied encryption keys.
- **Authentication:** Ensure that anyone who accesses your data has proper credentials.
- **Soft delete:** Prevent permanent loss of data against accidental or malicious deletion by retaining recently deleted objects and buckets. By default, Cloud Storage enables soft delete for all buckets with a seven day retention period.
- **Object Versioning:** When a live version of an object is replaced or deleted, it can be retained as a noncurrent version if you enable Object Versioning.
- **Bucket IP filtering:** With bucket IP filtering, you can restrict access to a bucket based on the source IP address of the request and secure your data from unauthorized access from specific IP addresses or Virtual Private Cloud (VPC).
- **Bucket Lock:** Govern how long objects in buckets must be retained by specifying a retention policy.

---

## Choose How to Store Your Data
A storage class sets costs for storage, retrieval, and operations, with minimal differences in uptime.  
Choose if you want objects to be managed automatically or specify a default storage class based on how long you plan to store your data and your workload or use case.
1. **Autoclass**
	- Automatically transitions each object to Standard or Nearline class based on object-level activity to optimize for cost and latency.  
	- Recommended if usage frequency may be unpredictable.  
	- Can be changed to a default class at any time.

2. **Set a Default Class**
	- Applies to all objects in your bucket unless you manually modify the class per object or set object lifecycle rules.  
	- Best when usage is highly predictable.

| Storage Class | Best Use Case                                                         | Access Pattern    | Minimum Storage Duration |
| ------------- | --------------------------------------------------------------------- | ----------------- | ------------------------ |
| **Standard**  | Short-term storage and frequently accessed data                       | Frequent access   | None                     |
| **Nearline**  | Backups and data accessed less than once a month                      | Infrequent access | 30 days                  |
| **Coldline**  | Disaster recovery and data accessed less than once a quarter          | Rare access       | 90 days                  |
| **Archive**   | Long-term digital preservation of data accessed less than once a year | Very rare access  | 365 days                 |

**Note** You upload a file today:
```yaml
Storage class: Nearline
Minimum duration: 30 days
```
After 10 days you move it to Standard , You will pay for remaining 20 days of Nearline storage. Google charges as if object stayed 30 days.

---

## Object Versioning
- Prevents accidental deletion & provides history, Enabled at bucket level, Can be turned on/off at any time
- Live version is the latest version
    - If you delete live object, it becomes a **noncurrent object version**
    - If you delete noncurrent object version, **it is deleted**

- Older versions are uniquely identified by: **Object key + generation number**
- Reduce costs by deleting older (noncurrent) versions

### Resource names
- Each resource has a unique name that identifies it, much like a filename. Buckets have a resource name in the form of `projects/_/buckets/BUCKET_NAME`, where **BUCKET_NAME** is the ID of the bucket. Objects have a resource name in the form of `projects/_/buckets/BUCKET_NAME/objects/OBJECT_NAME`, where **OBJECT_NAME** is the ID of the object.
- A `#Generation_NUMBER` appended to the end of the resource name indicates a specific generation of the object. `#0` is a special identifier refers to the latest version of an object. `#0` is useful to add when the name of the object ends in a string that would otherwise be interpreted as a generation number.

---

## 🌩 Object Lifecycle Management
- In Google Cloud Storage, a lifecycle refers to a set of rules that automate the management of objects over their lifecycle. These rules define actions to be taken based on the age of objects or other criteria, such as transitioning objects to different storage classes (e.g., from Standard to Coldline) or automatically deleting them after a certain period. Lifecycle management helps optimize storage costs and ensures that data is managed efficiently according to its relevance and usage.

### Lifecycle Actions (What Can Happen?)
1️⃣ Delete Object
Automatically remove objects, Example: `delete logs older than 90 days`

2️⃣ Change Storage Class
Move objects to cheaper storage, Example flow: `Standard → Nearline → Coldline → Archive`

---

## Cloud Storage – Encryption
- (Default) Cloud Storage encrypts data on **the server side**
- **Server-side encryption** -  Performed by GCS (Google Cloud Storage) after it receives data
    1. **Google-managed** - Default (No configuration needed)
    2. **Customer-managed encryption keys (CMEK)** - You manage the keys using Cloud Key Management Service
        - GCS Service Account **should have access** to customer-managed keys in KMS to encrypt and decrypt files
		
    3. **Customer-supplied (CSEK)** - Customer supplies the keys with every GCS operation
        - Cloud Storage does NOT store the key
        - Customer is responsible for storing and using it when making API calls
            - Use API headers when making API calls: x-goog-encryption-algorithm, x-goog-encryption-key (Base64 encryption key), x-goog-encryption-key-sha256 (encryption key hash)
            - OR when using gsutil: configure encryption_key under GSUtil section in the .boto configuration file

- **Client-side encryption** - Encryption performed by customer before upload
    - GCP does NOT know about the keys used, GCP is NOT involved in encryption or decryption

> **Warning** 
> If you use **customer-supplied** encryption keys or **client-side** encryption, you **must securely manage your keys** and ensure that they are not lost. If you lose your keys, you are no longer able to read your data, and you continue to be charged for storage of your objects until you delete them.


### Comparing encryption options

| Encryption Method | Key Management | Use Case |
|-------------------|---------------|----------|
| **Standard (Default)** | Google manages the encryption keys. | General purpose: Cloud Storage's standard encryption is ideal for most users who need their data encrypted at rest without managing encryption keys. It satisfies many compliance requirements automatically. |
| **CMEK** | You manage the keys using Cloud Key Management Service. | Compliance and control: Use CMEK when you need to control the lifecycle of your encryption keys to meet specific compliance standards (e.g., PCI-DSS or HIPAA). You can grant, revoke, and rotate keys on your own schedule. |
| **CSEK** | You provide your own encryption keys with each request to Cloud Storage. | External key management: Best when you already have a key management system outside Google Cloud and want to use those keys. The key is not stored by Google. |
| **Client-side encryption** | You encrypt the data and manage the keys entirely yourself before uploading to Cloud Storage. | Maximum secrecy: Ensures Google has no access to unencrypted data. Provides highest control but requires full responsibility for encryption, decryption, and key management. |

---

## Understanding Cloud Storage Metadata
- Each object in Cloud Storage can have Metadata associated with it
    - Key Value Pairs  
        - Example: `storageClass: STANDARD`

- Fixed-key metadata: Fixed key - Changing value
    - `Cache-Control: public, max-age=3600`
        - (Is caching allowed? If so, for how long?)
    - `Content-Disposition: attachment; filename="myfile.pdf"`
        - (Should content be displayed inline in the browser or downloaded as an attachment?)
    - `Content-Type: application/pdf`
        - (What kind of content does the object have?)
		
- Custom metadata: You can define your own keys and values
- Non-editable metadata: You cannot edit these directly: Storage class of the object , Customer-managed encryption keys , etc.

---

## Cloud Storage Bucket Lock - Meet Compliance Needs
How do you ensure that you comply with regulatory and compliance requirements around immutable storage in a Cloud Storage bucket?

A Retention Policy is:
> -  A rule that prevents objects in a bucket from being deleted or modified for a specified period of time.
> - It enforces data immutability.

- You can set it while creating a bucket or at a later point in time
- Retention policy is applied at: Bucket level , All objects inside follow it automatically. Applies automatically to existing objects in the bucket (as well as new objects added in)
- Google prevents you from deleting the entire bucket if it still contains objects that are protected by retention policy.
	- Bucket deletion is allowed ONLY when: Object age ≥ retention period

**Retention Lock**
Retention Lock makes the policy permanent.
> Once locked, the retention policy cannot be reduced or removed.

**Unlocked Retention Policy**: You can modify duration, You can remove policy


> **Summary**
> - **Retention Policy** → defines how long objects must be preserved.
> - **Retention Lock** → permanently enforces the policy for compliance by preventing modification or removal.

---

## Transferring data from on premises to cloud
- Most migrations move data into: 👉 Google Cloud Storage (GCS)
	- because it is: cheap, scalable, durable, entry point for analytics & AI

### 🧭 There are THREE main transfer approaches
```text
Online Transfer        → Internet upload
Managed Transfer       → Automated large transfers
Physical Transfer      → Ship hardware
```

1️⃣ **Online Transfer (gsutil / API):** You upload data over the internet using gsutil or API
**When to use**
- Small transfers
- One-time uploads
- Manual migration

==Recommended when: Data < 1 TB==

2️⃣ **Storage Transfer Service (STS):** 👉 Managed large-scale online transfer, Google runs the transfer job for you - You can set up a repeating schedule - Supports incremental transfer (only transfer changed objects)
Transfers from:
- On-premises
- AWS S3
- Azure
- Other GCS buckets

**Key Features**
- Scheduled transfers
- Incremental sync (copies only changes)
- Fault tolerant (resume on failure)
- Automated

==Recommended When: On-prem NAS → Cloud Storage==

3️⃣ **Transfer Appliance (Physical Migration):** Google sends you a physical storage device.
You:
- Copy data locally (fast LAN speed)
- Ship device back to Google
- Google uploads data into Cloud Storage

==Recommended when: Data > 20 TB or Online transfer takes > 1 week==

---

## Understanding Cloud Storage Best Practices
- Avoid use of sensitive info in bucket or object names
- Store data in the closest region (to your users)
- Ramp up request rate gradually
    - No problems up to 1000 write requests per second or 5000 read requests per second
    - Beyond that, take at least 20 minutes to double request rates - Do not suddenly send a huge number of requests to Cloud Storage. Increase traffic slowly over time so Google can scale internally - When traffic increases, `More requests → Google redistributes load → allocates resources`, This process takes time. If traffic spikes suddenly, throttling happens and you receive errors (429 / 5xx). so Wait ~20 minutes before doubling traffic again: `Minute 0  → 1k req/sec , Minute 20 → 2k req/sec, Minute 40 → 4k req/sec, Minute 60 → 8k req/sec`

- Use **Exponential backoff** if you receive 5xx (server error) or 429 (too many requests) errors
    - Retry after 1, 2, 4, 8, 16, ... seconds - When a request fails, wait before retrying, and increase the waiting time after each failure.

- Do NOT use sequential numbers or timestamp as object keys (object names)
	- **Sequential Names (Hotspot Problem):** Suppose you upload files like this: `log-0001.txt , log-0002.txt , log-0003.txt` or timestamps: `2026-02-20-10-00.log , 2026-02-20-10-01.log` All objects start with similar prefixes, Cloud Storage thinks: `Same prefix → same storage partition` , so the result: one backend server receives ALL traffic. This is called: Hotspotting.
    - Recommended to use completely random object names
    - Recommended to add a hash value before the sequence number or timestamp

- Use **Cloud Storage FUSE** to enable file system access to Cloud Storage
    - Mount Cloud Storage buckets as file systems on Linux or macOS systems

---

## Cloud Storage - Scenarios

| Scenario | Solution |
|----------|----------|
| I will frequently access objects in a bucket for 30 days. After that I don't expect to access objects at all. We have compliance requirements to store objects for 4 years. How can I minimize my costs? | **Initial Storage Class:** Standard<br>**Lifecycle policy:** Move to Archive class after 30 days.<br>Delete after 4 years. |
| I want to transfer 2 TB of data from Azure Storage to Cloud Storage | Use Cloud Storage Transfer Service |
| I want to transfer 40 TB of data from on premises to Cloud Storage | Use Transfer Appliance |
| Customer wants to manage their Keys | **Customer-managed** – Keys managed by customer in Cloud KMS |
| Regulatory compliance: Object should not modified for 2 years | Configure and lock data retention policy |

---
