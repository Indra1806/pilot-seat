> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine you are opening your **first bakery**.

You do everything yourself:

- You bake the bread
- You take orders from customers
- You manage the cash register
- You handle deliveries
- You clean up at the end of the day

One person. One shop. Everything in one place.

That is the essence of **Monolithic Architecture**.

In software, a **monolith** is a single, self-contained application where all of the functionality — the user interface, business logic, data access, and everything else — lives in **one unified codebase**, runs as **one process**, and is deployed as **one unit**.

The word "monolith" comes from Greek: *monos* (single) + *lithos* (stone). One solid block.

It is the **oldest and simplest** form of software architecture — and it is still widely used today for very good reasons.

---

# Why It Exists

When software development began, the idea of splitting an application into separate services was simply not practical.

- **Servers were expensive** — running multiple separate processes was a luxury
- **Networks were slow** — communicating between services added unacceptable latency
- **Teams were small** — one team of 3-5 developers didn't need separation
- **Deployment was hard** — every separate piece was one more thing to install and manage

So the natural approach was:

```
Write everything.
Put it in one place.
Deploy it once.
It works.
```

For small applications and early-stage products, this is still the **right choice**.

A monolith isn't a failure or a bad decision. It's an appropriate architecture for the right context.

---

# Problem It Solves

**The Problem:**
When you're starting a new product, you don't know:
- Exactly what features you'll need
- What will grow and what will be abandoned
- How users will actually use the system
- Whether your business idea will even succeed

**Monolithic Architecture solves the problem of building and shipping fast:**

```
BEFORE (trying to build microservices too early):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Set up 8 separate services...
Configure networking between them...
Set up 8 separate CI/CD pipelines...
Handle distributed failures...
...still no working product after 3 months

AFTER (starting with a monolith):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Build everything in one place.
Deploy once.
Ship in 3 weeks.
Iterate fast.
```

The monolith is the fastest way to go from **zero to working product**.

It also solves another problem: **simplicity of understanding**. New developers joining the project can read one codebase and understand the entire system.

---

# Core Concepts

## 1. Single Codebase

All code for the application lives in one repository and one project:

```
myapp/
├── users/        ← User management code
├── products/     ← Product management code
├── orders/       ← Order management code
├── payments/     ← Payment code
├── notifications/← Email/SMS code
└── main.js       ← One entry point
```

No network calls between components. Just function calls.

## 2. Single Deployment Unit

The entire application is packaged and deployed as one thing:

```
Build Process:
Source Code → Compile/Bundle → One Deployable Artifact
                                    ↓
                               Deploy to Server
                                    ↓
                          One Running Process
```

Compare this to microservices where you'd have 8 separate deployments for 8 services.

## 3. Shared Database

All parts of the application use the same database:

```
┌─────────────────────────────────┐
│         Application             │
│  ┌──────┐ ┌───────┐ ┌───────┐  │
│  │Users │ │Orders │ │Products│  │
│  └──┬───┘ └───┬───┘ └───┬───┘  │
│     └─────────┴─────────┘       │
│                ↓                │
└────────────────┼────────────────┘
                 ↓
         [ Single Database ]
```

This makes transactions easy — one database means no distributed transaction headaches.

## 4. In-Process Communication

Components talk to each other via **direct function calls**, not network requests:

```javascript
// In a monolith — just call the function directly
const user = userService.getUserById(123);
const orders = orderService.getOrdersByUser(user);
```

No HTTP calls. No retries. No timeouts. Just a function call that completes instantly.

---

# Architecture / Components

## The Monolith Structure

