---
tags:
  - gcp
---
## Resource Hierarchy in GCP
Organization → Folder → Project → Resources
- Resources are created inside **projects**
- A **Folder** can contain multiple projects
- An **Organization** can contain multiple folders

### Resource Hierarchy – Recommendations for Enterprises
- Create Separate Projects for Different Environments: 
	- Complete isolation between test and production environments

- Create Separate Folders for Each Department
    - Isolate production applications of one department from another
    - Create a shared folder for shared resources

- One Project per Application per Environment
- Example: Applications A1 && A2 - Environments DEV && PROD
	- **Ideal Setup**
		- A1-DEV
		- A1-PROD
		- A2-DEV
		- A2-PROD

- **Benefits**
	- Isolates environments from each other
	- DEV changes will NOT break PROD
	- Grant developers full access (create, delete, deploy) to DEV projects
	- Provide production access only to operations teams

---

## Billing Accounts
- Billing Account is **mandatory** for creating resources in a project
    - Billing Account contains the payment details
    - Every project with active resources must be associated with a Billing Account
	
- A Billing Account can be associated with **one or more projects**
- An Organization can have **multiple billing accounts**
- Recommendation: Create Billing Accounts representing your organization structure:
    - A startup can have just one Billing Account
    - A large enterprise can have separate billing accounts for each department
	
- Types of Billing Accounts
	- **Self Serve**: Billed directly to Credit Card or Bank Account
	- **Invoiced**: Generates invoices, Typically used by large enterprises

### Managing Billing - Budget, Alerts and Exports
- (RECOMMENDED) Configure alerts
- Default alert thresholds: 50% , 90%, 100%
    - Send alerts to Pub/Sub (Optional)
    - Billing admins and Billing Account users are alerted via e-mail
	
### Billing Data Export (Scheduled)
Billing data can be exported on a schedule to:
- **BigQuery**: Used for querying billing data or visualization
- **Cloud Storage**: Used for history and archiving

---

