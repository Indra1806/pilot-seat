> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine a **newspaper subscription**.

When a major news event happens:
- The newspaper **prints the story** (produces the event)
- Every **subscriber** gets a copy delivered to their door (consumes the event)
- The newspaper doesn't know who specifically will read the story — it just prints and distributes
- New subscribers can sign up any time — they'll get future papers without the newspaper changing how it works
- One story can trigger action in many readers simultaneously

Now compare this to a phone call:
- You call one specific person
- You wait for them to pick up
- If they're busy, you're stuck
- Only one conversation at a time

Newspaper = **Event-Driven**. Phone call = **Request-Response (REST)**.

**Event-Driven Architecture (EDA)** is a software design pattern where components communicate by **producing** and **consuming events** — rather than calling each other directly. An **event** is a record that something significant happened: "Order was placed", "Payment was processed", "User signed up".

Services that produce events don't know who will consume them. Services that consume events don't know who produced them. They are **completely decoupled** from each other — connected only by the events flowing through a **message broker** in the middle.

---

# Why It Exists

### The Problem with Direct Service Calls

In traditional architectures, services call each other **synchronously**:

```
Order placed → OrderService calls → InventoryService
                                  → NotificationService
                                  → AnalyticsService
                                  → LoyaltyService
                                  → AuditService
```

Problems with this approach:
1. **Tight coupling**: OrderService must know the address of every downstream service
2. **Cascading failures**: If AnalyticsService is down, the entire order flow fails
3. **Slow responses**: User waits while OrderService calls 5 other services
4. **Adding a new consumer**: You must modify OrderService to add a 6th call
5. **Traffic spikes**: All 5 services must handle the same peak load simultaneously

### The Event Solution

```
Order placed → OrderService publishes "order.created" event
                    ↓
             [Message Broker]
                    ↓
       ┌────────────────────────────┐
       ↓            ↓              ↓
InventoryService  NotifyService  AnalyticsService
(consumes)        (consumes)     (consumes)
```

Now:
1. **Loose coupling**: OrderService doesn't know or care who's listening
2. **Resilience**: If AnalyticsService is down, the event waits in the queue
3. **Fast response**: OrderService publishes and returns immediately
4. **Easy extension**: Add LoyaltyService tomorrow — subscribe to the event, zero changes to OrderService
5. **Independent scaling**: Each consumer scales based on its own load

---

# Problem It Solves

**Problem 1: Tight Coupling Between Services**

```
Before EDA:
  ❌ OrderService.createOrder() calls:
      → inventoryService.decrementStock()
      → notificationService.sendEmail()
      → loyaltyService.addPoints()
      → analyticsService.trackPurchase()
      
  If loyalty service API changes → OrderService must be updated.
  If analytics goes down → entire order fails.
  Adding new consumer → modify OrderService code.

After EDA:
  ✅ OrderService.createOrder() → publishes "order.created" event
  
  Each consumer independently listens and reacts.
  OrderService knows nothing about who's listening.
  New consumer? Subscribe to the event. Zero changes upstream.
```

**Problem 2: Traffic Spikes and Backpressure**

```
Before EDA:
  Black Friday: 100,000 orders/minute
  → NotificationService receives 100,000 email requests at once
  → It crashes under the load

After EDA:
  Black Friday: 100,000 orders/minute
  → 100,000 events queue up in Kafka
  → NotificationService processes at its own pace (10,000/minute)
  → Queue absorbs the spike → no crash → all emails eventually sent
```

**Problem 3: Cross-System Integration**

```
Before EDA:
  When a new order is placed, you need to notify:
    → Your internal shipping system (Node.js)
    → Your partner's warehouse API (different company, different language)
    → Your analytics platform (Snowflake / BigQuery)
    → Your CRM (Salesforce)
  
  Building 4 different direct integrations = fragile and expensive.

After EDA:
  Publish one "order.created" event.
  Each system subscribes and processes in its own language and format.
  Adding new system = subscribe to the event.
```

---

# Core Concepts

## 1. Event

An event is an **immutable record of something that happened** — past tense.

```
Good Event Names (past tense — things that occurred):
  ✅ order.created
  ✅ payment.processed
  ✅ user.registered
  ✅ shipment.dispatched
  ✅ password.reset

Bad Event Names (commands — telling something to do):
  ❌ create.order       (this is a command, not an event)
  ❌ send.email         (this is an instruction)
  ❌ process.payment    (imperative)

Event = "this happened"
Command = "do this"
```

