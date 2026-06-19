> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Software Architecture** is the high-level design, structure, and blueprint of a software system. It defines the system's core components, the relationships and communication pathways between them, and the strategic design constraints that govern how the application will scale, recover from failures, and be maintained over time.

# Why It Exists
Without a planned architecture, software projects inevitably collapse into a state called a "Big Ball of Mud" (or spaghetti code). As developers add features to a system without structural rules, components become tightly tangled together. A modification in one part of the codebase will quietly break several unrelated features in another. This makes updates slow, introduces bugs, and increases technical debt until the codebase becomes impossible to maintain. Software architecture is created to establish rigid rules, boundaries, and patterns that allow codebases to grow safely.

# Problem It Solves
Software Architecture solves the problems of runaway codebase complexity, scaling limitations, high maintenance costs, and team collaboration blocks.

### Before Software Architecture (Unstructured Codebase):
- An application worked well for 100 users, but crashed under 10,000 users because the database connections and server resources were not isolated.
- Adding a simple checkout coupon code required modifying files across ten separate directories, causing unexpected bugs in user profiles.
- As the engineering team grew, developers constantly overwrote each other's code because everyone worked inside the same giant, shared code file.

### After Software Architecture (Structured Blueprints):
- The system is decomposed into isolated modules with clear boundaries, allowing it to scale compute resources independently during traffic surges.
- New features are added quickly by editing specific, dedicated components without impacting the rest of the application.
- Teams work on separate services autonomously, deploying updates independently without coordinating release dates.

# Core Concepts
Understanding software architecture requires mastering the differences between planning and execution, and understanding component dependencies:

1. **Architecture vs. Design:**
   - **Architecture:** High-level, strategic, and difficult-to-change decisions. It answers: *How is the entire system structured?* (e.g. Choosing between Monolithic or Microservice architectures, selecting SQL or NoSQL databases).
   - **Design:** Low-level, tactical, and localized code choices. It answers: *How does this specific component work?* (e.g. Class definitions, code design patterns, query parameters).
2. **Coupling (Loose vs. Tight):** The degree of dependency between two separate system components:
   - **Tight Coupling:** Component A directly depends on the inner workings of Component B. If Component B changes, Component A breaks.
   - **Loose Coupling:** Components interact exclusively through public APIs or contracts. Component B's internal code can be rewritten without impacting Component A.
3. **Cohesion (High vs. Low):** The degree of focus inside a single component:
   - **High Cohesion:** A component only performs tasks that are closely related to its primary job (e.g. A Billing Service only handles credit card validation, invoicing, and refunds). High cohesion makes code reusable and easy to test.
   - **Low Cohesion:** A single component handles unrelated tasks (e.g. A user controller that validates emails, writes database queries, and formats HTML pages).
4. **Non-Functional Requirements (NFRs):** System characteristics that describe *how* a system behaves rather than *what* it does (e.g., Latency targets, security audits, availability SLAs).

# Architecture / Components
The difference in scope between Architectural Decisions and Design Decisions:

```text
  ┌────────────────────────────────────────────────────────┐
  │ ARCHITECTURAL LEVEL (The System Blueprint)             │
  │                                                        │
  │   [ Mobile App ] ──(HTTP)──► [ API Gateway ]           │
  │                                   │                    │
  │                        ┌──────────┴──────────┐         │
  │                        ▼                     ▼         │
  │                 [ Auth Service ]      [ Order Service ]│
  │                        │                     │         │
  │                        ▼                     ▼         │
  │                 [ User DB ]           [ Order DB ]     │
  └────────────────────────────────────────────────────────┘
                             │
            (Decomposes down to module level)
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ DESIGN LEVEL (Inside the Order Service Module)         │
  │                                                        │
  │  [ OrderController ] ──► [ OrderService ] ──► [ Repo ] │
  │                                                        │
  │  - Logic: Validate input details                       │
  │  - Design: Use Builder pattern to construct orders     │
  └────────────────────────────────────────────────────────┘
```

# Workflow
How an architect designs a system from initial requirements to implementation:

```text
Step 1: Gather functional requirements (features) and non-functional requirements (user count, latency targets).
                             ↓
Step 2: Identify system boundaries and separate the application into logical domains (e.g. Inventory, Billing).
                             ↓
Step 3: Define communication protocols between components (e.g. synchronous REST APIs or asynchronous event queues).
                             ↓
Step 4: Select storage solutions matching data structures (SQL for transactions, NoSQL for high-speed user profiles).
                             ↓
Step 5: Define security boundaries (firewalls, IAM) and monitoring checkpoints.
                             ↓
Step 6: Document decisions in Architecture Decision Records (ADRs) and hand the blueprint to developers.
```