```
┌─────────────────────────────────────────────────────┐
│                 MONOLITHIC APPLICATION               │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │            PRESENTATION LAYER                 │   │
│  │  Routes, Controllers, HTML Templates, APIs   │   │
│  └──────────────────────┬───────────────────────┘   │
│                         │                            │
│  ┌──────────────────────▼───────────────────────┐   │
│  │            BUSINESS LOGIC LAYER               │   │
│  │  User Logic, Order Logic, Payment Logic       │   │
│  │  All Features Live Here as Modules/Classes    │   │
│  └──────────────────────┬───────────────────────┘   │
│                         │                            │
│  ┌──────────────────────▼───────────────────────┐   │
│  │            DATA ACCESS LAYER                  │   │
│  │  One ORM / Database Client for everything     │   │
│  └──────────────────────┬───────────────────────┘   │
│                         │                            │
└─────────────────────────┼────────────────────────────┘
                          ↓
               ┌─────────────────┐
               │  Single Database│
               └─────────────────┘
```

## Types of Monoliths

Not all monoliths are the same. There are three common forms:

### Type 1: The Single-Process Monolith (Classic)
```
One server. One process. Everything runs together.

[Server]
  ├── Web Server (Express/Django)
  ├── Business Logic (all features)
  ├── Database Client
  └── Cron Jobs, Emails, etc.
```

### Type 2: The Modular Monolith (Structured)
```
One process. Internally organized into clear modules.
Modules cannot directly access each other's databases.

[Server]
  ├── [UserModule]    → UserService, UserRepository
  ├── [OrderModule]   → OrderService, OrderRepository
  ├── [ProductModule] → ProductService, ProductRepository
  └── Shared Database (with separate schemas per module)
```
This is a best-of-both-worlds approach (explored in depth in Chapter 04).

### Type 3: The Distributed Monolith (Anti-Pattern)
```
Looks like microservices. Behaves like a monolith. Worst of both worlds.

[Service A] ↔ [Service B] ↔ [Service C]
   ↑────────── All must deploy together ───────────↑
   ↑────── Tightly coupled, can't work alone ──────↑
```
This is what happens when you split services but keep tight dependencies. **Avoid this.**

---

# Workflow

**Scenario: A customer places an order on an e-commerce monolith.**

```
STEP 1 — HTTP Request Arrives
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /orders  (customer's browser sends the request)
  ↓
Express/Django router receives it
  ↓
OrderController.createOrder() is called

STEP 2 — Controller Delegates (Within Same Process)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderController
  → calls orderService.createOrder(data)  [direct function call]

STEP 3 — Business Logic Runs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
orderService.createOrder(data)
  → validates input
  → calls userService.getUserById(userId)         [direct call]
  → calls productService.checkStock(productId)    [direct call]
  → calls paymentService.charge(amount, card)     [direct call]
  → calls orderRepository.save(orderData)         [direct call]

STEP 4 — Database Operations
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
orderRepository.save() runs SQL on the shared database
productRepository.decrementStock() runs SQL on same database
  ↓
Both happen in one database TRANSACTION → either both succeed or both fail

STEP 5 — Response Flows Back
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Order saved → Controller formats JSON → 201 Created response sent

STEP 6 — Side Effects (Still In-Process)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
notificationService.sendOrderConfirmation(order)
  → sends email via SMTP (directly, no queue needed for simple cases)
```

**Total network calls: ZERO between internal components. Everything is a function call.**

---

# Real World Examples

## Example 1: The One-Room Workshop

```
A custom furniture workshop.
One craftsman. One room.
He designs, cuts, assembles, and delivers.

When he has 5 orders/day → perfect.
When he has 500 orders/day → he needs to hire specialists.
```

The monolith works until the volume demands specialization.

## Example 2: Basecamp (Project Management)

Basecamp — the company that made Ruby on Rails — runs their main product as a **monolith** with tens of thousands of businesses using it.

They famously wrote *"The Majestic Monolith"* blog post arguing that monoliths are underrated and often the right choice even at scale.

## Example 3: Early Airbnb, Uber, Twitter

All of them started as monoliths:

