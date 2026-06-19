> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine **Amazon's fulfillment operation** — not the software, but the physical warehouses.

A warehouse doesn't have one person doing everything. It has:
- A **receiving team** that accepts new stock
- A **sorting team** that organizes products
- A **picking team** that grabs items for orders
- A **packing team** that boxes them
- A **shipping team** that dispatches them
- A **returns team** that handles coming-back items

Each team:
- Works **independently** — the packing team doesn't wait for the receiving team to finish before starting
- Can **scale separately** — during peak season, you hire 10× more pickers but not more sorters
- Has their **own tools and processes** — pickers use scanners, packers use tape machines
- Can be **replaced** — if a new packing machine arrives, you change only the packing operation

That is **Microservices Architecture**.

In software, Microservices Architecture is a design approach where an application is built as a collection of **small, independent, autonomous services** — each running in its own process, owning its own data, deployable on its own, and communicating with other services over a network.

The word "micro" means **small and focused**. Each service does exactly one thing and does it extremely well.

Microservices is the architecture that powers **Netflix, Uber, Amazon, Airbnb, Twitter, LinkedIn, Spotify** — companies serving hundreds of millions of users globally.

---

# Why It Exists

### The Monolith Scaling Problem

As companies grew, their monolithic applications hit a wall:

```
Problem 1: All-or-Nothing Deployment
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Changing one feature → must redeploy the entire application.
A bug in the payment module → the entire site goes down.
Deployment time: 4 hours. Frequency: once a week, terrified.

Problem 2: Scaling the Whole Thing
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
The video encoding service is under heavy load.
But you can only add more servers for the WHOLE monolith.
You can't scale just the encoding — you scale everything.

Problem 3: Team Coordination
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
50 developers all modifying the same codebase.
Merge conflicts every day.
Team A waits for Team B to finish their feature before deploying.
One team's bug blocks everyone else.

Problem 4: Technology Lock-in
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
The entire monolith is Node.js.
The ML recommendation engine needs Python.
Impossible to introduce without rewriting everything.
```

Microservices emerged as the solution:

> "What if each **team** owned a small **service**, deployed it **independently**, and teams never blocked each other?"

---

# Problem It Solves

**Before — Single Monolith for Everything:**

```
One codebase. One deployment. One database.

Change payment feature → redeploy everything.
Checkout service busy → scale the entire monolith.
Python needed for ML → impossible, we're all Node.js.
Team B broke tests → Team A can't deploy their fix.
```

**After — Microservices:**

```
Each service = its own codebase, deployment, and database.

Change payment feature → only the Payment Service redeploys.
Checkout service busy → scale ONLY the Checkout Service.
ML model needs Python → Recommendation Service is in Python.
Team B broke their service → Team A's service is unaffected.
```

**Side by Side:**

```
MONOLITH                        MICROSERVICES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━    ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
┌─────────────────────────┐     ┌────────┐ ┌────────┐ ┌─────────┐
│  User + Order + Product │     │  User  │ │ Order  │ │ Product │
│  + Payment + Search +   │     │Service │ │Service │ │ Service │
│  + Notifications + ML   │     └────────┘ └────────┘ └─────────┘
│  + Reports + Admin...   │     ┌────────┐ ┌────────┐ ┌─────────┐
└─────────────────────────┘     │Payment │ │ Search │ │ Notify  │
Deploy: 4 hours                 │Service │ │Service │ │ Service │
Scale: all or nothing           └────────┘ └────────┘ └─────────┘
Team: 50 devs blocking          Deploy: 5 min each, independently
Language: one only              Scale: per service
                                Team: 6-person squads, autonomous
                                Language: best tool for each job
```

---

# Core Concepts

## 1. Single Responsibility per Service

Every microservice has **one clear business purpose**:

```
✅ Good Microservices:
  UserService      → manages user accounts and profiles
  OrderService     → handles order creation and lifecycle
  PaymentService   → processes payments and refunds
  SearchService    → handles product search and indexing
  NotifyService    → sends emails, SMS, push notifications

❌ Bad Microservices:
  UserOrderPaymentService → too much responsibility
  DataService             → too vague
  HelperService           → meaningless name and purpose
```

## 2. Independent Deployment

