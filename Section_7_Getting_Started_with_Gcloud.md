---
tags:
  - gcp
---
# Gcloud
- gcloud is the primary command-line interface (CLI) tool for Google Cloud Platform (GCP). for managing cloud resources without having to click through the web console. 
- The gcloud tool is part of the Google Cloud SDK and handles the heavy lifting for most GCP services: Compute, Containers, Storage, Network

## How it’s Structured
- Commands follow a predictable, hierarchical pattern: `gcloud <group> <subgroup> <action> <flags>`
- Example Command: `gcloud compute instances create my-server --machine-type=n1-standard-1`
- Tip: After installing, always run `gcloud auth login` , `gcloud init` , `gcloud config set project [PROJECT_ID]` to make sure
- lists all properties of the active configuration - show account and project id: `gcloud config list`

```bash
gcloud config set project my-project-id
gcloud services enable compute.googleapis.com
gcloud services enable container.googleapis.com
# gcloud services = manage APIs in your project.

# show running instances
gcloud compute instances list

# Examples
gcloud compute zones list
gcloud compute regions list
gcloud compute machine-types list
gcloud compute images list
gcloud compute disks list
gcloud compute addresses list
gcloud compute networks list
gcloud compute firewall-rules list
```

# Cloud Shell
- Important things you need to know about Cloud Shell:
	- Cloud Shell is backed by a VM instance (automatically provisioned by Google Cloud when you launch Cloud Shell)
	- 5 GB of free persistent disk storage is provided as your `$HOME` directory
	- Prepackaged with the latest version of Cloud SDK, Docker, etc.
	- (Remember) Files in your home directory persist between sessions  (scripts, user configuration files like `.bashrc` and `.vimrc`, etc.)
	- The instance is terminated if you are inactive for more than 20 minutes
	- Any modifications made outside your `$HOME` directory will be lost
	- (Remember) After 120 days of inactivity, even your `$HOME` directory is deleted
	
- Cloud Shell can be used to SSH into virtual machines using their private IP addresses

---
