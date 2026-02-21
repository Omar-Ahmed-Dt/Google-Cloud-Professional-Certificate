---
tags:
  - gcp
---
## IAM Overview
- IAM is a tool to manage fine-grained authorization for Google Cloud. In other words, it lets you control who can do what on which resources.
- Every action in Google Cloud requires certain permissions. When someone tries to perform an action in Google Cloud—for example, create a VM instance or view a dataset—IAM first checks if they have the required permissions. If they don't, then IAM prevents them from performing the action.
- Giving someone permissions in IAM involves the following three components:
    - **Principal - Member - Identity:** The **identity** of the person or system that you want to give permissions to.
    - **Role:** The collection of **permissions** that you want to give the principal.
    - **Resource:** The Google Cloud **resource** that you want to let the principal access.

---

## Authentication and Authorization in GCP
- Authentication (is it the right user?)
- Authorization (do they have the right access?)
- Identities can be
    - A GCP User (Google Account or Externally Authenticated User)
    - A Group of GCP Users
    - An Application running in GCP
    - An Application running in your data center
    - Unauthenticated users

- Provides very granular control
    - Limit a single user:
        - to perform a single action
        - on a specific cloud resource
        - from a specific IP address
        - during a specific time window

---

## Cloud IAM Role
- I want to provide access to manage a specific Cloud Storage bucket to a colleague of mine:
    - Important Generic Concepts:
        - **Member:** My colleague
        - **Resource:** Specific Cloud Storage bucket
        - **Action:** Upload/Delete objects

- **Roles:** ==A set of permissions (to perform specific actions on specific resources - What Action ? and What Resource? ), Roles do NOT know about members. It is all about permissions!==
- How do you assign permissions to a member?
    - **Policy:** ==You assign (or bind) a role to a member, Member (user , service account, group , domain) gets permissions through Role== 