Each microservice can be deployed, updated, or rolled back **without affecting any other service**:

```
Team A deploys UserService v2.1      → Payment Service still at v1.4
Team B deploys PaymentService v3.0   → UserService unaware of this
Team C rolls back OrderService       → Everything else unaffected
```

## 3. Decentralized Data (Database Per Service)

Each microservice owns its own database. No sharing.

```
UserService         → PostgreSQL (relational, good for structured user data)
OrderService        → PostgreSQL (transactional order records)
ProductService      → MongoDB (flexible product catalog)
RecommendationSvc   → Redis + custom ML store
SearchService       → Elasticsearch (full-text search)
```

Why? Because if services share a database:
- Schema change in one service breaks other services
- One service's heavy query slows down other services
- Services become tightly coupled through the database

## 4. API Communication

Services talk to each other via **network APIs**:

```
Option A: Synchronous (REST / gRPC)
  OrderService → HTTP GET → UserService/users/123
  Response: user data returned immediately

Option B: Asynchronous (Message Queue)
  OrderService → publishes "order.created" event to Kafka
  NotifyService → subscribes, sends email when it receives event
  No direct dependency between them
```

## 5. Fault Isolation

When one service fails, others continue working:

```
❌ Monolith: Payment service has a memory leak → whole site crashes
✅ Microservices: Payment service has a memory leak → 
     UserService, ProductService, SearchService all still work
     Users can browse and add to cart, just can't checkout
```

---

# Architecture / Components

## The Full Microservices Stack

```
                     CLIENTS
          ┌───────────────────────────┐
          │  Web  │  Mobile  │  API   │
          └────────────────┬──────────┘
                           │
                           ▼
          ┌────────────────────────────┐
          │        API GATEWAY         │
          │  Authentication │ Routing  │
          │  Rate Limiting  │ Logging  │
          └──────┬──────────────┬──────┘
                 │              │
        ┌────────▼──┐     ┌─────▼──────┐
        │   User    │     │   Order    │
        │  Service  │     │  Service   │
        │  [DB: PG] │     │  [DB: PG]  │
        └────────┬──┘     └──────┬─────┘
                 │               │
        ┌────────▼──┐     ┌──────▼──────┐
        │ Product   │     │  Payment    │
        │  Service  │     │  Service    │
        │ [DB:Mongo]│     │  [DB: PG]  │
        └────────┬──┘     └──────┬─────┘
                 │               │
                 ▼               ▼
          ┌──────────────────────────┐
          │     MESSAGE BROKER       │
          │     (Kafka / RabbitMQ)   │
          └──────────────┬───────────┘
                         │
               ┌─────────▼────────┐
               │  NotifyService   │
               │  (email/sms)     │
               └──────────────────┘

Supporting Infrastructure:
  ┌────────────────────────────────────┐
  │  Service Registry  │ Load Balancer │
  │  Config Server     │ Circuit Breaker│
  │  Distributed Trace │ Health Monitor │
  └────────────────────────────────────┘
```

## Key Components Explained

### API Gateway

The **single entry point** for all external clients:

```
Client → API Gateway → routes to correct service

Gateway Responsibilities:
  ├── Authentication (verify JWT token)
  ├── Authorization (does this user have permission?)
  ├── Rate Limiting (100 requests/minute per user)
  ├── Request Routing (POST /orders → OrderService)
  ├── SSL Termination
  ├── Request Logging
  └── Response Caching (optional)

Examples: Kong, Nginx, AWS API Gateway, Traefik
```

### Service Discovery

Services find each other via a **registry**:

```
Service Registry (e.g., Consul, Kubernetes DNS):
  UserService     → running at 10.0.0.1:8001 (3 instances)
  OrderService    → running at 10.0.0.2:8002 (5 instances)
  PaymentService  → running at 10.0.0.3:8003 (2 instances)

OrderService needs to call UserService:
  1. OrderService queries the registry: "Where is UserService?"
  2. Registry returns: ["10.0.0.1:8001", "10.0.0.1:8011", "10.0.0.1:8021"]
  3. OrderService picks one (load balancing)
  4. Makes the HTTP call
```

### Circuit Breaker

Prevents cascading failures:

```
Normal State:
  OrderService → PaymentService → works fine

PaymentService becomes slow/unavailable:
  Circuit Breaker OPENS after N failures:
  OrderService → Circuit Breaker → STOPS calling PaymentService
  Returns fallback response immediately
  
After recovery period:
  Circuit Breaker → probes PaymentService
  If healthy → CLOSES → normal operation resumes

Tools: Hystrix (Java), Resilience4j, Polly (.NET)
```

---

# Workflow

**Scenario: User orders a product on an e-commerce platform (microservices).**

```
STEP 1 — Client Sends Request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Browser: POST /orders { userId, productId, qty, paymentCard }

STEP 2 — API Gateway
━━━━━━━━━━━━━━━━━━━━
  → Validates JWT token (Auth check)
  → Rate limit OK
  → Routes to: OrderService

STEP 3 — OrderService (Synchronous calls via REST)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  → GET /users/123     → UserService  (verify user exists)
  → GET /products/456  → ProductService (check stock)
  → POST /payments     → PaymentService (charge card)

(Each call goes service-to-service over HTTP)

STEP 4 — OrderService Saves the Order
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  → INSERT into OrderService's own database
  → Returns orderId = 789

STEP 5 — OrderService Publishes an Event (Async)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  → Publishes "order.created" event to Kafka

STEP 6 — Downstream Services React (Asynchronous)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  NotificationService:  sends order confirmation email
  InventoryService:     decrements stock count
  AnalyticsService:     records the purchase event
  RecommendationSvc:    updates user purchase history for ML model

STEP 7 — Response to Client
━━━━━━━━━━━━━━━━━━━━━━━━━━━
  API Gateway returns: 201 Created { orderId: 789 }
  (Steps 6 happen asynchronously — user doesn't wait)
```

---

# Real World Examples

## Netflix

```
Scale: 250 million+ subscribers. 700+ microservices.

Key Services:
  UserService         → account management
  StreamingService    → video delivery
  RecommendationSvc   → ML-based "What to Watch Next"
  BillingService      → subscription payments
  SearchService       → title search
  EncodeService       → video transcoding into 20+ formats

Why Microservices?
  → Video encoding needs GPU-optimized servers
  → Recommendations need Python/ML infrastructure
  → Billing needs PCI-compliant isolated environment
  → Each can scale 100× independently during new releases
```

## Uber

```
Scale: 5+ million trips/day. 1000+ microservices.

Key Services:
  RiderService     → rider app, profile
  DriverService    → driver app, matching
  DispatchService  → match rider to nearest driver
  PricingService   → dynamic surge pricing
  MapService       → routing, ETAs
  PaymentService   → fare processing
  NotifyService    → SMS/push notifications

Why Microservices?
  → Dispatch needs real-time geo-spatial algorithms
  → Pricing needs ML for surge calculation
  → Each city can have different configurations
```

## Amazon

```
Amazon's 2002 Jeff Bezos API Mandate forced service separation.

Today:
  Every feature on Amazon.com is a separate service:
  SearchService, CartService, RecommendationService,
  ReviewService, PriceService, ShippingService, ...

Rule: "Two-Pizza Teams" — each service owned by a team small
      enough to be fed with two pizzas (~6-8 people).
      Team owns the service end-to-end: build, deploy, operate.
```

---

# Implementation

## Project Structure

```
services/
│
├── user-service/
│   ├── src/
│   │   ├── routes/
│   │   ├── services/
│   │   └── repositories/
│   ├── Dockerfile
│   ├── package.json
│   └── .env
│
├── order-service/
│   ├── src/
│   ├── Dockerfile
│   └── package.json
│
├── payment-service/
│   ├── src/
│   ├── Dockerfile
│   └── package.json
│
└── api-gateway/
    ├── nginx.conf
    └── Dockerfile
```

## Service Code Example (Order Service)