An event payload typically includes:
```json
{
  "eventId": "evt-8821",
  "eventType": "order.created",
  "timestamp": "2024-06-19T10:30:00Z",
  "version": "1.0",
  "data": {
    "orderId": 789,
    "userId": 123,
    "total": 99.99,
    "items": [{ "productId": 456, "qty": 2 }]
  }
}
```

## 2. Producer (Publisher)

The component that **creates and publishes** events:

```
OrderService creates an order → publishes "order.created" to Kafka
PaymentService charges card → publishes "payment.processed" to Kafka
UserService creates account → publishes "user.registered" to Kafka
```

The producer doesn't care who reads the event. It just sends it.

## 3. Consumer (Subscriber)

The component that **listens for and reacts** to events:

```
NotificationService listens for "order.created" → sends confirmation email
InventoryService listens for "order.created" → decrements stock
LoyaltyService listens for "order.created" → adds loyalty points
AnalyticsService listens for "order.created" → records purchase data
```

Multiple consumers can react to the same event independently.

## 4. Message Broker

The **intermediary** that receives events from producers and delivers them to consumers:

```
Producer → [Message Broker] → Consumer(s)

Broker Responsibilities:
  ├── Receive events from producers
  ├── Store events (persisted to disk)
  ├── Deliver events to subscribed consumers
  ├── Handle consumer failures (retry, dead letter queue)
  └── Support millions of events per second at scale
```

Popular brokers:
```
Apache Kafka   → High-throughput streaming; LinkedIn, Uber, Netflix
RabbitMQ       → Traditional message queue; flexible routing
AWS SQS/SNS    → Cloud-native managed queuing/pub-sub
Redis Pub/Sub  → Simple, fast but no persistence
Azure Service Bus → Microsoft cloud event system
```

## 5. Topic / Queue / Channel

The **named channel** through which events flow:

```
Kafka Topics:
  orders.created    ← All order creation events go here
  payments.processed ← All payment events go here
  users.registered   ← All user signup events go here

Consumers subscribe to specific topics.
One consumer can subscribe to multiple topics.
Multiple consumers can subscribe to the same topic.
```

---

# Architecture / Components

## Event-Driven Architecture Patterns

### Pattern 1: Pub/Sub (Publish-Subscribe)

One event published → delivered to **all subscribers** independently:

```
Publisher                Broker              Subscribers
━━━━━━━━━               ━━━━━━━━━           ━━━━━━━━━━━━━━━━━━━━━━━━━

OrderService  ──event──→ [orders.created] ──→ NotificationService
                               │
                               ├─────────────→ InventoryService
                               │
                               └─────────────→ AnalyticsService

Each subscriber gets a copy. Each processes independently.
```

### Pattern 2: Message Queue (Point-to-Point)

One event published → consumed by **exactly one** consumer (load distribution):

```
Publisher                Queue               Consumers
━━━━━━━━━               ━━━━━━━            ━━━━━━━━━━━━━━━━━━━━━━

OrderService  ──jobs──→ [email-queue] ──→ EmailWorker-1
                               │
                               └─────────→ EmailWorker-2
                               
Each message consumed by only ONE worker (load balanced).
Useful for distributing work across multiple workers.
```

### Pattern 3: Event Streaming (Kafka)

Events are stored as an **immutable, ordered log**:

```
Kafka Topic: orders.created
━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Event 1][Event 2][Event 3][Event 4][Event 5]... [Event 10,000,000]

Properties:
  → Events are NEVER deleted (by default, kept for 7 days)
  → Consumers read at their OWN pace (via offset)
  → Consumer A is at position 5,000
  → Consumer B is at position 9,999,000
  → New consumer can "replay" from position 0 (historical events)
  → Supports millions of events per second
```

## Full Event-Driven System Architecture

```
┌────────────────────────────────────────────────────────────┐
│                     PRODUCERS                              │
│  [OrderService]  [PaymentService]  [UserService]           │
└───────────┬──────────────┬────────────────┬───────────────┘
            │              │                │
            ▼              ▼                ▼
┌────────────────────────────────────────────────────────────┐
│                  MESSAGE BROKER (KAFKA)                     │
│                                                            │
│  [orders.created]  [payments.done]  [users.registered]    │
│  [orders.cancelled] [shipments.sent] [products.updated]   │
└───────────┬──────────────┬────────────────┬───────────────┘
            │              │                │
            ▼              ▼                ▼
┌────────────────────────────────────────────────────────────┐
│                     CONSUMERS                              │
│  [NotifyService]  [InventoryService]  [LoyaltyService]     │
│  [AnalyticsService] [ShippingService] [AuditService]       │
└────────────────────────────────────────────────────────────┘
```

