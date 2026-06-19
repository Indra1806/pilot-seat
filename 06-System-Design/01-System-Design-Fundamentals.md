> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**System Design Fundamentals** is the foundational practice of architectural planning, component coordination, and traffic routing in software engineering. It is the process of designing how separate software programs, servers, databases, and networks connect and work together to build a reliable system that can handle massive user traffic safely and efficiently.

# Why It Exists
When software developers first learn to write code, they build run-anywhere scripts or single-server applications (monoliths). However, as a business grows, a single application server cannot handle all operations. If a system handles millions of concurrent users, placing all features—from product search to payment processing to email shipping—on one server causes the system to run out of capacity and crash. Engineers created system design methodologies to break complex systems into independent, specialized components that cooperate over networks, allowing applications to grow infinitely without single-point-of-failure risks.

# Problem It Solves
System design fundamentals solve single-server capacity exhaustion, rigid application scaling, and single-point-of-failure vulnerability.

### Before System Design (Monolithic Single Server):
- All services shared the same memory and disk. If the payment code crashed, the entire website went down.
- Scaling required upgrading the entire system, which was extremely expensive and had physical limits.
- Traffic spikes (like holiday sales) overloaded the database, crashing the entire business.

### After System Design (Distributed Architecture):
- Features are separated into independent components. If the payment service has an issue, users can still search and browse products.
- Individual high-traffic services can be scaled independently without affecting the rest of the system.
- Load balancers and cache layers protect databases from traffic spikes, maintaining high speeds.

# Core Concepts
To design complex systems, you must understand three core architectural foundations:

1. **System Components:** The individual building blocks of software systems, such as web servers (processing logic), databases (permanent storage), caches (fast temporary memory), and message queues (asynchronous coordination).
2. **Client-Server Communication:** The protocols and network links (like HTTP, WebSockets, or RPC) that allow frontends, backends, and databases to talk to each other across different computers.
3. **High Availability (Redundancy):** The practice of duplicating critical system components so that if one server fails or catches fire, another duplicate server instantly takes over the workload, preventing application downtime.

# Architecture / Components
The high-level architecture of a fundamental multi-tier web application system:

```text
                                [ User Clients ]
                                       │
                                       ▼ (Public HTTP Traffic)
                               [ Load Balancer ]
                                       │
                ┌──────────────────────┴──────────────────────┐
                ▼ (Distributed Requests)                      ▼
       [ Web Server Node A ]                         [ Web Server Node B ]
                │                                             │
                ├──────────────────────┬──────────────────────┤
                ▼                      ▼                      ▼
      [ In-Memory Cache ]      [ Write Database ]     [ Read Database ]
      - Fast RAM lookups       - Processes edits      - Serves search lists
```

- **Load Balancer:** The entry gateway that intercepts incoming user traffic and distributes it evenly among available web servers.
- **Web Application Servers:** The processing layer that executes business logic and prepares data.
- **Data Tier:** The combination of caches (for speed) and databases (for durability) that manages application records.

# Workflow
The request-response lifecycle in a designed distributed system:

```text
Step 1: A user clicks "Buy Now" on their phone screen.
                             ↓
Step 2: The request hits the Load Balancer, which routes it to Web Server Node A (the least busy server).
                             ↓
Step 3: Web Server Node A verifies the item inventory by checking the fast In-Memory Cache first.
                             ↓
Step 4: If cache matches, the server sends a write query to the Write Database to record the purchase.
                             ↓
Step 5: The Write Database updates the records and synchronizes the update with the Read Database.
                             ↓
Step 6: Web Server Node A sends a success confirmation back through the load balancer to the user's phone.
```

# Real World Examples
Think of system design fundamentals as **town planning for a new city**.
- An un-designed application is like a single enormous building that houses the city hall, the supermarket, the hospital, the fire station, and the police station under one roof. If the kitchen in the city hall catches fire, the entire city's infrastructure burns down. If everyone in town tries to visit the supermarket at 5:00 PM, the single doorway gets jammed, and no one can access the hospital.
- A designed system is like a planned city.
  - **Services (Buildings):** The supermarket (Product database) is separate from the hospital (User account service). If one building closes for maintenance, the others remain open.
  - **APIs/Networks (Roads):** Street networks allow delivery trucks (data packets) to travel from the warehouse (database) to the storefronts (backend servers).
  - **Load Balancers (Traffic Roundabouts):** Traffic lights and roundabouts route incoming cars down alternative roads so no single intersection gets gridlocked.
  - **Caches (Water Towers):** Keeping water reserves in local neighborhood water towers so you don't have to pump water from the distant mountain lake (primary database) every time someone turns on a tap.

