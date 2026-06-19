> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Microservices** is an architectural pattern where a large software system is split into a collection of small, independent, and specialized services. Each service runs in its own process, manages its own private database, and communicates with other services over a network using lightweight protocols like HTTP or gRPC.

# Why It Exists
In early software development, applications were built as **Monoliths**—single, unified codebases containing all business features. As teams grew to hundreds of developers, working on a single codebase became extremely difficult. A change made by an inventory developer could accidentally break the payment system, halting all deployments. Furthermore, scaling the application required upgrading the entire system, even if only one feature was experiencing heavy traffic. Engineers created microservices to establish clear boundaries, allowing different teams to build, deploy, and scale separate features independently.

# Problem It Solves
Microservices solve deployment coordinate blockages, single-point-of-failure vulnerabilities, and inefficient system resource utilization.

### Before Microservices (Monolithic Architecture):
- A single bug in one minor feature (like a profile review form) crashed the entire website.
- Deploying updates required rebuilding the entire system, leading to slow release cycles.
- Scaling the application meant duplicating the entire codebase across more servers, wasting memory.

### After Microservices (Decoupled Services):
- If the Review Service crashes, the payment and shopping services continue running unaffected.
- Teams deploy updates to their specific services daily without coordinating with other teams.
- High-traffic services (like search) are scaled up onto larger servers, while quiet services remain on cheap virtual nodes.

# Core Concepts
To design microservice architectures, you must understand gateways, discovery, and boundaries:

1. **API Gateway:** A single, central entryway server that sits in front of all internal microservices. It intercepts public client requests, handles authentication, filters traffic, and routes requests to the correct service.
2. **Service Discovery:** An internal database registry that maps the dynamic IP addresses of running microservice instances, allowing services to find and communicate with each other over the network.
3. **Database-per-Service:** A core design rule stating that each microservice must own its private database. Services cannot access other services' databases directly; they must query them via APIs.

# Architecture / Components
The topology of a microservices architecture routing incoming queries through a central API Gateway:

```text
                               [ User Clients ]
                                       │
                                       ▼ (Public HTTP Requests)
                               [ API Gateway ]
                                       │
               ┌───────────────────────┼───────────────────────┐
               ▼ (Route to /users)     ▼ (Route to /products)  ▼ (Route to /orders)
        [ User Service ]       [ Product Service ]     [ Order Service ]
               │                       │                       │
               ▼                       ▼                       ▼
        [ User SQL DB ]        [ Catalog Mongo DB ]     [ Order Redis DB ]
```

- **API Gateway:** The gatekeeper handling SSL, auth, and routing.
- **Service Registry:** The dynamic directory holding service IP addresses.
- **Microservices:** Specialized worker applications handling distinct business features.

# Workflow
The request flow through an API Gateway and Service Registry:

```text
Step 1: A client requests `example.com/api/products`.
                             ↓
Step 2: The API Gateway intercepts the request and verifies the client's JWT auth token.
                             ↓
Step 3: The API Gateway queries the Service Registry: "What are the IP addresses of the Product Service?"
                             ↓
Step 4: The Service Registry returns active IPs: `[10.0.1.12, 10.0.1.13]`.
                             ↓
Step 5: The API Gateway selects `10.0.1.12` and forwards the request.
                             ↓
Step 6: The Product Service fetches details from its Catalog Database and returns the response to the Gateway.
```

# Real World Examples
Think of microservices as a **large department store** and a monolith as a **single-owner general store**.
- **Monolith Store:** One person runs the shop. They handle checking out customers, stocking shelves, locking the vault, and doing accounting. If they catch a cold, the store closes. If the checkout line gets long, they cannot clone themselves to work multiple cash registers.
- **Microservices Department Store:** The business is split into independent departments.
  - **Departments (Services):** The Cashier Desk, the Shoe Inventory Room, the Clothing Inventory Room, and the Shipping Center. Each has its own staff and tools.
  - **API Gateway (Receptionist):** When a customer walks in, they don't wander into the employee break room. They ask the front desk receptionist: *"Where are the shoes?"* The receptionist says: *"Go to counter 4."* The receptionist also checks security IDs at the door.
  - **Service Discovery (Staff Phonebook):** If you add 3 new cash registers to counter 4, the staff registers their extension numbers in the store's phone directory.
  - **Database-per-Service (Private ledgers):** The Shoe department keeps its own inventory ledger. The Cashier desk cannot walk in and edit the ledger. They must call the Shoe clerk and say: *"Did you sell item #5?"* and let the clerk update the book.

# Implementation
Here is a simplified look at how an API Gateway routing configuration is written in Javascript using Express:

