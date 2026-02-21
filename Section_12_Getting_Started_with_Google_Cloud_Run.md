---
tags:
  - gcp
---
## Cloud Run - Serverless Overview
- Cloud Run is a fully managed application platform for running your code, function, or container on top of Google's highly scalable infrastructure.
- You can deploy code written in any programming language on Cloud Run **if you can build a container image from it. In fact, building container images is optional. If you're using Go, Node.js, Python, Java, .NET, Ruby, or a supported framework you can use the source-based deployment option** that builds the container for you, using the best practices for the language you're using.
- You give it a container image. Google runs it, scales it, and exposes it over HTTPS.
- You don’t manage: Servers, VMs, Kubernetes, Load balancers, Autoscaling, `You just provide a container`.
- Request-Driven: The service stays idle (at zero cost) until a request hits its URL. Google then instantly starts your container to handle the traffic.
- Built-in Load Balancing: Every service gets a stable HTTPS URL automatically.
- **Cloud Run** – Container to Production in Seconds
    - Fully managed serverless platform for containerized applications
        - ZERO infrastructure management
        - Pay-per-use (CPU, Memory, Requests, Networking)

## Cloud Run types

| Resource    | Description                                                                                                                                                                        |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Service     | Responds to HTTP requests sent to a unique and stable endpoint, using stateless instances that autoscale based on a variety of key metrics. Also responds to events and functions. |
| Job         | Executes parallelizable tasks that are executed manually, or on a schedule, and run to completion.                                                                                 |
| Worker pool | Handles always-on **background workloads** such as pull-based workloads, for example, Kafka consumers, Pub/Sub pull queues, or RabbitMQ consumers.                                 |

## 🏗 How Cloud Run Works Conceptually
0. You build your application inside a container.
1. You push that container image to a container registry.
2. You deploy the image to Cloud Run.
3. Cloud Run creates a service.
4. That service gets a public HTTPS endpoint.
5. When requests arrive: Cloud Run starts containers (if needed), Handles traffic, Scales automatically
6. When traffic stops:
	- It scales down (even to zero)

## Cloud Run – From the Command Line

| Description                | Command                                                                                                                                                                                                   |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Deploy a new container     | `gcloud run deploy SERVICE_NAME --image IMAGE_URL --revision-suffix v1`  <br><br> First deployment creates a service and first revision. <br> Next deployments for the same service create new revisions. |
| List available revisions   | `gcloud run revisions list`                                                                                                                                                                               |
| Adjust traffic assignments | `gcloud run services update-traffic myservice --to-revisions=v2=10,v1=90`                                                                                                                                 |

---