---

# Workflow

**Scenario: A customer places an order on an e-commerce platform.**

```
STEP 1 — Customer Sends Request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /orders { userId: 123, productId: 456, qty: 2 }
  ↓
OrderService receives the request

STEP 2 — OrderService Processes and Publishes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderService:
  → Validates the request
  → Saves order to its database
  → Publishes event to Kafka:
      Topic: "orders.created"
      Payload: { orderId: 789, userId: 123, productId: 456, qty: 2, total: 99.99 }
  → Returns 201 Created to the customer (FAST — no waiting)

STEP 3 — Broker Stores and Distributes
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Kafka receives the event.
Stores it to disk (durable).
Notifies all subscribers of "orders.created".

STEP 4 — Consumers React Independently (Async)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NotificationService reads event:
  → Sends order confirmation email to user 123

InventoryService reads event:
  → Decrements stock for product 456 by 2

LoyaltyService reads event:
  → Adds 99 loyalty points to user 123's account

AnalyticsService reads event:
  → Records the purchase for reporting

ShippingService reads event:
  → Creates a shipping job for order 789

ALL of these happen concurrently and independently.
OrderService knows nothing about any of them.
If any consumer is temporarily down → events queue up → processed on recovery.

STEP 5 — Chain of Events (Optional)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ShippingService creates a shipment → publishes "shipment.created" event
  → NotificationService sends "Your order has shipped!" email
  → AuditService logs the shipment
```

---

# Real World Examples

## Example 1: The Newspaper Subscription

```
Event = News Story Published
Producer = Newspaper Editorial Team
Broker = Distribution Network
Consumers = Subscribers (Home Delivery, Newsstand, Digital Readers)

Each subscriber gets the same story.
Each does something different with it.
The editor doesn't call each subscriber — they just publish.
```

## Example 2: Uber — Real-Time Ride Events

```
Events in Uber's system:
  driver.location.updated → every 5 seconds per driver
                         → MapService updates map
                         → DispatchService updates ETAs
                         → RiderService shows real-time position

  ride.completed         → PaymentService charges the fare
                         → DriverService updates earnings
                         → RatingService prompts both parties for rating
                         → AnalyticsService records trip data

Volume: Millions of location events per minute — only Kafka can handle this.
```

## Example 3: LinkedIn

LinkedIn uses Kafka for **7 trillion events per day**:

```
Events flowing through LinkedIn's Kafka:
  → Profile views
  → Post impressions
  → Connection requests accepted
  → Job application events
  → Message reads

Each event triggers:
  → Feed algorithm updates
  → Recommendation model updates
  → Analytics dashboards
  → Notification triggers
```

## Example 4: E-Commerce Order Pipeline

```
order.placed
  ↓ [Kafka]
  ├→ inventory.reserved    (InventoryService reacts)
  │    ↓ [Kafka]
  │    └→ shipment.created  (ShippingService reacts)
  │         ↓ [Kafka]
  │         └→ shipment.dispatched (triggers delivery updates)
  │
  ├→ payment.charged       (PaymentService reacts)
  │    ↓ [Kafka]
  │    └→ receipt.sent     (NotifyService sends email)
  │
  └→ loyalty.updated       (LoyaltyService reacts)

Events trigger events. The system flows forward automatically.
```

---

# Implementation

## Setting Up Kafka (Docker)

```yaml
# docker-compose.yml
services:
  kafka:
    image: confluentinc/cp-kafka:latest
    ports: ["9092:9092"]
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092

  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    ports: ["2181:2181"]
```

## Producer (Publishing Events — Node.js)

```javascript
// order-service/src/eventPublisher.js

const { Kafka } = require('kafkajs');

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const producer = kafka.producer();

async function publishEvent(topic, event) {
    await producer.connect();
    await producer.send({
        topic,
        messages: [{
            key: String(event.orderId),       // Key for partitioning
            value: JSON.stringify({
                eventId: `evt-${Date.now()}`,
                eventType: topic,
                timestamp: new Date().toISOString(),
                version: '1.0',
                data: event
            })
        }]
    });
    console.log(`Event published to ${topic}:`, event);
}

module.exports = { publishEvent };
```

