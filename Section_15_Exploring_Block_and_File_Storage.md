---
tags:
  - gcp
  - storage
---
# GCP Storage
Google Cloud offers three main types of data storage: **block, file, and object storage.**
1. **Block storage** = a hard disk attached to one computer
	- Two popular types of **block storage** can be attached to VM instances:
		- **Local SSDs**
		- **Persistent Disks**
		
2. **File storage** = A shared filesystem accessible by multiple machines, Like a company shared network folder.
3. **Stores files** = as objects inside buckets, Like Google Drive or Amazon S3.

---

## Persistent Disks
- A Persistent Disk is Google Cloud’s durable, block-level storage that you can attach to virtual machines (VMs) or use with container nodes in Google Kubernetes Engine (GKE).
- It is the standard disk storage used for operating systems, applications, and data on Compute Engine VMs.
- Network block storage attached to your VM instance - **Network storage**, Your VM accesses it over Google’s internal high-speed network.
- Lifecycle **NOT tied** to VM instance - Independent lifecycle from VM instance , Attach/Detach from one VM instance to another
- Typically, ONE Block Storage device can be connected to ONE virtual server
- **(EXCEPTIONS)** You can attach **read only block devices** with multiple virtual servers and certain cloud providers are exploring multi-writer disks as well!
- HOWEVER, you can connect multiple different block storage devices to one virtual server
- Deletion rule ( Vm instance => Boot disk configs )
	- When deleting instance:  Keep boot disk || Delete boot disk 

### Persistent Disks - Standard vs Balanced vs SSD

| Disk Type                                      | Description                                                                                                                                                         | Performance Level | Storage Technology | Best Use Cases                                              | Special Notes                                          |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ------------------ | ----------------------------------------------------------- | ------------------------------------------------------ |
| **Balanced Persistent Disk (pd-balanced)**     | **default option** - Persistent Disk with balance between performance and cost. Provides same maximum IOPS as SSD PD for most machine types but lower IOPS per GiB. | Medium–High       | SSD                | General-purpose applications, web servers, backend services | Price between Standard and SSD PD                      |
| **SSD (Performance) Persistent Disk (pd-ssd)** | Designed for enterprise applications requiring lower latency and higher IOPS than standard disks.                                                                   | High              | SSD                | High-performance databases, enterprise applications         | Optimized for low latency workloads                    |
| **Standard Persistent Disk (pd-standard)**     | Suitable for workloads mainly performing sequential read/write operations.                                                                                          | Low–Medium        | HDD                | Large data processing, backups, batch jobs                  | Lowest cost option                                     |
| **Extreme Persistent Disk (pd-extreme)**       | Provides consistently high performance for random access and high throughput workloads with configurable IOPS.                                                      | Very High         | SSD                | High-end databases, mission-critical enterprise systems     | Custom IOPS provisioning; limited machine type support |

---

## Local SSDs
- physically attached to the host of the VM instance
- Ephemeral Storage - Temporary data, Lifecycle tied to VM instance , Data persists only while instance is running
- Provide very high IOPS and very low latency
- Enable live migration for maintenance events
- Data automatically encrypted: ==Encryption keys cannot be configured==
- Only some machine types support Local SSDs
- Supports SCSI and NVMe interfaces - `When a VM talks to a disk, it uses a disk interface (driver/protocol). NVMe = best performance, Multi-queue SCSI = good fallback (Better parallelism → better throughput)`

**Remember:**
- Choose NVMe-enabled and multi-queue SCSI images for best performance
- Larger Local SSDs and more vCPUs attached to VM ⇒ better performance

**Advantages**
- Very fast I/O (10–100x compared to Persistent Disks): Higher throughput and lower latency
- Ideal for high IOPS temporary workloads: Examples: caches, temporary data, scratch files

**Disadvantages**: Ephemeral storage: Lower durability, Lower availability, Lower flexibility compared to Persistent Disks

---

## Persistent Disks vs Local SSDs

| Feature                   | Persistent Disks          | Local SSDs              |
| ------------------------- | ------------------------- | ----------------------- |
| Attachment to VM instance | As a network drive        | Physically attached     |
| Lifecycle                 | Separate from VM instance | Tied with VM instance   |
| I/O Speed                 | Lower (network latency)   | 10–100x faster than PDs |
| Snapshots                 | Supported                 | Not supported           |
| Use case                  | Permanent storage         | Ephemeral storage       |

