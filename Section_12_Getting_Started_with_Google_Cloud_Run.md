---
tags:
  - gcp
---
# Cloud Run - Serverless
- Cloud Run is a fully managed serverless platform that lets you run containerized applications.
- You give it a container image. Google runs it, scales it, and exposes it over HTTPS.
- You don’t manage: Servers, VMs, Kubernetes, Load balancers, Autoscaling, `You just provide a container`.
- Request-Driven: The service stays idle (at zero cost) until a request hits its URL. Google then instantly starts your container to handle the traffic.
- Built-in Load Balancing: Every service gets a stable HTTPS URL automatically.
- **Cloud Run** – Container to Production in Seconds
    - Built on top of an open standard – **Knative**
    - Fully managed serverless platform for containerized applications
        - ZERO infrastructure management
        - Pay-per-use (CPU, Memory, Requests, Networking)

- Fully integrated end-to-end developer experience:
    - No limitations in languages, binaries, and dependencies
    - Easily portable because of container-based architecture
    - Integrations with Cloud Code, Cloud Build, Cloud Monitoring, and Cloud Logging

- **Anthos** – Run Kubernetes clusters anywhere: Cloud, Multi-cloud, On-premise - Anthos = Kubernetes anywhere (but managed by Google)
- **Cloud Run for Anthos** Deploy your workloads to Anthos clusters, Run on-premises or on Google Cloud

## 🏗 How Cloud Run Works Conceptually
0. You build your application inside a container.
1. You push that container image to a container registry.
2. You deploy the image to Cloud Run.
3. Cloud Run creates a service.
4. That service gets a public HTTPS endpoint.
5. When requests arrive: Cloud Run starts containers (if needed), Handles traffic, Scales automatically
6. When traffic stops:
7. It scales down (even to zero)

## Cloud Run – From the Command Line

| Description                | Command                                                                                                                                                                                                   |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Deploy a new container     | `gcloud run deploy SERVICE_NAME --image IMAGE_URL --revision-suffix v1`  <br><br> First deployment creates a service and first revision. <br> Next deployments for the same service create new revisions. |
| List available revisions   | `gcloud run revisions list`                                                                                                                                                                               |
| Adjust traffic assignments | `gcloud run services update-traffic myservice --to-revisions=v2=10,v1=90`                                                                                                                                 |

---