### Simple API Gateway Router Config (Node.js)
```javascript
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

// 1. Authenticate all requests at the gateway level
app.use((req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) {
    return res.status(401).send("Unauthorized: Missing security token.");
  }
  // Validate token...
  next();
});

// 2. Route traffic dynamically to target microservices based on URL paths
app.use('/api/users', createProxyMiddleware({
  target: 'http://user-service-cluster:3000', // Internal network IP
  changeOrigin: true
}));

app.use('/api/products', createProxyMiddleware({
  target: 'http://product-service-cluster:3001',
  changeOrigin: true
}));

app.use('/api/orders', createProxyMiddleware({
  target: 'http://order-service-cluster:3002',
  changeOrigin: true
}));

app.listen(80, () => {
  console.log("API Gateway running on Port 80.");
});
```

# Best Practices
- **Enforce Database Isolation:** Never let two microservices share or read from the same database tables directly. Shared databases create hidden dependencies that prevent services from being updated or scaled independently.
- **Implement Circuit Breakers:** If Service A queries Service B, and Service B is down, Service A should trip a "circuit breaker" and fail immediately with a default fallback response. This prevents requests from queueing up and crashing Service A.
- **Utilize Distributed Tracing:** In a microservices system, a single user request can trigger a chain of calls across 5 different services. Use tracing tools (like Jaeger or Zipkin) to attach a unique `Correlation ID` to the request so you can trace its path and identify performance bottlenecks.

# Industry Standards
Modern microservice architectures are containerized using **Docker** and orchestrated using **Kubernetes**. Internal service-to-service communication is optimized using a **Service Mesh** (like Istio), which automatically handles network security, load balancing, and failure retries between services.

# Common Mistakes
- **Building a "Distributed Monolith":** Splitting your application into separate services but keeping them tightly coupled (e.g. Service A cannot do anything without calling Service B synchronously). You inherit the network lag of microservices with the deployment lock-ins of a monolith.
- **Under-estimating Network Latency:** Forgetting that communicating over network cables is thousands of times slower than calling a function in memory. If your page load requires 50 synchronous HTTP calls between microservices, the page will load incredibly slowly.
- **Forgetting Automated Testing pipelines:** Trying to coordinate microservices manually. Because services depend on each other's APIs, teams must enforce strict contract testing (e.g. Pact) to guarantee that changing Service A's code doesn't break Service B.

# Security & Performance Considerations
- **Internal Security (Zero Trust):** Once an attacker breaches the API Gateway, they should not have free rein over the internal network. Relational service calls should be encrypted using Mutual TLS (mTLS) so services verify each other's identity before sharing data.
- **Network Serialization Overhead:** Relational microservices spend significant CPU power turning data into JSON strings to send over HTTP. High-performance systems use binary serialization protocols like **gRPC** (Protobuf) to compress data and speed up internal network speeds.

# Related Technologies
- **Kubernetes:** The industry standard for orchestrating containerized microservices.
- **gRPC:** A high-performance, low-latency remote procedure call framework using binary serialization.
- **Istio:** A service mesh used to secure and manage service-to-service communication.

# Summary

## What We Learned
- Microservices divide large applications into specialized, independently deployable services to improve scaling and developer coordination.
- API Gateways act as centralized gatekeepers handling entry authentication, rate limiting, and request routing.
- Database isolation is mandatory: each microservice must own its data tier to prevent tight coupling.

## Key Takeaways
- Prevent cascading failures by wrapping inter-service calls in circuit breakers.
- Implement distributed tracing with Correlation IDs to diagnose slow requests across service boundaries.
- Utilize gRPC for internal service-to-service calls to reduce network serialization latency.

# Keywords
- Microservices
- Monolith
- API Gateway
- Service Discovery
- Database-per-Service
- Circuit Breaker
- Distributed Tracing
- Service Mesh
- gRPC
- Correlation ID

# Glossary

| Term | Meaning |
|---|---|
| API Gateway | A server that acts as a single entry point for all API requests, routing them to target microservices. |
| Service Registry | An internal database containing the network locations (IPs) of all running microservices. |
| Database-per-Service | An architectural rule where each microservice manages its own database, preventing shared storage access. |
| Circuit Breaker | A design pattern that blocks queries to a failing service immediately, preventing cascading system failure. |
| Correlation ID | A unique tracking identifier attached to a client request used to track its path across multiple microservices. |
| Service Mesh | A dedicated infrastructure layer that manages secure service-to-service network communication. |

## Next Recommended Chapters
- 09-Distributed-Systems.md