```javascript
// order-service/src/app.js

const express = require('express');
const axios = require('axios');
const { publishEvent } = require('./eventBus');

const app = express();
app.use(express.json());

app.post('/orders', async (req, res) => {
    const { userId, productId, quantity } = req.body;

    // Call UserService over HTTP (Service-to-Service)
    const userRes = await axios.get(
        `http://user-service:8001/users/${userId}`
    );
    if (!userRes.data) return res.status(404).json({ error: 'User not found' });

    // Call ProductService over HTTP
    const productRes = await axios.get(
        `http://product-service:8002/products/${productId}`
    );
    if (productRes.data.stock < quantity) {
        return res.status(400).json({ error: 'Insufficient stock' });
    }

    // Call PaymentService over HTTP
    await axios.post('http://payment-service:8003/charges', {
        userId, amount: productRes.data.price * quantity
    });

    // Save to THIS service's OWN database
    const order = await db.query(
        'INSERT INTO orders (user_id, product_id, qty) VALUES ($1, $2, $3) RETURNING *',
        [userId, productId, quantity]
    );

    // Publish async event (fire and forget)
    await publishEvent('order.created', order.rows[0]);

    res.status(201).json(order.rows[0]);
});

app.listen(8004, () => console.log('Order Service on port 8004'));
```

## Docker Compose (Run All Services)

```yaml
# docker-compose.yml

version: '3.8'
services:

  user-service:
    build: ./user-service
    ports: ["8001:8001"]
    environment:
      DB_URL: postgres://user-db/users
    depends_on: [user-db]

  order-service:
    build: ./order-service
    ports: ["8004:8004"]
    environment:
      DB_URL: postgres://order-db/orders
    depends_on: [order-db, kafka]

  payment-service:
    build: ./payment-service
    ports: ["8003:8003"]

  api-gateway:
    image: nginx:alpine
    ports: ["80:80"]
    volumes: ["./nginx.conf:/etc/nginx/conf.d/default.conf"]

  user-db:
    image: postgres:15
    environment: { POSTGRES_DB: users }

  order-db:
    image: postgres:15
    environment: { POSTGRES_DB: orders }

  kafka:
    image: confluentinc/cp-kafka:latest
```

---

# Best Practices

## ✅ DO

| Practice | Why |
|---|---|
| **One service per business capability** | UserService, OrderService — not too small, not too large |
| **Database per service** | Prevents coupling through shared schema |
| **Communicate async when possible** | Events decouple services; services don't block each other |
| **Design for failure** | Every network call can fail — use retries, timeouts, circuit breakers |
| **Implement health endpoints** | `GET /health` on each service for load balancers and orchestrators |
| **Use centralized logging and tracing** | Trace a request across 10 services — you NEED distributed tracing |
| **Keep services small enough for one team** | A service owned by two teams becomes nobody's responsibility |
| **Version your APIs** | `/v1/orders`, `/v2/orders` — never break existing consumers |
| **Use an API gateway for external traffic** | Single entry point, central auth, rate limiting |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| **Start with microservices on day 1** | Over-engineering; complexity without benefit |
| **Share a database across services** | Coupling through data; defeats the purpose |
| **Synchronous calls for everything** | Services form a chain — one slow service blocks them all |
| **No distributed tracing** | A request fails and you have no idea which of 10 services caused it |
| **Ignore the network** | Services communicate over a network — it can fail, be slow, be lossy |
| **Too many tiny services** | 200 services for a team of 10 = unmanageable |
| **No circuit breakers** | One failing service takes down the entire chain via cascading failure |

---

# Industry Standards

## Key Patterns and Tools

```
Communication:
  REST/HTTP          → Most common inter-service communication
  gRPC               → High-performance binary protocol (Google-created)
  GraphQL Federation → When frontend needs data from multiple services
  Apache Kafka       → Async event streaming between services
  RabbitMQ           → Async message queuing

Service Mesh (Advanced):
  Istio / Linkerd    → Handles retries, circuit breaking, mTLS automatically
  → A service mesh adds a "sidecar proxy" next to each service
  → The proxy handles all network concerns automatically

Container Orchestration:
  Kubernetes          → Industry standard for running/scaling microservices
  Docker Compose      → For local development

API Gateway:
  Kong               → Open-source, popular
  AWS API Gateway    → Fully managed on AWS
  Nginx              → Simple, high-performance proxy

Observability (see, understand, debug):
  Jaeger / Zipkin    → Distributed tracing (trace requests across services)
  Prometheus + Grafana → Metrics and dashboards
  ELK Stack          → Centralized logging (Elasticsearch, Logstash, Kibana)
