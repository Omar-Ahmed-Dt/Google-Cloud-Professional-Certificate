---
tags:
  - gcp
---
## Asynchronous Communication
- Create a topic and have your applications publish messages to that topic.
- A separate service (e.g., Logging Service) subscribes and processes messages when ready.

**Flow**
```text
Web Server (Publisher)
    ↓
Topic / Message Queue
    ↓
Logging Service (Subscriber)
    ↓
Database / Storage
```

**Advantages**
- Decoupling: Publisher (Apps) does not care who is listening && No direct dependency between services
- Availability: Publisher stays up even if Subscriber is down
- Scalability: Scale consumer instances (Logging Service) under high load
- Durability: Messages are not lost even if Subscriber is temporarily unavailable

---

## Pub/Sub
- Is an asynchronous and scalable messaging service that decouples services producing messages from services processing those messages
- Backbone for Highly Available and Highly Scalable solutions
    - Auto-scales to process billions of messages per day
    - Low cost (Pay for use)

- Message Delivery Models: Supports **Push** delivery and **Pull** delivery
- Use Cases: Event ingestion and delivery for streaming analytics pipelines

**Publishers  →  Topic  →  Multiple Subscriptions  →  Multiple Consumers**
```text
                    +-----------------------+
                    |      Cloud Pub/Sub    |
                    |                       |
Mobile  ----------- |                       |
                    |      +-----------+    |
On-Prem / 3rd ----> | ---> |  Topic 1  | ---+----> [Subscription 1] ----> Datawarehouse
Party               |      +-----------+    | 
                    |                       +----> [Subscription 2] ----> App Engine (Microservices)
IoT  -------------->|                       |
                    |                       +----> [Subscription 3] ----> Legacy / 3rd Party Services
                    +-----------------------+
```

### Pub/Sub – How does it work?
#### Publisher
- Publisher = Sender of a message - Publishers send messages by making HTTPS requests to: `pubsub.googleapis.com`
#### Subscriber
- Subscriber = Receiver of the message
##### Pull Model
- Subscriber pulls messages when ready - subscribers must request delivery
- Subscriber makes HTTPS requests to:  `pubsub.googleapis.com`

##### Push Model
- Messages are automatically sent to subscribers - delivers messages immediately after publish
- Subscriber provides a web hook endpoint at the time of registration
- When a message is received on the topic,  an HTTPS POST request is sent to the subscriber's web hook endpoint

Pub/Sub supports flexible publisher and subscriber relationships: **One to Many, Many to One , Many to Many**

### Pub/Sub concepts
**Step 1 — Topic is Created**
- A Topic is the message channel.
- Think of it as: A broadcast channel where publishers send messages.
- Publishers send messages to the topic: `App → Topic`
- The topic does NOT store messages for users directly - it only receives messages.

**Step 2 — Subscriptions are Created**
- Subscription = message delivery pipeline
- A Subscription connects a subscriber to a topic - Subscribers register to the topic using subscriptions. `Publisher → Topic → Subscription → Subscriber`
- Each subscription receives messages independently - **Each Subscription Represents a Separate Message Stream**

#### Two Important Scenarios
**Scenario 1 — Multiple Clients Use SAME Subscription**
```text
Topic
  ↓
Subscription A
  ↓
Worker 1
Worker 2
Worker 3
```

Result:
 - Messages are split between clients
 - Load balancing happens automatically
 
```text
Message 1 → Worker 1
Message 2 → Worker 2
Message 3 → Worker 3
```

**Scenario 2 — Each Client Has Its OWN Subscription**
```text
Topic
 ├── Subscription A → Service A
 ├── Subscription B → Service B
 └── Subscription C → Service C
```

Result:
- Every client receives ALL messages

```text
Message 1 → A, B, C
Message 2 → A, B, C
```

### Pub/Sub – Sending and Receiving a Message
**Message Flow**
- Publisher sends a message to a **Topic**
- Message is **individually delivered** to each and every subscription, Subscribers can receive messages using:
	- **Push**: Pub/Sub sends the message directly to the subscriber
	- **Pull**: Subscribers poll Pub/Sub for messages
	
- Subscribers send acknowledgement(s) to Subscription after processing the message
- Message Removal: Message(s) are removed from the subscription message queue after acknowledgement: Pub/Sub ensures the message is retained **per subscription** until it is acknowledged

---

## Understanding Cloud Pub/Sub Best Practices
**Use Cases**
- ==Convert synchronous workflows to asynchronous workflows==
    - Useful when the consumer cannot keep up with the producer (buffer data)
    - Alternatives include: RabbitMQ, Apache Kafka

- ==Apply transformations to IoT data streams==
- ==Ordering and Exactly-Once Processing==: **In-order processing** and **Exactly-once processing** (de-duplication)
	- In-Order Processing: Pub/Sub supports message ordering, Enable using: `--enable-message-ordering` (on subscription)
	- Exactly-Once Processing (De-duplication)
		- Add **Dataflow** into the pipeline to enable message de-duplication: Maintains a list of message IDs for a time period and If a message ID repeats, it is discarded (assumed duplicate) 
		
---

## Cloud Dataflow
Cloud Dataflow is a fully managed data processing service used to:
- Transform data, Filter data, Aggregate data, Move data between systems, Process streaming and batch workloads
- Serverless and Autoscaling
- Because Dataflow is not a database, not a queue, not storage. It is a **Data Processing Engine** , It sits between systems and transforms data.
- Think of Dataflow as the middle processing layer:
	- `Pub/Sub → Dataflow → BigQuery` 
	- `Pub/Sub → Dataflow → Cloud Storage`
	- `Cloud Storage → Dataflow → Bigtable / Spanner / BigQuery`

---