# Real World Examples
Think of software architecture as **constructing a commercial office building**.
- **Architecture (Structural Blueprints):** The blueprints created by a structural architect defining the concrete foundation, where the structural steel columns stand, the location of utility shafts, and fire exit placements. These decisions are difficult and expensive to change once construction begins.
- **Design (Interior layout):** Choosing where to place desks, what color to paint the walls, selecting the light fixtures, and arranging the office furniture. These can be modified easily without endangering the building's stability.
- **Tight Coupling (Electricity glued to plumbing):** Imagine a house where electrical wires are run directly inside the water pipes. If the pipe leaks, the power shorts. If you need to replace a pipe, you must cut the electrical line.
- **Loose Coupling (Plumbing and electrical in separate conduits):** The pipes travel in one channel; the electrical wires travel in a separate plastic conduit. You can repair the faucet without touching the light switches.
- **High Cohesion (Specialized office rooms):** The kitchen contains the stove, fridge, and sink. The server room contains the racks and AC. If you start storing server racks inside the kitchen fridge (Low Cohesion), both cooking food and cooling servers fail.

# Implementation
Here is how developers translate coupling concepts into code. Example A shows tightly coupled code where a service directly creates its own database dependency. Example B shows loosely coupled code using **Dependency Injection** (passing the database helper through a contract interface), which allows the database to be swapped easily during unit testing:

### Example A: Tightly Coupled Code (Fragile)
```javascript
// OrderService.js
const PostgresDatabase = require('./PostgresDatabase');

class OrderService {
  constructor() {
    // 1. Hardcoded dependency: OrderService directly creates Postgres connection
    this.db = new PostgresDatabase();
  }

  async createOrder(orderData) {
    // 2. Direct execution: If database method changes, this code breaks
    return this.db.save('orders', orderData);
  }
}

module.exports = OrderService;
```

### Example B: Loosely Coupled Code (Flexible & Testable)
```javascript
// OrderService.js
class OrderService {
  // 1. Dependency Injection: Pass the database client via constructor
  // The client must obey a common interface contract (e.g., holding a .save method)
  constructor(databaseClient) {
    this.db = databaseClient;
  }

  async createOrder(orderData) {
    // 2. Loose dependency: Works with Postgres, MySQL, or a Mock database
    return this.db.save('orders', orderData);
  }
}

module.exports = OrderService;
```

# Best Practices
- **Enforce Separation of Concerns:** Divide your code into logical layers (e.g. presentation, business logic, data access). Each layer should only handle its specific concern and communicate with adjacent layers via defined interfaces.
- **Prefer Loose Coupling:** Avoid direct dependencies between modules. Use dependency injection, interfaces, or messaging systems so that components remain independent and easily testable.
- **Design for Change:** Business requirements will change. Build structures that allow you to swap database engines, update APIs, or replace libraries without having to rewrite the entire application.

# Industry Standards
Modern architectures follow **Clean Architecture** and **SOLID design principles**. These standards enforce the dependency rule: high-level business logic must never depend on low-level tools (like databases, web frameworks, or third-party APIs). Instead, both must depend on abstract contracts.

# Common Mistakes
- **Overengineering Startups:** Building a complex distributed microservices architecture with message brokers for a product that has zero active users. Start with a clean monolith and evolve as traffic grows.
- **Database Integration Coupling:** Allowing multiple separate applications to write directly to the same database tables. If you update the database structure for App A, App B crashes. App B must access the data via App A's API.
- **Neglecting Architecture Reviews:** Writing code without defining architectural rules, leading to different developers implementing contradictory patterns (e.g. mixing MVC and Hexagonal structures in one app).

# Security & Performance Considerations
- **Boundary Validation:** Never trust data passing across system boundaries (e.g. from frontend client to API gateway). Validate and sanitize all inputs at every gateway interface.
- **Latency at Integration Boundaries:** Performance bottlenecks rarely occur in CPU calculations; they occur when data travels over network connections between decoupled services. Minimize cross-service network jumps to keep latencies low.

# Related Technologies
- **UML (Unified Modeling Language):** Standardized diagram notation used to visualize system architectures.
- **ArchiMate:** A modeling language used to design enterprise architectures.
- **Terraform:** Infrastructure as Code utility used to deploy structural architecture blueprints automatically.

# Summary

## What We Learned
- Software architecture establishes high-level rules, blueprints, and structures to keep codebases maintainable.
- Architecture focuses on strategic, long-term decisions; design focuses on tactical, localized code implementation.
- Loose coupling allows components to remain independent, while high cohesion ensures modules stay focused on a single job.
- Separation of concerns divides applications into logical layers communicating through defined interfaces.

## Key Takeaways
- Enforce loose coupling using dependency injection to make code testable and flexible.
- Group related features inside cohesive services, keeping unrelated logic separated.
- Avoid integrating separate systems directly through database tables; use public API contracts instead.

# Keywords
- Software Architecture
- Software Design
- Loose Coupling
- Tight Coupling
- High Cohesion
- Non-Functional Requirements
- Separation of Concerns
- System Boundary
- Dependency Injection
- ADR

# Glossary

| Term | Meaning |
|---|---|
| Software Architecture | The high-level structure, core components, and relationship guidelines of a software system. |
| Loose Coupling | An design pattern where system components interact via defined contracts, minimizing dependency. |
| High Cohesion | A design metric where all functions inside a module are closely related to a single purpose. |
| Non-Functional | System quality attributes (like security, speed, uptime) describing how the system performs. |
| Dependency Injection | A technique where a component receives its required dependencies from the outside rather than creating them itself. |
| ADR | Architecture Decision Record; a document capturing an architectural decision, its context, and consequences. |

## Next Recommended Chapters
- 02-Layered-Architecture.md
