> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **System Design Interview** is a structured, open-ended technical evaluation used by software engineering teams to assess a developer's architectural thinking, scalability planning, and system coordination skills. Instead of writing code, candidates design the blueprints for a large-scale system (like a chat app or payment gateway) on a whiteboard.

# Why It Exists
Writing code is only one part of software engineering. As applications scale, the way components (servers, databases, networks) are arranged and connected becomes more important than individual lines of code. An engineer who writes excellent code but designs a system with a single point of failure will cause application outages. Companies use system design interviews to ensure candidates can identify bottlenecks, estimate system resources mathematically, and make logical engineering trade-offs under production conditions.

# Problem It Solves
System design interviews solve poor architectural choices, developer miscommunications, and unscalable codebase layouts.

### Before System Design Interviews (Code-Only Focus):
- Companies hired developers who wrote fast algorithms but built backend architectures that crashed under basic user traffic spikes.
- Engineers jumped straight into writing database schemas without estimating storage growth, leading to disk exhaustion weeks after launch.
- Design reviews were chaotic because teams lacked a shared framework to discuss architectural trade-offs.

### After System Design Interviews (Architectural Alignment):
- Candidates are evaluated on their ability to think holistically about system reliability, bottlenecks, and costs.
- Engineers are trained to mathematically calculate system sizes (bandwidth, RAM, storage) before writing code.
- Teams use a structured, step-by-step framework to propose, review, and align on new architectural designs.

# Core Concepts
To succeed in a system design interview, you must follow a structured **4-Step System Design Framework**:

1. **Step 1: Clarify Requirements (5-10 mins):** Ask clarifying questions to define the boundaries of the system. Split requirements into *Functional* (features, e.g. "users can send messages") and *Non-Functional* (scaling, e.g. "messages must deliver in under 200ms").
2. **Step 2: High-Level Design (10-15 mins):** Sketch a simple, end-to-end flowchart of the core components. Focus on routing requests from the client's screen to the web server, cache layer, and database.
3. **Step 3: Deep Dive (15-20 mins):** Focus on the specific technical challenges of the system. Detail the database schema, select specific indexing keys, design the caching policy, or describe the API endpoints.
4. **Step 4: Scale and Address Bottlenecks (5-10 mins):** Stress-test your design. Identify the Single Points of Failure (SPOF) and explain how you would scale the system if traffic multiplied by 100 (e.g. adding load balancers, database replica nodes, or message queues).

# Architecture / Components
The timeline division of a standard 45-minute system design interview session:

```text
       [ Start: 45 Minute Interview Timer ]
                        │
                        ▼ (Step 1: 5-10 mins)
            [ Clarify Requirements ] ──────> Ask: "Who is the user? How many Daily Active Users?"
                        │
                        ▼ (Step 2: 10 mins)
            [ High-Level Design ]    ──────> Draw: Client -> Load Balancer -> Web Server -> DB
                        │
                        ▼ (Step 3: 15-20 mins)
            [ Deep Dive Details ]    ──────> Explain: Schema, caching keys, API routing details
                        │
                        ▼ (Step 4: 5 mins)
            [ Scale & Bottlenecks ]  ──────> Add: Replicas, sharding, monitoring agents
```

- **Functional Requirements:** The feature checklist you must design.
- **Non-Functional Requirements:** The performance boundaries (Latency, Availability, Consistency) you must enforce.

# Workflow
The calculation workflow for back-of-the-envelope math during the requirements phase:

```text
Step 1: The interviewer says: "Design a photo sharing app. We have 10 million Daily Active Users (DAU)."
                             ↓
Step 2: Calculate write volume: "If 10% of users upload 1 photo per day, that is 1 million uploads/day."
                             ↓
Step 3: Convert to Writes Per Second: 1 million uploads / 86,400 seconds = ~12 writes/second.
                             ↓
Step 4: Calculate storage: "If each photo averages 2MB, daily storage is 1 million * 2MB = 2 Terabytes (TB)/day."
                             ↓
Step 5: Project 5-year storage: 2TB/day * 365 days * 5 years = 3.65 Petabytes (PB) of S3 storage.
                             ↓
Step 6: Confirm design choice: "Since we need 3.65PB, we must use object storage (S3) for photos and a SQL DB for metadata."
```

# Real World Examples
Think of a system design interview as a **commercial architect presenting a proposal to build a new shopping mall**.
- **Bad Presentation:** The client asks you to design a shopping mall. You immediately grab a pencil, draw a single brick, and start explaining how to mix concrete. You don't ask how many shops the mall needs, how many shoppers will visit, or if there is a parking garage. The client walks away because you have no blueprint.
- **Good Presentation (Using the Framework):**
  - **Step 1 (Clarify):** You ask the client: *"How many shoppers visit per day? Do we need a food court? Are there specific safety rules?"*
  - **Step 2 (High-Level Sketch):** You draw a simple outline: *"Here is the main entrance (Load Balancer), here is the main hallway leading to shops (API Router), and here is the loading dock where deliveries arrive (Database writes)."*
  - **Step 3 (Deep Dive):** You detail the specific blueprints: *"We will build escalators capable of moving 5,000 people per hour. The plumbing pipes will be 6 inches wide to handle peak bathroom usage, and the security office will monitor all entrances."*
  - **Step 4 (Scale & Safety):** You address failures: *"If the city power grid fails, we have backup generators (Redundancy/Failover). If holiday traffic doubles, we have extra parking lots we can open."*