```

## The Scale-to-Services Journey

```
Stage 1: Startup / Early Product
  → Monolith. Fast. Simple. Ship it.
  → "Majestic Monolith" era

Stage 2: Growing Product, Growing Team
  → Modular Monolith. Clean up internally.
  → Team boundaries emerging.

Stage 3: Multiple Teams, Clear Pain
  → Extract first microservice (usually the high-load or specialized one)
  → e.g., Extract NotificationService (no business risk)
  → Then PaymentService (compliance isolation)

Stage 4: Full Microservices
  → When the organizational structure demands it
  → Conway's Law: "A system's design mirrors its organization's structure"
```

---

# Common Mistakes

## Mistake 1: Nanoservices (Too Fine-Grained)

```
❌ Wrong:
  GetUserFirstNameService
  GetUserLastNameService
  GetUserEmailService
  UpdateUserEmailService

These are methods, not services.
100+ of these = impossible to manage.

✅ Right:
  UserService (handles all user operations)
```

## Mistake 2: The Distributed Monolith

```
❌ Services that can't deploy independently:
  PaymentService MUST deploy before OrderService.
  OrderService DIRECTLY reads from UserService's database.
  All services must deploy together every time.

This is not microservices. It's a monolith split across the network.
You get all the complexity of microservices, none of the benefits.
```

## Mistake 3: No Distributed Tracing

```
❌ Without tracing:
  User reports: "My order failed."
  Developer: Checks OrderService logs... nothing obvious.
  Checks UserService logs... looks fine.
  Checks PaymentService logs... hard to correlate.
  Checks NotifyService logs... etc.
  → Spends 4 hours debugging what took 200ms to fail.

✅ With Jaeger/Zipkin:
  Open the trace dashboard.
  See the full request journey: OrderService → UserService → PaymentService.
  See: PaymentService call timed out at 2000ms.
  Fixed in 5 minutes.
```

## Mistake 4: Synchronous Chains

```
❌ Anti-Pattern: Synchronous Chain
  API → Service A → Service B → Service C → Service D

If Service D is slow:
  → C waits
  → B waits
  → A waits
  → User waits
  → Everything blocked

✅ Better: Break the chain with async events where possible
  API → Service A (saves, publishes event)
  Service B, C, D react asynchronously to the event
```

---

# Security & Performance Considerations

## Security

```
Microservices Security Challenges
│
├── Service-to-Service Authentication
│   → Services must authenticate each other, not just clients
│   → Use: mTLS (mutual TLS), JWT tokens, API keys
│   → Service Mesh (Istio) can automate mTLS between services
│
├── API Gateway as Security Gate
│   → All external requests authenticated here
│   → Rate limiting prevents DDoS
│   → WAF (Web Application Firewall) at the gateway
│
├── Secrets Management
│   → Each service has its own DB credentials, API keys
│   → Use: HashiCorp Vault, AWS Secrets Manager, Kubernetes Secrets
│   → Never hardcode credentials
│
└── Blast Radius Containment
    → A compromised UserService shouldn't be able to talk to
      PaymentService — use network policies (Kubernetes Network Policy)
    → Least privilege: each service has minimum required permissions
```

## Performance

```
Microservices Performance Considerations
│
├── Network Latency is Real
│   → What was a local function call is now an HTTP request
│   → Add 1-10ms per service call in the same datacenter
│   → Minimize synchronous cross-service calls in hot paths
│
├── Use Caching Strategically
│   → Redis in front of frequently called services
│   → Cache user profiles, product data, configuration
│
├── Prefer Async Communication
│   → Kafka/RabbitMQ: decouple services, absorb traffic spikes
│
├── Connection Pooling
│   → Each service maintains a pool of DB connections
│   → With 200 services × 10 connections each = 2000 DB connections
│   → Use PgBouncer or similar connection pooler
│
└── Scale Individually
    → Identify hot services (search, checkout)
    → Scale those specifically with Kubernetes HPA
    → (Horizontal Pod Autoscaler)