```javascript
// order-service/src/orderService.js

const { publishEvent } = require('./eventPublisher');
const orderRepository = require('./orderRepository');

async function createOrder(orderData) {
    // 1. Save the order
    const order = await orderRepository.save(orderData);

    // 2. Publish the event (fire and forget — async)
    await publishEvent('orders.created', {
        orderId: order.id,
        userId: order.userId,
        productId: order.productId,
        quantity: order.quantity,
        total: order.total,
    });

    // 3. Return immediately — consumers react asynchronously
    return order;
}
```

## Consumer (Listening to Events — Node.js)

```javascript
// notification-service/src/eventListener.js

const { Kafka } = require('kafkajs');
const emailService = require('./emailService');

const kafka = new Kafka({ brokers: ['localhost:9092'] });
const consumer = kafka.consumer({ groupId: 'notification-service' });

async function startListening() {
    await consumer.connect();
    await consumer.subscribe({ topic: 'orders.created' });

    await consumer.run({
        eachMessage: async ({ topic, partition, message }) => {
            const event = JSON.parse(message.value.toString());
            console.log('Received event:', event.eventType);

            // React to the event
            if (event.eventType === 'orders.created') {
                await emailService.sendOrderConfirmation({
                    userId: event.data.userId,
                    orderId: event.data.orderId,
                    total: event.data.total,
                });
            }
        }
    });
}

startListening();
```

## RabbitMQ Alternative (Simpler Queue)

```javascript
// Simple RabbitMQ publisher
const amqp = require('amqplib');

async function publish(queue, message) {
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    await channel.assertQueue(queue, { durable: true });
    channel.sendToQueue(queue, Buffer.from(JSON.stringify(message)));
}

// Simple RabbitMQ consumer
async function consume(queue, handler) {
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    await channel.assertQueue(queue, { durable: true });
    channel.consume(queue, (msg) => {
        if (msg) {
            handler(JSON.parse(msg.content.toString()));
            channel.ack(msg); // Acknowledge processing
        }
    });
}
```

---

# Best Practices

## ✅ DO

| Practice | Why |
|---|---|
| **Name events in past tense** | "order.created" not "create.order" — events are facts, not commands |
| **Make events self-contained** | Include all necessary data in the event payload — consumers shouldn't need to call back |
| **Use event versioning** | "order.created.v1" and "order.created.v2" — evolve without breaking consumers |
| **Implement dead letter queues (DLQ)** | Failed-to-process events go here for investigation — nothing is silently lost |
| **Idempotent consumers** | Consumers should handle receiving the same event twice without side effects |
| **Use message acknowledgment** | Consumer only ACKs after successfully processing — guarantees at-least-once delivery |
| **Monitor queue depth** | A growing queue = consumer is falling behind — alert and scale |
| **Log the event trail** | A record of all events = a natural audit log |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| **Use events for commands** | "send.email" is a command → tightly couples producer to outcome |
| **Assume exactly-once delivery** | Most brokers guarantee at-least-once — design for duplicates |
| **Skip error handling in consumers** | Failed messages lost silently → data inconsistency |
| **Make events too large** | Don't put entire database records in events — use reference IDs |
| **Ignore ordering guarantees** | Kafka guarantees order within a partition; don't assume global order |
| **No monitoring on broker** | Kafka/RabbitMQ clusters can degrade silently — always monitor |

---

# Industry Standards

## Event Schema Standards

```
CloudEvents (CNCF Standard):
  The Cloud Native Computing Foundation defined a standard event format:
  
{
  "specversion": "1.0",
  "type": "com.myshop.order.created",
  "source": "https://myshop.com/orders",
  "id": "evt-8821",
  "time": "2024-06-19T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "orderId": 789,
    "total": 99.99
  }
}

Used by: Google Cloud Pub/Sub, Azure Event Grid, Knative
```

## Broker Comparison

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│    Kafka     │   RabbitMQ   │   AWS SQS    │  Redis Pub   │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ High volume  │ Moderate vol │ Managed/easy │ Very fast    │
│ Stream log   │ Flexible     │ Cloud-native │ No persist.  │
│ Replay       │ No replay    │ No replay    │ No replay    │
│ Persistent   │ Persistent   │ Managed      │ In-memory    │
│ Complex setup│ Medium setup │ Easy setup   │ Simple setup │
│ LinkedIn,    │ Internal     │ AWS-native   │ Simple apps  │
│ Netflix,Uber │ enterprise   │ apps         │              │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

