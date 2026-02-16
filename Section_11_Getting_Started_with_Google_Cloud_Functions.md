---
tags:
  - gcp
---
# Cloud Functions
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