```
Airbnb (2008):
  One Rails application.
  Users, listings, bookings, payments — all in one app.
  Scaled to millions of users on this monolith.
  Only later split into services when the pain became undeniable.

Uber (2010):
  One Python application.
  All of dispatch, drivers, payments in one codebase.

Twitter (2006):
  One Ruby on Rails monolith called "twttr".
  Struggled with scale around 2008-2010 (the "Fail Whale" era).
  Eventually moved away from the monolith.
```

The pattern: **Start monolith → Grow → Hit scale/team limits → Migrate.**

## Example 4: GitHub

GitHub itself ran as a **massive Ruby on Rails monolith** for over a decade. They only recently started breaking it apart — and even then, they describe it as an "organized monolith."

---

# Implementation

## Basic Monolith Structure (Node.js / Express)

```
myapp/
│
├── src/
│   ├── routes/            ← HTTP routing
│   │   ├── users.js
│   │   ├── orders.js
│   │   └── products.js
│   │
│   ├── services/          ← Business logic
│   │   ├── userService.js
│   │   ├── orderService.js
│   │   └── productService.js
│   │
│   ├── repositories/      ← Database access
│   │   ├── userRepo.js
│   │   └── orderRepo.js
│   │
│   ├── models/            ← Database models/schema
│   │   ├── User.js
│   │   └── Order.js
│   │
│   └── app.js             ← One entry point
│
├── package.json
└── Dockerfile             ← One container, all of the above
```

## Starting the Application

```javascript
// src/app.js — Single Entry Point

const express = require('express');
const userRoutes = require('./routes/users');
const orderRoutes = require('./routes/orders');
const productRoutes = require('./routes/products');
const db = require('./database/db');

const app = express();

// All routes registered in one place
app.use('/users', userRoutes);
app.use('/orders', orderRoutes);
app.use('/products', productRoutes);

// One process, one port
app.listen(3000, () => {
    console.log('Monolith running on port 3000');
});
```

## Direct Service-to-Service Communication (No Network)

```javascript
// services/orderService.js

const userService = require('./userService');     // Direct import
const productService = require('./productService'); // Direct import
const paymentService = require('./paymentService'); // Direct import

async function createOrder(orderData) {
    // All of these are local function calls — no HTTP, no retries
    const user = await userService.getUser(orderData.userId);
    const product = await productService.getProduct(orderData.productId);
    await paymentService.charge(user.card, product.price);

    return await orderRepository.save(orderData);
}
```

## Deploying as One Unit

```dockerfile
# Dockerfile — One container for the entire application

FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "src/app.js"]
```

```yaml
# docker-compose.yml — Simple deployment

version: '3'
services:
  app:
    build: .
    ports:
      - "3000:3000"
  database:
    image: postgres:15
```

---

# Best Practices

## ✅ DO

| Practice | Why |
|---|---|
| **Organize by domain/feature** | Group user code, order code, product code into modules |
| **Use layered architecture internally** | Even a monolith should have controllers → services → repositories |
| **Write automated tests** | Test services and repositories independently |
| **Use database migrations** | Track schema changes with tools like Flyway or Liquibase |
| **Set up monitoring early** | Logs, metrics, and error tracking from day one |
| **Plan for future extraction** | Write clean interfaces even if everything is in one place now |
| **Keep modules loosely coupled internally** | Avoid modules directly importing each other's repositories |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| **No internal structure ("Big Ball of Mud")** | All files at the root, no organization → unmanageable quickly |
| **All logic in route handlers** | Impossible to test or reuse |
| **Shared database tables across all features** | Schema changes break everything at once |
| **Ignoring growth signals** | When deployment takes 30+ minutes or deploy frequency slows → time to consider splitting |
| **Fear of the monolith** | Starting with microservices when a monolith would suffice is over-engineering |

---

# Industry Standards

## When the World's Best Companies Chose the Monolith