## Patterns in Production

```
Outbox Pattern:
  Problem: OrderService saves to DB and publishes event
           → What if the publish fails after the DB save?
           → Order saved but no event → downstream systems unaware
  
  Solution: Save the event to an "outbox" table in the same transaction.
            A background job reads the outbox and publishes to Kafka.
            Guarantees: if order is saved, event will eventually be published.

Saga Pattern:
  Problem: Order requires: reserve inventory + charge payment + create shipment
           All must succeed or all must roll back.
           But they're in separate services with separate databases.
  
  Solution: Choreographed saga — each step publishes an event.
            If payment fails → publish "payment.failed" event
            InventoryService listens → publishes "inventory.released" event
            Compensating transactions undo the previous steps.
```

---

# Common Mistakes

## Mistake 1: Commands Masquerading as Events

```
❌ Wrong:
  OrderService publishes: "send.welcome.email"
  NotificationService is the only possible consumer.
  
  This is a point-to-point command in event clothing.
  You've coupled OrderService to NotificationService.

✅ Right:
  OrderService publishes: "user.registered"
  NotificationService CHOOSES to listen and send a welcome email.
  Tomorrow, AnotherService can also listen — zero changes to OrderService.
```

## Mistake 2: Not Handling Duplicate Events

```
Brokers guarantee "at least once" delivery.
The same event CAN be delivered twice (network retry, consumer restart).

❌ If NotificationService sends email on every "order.created" event:
  Event delivered twice → user receives 2 confirmation emails.

✅ Idempotent Consumer:
  NotificationService checks: "Did I already process event evt-8821?"
  If yes → skip. If no → process and record that I did.

Implementation: Store processed event IDs in Redis or a DB.
```

## Mistake 3: Ignoring the Dead Letter Queue

```
❌ No DLQ:
  Event fails to process → message lost → nobody knows
  Database inconsistency silently accumulates

✅ With DLQ:
  Event fails to process → goes to Dead Letter Queue
  Alert fires → team investigates → event replayed after fix
```

## Mistake 4: Massive Event Payloads

```
❌ Wrong:
  "order.created" event contains:
    → Full user profile (50 fields)
    → Full product details (100 fields)
    → Historical orders (1000 records)
    → Large binary data
  Event size: 5MB

  Kafka performance degrades. Consumers receive huge blobs.
  Schema changes break everything.

✅ Right:
  "order.created" event contains:
    → orderId, userId, productId, total, timestamp
  Event size: 200 bytes

  Consumers that need more data → call the respective service.
```

---

# Security & Performance Considerations

## Security

```
EDA Security Considerations
│
├── Broker Security
│   ├── Authentication → SASL/SCRAM for Kafka
│   ├── Encryption → TLS for all broker communication
│   └── Authorization → ACLs: which services can publish/consume which topics
│
├── Event Data Security
│   ├── Don't include sensitive data in events (PII, card numbers)
│   ├── Use reference IDs → consumers call secure APIs for sensitive data
│   └── Encrypt sensitive fields if they must appear in events
│
└── Consumer Security
    ├── Validate event schema before processing
    ├── Don't trust event data blindly (could be tampered)
    └── Idempotency protects against replay attacks
```

## Performance

```
EDA Performance Characteristics
│
├── Kafka Throughput
│   → Handles millions of events per second per partition
│   → LinkedIn: 7 trillion events/day on Kafka
│   → Key: partition by entity ID (e.g., userId) for ordering
│
├── Consumer Group Scaling
│   → Add more consumer instances to a group → Kafka distributes partitions
│   → Scale consumers independently of producers
│
├── Backpressure Handling
│   → If consumers are slow → events accumulate in the queue
│   → Queue acts as a buffer (absorbs traffic spikes)
│   → Alert when queue depth grows (consumer falling behind)
│
└── Latency Considerations
    → Async events = eventual consistency (not immediate)
    → For truly real-time requirements → consider synchronous calls instead
    → Typical Kafka end-to-end latency: 5-20ms
```

---

# Related Technologies