---

## Persistent Disks – Snapshots
- Take **point-in-time snapshots** of your Persistent Disks
- You can also schedule snapshots (configure a schedule): You can also auto-delete snapshots after X days
- Snapshots can be **Multi-regional** and **Regional**
- You can share snapshots across projects
- You can create new disks and instances from snapshots
- Snapshots are **incremental** - only changes are stored after the first snapshot, saving time and cost
- Deleting a snapshot only deletes data which is **NOT** needed by other snapshots
- Keep similar data together on a Persistent Disk:
    - Separate your operating system, volatile data, and permanent data
    - Attach multiple disks if needed
    - This helps to better organize your snapshots and images

**Snapshots allow:**
- Backing up disk data
- Restoring disks to a previous state
- Creating new disks from existing data

### Persistent Disks — Snapshots — Recommendations
- Avoid taking snapshots more often than once an hour.
- Disk volume is available for use **but snapshots reduce performance** , **(RECOMMENDED)** Schedule snapshots during off-peak hours.
- Can create a Vm instance or disk from the snapshot
- Creating snapshots from disks is faster than creating from images:
    - Creating disks from an image is faster than creating from snapshots.
    - **(RECOMMENDED)** If you repeatedly create disks from a snapshot:
        - Create an **image** from the snapshot.
        - Use the image to create disks.

- Snapshots are **incremental**:
    - You do not lose data by deleting older snapshots.
    - Deleting a snapshot only removes data **not required by other snapshots**.
    - **(RECOMMENDED)** Do not hesitate to delete unnecessary snapshots.

```text
# You do this repeatedly: 
Snapshot → create disk
Snapshot → create disk
Snapshot → create disk

# Each time Google reconstructs disk data again.
# Recommended Solution: Create image once, Use image many times:
Snapshot → Image

Image → Disk
Image → Disk
Image → Disk
```

---

## Mounting a Data Persistent Disk on a GCE VM
Steps to attach a newly created Persistent Disk with an already running VM:
1. Attach Disk to running or stopped VM
```bash
gcloud compute instances attach-disk INSTANCE_NAME --disk DISK_NAME
```

2. Format the disk
```bash
sudo lsblk

# Format to file system of your choice (example: ext4)
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
```

3. Mount the disk
```bash
# Create directory to mount disk
sudo mkdir -p /mnt/disks/MY_DIR

# Mount the disk
sudo mount -o discard,defaults /dev/sdb /mnt/disks/MY_DIR

# Provide permissions
sudo chmod a+w /mnt/disks/MY_DIR
```

```bash
~ > gcloud config set project gcp-professional-2026
Updated property [core/project].

~ > gcloud compute disks create testdisk \                                                              1
    --size=20GB \
    --type=pd-balanced \
    --zone=us-central1-a
Created [https://www.googleapis.com/compute/v1/projects/gcp-professional-2026/zones/us-central1-a/disks/testdisk].
NAME      ZONE           SIZE_GB  TYPE         STATUS
testdisk  us-central1-a  20       pd-balanced  READY

New disks are unformatted. You must format and mount a disk before it
can be used. You can find instructions on how to do this at:


~ > gcloud compute instances attach-disk instance-20260219-181134  --disk testdisk
No zone specified. Using zone [us-central1-a] for instance: [instance-20260219-181134].
Updated [https://www.googleapis.com/compute/v1/projects/gcp-professional-2026/zones/us-central1-a/instances/instance-20260219-181134].
~ >
                                                                                               
~ > gcloud compute ssh instance-20260219-181134 --zone=us-central1-a                                


omar@instance-20260219-181134:~$
omar@instance-20260219-181134:~$ sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

omar@instance-20260219-181134:~$ sudo lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda       8:0    0   10G  0 disk
├─sda1    8:1    0  9.9G  0 part /
├─sda14   8:14   0    3M  0 part
└─sda15   8:15   0  124M  0 part /boot/efi
sdb       8:16   0   20G  0 disk

omar@instance-20260219-181134:~$ sudo mkdir -p /mnt/data
omar@instance-20260219-181134:~$ sudo mount -o discard,defaults /dev/sdb /mnt/data
omar@instance-20260219-181134:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.0G     0  2.0G   0% /dev
tmpfs           393M  536K  392M   1% /run
/dev/sda1       9.7G  2.7G  6.5G  30% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda15      124M   12M  113M  10% /boot/efi
tmpfs           393M     0  393M   0% /run/user/1000
/dev/sdb         20G   24K   20G   1% /mnt/data
omar@instance-20260219-181134:~$ sudo chmod a+w /mnt/data
omar@instance-20260219-181134:~$ sudo blkid /dev/sdb
/dev/sdb: UUID="4b0550bc-aab5-4dd3-9b80-d05a4f6dafb5" BLOCK_SIZE="4096" TYPE="ext4"
omar@instance-20260219-181134:~$
```