```

---

# Related Technologies

| Technology | Relationship |
|---|---|
| **Docker** | Each microservice runs in its own container |
| **Kubernetes** | Orchestrates, scales, and manages microservice containers |
| **API Gateway (Kong, Nginx)** | Single entry point that routes to microservices |
| **Apache Kafka** | Async event streaming backbone between services |
| **RabbitMQ** | Message queue for async inter-service communication |
| **gRPC** | High-performance binary protocol for service-to-service calls |
| **Istio / Linkerd** | Service mesh that handles mTLS, retries, circuit breaking automatically |
| **Consul / Eureka** | Service discovery and registry tools |
| **Jaeger / Zipkin** | Distributed tracing for following requests across services |
| **Prometheus + Grafana** | Metrics collection and dashboard visualization |
| **SOA** | The predecessor — microservices = SOA without the ESB |
| **Event-Driven Architecture** | The async communication pattern used between microservices |
| **Domain-Driven Design (DDD)** | The bounded context concept directly maps to microservice boundaries |

---

# Summary

## What We Learned

- **Microservices Architecture** is a design where an application is built as **many small, independent services** — each with one responsibility, its own database, and its own deployment pipeline.
- Each service can be **deployed independently** — no coordinated releases.
- Services each own their **own database** — no sharing.
- Services communicate via **REST APIs** (synchronous) or **message brokers** (asynchronous).
- An **API Gateway** is the single entry point for external clients.
- **Service Discovery** lets services find each other dynamically.
- **Circuit Breakers** prevent cascading failures.
- **Distributed Tracing** (Jaeger/Zipkin) is essential to debug cross-service issues.
- Companies like Netflix, Amazon, Uber, and Spotify run on hundreds to thousands of microservices.
- The biggest mistake: starting with microservices from day one, or building a Distributed Monolith.

## Key Takeaways

- Think of microservices like Amazon's fulfillment warehouse: specialized teams, independently operable, scaling only what's busy.
- The shift from monolith to microservices is organizational, not just technical — teams must be small and autonomous.
- Async is better than sync where possible — it breaks chains and absorbs spikes.
- Observability is not optional — distributed tracing, metrics, and centralized logging are mandatory.
- Follow Conway's Law: design your services to mirror your team structure.

---

# Keywords

- Microservices Architecture
- Service Independence
- Database Per Service
- API Gateway
- Service Discovery
- Circuit Breaker
- Service Mesh
- Event-Driven Communication
- Fault Isolation
- Distributed Tracing
- Conway's Law
- Two-Pizza Team
- gRPC
- Kafka
- Kubernetes
- Nanoservices Anti-Pattern
- Distributed Monolith Anti-Pattern

---

# Glossary

| Term | Meaning |
|---|---|
| **Microservice** | A small, independently deployable service that handles one business capability |
| **API Gateway** | The single entry point for external clients; handles routing, auth, rate limiting |
| **Service Discovery** | A mechanism (registry) that lets services find each other's network addresses dynamically |
| **Circuit Breaker** | A pattern that stops calling a failing service and returns a fallback to prevent cascading failures |
| **Service Mesh** | Infrastructure layer (Istio/Linkerd) that manages all network communication between services automatically |
| **Distributed Tracing** | Tracking a request as it flows through multiple services to see timing and failures |
| **Database Per Service** | Each microservice owns its own database — sharing is forbidden |
| **Fault Isolation** | When one service fails, others continue operating independently |
| **Conway's Law** | Systems are designed to mirror the communication structure of the organization that built them |
| **Two-Pizza Team** | Amazon's principle: each service should be owned by a team small enough to be fed by two pizzas |
| **Nanoservices** | An anti-pattern: services so small (a single method) they become unmanageable |
| **Distributed Monolith** | An anti-pattern: services that look separate but can't be deployed independently due to tight coupling |
| **gRPC** | Google's high-performance, binary Remote Procedure Call protocol — used for inter-service communication |
| **Kafka** | Apache Kafka — a distributed event streaming platform used for async microservice communication |
| **Sidecar Proxy** | In a service mesh, a proxy container runs alongside each service and handles all networking |

## Next Recommended Chapters

- [07. Event-Driven Architecture](./07-Event-Driven-Architecture.md)
- [10. Cloud-Native Architecture](./10-Cloud-Native-Architecture.md)
- [12. Architecture Patterns](./12-Architecture-Patterns.md)
