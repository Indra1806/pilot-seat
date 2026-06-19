> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Microservices Architecture** is a software design pattern where a single, large backend application (a monolith) is split into a collection of small, self-contained, and independently running services. Each service is responsible for a single business capability (such as user management, payment processing, or shipping) and communicates with other services over standard network protocols (like REST APIs, gRPC, or message queues).

# Why It Exists
In traditional development, backend apps are built as a **Monolith**—meaning all code, routing, and database queries live in a single program. For small teams, this is simple and fast. However, as an enterprise grows, having 100+ developers edit the same monolithic application code results in development traffic jams. Deploying a minor update requires rebuilding and shipping the entire massive app, which takes hours. If a single developer makes a memory leak error in a minor reporting feature, it will crash the entire server, taking down payment processing and user logins. Engineers created microservices to split this monolith into autonomous, fault-isolated units so teams could build and deploy code independently.

# Problem It Solves
Microservices architecture solves monolithic scaling bottlenecks, single points of failure (cascading crashes), and release coordination delays.

### Before Microservices (The Monolith):
- A bug in one minor feature could crash the entire website (single point of failure).
- Scaling required duplicating the entire monolith onto larger servers, which was highly expensive.
- Different development teams had to coordinate release schedules, slowing down feature launches.

### After Microservices (Decoupled Services):
- Fault Isolation: If the `Recommendation Service` crashes, the checkout and login pages continue working normally.
- Granular Scaling: You can scale up the `Payment Service` on larger servers during sales without wasting money duplicating the rest of the application.
- Release Independence: The "Payments" team can deploy updates 10 times a day without asking permission from the "Shipping" or "Auth" teams.

# Core Concepts
To architect microservices, you must master three organizational patterns:

1. **Database-Per-Service:** Each microservice must own its own private database. No service is allowed to read or write to another service's database directly. If the `Order Service` needs user details, it must request them via the `User Service` API. This prevents database schema changes in one service from breaking others.
2. **Inter-Service Communication:** Services communicate over the network using:
   - **Synchronous (HTTP/gRPC):** Service A requests data and waits for Service B to respond (great for instant validations).
   - **Asynchronous (Message Brokers):** Service A publishes a message and walks away; Service B reads it later (great for workflows).
3. **API Gateway:** The single entry router for client requests. The frontend talks only to the API Gateway, which routes requests to the correct microservice in the background.

# Architecture / Components
The flow of client traffic through an API Gateway to microservices:

```text
  [ User Frontend ]
          │
          ▼ (Single API call: GET /api/orders)
  [ API Gateway ] (Routes and authorizes traffic)
     ┌────┴───────────────────────────┐
     ▼ (HTTP/gRPC)                    ▼ (Event Publish)
  [ Order Service ]             [ Payment Service ]
  (Database A)                  (Database B)
```

- **API Gateway:** The front door routing layer that handles authentication, rate limiting, and request forwarding.
- **Service Registry:** The directory manager (like Consul) where active microservice instances register their IP addresses so the API Gateway can locate them dynamically.
- **Service Mesh:** A dedicated network infrastructure layer (like Istio) that manages secure, encrypted communication between microservices.

# Workflow
How a client purchase request is processed across microservices:

```text
Step 1: The user clicks "Checkout" in their browser.
                             ↓
Step 2: The browser sends a request to the API Gateway: `POST /api/checkout`.
                             ↓
Step 3: The API Gateway routes the request to the `Order Service`.
                             ↓
Step 4: The `Order Service` makes a synchronous HTTP/gRPC call to `User Service` to verify the user.
                             ↓
Step 5: The `Order Service` saves the order, then publishes an `OrderCreated` event to a message broker.
                             ↓
Step 6: The `Payment Service` (listening to the broker) picks up the event and charges the customer.
```

# Real World Examples
Think of microservices as a **major city hospital** versus a **general practitioner (GP) clinic**.
- A monolithic application is like a single GP doctor operating in a small town. They diagnose your throat, set your broken bone, check your heart, and run the x-ray machine. It is simple, cheap, and fast for a small scale. But if 1,000 patients arrive, the single doctor is overwhelmed, and if the doctor gets sick, the clinic closes (single point of failure).
- A microservices architecture is like a large city hospital. It is split into independent departments: Cardiology, Radiology, Pediatrics, and Pharmacy. Each department has its own specialized doctors (Services), its own equipment (Hardware), and its own private medical filing cabinet (Database-per-Service).
- If the Radiology department's x-ray machine breaks, the Cardiology department continues performing heart checks (Fault Isolation). They communicate over standard hospital phones (API networks) when coordinating patient care.

