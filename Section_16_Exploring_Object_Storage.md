---
tags:
  - gcp
  - storage
---
## Cloud Storage Overview
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
	
-  Pay for Storage , data retrieval , operations and network egress traffic (download a object)

**Location type**
- **Region:** A single region - Lowest Availability
- **Dual-region:** 2 Regions - Higher Availability
- **Multi-region:**  Multi Regions - Global - Highest availability 

---

## Securing your data
Once you upload your objects to Cloud Storage, you have fine-grained control over how you secure and share your data. Here are some ways to secure the data you upload to Cloud Storage:
- **Identity and Access Management - IAM** 
- **Data encryption** 
- **Authentication:** Ensure that anyone who accesses your data has proper credentials.
- **Soft delete:** Prevent permanent loss of data against accidental or malicious deletion by retaining recently deleted objects and buckets. By default, Cloud Storage enables soft delete for all buckets with a seven day retention period.
- **Object Versioning:** When a live version of an object is replaced or deleted, it can be retained as a noncurrent version if you enable Object Versioning.
- **Bucket IP filtering:** With bucket IP filtering, you can restrict access to a bucket based on the source IP address of the request and secure your data from unauthorized access from specific IP addresses or Virtual Private Cloud (VPC).
- **Bucket Lock:** Govern how long objects in buckets must be retained by specifying a retention policy.
- **Prevent Public Access (Default)** A bucket is public only if disabled and IAM permission - **Storage Object Viewer** to **allUsers** exists.

---

## Choose How to Store Your Data
Based on data types: **active data, analytics data, or archived data**, we will choose the appropriate class.

| Storage Class | Best Use Case                                                         | Access Pattern    | Minimum Storage Duration |
| ------------- | --------------------------------------------------------------------- | ----------------- | ------------------------ |
| **Standard**  | Short-term storage and frequently accessed data                       | Frequent access   | None                     |
| **Nearline**  | Backups and data accessed less than once a month                      | Infrequent access | 30 days                  |
| **Coldline**  | Disaster recovery and data accessed less than once a quarter          | Rare access       | 90 days                  |
| **Archive**   | Long-term digital preservation of data accessed less than once a year | Very rare access  | 365 days                 |

**Minimum Storage Duration** You upload a file today:
```yaml
Storage class: Nearline
Minimum duration: 30 days
```
After 10 days you move it to Standard , You will pay for remaining 20 days (Pay for 30 days - minimum storage duration for the calss) of Nearline storage. Google charges as if object stayed 30 days.

### Google Cloud Storage classes
> The less frequently you access data, the cheaper storage becomes - but retrieval cost increases.

| Storage Class | Access Frequency | Storage Cost (Approx) | Retrieval Cost (Approx) | Typical Use Cases |
|---|---|---|---|---|
| Standard | Frequently accessed (Hot data) | ~$25 / TB / month | ~$0 / TB | Active applications, websites, streaming, analytics |
| Nearline | About once per month | ~$10 / TB / month | ~$10 / TB | Backups, disaster recovery, long-lived data |
| Coldline | About once per quarter | ~$7 / TB / month | ~$20 / TB | Compliance storage, long-term backups, audit data |
| Archive | About once per year or less | ~$1 / TB / month | ~$50 / TB | Legal archives, historical records, long-term retention |

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

## Object Lifecycle Management
1. **Lifecycle Management Config**
	- Set of rules You define in **the bucket level.**
	- You manage transitions - **respect minimum storage durations**
	- Ideal for predictable data lifecycles

2. **Autoclass**
	- Automatically transitions each object to Standard or Nearline class based on object-level activity to optimize for cost and latency.  
	- Recommended if usage may be unpredictable.  
	- Can be changed to a default class at any time.
	- Set in **the bucket level.**
	
### Lifecycle Actions
1️⃣ **Delete Object**
Automatically remove objects, Example: `delete logs older than 90 days`

2️⃣ **Change Storage Class**
Move objects to cheaper storage, Example flow: `Standard → Nearline → Coldline → Archive`

---

## Cloud Storage – Encryption
- Encryption **in Transit**: Protects data moving across networks between clientand server, or between services. (TLS or VPN tunnels). ==it's not related to this topic==
- Encryption **at Rest**: Protects data stored on disks or databases, etc.. (Encrypting files in a cloud Storage)
	- **Client** Side Encryption and **Server** Side Encryption
	
- (Default) Cloud Storage encrypts data on **the server side**
1. **Server-side encryption (SSE)** -  Performed by GCS (Google Cloud Storage) after it receives data
    1. **Google-managed** - Default (No configuration needed)
    2. **Customer-managed encryption keys (CMEK)** - You manage the keys using Cloud Key Management Service
        - GCS Service Account **should have access** to customer-managed keys in KMS to encrypt and decrypt files
        - Customer have Full Control over the Keys, Rotation and audit logging and Role Separation as big advantage (so If a user has Cloud Storage Admin permissions, they can perform operations on the bucket but cannot read its contents because they do not have permission to use my KMS key).
        - You pay to use KMS
		
    3. **Customer-supplied (CSEK)** - Customer supplies the keys with every GCS operation
        - Cloud Storage does NOT store the key - Google discard the key after encryption operation.
        - Customer is responsible for storing and using it when making API calls:
            - Use API headers when making API calls: x-goog-encryption-algorithm, x-goog-encryption-key (Base64 encryption key), x-goog-encryption-key-sha256 (encryption key hash)