- You can't directly grant permissions to a principal. Instead, you give principals permissions by granting them roles.
- Roles are collections of permissions. When you grant a role to a principal, you give that principal all of the permissions in that role.
- There are three types of roles:
    - **Basic roles:**  **Owner**(`roles/owner`: Editor + Manage Roles and permissions + Billing) /**Editor** (`roles/editor`: Viewer + Edit actions) /**Viewer** (`roles/viewer`: Read Only actions) - Highly permissive roles that provide **broad access (not for a specific resource, it's for all resources)** to Google Cloud services. Not Recommended use in production 
    - **Predefined roles:** Roles that are managed by Google Cloud services. These roles contain the permissions needed to perform common tasks for each given service. For example, `Storage Admin, Storage Object Admin, Storage Object Viewer, Storage Object Creator`
    - **Custom roles:** Roles that you create that contain only the permissions that you specify. You have complete control over the permissions in these roles.

### IAM – Predefined Roles – Example Permissions
- Important Cloud Storage Roles:
    - Storage Admin (`roles/storage.admin`)
        - `storage.buckets.*`
        - `storage.objects.*`

    - Storage Object Admin (`roles/storage.objectAdmin`)
        - `storage.objects.*`

    - Storage Object Creator (`roles/storage.objectCreator`)
        - `storage.objects.create`

    - Storage Object Viewer (`roles/storage.objectViewer`)
        - `storage.objects.get`
        - `storage.objects.list`

- All four roles have these permissions: All Cloud Storage predefined roles include some Project-level read permissions.
    - `resourcemanager.projects.get`
    - `resourcemanager.projects.list`

---

## Service Account
- A Service Account is a special type of Google Cloud identity used by applications, services, or virtual machines — NOT humans.
- A user account for machines instead of people.
- Identified by an Email address (Ex: id-compute@developer.gserviceaccount.com)

### Types of Service Accounts
1️⃣ **Default Service Account**
- Automatically created by GCP services - Automatically created when some services are used
- (NOT RECOMMENDED) Has Editor role by default

2️⃣ **User-Managed Service Accounts - User Created**
- You create and control them.
- (RECOMMENDED) Provides fine grained access control
- Used for:
	- applications
	- microservices
	- automation scripts

3️⃣ **Google-Managed Service Accounts**
- Used internally by Google services.
- You normally don’t manage these.

### Is a Service Account an Identity or a Resource?
**Answer: BOTH**
A Service Account (SA) plays two roles at the same time in Google Cloud IAM: 
- A service account can act like a user **(a principal)** that performs actions. When SA is an Identity, You attach roles TO the service account so it can access resources.
- Service Account as **a Resource**. Other identities can be granted permission to **USE or MANAGE the service account (When creating a SA , we configure these options)**
	- Developer wants to deploy using a powerful service account.
	- Instead of giving developer admin permissions directly, `Developer → allowed to USE → Service Account , Service Account → has powerful roles` , So developer impersonates the SA.

---

## Use a Service Account Key (External Authentication)
You CANNOT assign Service Account directly to an On Prem App, **The Solution:**
1. Create a service account with required permissions.
2. Create a User-Managed Key - `Open Service Accounts page, Select your Service Account , Open Keys tab , Add Key , Choose Key Type (Json) , Then Create`
3. Make the service account key file accessible to your application - Place the JSON securely on your server, Then set environment variable: `export GOOGLE_APPLICATION_CREDENTIALS="/path/key.json"`
4. Use Google Cloud Client Libraries (ADC): ADC uses the service account key file if env var `GOOGLE_APPLICATION_CREDENTIALS` exists. 

- This allows external apps to authenticate as the service account.
- **long-lived & risky**

---

## Short-lived authentication outside GCP
1. **OAuth 2.0 Access Token** , When a member needs elevated permissions, he can assume the service account role (Create OAuth 2.0 access token for service account)
	- [lab](https://www.youtube.com/watch?v=D8DMj2lQMwo)
2. **OpenID Connect (OIDC) Token**, OpenID Connect tokens is recommended for service to service authentications: `A service in GCP needs to authenticate itself to a service in other cloud`

---
## Service Account Use case Scenarios

| Scenario | Solution |
|---|---|
| Application on a VM wants to talk to a Cloud Storage bucket | Configure the VM to use a Service Account with right permissions |
| Application on a VM wants to put a message on a Pub/Sub Topic | Configure the VM to use a Service Account with right permissions |
| Is Service Account an identity or a resource? | It is both. You can attach roles with Service Account (identity). You can let other members access a SA by granting them a role on the Service Account (resource). |
| VM instance with default service account in Project A needs to access Cloud Storage bucket in Project B | In project B, add the service account from Project A and assign **Storage Object Viewer** permission on the bucket |

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
    - **Uniform (Recommended)**, Uniform bucket-level access using IAM.
    - **Fine-grained**, Use IAM and ACLs to control access: Both bucket-level and individual object-level permissions.

- Use **Uniform access** when all users have the same level of access across all objects in a bucket.
- **Fine-grained access** with ACLs can be used when you need to customize access at an object level.

---

## Cloud Storage – Signed URL
- You would want to **allow a user limited time access** to your objects: Users do **NOT** need Google accounts
- Use **Signed URL** functionality, A URL that gives **permissions for limited time duration** to perform specific actions
- **To create a Signed URL:**
    1. Create a key (`YOUR_KEY`) for the Service Account/User with the desired permissions
    2. Create Signed URL with the key:
	
```bash
gsutil signurl -d 10m YOUR_KEY gs://BUCKET_NAME/OBJECT_PATH
```

---

## Cloud Storage – Static Website
1. **Create a bucket with the same name as the website name**
	- **The bucket name should match the DNS name of the website**
	- Verify that the domain is owned by you
	
2. **Copy the files to the bucket**: Add `index.html` and `error.html` files for a better user experience
3. **Add member `allUsers` and grant `Storage Object Viewer` role**: This allows public read access to website files

---

## Scenarios

| Scenario | Solution |
|----------|----------|
| An Application on a GCE VM needs access to Cloud Storage | Use a Service Account (Google Cloud-managed keys) |
| An Application on premises needs access to Cloud Storage | Use Service Account User Managed Key |
| Allow a user limited time access to your objects | Signed URL |
| Customize access to a subset of objects in a bucket | Use ACL (Access Control Lists) |
| Permission is allowed by IAM but NOT by ACL. Will user be able to access the object? | Yes |

---