# Implementation
Here is how to structure a written summary page of your system design proposal during an interview or design review document:

### Sample System Design Document Template
```markdown
# System Proposal: Real-Time Chat System

## 1. Requirements & Scope
- **Functional:** 
  - Users can send 1-to-1 private text messages.
  - Users can see online/offline status indicators.
- **Non-Functional:**
  - Messages must deliver in < 200ms (Low Latency).
  - Messages must not be lost (High Durability).
- **Scale:** 50 Million Daily Active Users, averaging 100 messages/day.

## 2. Capacity Estimations
- **Message Volume:** 50M * 100 = 5 Billion messages/day.
- **Writes Per Second:** 5B / 86,400 = ~58,000 requests/second.
- **Storage Growth:** 5B messages * 100 bytes = 500GB/day of text database writes.

## 3. High-Level Architecture
- **WebSockets:** Used for persistent bi-directional connection between clients and Chat Servers.
- **Load Balancer:** Distributes WebSocket connections across a pool of Chat Servers.
- **Cache (Redis):** Stores active user presence status (online/offline).
- **Database (Cassandra):** Stores message history logs (NoSQL optimized for fast writes).

## 4. Deep Dive & Bottlenecks
- **Single Point of Failure (SPOF):** The presence Redis database. We will deploy Redis Sentinel to automate failover.
- **Write Saturation:** Cassandra handles heavy writes efficiently, but we will shard tables by `conversation_id` to distribute storage evenly across database nodes.
```

# Best Practices
- **Never Design in Silence:** Talk out loud constantly during the interview. The interviewer wants to hear your thought process, how you weigh trade-offs, and why you choose one database over another.
- **Do Not Guess Math:** If you are estimating numbers, write the math steps clearly on the board. Use round numbers (e.g. 100,000 seconds in a day instead of 86,400) to keep calculations simple and prevent errors.
- **State Your Trade-offs:** Every design choice has a cost. If you choose MongoDB, explain *why* (flexible schema, fast development) and acknowledge the trade-off (no native complex joins, high memory usage). This shows maturity.

# Industry Standards
Professional teams use standard design documentation called **RFCs (Request for Comments)** or **Architecture Design Records (ADRs)**. These documents use the exact same structured steps (Requirements -> High-Level Design -> Deep Dive -> Scaling) to propose and review system changes before writing code.

# Common Mistakes
- **Jumping to Whiteboard Diagrams Instantly:** Drawing servers and databases the second the interviewer finishes speaking. If you don't know the read/write ratios, you might design an SQL database when a cache-heavy CDN was required.
- **Over-complicating the Design:** Adding Kafka queues, Kubernetes clusters, and sharded databases to a design for a simple blog platform. Keep the design as simple as possible; only add complex components when scaling requirements force you to.
- **Ignoring the Interviewer's Hints:** Continuing down a path when the interviewer asks: *"Are you sure you want to use a relational database for this search logs table?"* Stop, listen, and re-evaluate your design based on their feedback.

# Security & Performance Considerations
- **Authentication Check Bottlenecks:** If your API Gateway checks user sessions by querying the database on every single request, the auth query will saturate the database. In your design, explain how you will cache session tokens in Redis or use self-validating JWTs to protect database performance.
- **Rate Limiting Security:** An open API is a target for script attacks. Explain where you will place rate-limiters (e.g., at the API Gateway) to block automated brute-force attacks.

# Related Technologies
- **System Design Primer:** A famous open-source GitHub repository detailing system design interview concepts.
- **Excalidraw / draw.io:** Collaborative digital whiteboarding tools used for system design sketches.
- **Architecture Design Records (ADR):** The corporate documentation standard for recording architectural decisions.

# Summary

## What We Learned
- System design interviews evaluate a developer's ability to think architecturally and plan resource requirements.
- The 4-step framework guides candidates through Clarifying Requirements, High-Level Sketching, Deep-Diving, and Scaling.
- Every architectural choice represents a trade-off; designers must justify their choices based on business metrics.

## Key Takeaways
- Always establish functional and non-functional requirement boundaries before drawing diagrams.
- State your design trade-offs explicitly (e.g., CP vs. AP database profiles).
- Avoid premature complexity; keep blueprints simple and add scaling layers only where bottlenecks appear.

# Keywords
- System Design Interview
- 4-Step Framework
- Functional Requirement
- Non-Functional Requirement
- Back-of-the-Envelope Math
- Single Point of Failure (SPOF)
- Deep Dive
- Architecture Design Record (ADR)
- RFC
- Trade-off

# Glossary

| Term | Meaning |
|---|---|
| Functional Requirement | A specific action or feature that the software system must perform. |
| Non-Functional Requirement | A quality attribute or performance boundary (like speed, uptime, or security) that the system must meet. |
| Deep Dive | The phase of a design review where specific component details (like database schemas or API endpoints) are analyzed. |
| SPOF | Single Point of Failure; any individual component that will disable the entire system if it crashes. |
| Back-of-the-Envelope Math | Quick, high-level mathematical calculations used to estimate system scale and resource needs. |
| ADR | Architecture Design Record; a document recording technical decisions, rationale, and trade-offs. |

## Next Recommended Chapters
- (Next Section: DevOps)