2. **Client-side encryption** - Encryption performed by customer before upload
    - GCP does NOT know about the keys used, GCP is NOT involved in encryption or decryption
	- The customer encrypts the data and then sends it to Cloud Storage, Cloud Storage encrypts the data again using **SSE** mode. 

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
### Retention Policy
- A Retention Policy defines how long objects must be kept in a bucket before they can be deleted or overwritten.
- When you set a retention policy:
	- Every object in the bucket gets a retention timer
	- Timer starts when the object is created
	- Until expiration: Cannot delete object , Cannot overwrite object , Cannot replace object version
	
- You can set it while creating a bucket or at a later 
- Bucket level , All objects inside follow it automatically. Applies automatically to existing objects in the bucket and new objects added.
- Google prevents you from deleting the entire bucket if it still contains objects that are protected by retention policy.
	- Bucket deletion is allowed ONLY when: **Object age ≥ retention period**
	
### Bucket Lock (Retention Policy Lock)
- Bucket Lock makes the retention policy PERMANENT and IMMUTABLE.
- Nobody - not even admins - can reduce or remove retention. This is required for regulatory compliance.
- Once Locked
- You CANNOT:
	- Remove retention policy
	- Reduce retention duration
	- Disable retention

> **Summary**
> - **Retention Policy** → defines how long objects must be preserved.
> - **Bucket Lock** → permanently enforces the policy for compliance by preventing modification or removal.

### Object Retention Lock
- retention at the **OBJECT level**, not only the bucket level.
- Each object can have:
	- **Retain Until** (date)
	- **Retention Mode**
		- **Unlocked (Governance mode)**: allows authorized users to modify or remove an object's retention.
		- **Locked (Compliance mode)**: permanently prevents the retention date from being reduced or removed.


---

## Transferring data from on premises to cloud
- Most migrations move data into Google Cloud Storage (GCS) because it is cheap, scalable, durable, entry point for analytics & AI

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

## ACL (Access Control Lists)
- **ACL**: Define **who** has access to your buckets and objects, as well as **what level of access** they have.

**How is this different from IAM?**
- IAM permissions apply to **all objects within a bucket**.
- ACLs can be used to **customize specific access** to different objects.
- A user gets access if they are allowed by **either IAM or ACL**!

**✅ Remember**
- Use **IAM** for **common permissions** to all objects in a bucket.
- Use **ACLs** if you need to **customize access to individual objects**.

### Access Control – Overview
- How do you control access to objects in a Cloud Storage bucket?
- Two types of access controls:
    - **Uniform (Recommended)**, Uniform bucket-level access using **IAM**
    - **Fine-grained**, Use **IAM and ACLs** to control access **Both bucket-level and individual object-level permissions**

- Use **Uniform access** when all users have the same level of access across all objects in a bucket.
- **Fine-grained access** with ACLs can be used when you need to customize access at an object level.

---

## Cloud Storage – Signed URL
- You would want to **allow a user limited time access** to **a specific object** in the bucket, Users do **NOT** need Google accounts
- **To create a Signed URL:**
    1. Create a key (`YOUR_KEY`) for the Service Account/User with the desired permissions
    2. Create Signed URL with the key:
	
```bash
gsutil signurl -d 10m YOUR_KEY gs://BUCKET_NAME/OBJECT_PATH
```

**Example**
```bash
# Create a SA 
$ gcloud iam service-accounts create signed-url-test-sa

# Give <storage object viewer> permissin to SA
$ gcloud projects add-iam-policy-binding gcp-professional-2026 \
    --member="serviceAccount:<SA Name>@<Project Name>.iam.gserviceaccount.com" \
    --role="roles/storage.objectViewer"

# Create a key	
$ gcloud iam service-accounts keys create key.json \
    --iam-account=<SA Name>@<Project Name>.iam.gserviceaccount.com
	
$ ls
key.json  README-cloudshell.txt

# Create signed URL
$ gsutil signurl -d 10m key.json \
gs://enkidutestingbucket/17cd40f2545ac328470a039abe848872.jpg

## the URL will show here ...
```

---

## Cloud Storage – Static Website
1. **Create a bucket with the same name as the website name**
	- **The bucket name should match the DNS name of the website**
	- Verify that the domain is owned by you
	
2. **Copy the files to the bucket**: Add `index.html` and `error.html` files for a better user experience
3. **Add member `allUsers` and grant `Storage Object Viewer` role**: This allows public read access to website files

---