```
Stack Overflow (2023):
  One of the highest-traffic tech sites in the world.
  Runs on a single application server (with replicas for availability).
  500 million+ page views/month. Still a monolith.

Shopify:
  Runs an enormous Ruby on Rails monolith called "Shopify Core".
  Serves millions of merchants.
  They've invested heavily in making their monolith scale rather than splitting it.

GitHub:
  Rails monolith for over a decade.
  150+ million repos. Still largely monolithic.
```

**The takeaway from industry:** Monoliths don't automatically fail at scale. Poor engineering fails at scale.

## When Companies Moved Away

```
Amazon (early 2000s):
  Moved from a monolith to services after hitting team coordination walls.
  The famous "two-pizza team" rule required independent deployable services.

Netflix (2009-2012):
  A single DVD-delivery monolith → streaming platform needed independent scaling.
  Moved to microservices and built much of the modern microservices tooling.

Uber (2014-2016):
  Outgrew the monolith due to global expansion requiring independent teams.
```

**Pattern:** The trigger to move was usually **team scale** and **organizational complexity**, not technical scale alone.

---

# Common Mistakes

## Mistake 1: The Big Ball of Mud

```
What it looks like:

myapp/
├── user.js          ← Contains order logic too
├── order.js         ← Imports from user.js and product.js
├── product.js       ← Calls payment code directly
├── payment.js       ← Also handles email sending
└── helpers.js       ← 5000 lines, everything else

Result:
  No clear ownership.
  Changing one file breaks three others.
  New developers quit.
```

**Fix:** Enforce clear module boundaries from day one.

## Mistake 2: Ignoring the Signs That It's Time to Split

```
Warning Signs:
  ✗ Build takes > 20 minutes
  ✗ Tests take > 30 minutes
  ✗ Deploying takes > 1 hour
  ✗ One team's changes constantly break another team's work
  ✗ Cannot scale just one hot component (e.g., image processing)
  ✗ Different features need different technology choices
```

## Mistake 3: Building a "Distributed Monolith"

Splitting code into services but keeping tight dependencies:

```
❌ Anti-Pattern:
Service A must be deployed before Service B.
Service C directly imports Service A's database schema.
Service D must call Service B synchronously or it fails.

This is a monolith in distributed clothing.
You get all the complexity of microservices
and none of the benefits.
```

## Mistake 4: Starting with Microservices

```
A new startup builds 12 microservices for a product
they haven't validated yet.

6 months later:
  → 3 of the 12 services were for features nobody wanted
  → Still no working product
  → Team burned out managing 12 CI/CD pipelines
  → Would have shipped in 6 weeks with a monolith
```

---

# Security & Performance Considerations

## Security

```
Monolith Security Profile
│
├── Advantages
│   ├── Single attack surface to secure
│   ├── No inter-service network traffic to encrypt
│   └── Authentication happens in one place
│
└── Risks
    ├── One compromise = all features affected
    │   → A bug in the payment code can affect user data
    │
    ├── All features share the same process/memory
    │   → A memory leak in one feature can crash everything
    │
    └── Database permissions are shared
        → One module can read another module's sensitive tables
```

**Mitigation:**
- Use database views or schemas to isolate sensitive data
- Apply middleware-level authorization checks per route
- Use environment variables for secrets (never hardcode)

## Performance

```
Monolith Performance Profile
│
├── Advantages
│   ├── No network latency between components
│   ├── Shared memory → fast inter-module communication
│   └── Database transactions across the whole app
│
└── Challenges
    ├── Vertical Scaling Only (initially)
    │   → Scale the whole app, not just the busy part
    │
    ├── Memory bloat
    │   → All features loaded in memory even if idle
    │
    └── One bottleneck affects everything
        → Slow report generation slows down user logins
```

**Performance Solutions:**
```
1. Horizontal Scaling (run multiple instances behind a load balancer)
   [Load Balancer]
     ├── [App Instance 1]
     ├── [App Instance 2]
     └── [App Instance 3]
         All share the same database

2. Caching (Redis) for hot data

3. Background Jobs — offload slow tasks (emails, reports, exports)
   [App] → enqueue → [Background Worker] → process separately

4. Read Replicas — separate read traffic from write traffic
```

