---
tags:
  - gcp
---
## Cloud Functions Overview
- Imagine you want to **execute some code when an event happens?**
    - A file is uploaded in Cloud Storage  
    - An error log is written to Cloud Logging  
    - A message arrives to Cloud Pub/Sub  
    - An HTTP/HTTPS invocation is received  

- Enter **Cloud Functions**
    - **Run code in response to events**
        - Write your business logic in Node.js, Python, Go, Java, .NET, and Ruby
        - Don’t worry about servers, scaling, or availability (only focus on your code)
		
    - **Pay only for what you use**
        - Number of invocations
        - Compute time of the invocations
        - Memory and CPU provisioned
		
    - **Time Bound**
        - Default: 1 minute
        - Maximum: 60 minutes (3600 seconds)
		
    - **2 product versions**
        - Cloud Functions (1st gen): First version
        - Cloud Functions (2nd gen): New version built on top of Cloud Run and Eventarc

---

## Concepts

- **Event**: An Event is something that happens in your cloud environment - Upload object to Cloud Storage. 
- **Trigger**: A Trigger connects the event to your function - When this event happens, which function should run?
    - Trigger – Which function to trigger when an event happens?
    - Functions – Take event data and perform action 
    - Event → Trigger → Function
	
- Example:
	- Event: File uploaded to bucket images-bucket
	- Trigger: Listen to upload events in that bucket
	- Result: Call your function

- Cloud Functions can be triggered by:
    - Cloud Storage
    - Cloud Pub/Sub
    - HTTP POST / GET / DELETE / PUT / OPTIONS
    - Firebase
    - Cloud Firestore
    - Stackdriver Logging

---

## Cloud Functions – Second Generation – What's New?
- **2 Product Versions:**
    - Cloud Functions (1st gen): First version
    - Cloud Functions (2nd gen): New version built on top of Cloud Run and Eventarc

- **Recommended:** Use Cloud Functions (2nd gen)
- **Key Enhancements in 2nd gen:**
    - **Longer Request Timeout:** Up to 60 minutes for HTTP-triggered functions
    - **Larger Instance Sizes:** Up to 16 GiB RAM with 4 vCPU  
      (v1: Up to 8 GB RAM with 2 vCPU)
    - **Concurrency:** Up to 1000 concurrent requests per function instance  
      (v1: 1 concurrent request per function instance)
    - **Multiple Function Revisions and Traffic Splitting supported**  
      (v1: NOT supported)

---

## Cloud Functions – Scaling and Concurrency

- **Typical serverless functions architecture:**
    - **Autoscaling** – As new invocations come in, new function instances are created
    - **One function instance handles ONLY ONE request at a time:** ==3 concurrent function invocations ⇒ 3 function instances==
        - If a 4th invocation occurs while others are in progress, a new function instance is created
        - HOWEVER, a function instance that completed execution may be reused for future requests
    - **(Typical Problem) Cold Start:**
        - New function instance initialization from scratch can take time
        - (Solution) Configure minimum number of instances (increases cost)

- **1st Gen** uses the typical serverless functions architecture
- **2nd Gen** adds a very important new feature:
    - ==One function instance can handle multiple requests AT THE SAME TIME==
        - **Concurrency:** ==How many concurrent invocations can one function instance handle? (Max 1000)==
        - (IMPORTANT) Your function code should be safe to execute concurrently

---
