---
tags:
  - gcp
  - compute_engine
---
## SSH into Linux VMs - Overview
- Google Compute Engine Linux VMs use Key-based SSH authentication (NOT passwords)
- Meaning:
	- You connect using SSH key pairs
	- Private key → on your machine
	- Public key → stored in GCP metadata or managed via OS Login

### Two SSH Access Options
1️⃣ **Metadata-Managed SSH Keys (Traditional Method)**
How it works: You manually manage SSH keys.
Steps:
- Generate SSH key locally
- Upload public key to VM metadata
- Connect using private key

Where keys are stored
- Instance metadata OR
- Project metadata

```text
# instance level 
Create VM => Serurity => Manage Access => Add manually generated SSH keys => Add Item => Add you Public key

or 
# project level
Compute Engine => Settings => metadata => SSH keys

then
$ ssh -i ~/.ssh/<private key> username@vm_external_ip
```

2️⃣ **OS Login (Recommended ✅)**
OS Login manages SSH access using Google IAM instead of SSH keys.

```text
	Google Identity (IAM user)
        ↓
	OS Login
        ↓
	Temporary SSH credentials
        ↓
	Linux VM access
```

**Linux user account is directly tied to a user's Google identity - Your Linux username becomes linked to your Google account.**
Enable OS Login: 
```bash
# For Entire Project (Recommended)
gcloud compute project-info add-metadata \
    --metadata enable-oslogin=TRUE
    
# For Single VM
gcloud compute instances add-metadata VM_NAME \
    --zone=ZONE \
    --metadata enable-oslogin=TRUE

# check: Compute Engine => Settings => metadata => Metadata => will find key value pairs added (key: enable-oslogin , value: true) , we can add this key value pairs manually it Vm level or Project Level
```

The user must have one of these roles: `roles/compute.osLogin` (SSH access non-root) or `roles/compute.osAdminLogin` (SSH access with sudo), Grant role: 
```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="user:your-email@gmail.com" \
  --role="roles/compute.osLogin"
```

If the VM uses a service account, the user also needs `roles/iam.serviceAccountUser` on that service account. 

```bash
gcloud compute os-login ssh-keys add \
    --key-file ~/.ssh/id_rsa.pub
	
gcloud compute ssh VM_NAME --zone ZONE
```

**Windows VM Access (Different)**
- Windows instances do NOT use SSH, They use: `Username + Password authentication (RDP)`
- Generate Windows Password:
	```bash
			gcloud compute reset-windows-password INSTANCE_NAME
	```

---

## Moving VM Instances Between Zones and Regions

### Move VM Between Zones (Same Region)
VM instances can be moved between zones in the same region:
```bash
    gcloud compute instances move my-instance \
        --zone us-central1-a \
        --destination-zone us-central1-b
```

### Restrictions
You cannot use the `move` command for:
- Instances that are part of a Managed Instance Group (MIG)
- Instances attached with local SSDs
- Instances in TERMINATED status
- Moving instances across regions

### Manual Approach (For Cross-Region Move)
Since moving across regions is not supported directly, follow this manual approach:
1. Create snapshot of attached persistent disks
	```bash
	    gcloud compute disks snapshot my-disk-a \
	        --snapshot-names my-pd-snapshot \
	        --zone ZONE
	```
	
2. Create copy of persistent disk in destination zone (or new region)
	```bash
	    gcloud compute disks create my-disk-b \
	        --source-snapshot my-pd-snapshot \
	        --zone DESTINATION_ZONE
	```
	
3. Create new VM in destination zone (or new region)
	- Create a new instance
	- Attach the newly created persistent disks

---

> [!Info] 
> - Your application deployed on a GCE VM  (Project A) needs to access cloud storage bucket from a different project (Project B)
> 	- In Project B, assign the right role to GCE VM service account from Project A

---