---

## Resizing Data Persistent Disks
1. Resize the Disk
```bash
gcloud compute disks resize DISK_NAME --size DISK_SIZE

# Ex: 
gcloud compute disks resize testdisk \
    --size=50GB \
    --zone=us-central1-a
```
2. Take a Snapshot: Recommended backup step in case something goes wrong
3. Resize the File System and Partitions
```bash
# For ext4 filesystem
sudo resize2fs /dev/sdb

# For xfs filesystem
sudo xfs_growfs /dev/sdb
```

---

## Image (Disk Image) vs Machine Image
- Machine Image is different from Image

### Image ( Disk Image ) 
- An Image is created from a boot Persistent Disk only.
- It is basically a template of an operating system disk.
- **Boot Disk → Image → New VM** 
- Create an image → now you can create many VMs with same **OS setup.**

**What an Image contains**
- ✅ OS
- ✅ Installed software
- ✅ System configuration (inside disk)
- ❌ VM settings
- ❌ network config
- ❌ metadata
- ❌ extra disks

### Machine Image
- A machine image is a Compute Engine resource that stores **all the configuration (metadata, permissions, and data from multiple disks if there are multi disks that are attached to the VM instance)** of a virtual machine (VM) instance - A Machine Image is created from the entire VM instance.
- It captures EVERYTHING needed to recreate the VM.
- Complete VM snapshot

**Machine Image contains**
- ✅ Configuration (machine type, CPU, memory)
- ✅ Metadata
- ✅ Permissions
- ✅ Boot disk data
- ✅ Additional disks data
- ✅ Network settings