# Implementation
Here is how an API Gateway routes requests to separate microservices in Node.js:

```javascript
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');
const app = express();

// 1. Define the internal network addresses of our microservices
const SERVICES = {
  userService: 'http://localhost:3001',
  orderService: 'http://localhost:3002'
};

// 2. Setup the API Gateway routing rules
// Route any request to /api/users to the User Microservice
app.use('/api/users', createProxyMiddleware({
  target: SERVICES.userService,
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' } // Strips prefix before forwarding
}));

// Route any request to /api/orders to the Order Microservice
app.use('/api/orders', createProxyMiddleware({
  target: SERVICES.orderService,
  changeOrigin: true,
  pathRewrite: { '^/api/orders': '' }
}));

console.log("API Gateway is active on port 3000...");
app.listen(3000);
```

# Best Practices
- **Never Share Databases:** Sharing a database is the #1 reason microservice projects fail. If Team A changes a column name in a shared database, Team B's service will instantly crash in production. Enforce strict database ownership.
- **Build Services Around Business Domains:** Do not split services by technical layers (e.g. do not make a "Database Service" and a "Javascript Service"). Split them by business capability, following Domain-Driven Design (DDD) (e.g. `BillingService`, `CatalogService`).
- **Implement Distributed Tracing:** When a request fails, it can be difficult to tell which service in the chain crashed. Use distributed tracing headers (like correlation IDs) that travel with the request across all services so you can search logs easily.

# Industry Standards
Almost all massive digital platforms—like Netflix, Uber, Amazon, and Spotify—use microservices. They run containerized microservices in Docker containers orchestrated by **Kubernetes**, which manages scaling, service discovery, and routing automatically.

# Common Mistakes
- **Creating a Distributed Monolith:** Splitting your code into separate repositories, but keeping them so tightly coupled that you cannot deploy Service A without also deploying Service B and C at the same time. If they must be deployed together, keep them in the same codebase.
- **Choosing Microservices Too Early:** Designing a startup's MVP using 15 microservices. Microservices introduce massive operational overhead (network delays, complex deployments, debugging difficulties). Build a clean monolith first; only split it into microservices when your team size or traffic scales too large to manage.

# Security & Performance Considerations
- **Network Latency:** In a monolith, calling another function takes nanoseconds. In microservices, calling another service requires a network jump, which can add 20ms of latency. Keep service call chains short.
- **Service-to-Service Authentication:** Secure internal network communications. Use internal OAuth client credentials or mTLS (mutual TLS) certificates so that a compromised service cannot make unauthorized requests to other services.

# Related Technologies
- **Kubernetes:** The container orchestration system used to run fleets of microservices.
- **gRPC:** A high-speed, binary network protocol developed by Google, popular for fast synchronous inter-service communication.
- **Istio:** A popular Service Mesh tool used to manage security and telemetry between microservices.

# Summary

## What We Learned
- Microservices decouple monolithic applications into autonomous, domain-focused network services.
- Database-per-service isolates data structures and prevents schema coupling.
- API Gateways streamline client requests, while service discovery manages dynamic IP routing.

## Key Takeaways
- Enforce strict database boundaries; never share database access across microservices.
- Do not adopt microservices for small teams; the operational overhead outweighs the benefits at small scale.

# Keywords
- Microservices
- Monolith
- Decoupling
- API Gateway
- Database-per-Service
- gRPC
- Service Discovery
- Domain-Driven Design

# Glossary

| Term | Meaning |
|---|---|
| Monolith | A single, unified software application where all components share code and database access. |
| API Gateway | The router entry point that acts as a reverse proxy to manage and forward client requests to internal microservices. |
| gRPC | A high-performance, open-source universal RPC framework developed by Google using binary protocols. |
| Fault Isolation | The design capability that ensures a crash in one software module does not affect or bring down other modules. |
| Service Discovery | The system that automatically detects the network locations (IP addresses) of active service instances. |

## Next Recommended Chapters
- 08-Message-Queues-And-Events.md
- 11-Containerization-And-Docker.md