| Technology | Relationship |
|---|---|
| **Apache Kafka** | The most popular event streaming platform — backbone of EDA at scale |
| **RabbitMQ** | Traditional message broker for EDA — flexible routing, point-to-point or pub/sub |
| **AWS SQS / SNS** | Cloud-managed message queue and pub/sub for AWS-native EDA |
| **Azure Service Bus** | Microsoft's managed message broker for EDA |
| **Redis Pub/Sub** | Lightweight in-memory pub/sub — fast but no persistence |
| **Microservices Architecture** | EDA is the async communication pattern between microservices |
| **CQRS** | Command Query Responsibility Segregation — often paired with EDA |
| **Event Sourcing** | Store state as a sequence of events — related to EDA |
| **Saga Pattern** | Managing distributed transactions via event choreography |
| **Outbox Pattern** | Guaranteeing event delivery even if the broker is temporarily unavailable |
| **CloudEvents (CNCF)** | The standard specification for event format and metadata |
| **Serverless (AWS Lambda)** | Lambda functions are a natural consumer for events (e.g., S3 events trigger Lambda) |

---

# Summary

## What We Learned

- **Event-Driven Architecture (EDA)** is a pattern where components communicate by **producing and consuming events** through a **message broker** — rather than calling each other directly.
- An **event** is an immutable record of something that happened (past tense) — "order.created", not "create.order".
- **Producers** publish events. **Consumers** subscribe and react. Neither knows about the other.
- A **message broker** (Kafka, RabbitMQ) stores and delivers events reliably.
- EDA provides: **loose coupling**, **resilience**, **scalability**, **easy extensibility**, and **natural audit trails**.
- Kafka is the dominant choice for high-volume event streaming (LinkedIn: 7 trillion events/day).
- **Idempotent consumers** are essential — events can be delivered more than once.
- The **Outbox Pattern** guarantees events are published even if the broker is temporarily unavailable.
- The **Saga Pattern** manages distributed transactions across multiple services via event choreography.

## Key Takeaways

- Think of EDA like a newspaper: the publisher doesn't call each subscriber — they just print and distribute.
- Events are facts, not commands. "order.created" not "send.email".
- The queue is your shock absorber — it absorbs traffic spikes so consumers don't crash.
- Make consumers idempotent — assume every event will arrive at least twice.
- EDA + Microservices = the dominant pattern for large-scale distributed systems today.

---

# Keywords

- Event-Driven Architecture (EDA)
- Event
- Producer / Publisher
- Consumer / Subscriber
- Message Broker
- Apache Kafka
- RabbitMQ
- Pub/Sub
- Message Queue
- Dead Letter Queue (DLQ)
- Idempotent Consumer
- Event Streaming
- Outbox Pattern
- Saga Pattern
- CQRS
- Event Sourcing
- CloudEvents
- Backpressure

---

# Glossary

| Term | Meaning |
|---|---|
| **Event** | An immutable record that something happened — e.g., "order.created" with timestamp and data |
| **Producer** | A service that creates and publishes events to a message broker |
| **Consumer** | A service that subscribes to topics and reacts to events |
| **Message Broker** | Middleware (Kafka, RabbitMQ) that receives, stores, and delivers events between producers and consumers |
| **Topic** | A named channel in Kafka to which events are published and from which consumers read |
| **Pub/Sub** | Publish-Subscribe pattern — one event published, all subscribers receive a copy |
| **Dead Letter Queue (DLQ)** | A special queue where failed-to-process messages are sent for investigation |
| **Idempotent** | An operation that can be performed multiple times with the same result — essential for at-least-once delivery |
| **Backpressure** | When consumers can't keep up with producers — events accumulate in the queue |
| **Event Streaming** | Treating events as a continuous, ordered log (Kafka model) — events are stored and can be replayed |
| **Outbox Pattern** | Save the event to a database "outbox" table atomically, then publish to broker asynchronously |
| **Saga Pattern** | A sequence of local transactions coordinated via events — with compensating transactions for rollback |
| **CQRS** | Command Query Responsibility Segregation — separate write and read models, often used with EDA |
| **Event Sourcing** | Storing application state as a sequence of events rather than current state |
| **CloudEvents** | CNCF standard specification for event format and metadata |
| **Consumer Group** | A group of consumer instances in Kafka that share the load of processing a topic |

## Next Recommended Chapters

- [08. Clean Architecture](./08-Clean-Architecture.md)
- [06. Microservices Architecture](./06-Microservices-Architecture.md)
- [12. Architecture Patterns](./12-Architecture-Patterns.md)