Lab: [Playing with Machine Images](https://scribehow.com/viewer/Playing_with_Machine_Images__LuPBTrC2SMifZyAexG2BZA)

---

## When to use a machine image
Machine images are ideal when you need:
- ✅ Full VM Backup
	- Instance configuration
	- Metadata
	- ALL disks

- ✅ Disaster Recovery
	- Restore entire VM quickly

- ✅ Multi-disk consistency
	- Database + app disks together

- ✅ VM cloning
	- Create identical environments

```text
Snapshot          → Backup ONE disk
Custom Image      → Reusable OS template
Instance Template → Scaling blueprint
Machine Image     → FULL VM backup + clone
```

| Resource | What it captures |
|---|---|
| Machine Image | Full VM snapshot (instance + disks + config) |
| Disk Snapshot | Backup of ONE disk only |
| Custom Image | Reusable OS + software template |
| Instance Template | VM configuration blueprint for scaling |

### 📊 Now Let’s Explain Each Row (Scenario)
#### ✅ 1️⃣ Single Disk Backup

| Resource | Why |
|---|---|
| Machine Image | Includes disks → works |
| Disk Snapshot | Designed exactly for this |
| Custom Image | Can capture boot disk |
| Instance Template | ❌ No disk backup capability |

**Meaning**
- If you only want to backup one disk, snapshots are usually best.

**Example:**  
- Backup database disk daily.

#### ✅ 2️⃣ Multiple Disk Backup

| Resource | Why |
|---|---|
| Machine Image | ✅ Captures ALL attached disks together |
| Disk Snapshot | ❌ Only one disk at a time |
| Custom Image | ❌ Only boot disk |
| Instance Template | ❌ No backups |

**Meaning**
- Machine Image = full VM backup.

**Example:**

```
VM
├── boot disk
├── database disk
└── uploads disk
```

- Machine image saves ALL together consistently.
- 👉 Best for disaster recovery.

#### ✅ 3️⃣ Differential Backup
- Differential = only changed data is stored

| Resource | Why |
|---|---|
| Machine Image | Incremental internally |
| Disk Snapshot | Incremental snapshots |
| Custom Image | Full copy every time |
| Instance Template | Not backup |

**Meaning**
- Snapshots & machine images save storage because:
- Only changed blocks are stored.
- This reduces cost.

#### ✅ 4️⃣ Instance Cloning
- Create identical VM

| Resource | Why |
|---|---|
| Machine Image | Full clone including disks & config |
| Disk Snapshot | Missing VM config |
| Custom Image | Can recreate VM from OS image |
| Instance Template | Designed for cloning/scaling |

**Meaning**
- You want another identical server.

**Example:**
- Clone production VM into staging.
- Machine Image copies:
	- disks
	- metadata
	- configs
	- machine type
	- networking

#### ✅ 5️⃣ Base Image for Replication

| Resource | Why |
|---|---|
| Machine Image | ❌ Not reusable base OS |
| Disk Snapshot | ❌ Backup only |
| Custom Image | ✅ Designed for reuse |
| Instance Template | ❌ Needs image source |

**Meaning**
- You want a golden OS image.

**Example:**
- Ubuntu + Node.js + security patches
- Used to create many VMs
- 👉 Use Custom Image.

#### Summary: The following table compares the use of machine images, standard disk snapshots, instance templates, and custom images.

| Scenarios                 | Machine image | Standard disk snapshot | Custom image | Instance template |
|---------------------------|---------------|------------------------|--------------|-------------------|
| Single disk backup        | Yes           | Yes                    | Yes          | No                |
| Multiple disk backup      | Yes           | No                     | No           | No                |
| Differential backup       | Yes           | Yes                    | No           | No                |
| Instance cloning          | Yes           | No                     | Yes          | Yes               |
| Base image for replication| No            | No                     | Yes          | No                |

---

## Storage – Scenarios – Persistent Disks

| Scenario | Solution |
|----------|----------|
| You want to improve performance of Persistent Disks (PD) | Increase size of PD or add more PDs. Increase vCPUs in your VM. |
| You want to increase durability of Persistent Disks (PD) | Go for Regional PDs (2× cost but replicated in 2 zones). |
| You want to take hourly backup of Persistent Disks (PD) for disaster recovery | Schedule hourly snapshots. |
| You want to delete old snapshots created by scheduled snapshots | Configure it as part of your snapshot scheduling. |

---

## Filestore
- Fully managed Network File System (NFS) file storage service on Google Cloud.
- It provides shared file storage that multiple machines can mount and access simultaneously - Filestore supports multiple concurrent application instances accessing the same file system simultaneously.
- File-based storage (not block, not object)
- Shared read/write filesystem
- Filestore instances act as managed NFS file servers accessible from: Compute Engine VMs , Google Kubernetes Engine (GKE) , VMware Engine, On-prem systems (via networking)

**Why Filestore Exists (Problem It Solves)**
Some applications require:
- Shared filesystem
- POSIX file access
- Multiple clients writing same files
- Legacy applications expecting NFS

Persistent Disk ❌ cannot be shared easily. Filestore ✅ solves shared storage.

```text
Filestore Instance (Managed NFS Server)
        │
        ├── VM 1 mounts share
        ├── VM 2 mounts share
        ├── GKE Pods mount share
        └── Multiple clients read/write same files
```

### Service tiers
**Why Filestore Has Tiers**: Not all workloads need the same:
- Performance (IOPS / throughput)
- Availability
- Replication
- Cost

So Google created tiers optimized for different scenarios - Cheap → Fast → Highly Available → Enterprise-grade

Filestore offers multiple service tiers that vary in capacity, performance, and features. Each service tier is tailored for specific use cases:
- Basic tier: Entry-level Filestore designed for general-purpose file sharing.
- Zonal tier: High-performance storage within **one zone** - Designed for heavy compute workloads needing high throughput.
- Regional tier: High-availability Filestore - Data is replicated across **multiple zones in a region.**
- Enterprise tier: Highest level Filestore offering for enterprise-grade workloads - Extreme reliability, Advanced performance, Large-scale enterprise systems

---
## Storage - Scenarios

| Scenario | Solution |
|----------|----------|
| You want very high IOPS but your data can be lost without a problem | Local SSDs |
| You want to create a high performance file sharing system in GCP which can be attached with multiple VMs | Filestore |
| You want to backup your VM configuration along with all its attached Persistent Disks | Create a Machine Image |
| You want to make it easy to launch VMs with hardened OS and customized software | Create a Custom Image |

---