# Implementation
Here is a conceptual look at how a developer structures a basic multi-service system configuration in code (using a Docker Compose configuration file to coordinate separate service containers):

### Multi-Service System Blueprint (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  # 1. Load Balancer Component (Entrypoint)
  gateway:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - web_app

  # 2. Web Application Server Component
  web_app:
    image: node-backend-app:latest
    environment:
      - DATABASE_URL=postgres://db_user:password@db_server:5432/store
      - CACHE_URL=redis://cache_server:6379
    depends_on:
      - db_server
      - cache_server

  # 3. Cache Tier Component
  cache_server:
    image: redis:alpine

  # 4. Database Tier Component
  db_server:
    image: postgres:alpine
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

# Best Practices
- **Design for Failure:** Assume that every server, hard drive, and network cable in your system will eventually fail. Build duplicate backups (redundancy) and automatic switch-overs (failover) for every critical component.
- **De-couple Services:** Ensure that different components of your system know as little about each other as possible. Communication should happen through clean, versioned APIs so you can update one service without breaking another.
- **Keep Application Servers Stateless:** Never store user sessions or uploaded files directly on a web application server's local hard drive. Keep application servers "stateless" so they can be turned off, rebooted, or scaled up instantly, storing files in central cloud buckets and sessions in Redis.

# Industry Standards
Modern internet enterprises use standard patterns like **Microservices** and **Service-Oriented Architecture (SOA)**. Communication between these services is standardized using protocols like HTTP/JSON, gRPC, or asynchronous message channels (like Apache Kafka).

# Common Mistakes
- **Premature Complexity:** Building a massive distributed system with microservices and load balancers on Day 1 for a project that has no users yet. Start with a clean monolith, design it cleanly, and split it into services only when scaling demands it.
- **Single Point of Failure (SPOF):** Forgetting to duplicate a critical component. If you have 10 web servers but only 1 database with no replicas, a database crash will take down the entire system regardless of your web servers' health.
- **Ignoring Network Latency:** Assuming that communication between separate servers over the network is as fast as communication inside a single server's memory. Network queries take milliseconds, which can add up and slow down user requests if not managed.

# Security & Performance Considerations
- **API Gateways & Rate Limiting:** Exposing all internal database and backend servers directly to the public internet is a major security risk. System designers use an **API Gateway** as a single firewall entry point that filters and limits traffic before it reaches internal services.
- **Distributed Failure Cascades:** If one internal service runs slowly, requests will pile up, consuming memory on upstream web servers and eventually causing a system-wide crash. System designers implement **Circuit Breakers** to cut off connection attempts to failing services immediately, keeping the rest of the application running.

# Related Technologies
- **Docker Compose:** A tool used to orchestrate and run multi-container applications locally.
- **Kubernetes:** The industry-standard system for automating deployment, scaling, and management of containerized distributed systems.
- **Nginx / HAProxy:** High-performance software used to run load balancers and reverse proxies.

# Summary

## What We Learned
- System Design is the architectural planning of how separate hardware and software systems coordinate to process traffic at scale.
- Breaking systems into independent components prevents capacity exhaustion and single-point-of-failure vulnerabilities.
- State should be kept off application servers, using central caches and databases to allow easy scaling.

## Key Takeaways
- Always identify and eliminate Single Points of Failure (SPOF) by adding redundant servers.
- Keep web application servers stateless so they can scale up or down automatically.
- Secure your internal servers by restricting access behind an API Gateway and Load Balancer.

# Keywords
- System Design
- Monolith
- Distributed System
- High Availability
- Redundancy
- Single Point of Failure (SPOF)
- Stateless
- API Gateway
- Failover
- Docker Compose

# Glossary

| Term | Meaning |
|---|---|
| Monolith | An application model where all functions, code, and services are packaged into a single, unified program. |
| Distributed System | A collection of separate computer servers connected over a network that coordinate actions to act as a single system. |
| High Availability | A design goal to keep a system running continuously without interruption for long periods. |
| Single Point of Failure | Any single component in a system that, if it fails, will stop the entire system from working. |
| Stateless | An architecture style where servers do not save user request data locally, allowing any server to handle any request. |
| Failover | The automatic transfer of operations to a standby backup server when the primary server fails. |

## Next Recommended Chapters
- 02-Scalability.md
