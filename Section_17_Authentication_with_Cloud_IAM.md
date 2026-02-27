---
tags:
  - gcp
---
## IAM Overview
- IAM is a tool to manage fine-grained **authorization** (IAM is for Authorization, Authentication is handled by Identity systems) for Google Cloud. In other words, it lets you control who can do what on which resources.
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
- NOT Recommended to use it - Use SA Keys only for use-cases which cannot use a more secure alternative.
- **Long-lived & Risky**

### Service Account Impersonation
- Service Account Impersonation allows a user or another service account to generate short-lived credentials and perform actions as a target service account, based on IAM permissions.
- Borrow the identity of a service account temporarily instead of sharing credentials.
- Problem It Solves - Old way (❌ insecure): Create service account key JSON , Download key - anyone have a SA key can act as a service account without any `gcloud auth`
- No keys created, Access granted via IAM, Credentials are temporary tokens
- default service account impersonation lifetime in GCP is: **1 hour (3600 seconds)**

```text
User / VM / CI/CD Identity
        │
        │ (has permission: iam.serviceAccountTokenCreator)
        ▼
Impersonate Service Account
        │
        ▼
Temporary Access Token (short-lived)
        │
        ▼
Access GCP resources AS that service account

```

#### Step by Step:
-  Admin grants impersonation permission
	```bash
			gcloud iam service-accounts add-iam-policy-binding SA_EMAIL \
			  --member="user:USER_EMAIL" \
			  --role="roles/iam.serviceAccountTokenCreator"
  
	    	  # Example 
			  gcloud iam service-accounts add-iam-policy-binding \
				deploy-sa@my-project.iam.gserviceaccount.com \
				  --member="user:omar@gmail.com" \
				  --role="roles/iam.serviceAccountTokenCreator"
				  # Omar can now impersonate deploy-sa.
	```
	- `SA_EMAIL` → service account email, `USER_EMAIL` → human user email
	- This command allows a user to impersonate a service account.

-  User authenticates locally and Run commands using impersonation
	```bash
		gcloud auth login
		gcloud COMMAND \
		  --impersonate-service-account=SA_EMAIL
		
		# Example 
		gcloud compute instances list \
		  --impersonate-service-account=deploy-sa@my-project.iam.gserviceaccount.com
		  # The command runs with the permissions of the service account, not your user.
	```

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

## Scenarios

| Scenario | Solution |
|----------|----------|
| An Application on a GCE VM needs access to Cloud Storage | Use a Service Account (Google Cloud-managed keys) |
| An Application on premises needs access to Cloud Storage | Use Service Account User Managed Key |
| Allow a user limited time access to your objects | Signed URL |
| Customize access to a subset of objects in a bucket | Use ACL (Access Control Lists) |
| Permission is allowed by IAM but NOT by ACL. Will user be able to access the object? | Yes |

---
