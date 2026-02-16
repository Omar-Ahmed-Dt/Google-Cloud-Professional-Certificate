---
tags:
  - gcp
---
# Google Kubernetes Engine (GKE) - GKE
GKE has two main modes:
- **Standard**
	- Google manages: **Control plane (master nodes) , Kubernetes API**
	- You manage: **Node pools, Machine types, Scaling rules, OS upgrades (mostly), Networking settings**
	- You pay for: VM instances (nodes), Even if pods aren’t using full capacity

- **Autopilot** This is fully managed Kubernetes. You don’t manage nodes. You only deploy pods.
	- Google automatically:
		-  Creates nodes
		-  Scales nodes
		-  Handles upgrades
		-  Handles security patches

	- Billing: You pay per: Pod CPU, Pod memory, Not per VM. More granular billing.

```bash
gcloud projects list
gcloud config set project my-kubernetes-project-304910
gcloud container clusters get-credentials my-cluster --zone us-central1-c --project my-kubernetes-project-304910

# Increase number of instances
gcloud container clusters resize my-cluster --node-pool <node pool name: default-pool> --num-nodes=2 --zone=us-central1-c

# scale the deployment based on the cup utilization
kubectl autoscale deployment hello-world-rest-api --max=4 --cpu-percent=70 # will create hpa
kubectl get hpa

# scale nodes 
gcloud container clusters update cluster-name \
  --zone=us-central1-a \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10

# Create a new node pool (e.g., GPU node pool)
gcloud container node-pools create POOL_NAME \
  --cluster CLUSTER_NAME # will create a node pool with default configs unless you specify options

# create a new node pool with a custom configs  
gcloud container node-pools create web-pool \
  --cluster CLUSTER_NAME \
  --zone ZONE \
  --machine-type e2-standard-4 \
  --disk-type pd-balanced \
  --disk-size 100 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 10 \
  --labels=workload=web \
  --node-taints=nvidia.com/gpu=present:NoSchedule
  
  
# List node pools in a cluster
gcloud container node-pools list \
  --cluster CLUSTER_NAME

# deployment.yaml (schedule workload to specific node pool)
# nodeSelector:
#         cloud.google.com/gke-nodepool: POOL_NAME
 
gcloud container clusters delete my-cluster --zone us-central1-c
```

---

## What is the default node pool in GKE?
- When you create a GKE **Standard cluster**, Google automatically creates `default-pool` , This is just the first node pool created inside your cluster.
- A node pool = a group of VM instances (nodes)
- All nodes inside a pool:
	- Have the same machine type
	- Same disk type
	- Same configuration
	- Same autoscaling settings

So basically:
- Cluster = Control plane
- Node pool = Group of worker VMs
- Nodes = Actual Compute Engine VMs

When you run:
```bash
gcloud container clusters create my-cluster
```
GKE automatically creates: Usually the default node pool contains 3 nodes by default.
```text
Cluster
 └── default-pool
       ├── node-1
       ├── node-2
       └── node-3
```

In GKE **Autopilot:**
🚫 There is NO default node pool, Because:
- You do NOT manage nodes
- Google creates and deletes nodes automatically
- You never see or configure node pools
- So default node pool only applies to Standard mode.

**Best practice in production:**
1. Create cluster with minimal default pool
2. Create custom node pools
3. Delete default-pool


- Cluster: Group of Compute Engine instances:
    - Master Node(s) - Manages the cluster
    - Worker Node(s) - Run your workloads (pods)

- Master Node (Control plane) components:
    - API Server - Handles all communication for a K8S cluster (from nodes and outside)
    - Scheduler - Decides placement of pods
    - Control Manager - Manages deployments & replicasets
    - etcd - Distributed database storing the cluster state

- Worker Node components:
    - Runs your pods

---

## Kubernetes Components 
- Cluster: Group of Compute Engine instances:
    - **Master Node(s)** - Manages the cluster
    - **Worker Node(s)** - Run your workloads (pods)

- Master Node (Control plane) components:
    - **API Server** - Handles all communication for a K8S cluster (from nodes and outside)
    - **Scheduler** - Decides placement of pods
    - **Control Manager** - Manages deployments & replicasets
    - **etcd** - Distributed database storing the cluster state

- Worker Node components:
    - **Runs your workloads**

---

## GKE Cluster Types

| Type            | Description |
|-----------------|-------------|
| Zonal Cluster  | **Single Zone** - Single Control plane. Nodes running in the same zone. |
| Multi-zonal    | **Single Control plane** but nodes running in multiple zones. |
| Regional cluster | Replicas of the control plane run in multiple zones of a given region. Nodes also run in the same zones where the control plane runs. |
| Private cluster | VPC-native cluster. Nodes only have internal IP addresses. |
| Alpha cluster  | Clusters with alpha APIs - early feature APIs. Used to test new K8S features. |

---

## What is a Container Registry
A Container Registry is a service that stores and manages Docker container images.
1. **Google Container Registry (GCR) – Legacy:**  `gcr.io/PROJECT-ID/my-app:tag`=> gcr.io/my-project/my-app:v1
2.  **Artifact Registry (Recommended):** `REGION-docker.pkg.dev/PROJECT-ID/REPO-NAME/IMAGE:TAG`=> us-central1-docker.pkg.dev/my-project/my-repo/my-app:v1 

---

## Scenarios

| Scenario | Solution |
|----------|----------|
| You want to keep your costs low and optimize your GKE implementation | Consider Preemptible VMs, appropriate region selection, and Committed-use discounts. E2 machine types are cheaper than N1. Choose the right environment to fit your workload type (use multiple node pools if needed). |
| You want an efficient, completely auto-scaling GKE solution | Configure Horizontal Pod Autoscaler for deployments and Cluster Autoscaler for node pools. |
| You want to execute untrusted third-party code in Kubernetes Cluster | Create a new node pool with GKE Sandbox. Deploy untrusted code to the Sandbox node pool. |
| You want enable ONLY internal communication between your microservice deployments in a Kubernetes Cluster | Create a Service of type ClusterIP. If you don't want to expose your deployment externally, use ClusterIP (default service type). |
| My pod stays pending | Most probably the Pod cannot be scheduled onto a node (insufficient resources). |
| My pod stays waiting | Most probably failure to pull the container image. |

---