---

# Related Technologies

| Technology | Relationship |
|---|---|
| **Modular Monolith** | An evolution — a monolith organized into strict internal modules |
| **Microservices** | The architecture pattern most monoliths eventually migrate toward |
| **Docker** | Monoliths are commonly containerized with one Dockerfile |
| **Ruby on Rails** | The most famous monolith-friendly framework (used by GitHub, Shopify) |
| **Django** | Python's equivalent — a batteries-included framework ideal for monoliths |
| **Spring Boot (Java)** | Common for enterprise Java monoliths |
| **Layered Architecture** | The internal structure pattern most monoliths use |
| **Horizontal Scaling** | Running multiple copies of the monolith behind a load balancer |
| **Background Workers (Sidekiq, Celery)** | Offload slow work from the main monolith process |
| **Database Migrations (Flyway, Liquibase)** | Manage database schema changes in a monolith safely |

---

# Summary

## What We Learned

- A **monolith** is a single application where all features, all logic, and all data access live in one codebase, run in one process, and deploy as one unit.
- It is the **oldest, simplest, and fastest-to-build** architecture.
- All components communicate via **direct function calls** — no network, no latency.
- They share a **single database**, making transactions trivial.
- Three forms exist: Classic Monolith, Modular Monolith, and Distributed Monolith (anti-pattern).
- **Monoliths are not "bad"** — companies like Stack Overflow, Shopify, and GitHub run them successfully at enormous scale.
- The right time to move away is driven by **team size and organizational complexity**, not just technical load.
- The **biggest mistake** is starting with microservices when a monolith would suffice.

## Key Takeaways

- Think of a monolith as a one-room workshop — perfect when you're starting out.
- Use layered architecture internally so the monolith stays organized.
- Watch for the warning signs: slow builds, slow deploys, team conflicts on every merge.
- Resist the urge to over-engineer early. Validate the product first.
- The path is: Monolith → Modular Monolith → Selective Service Extraction → Microservices (if needed).

---

# Keywords

- Monolithic Architecture
- Single Codebase
- Single Deployment Unit
- Shared Database
- In-Process Communication
- Modular Monolith
- Distributed Monolith
- Horizontal Scaling
- Big Ball of Mud
- Vertical Scaling
- Background Workers
- Majestic Monolith
- Two-Pizza Team
- Over-Engineering

---

# Glossary

| Term | Meaning |
|---|---|
| **Monolithic Architecture** | An application where all functionality lives in one codebase and deploys as one unit |
| **Single Deployment Unit** | The entire application is packaged and deployed as one artifact |
| **In-Process Communication** | Components talk via direct function calls, not network requests |
| **Shared Database** | All parts of the monolith use the same database instance |
| **Modular Monolith** | A monolith with strict internal module boundaries — organized, clean, but still one deployment |
| **Distributed Monolith** | An anti-pattern: multiple services that are tightly coupled and can't be deployed independently |
| **Big Ball of Mud** | An anti-pattern: a system with no internal structure — everything depends on everything |
| **Vertical Scaling** | Making a single server bigger (more CPU, RAM) to handle more load |
| **Horizontal Scaling** | Running multiple copies of the application behind a load balancer |
| **Background Worker** | A separate process that handles slow/async tasks (emails, exports, reports) |
| **Technical Debt** | The future cost of taking shortcuts now — monoliths accumulate it if not organized |
| **Two-Pizza Team** | Amazon's rule: a team should be small enough to be fed by two pizzas — a driver for service independence |
| **Over-Engineering** | Building a more complex system than the problem requires |

## Next Recommended Chapters

- [04. Modular Monolith](./04-Modular-Monolith.md)
- [06. Microservices Architecture](./06-Microservices-Architecture.md)
- [02. Layered Architecture](./02-Layered-Architecture.md)
