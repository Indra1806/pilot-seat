> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine a **shopping mall**.

From the outside, it looks like one big building — one roof, one address, one parking lot. That's the monolith part.

But inside, the mall is divided into clearly separated **stores**. Each store:
- Has its own staff
- Has its own inventory
- Has its own cash register
- Has its own rules and products

Stores don't go into each other's back-rooms. They don't share inventory. They are **independent on the inside** while still being **together on the outside**.

That is the **Modular Monolith**.

A Modular Monolith is a single application (one codebase, one deployment) that is internally divided into **well-defined, loosely coupled modules** — each representing a distinct business capability.

Unlike a classic monolith where everything is tangled together, a modular monolith is **structured, organized, and disciplined** on the inside.

Unlike microservices, it is still **one deployable unit** — no distributed system complexity.

It is widely regarded as the **ideal middle ground** between simplicity and structure.

---

# Why It Exists

Two pain points drove the creation of the Modular Monolith pattern:

### Pain Point 1: The Classic Monolith Got Messy

Classic monoliths often degraded into the **Big Ball of Mud**:

```
myapp/
├── user.js           → but also contains order logic
├── order.js          → imports directly from product.js and payment.js
├── product.js        → secretly reads the user database table
├── payment.js        → sends emails AND processes payments
└── helpers.js        → 4000 lines of "stuff"
```

No structure. No rules. No boundaries. Changing anything broke everything.

### Pain Point 2: Microservices Were Too Complex Too Soon

Engineers trying to fix the messy monolith moved to microservices — and discovered:

```
New Problems with Microservices Too Early:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ 12 separate services to manage
→ 12 CI/CD pipelines
→ Distributed transaction nightmares
→ Network failures between services
→ Service discovery, load balancing, health checks
→ All this overhead for a 5-person team
```

The Modular Monolith emerged as the answer:

> "What if we got the **discipline of microservices** (clear boundaries, single responsibility) without the **operational complexity of microservices** (distributed systems)?"

---

# Problem It Solves

**Before — Classic Monolith (No Structure):**

```
Any code can call any other code.
Any module can read any database table.
Changes in one area ripple unpredictably.
Teams step on each other constantly.
```

**After — Modular Monolith:**

```
Modules have clear boundaries.
A module's internal code is private.
Modules communicate through defined public APIs.
Each module owns its own database tables/schema.
Teams work independently on their modules.
```

**Visually:**

```
❌ Classic Messy Monolith         ✅ Modular Monolith
━━━━━━━━━━━━━━━━━━━━━━━           ━━━━━━━━━━━━━━━━━━━━━━━
[users] ←→ [orders]               [UserModule] | [OrderModule]
   ↓     ↗     ↓                     ↓ API only     ↓ API only
[payments] ←→ [products]          [PaymentModule] | [ProductModule]
   ↑  ↙          ↗                   (modules don't cross borders)
[notifications] ←→ [email]
(everything knows about everything)
```

---

# Core Concepts

## 1. Module — The Core Unit

A module is a self-contained unit that represents **one business capability**:

```
Module = Business Capability + Code + Data Ownership

Examples:
  UserModule     → everything about users
  OrderModule    → everything about orders
  ProductModule  → everything about products
  PaymentModule  → everything about payments
```

Each module has:
- **Public API** — what it exposes to other modules
- **Private internals** — its services, repositories, and internal logic
- **Owned data** — its own database tables or schema

## 2. Boundaries — The Sacred Rule

Modules enforce strict boundaries:

```
✅ Allowed:
  OrderModule calls UserModule's PUBLIC API
  → orderService.createOrder() → UserModule.getUser(id)

❌ Forbidden:
  OrderModule directly queries the users database table
  → SELECT * FROM users WHERE id = ?   ← a module cannot do this for another module's table

❌ Forbidden:
  OrderModule imports UserModule's private repository
  → const userRepo = require('../user/userRepository')  ← violates boundaries
```

## 3. Public API — The Contract

Each module exposes only what is necessary:

```
UserModule/
├── index.js        ← PUBLIC API (what other modules can use)
│   └── exports: { getUser, getUserProfile }
│
├── userService.js  ← PRIVATE (not visible to other modules)
├── userRepo.js     ← PRIVATE
└── userModel.js    ← PRIVATE
```

If another module needs user data, it calls `UserModule.getUser()` — not the repository directly.

## 4. Data Ownership — Each Module Owns Its Tables

```
Database
├── users             ← Owned by UserModule ONLY
├── user_sessions     ← Owned by UserModule ONLY
├── orders            ← Owned by OrderModule ONLY
├── order_items       ← Owned by OrderModule ONLY
├── products          ← Owned by ProductModule ONLY
└── payments          ← Owned by PaymentModule ONLY

Rule: No module writes to another module's tables.
      No module reads another module's tables directly.
      Cross-module data access goes through the public API.
```

---

# Architecture / Components

## Overall Structure

```
┌────────────────────────────────────────────────────────────────┐
│                    MODULAR MONOLITH                             │
│                                                                 │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                  Presentation Layer                     │    │
│  │    HTTP Router / REST API / GraphQL / Event Handlers   │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────┐ ┌────────┐  │
│  │  UserModule │ │ OrderModule │ │ProductModule │ │Payment │  │
│  │─────────────│ │─────────────│ │──────────────│ │Module  │  │
│  │ Public API  │ │ Public API  │ │ Public API   │ │────────│  │
│  │─────────────│ │─────────────│ │──────────────│ │Pub API │  │
│  │  [private]  │ │  [private]  │ │   [private]  │ │[priv.] │  │
│  │  services   │ │  services   │ │   services   │ │service │  │
│  │  repos      │ │  repos      │ │   repos      │ │repos   │  │
│  └──────┬──────┘ └──────┬──────┘ └──────┬───────┘ └───┬────┘  │
│         └───────────────┴───────────────┴─────────────┘        │
│                              ↓                                  │
└──────────────────────────────┼─────────────────────────────────┘
                               ↓
               ┌────────────────────────────┐
               │         Database           │
               │  ┌──────┐  ┌──────┐        │
               │  │users │  │orders│  ...   │
               │  └──────┘  └──────┘        │
               └────────────────────────────┘
```

## Module Internal Structure

Each module follows layered architecture internally:

```
UserModule/
├── index.js            ← Public API boundary
├── api/
│   └── userController.js  ← HTTP handlers
├── application/
│   └── userService.js     ← Business logic (private)
├── domain/
│   └── User.js            ← Domain entity/model
└── infrastructure/
    ├── userRepository.js  ← Database access (private)
    └── userMapper.js      ← Map DB rows to domain objects
```

## Cross-Module Communication

```
Option 1: Direct Function Call (Synchronous)
  OrderModule → UserModule.getUser(id) → returns user data

Option 2: Internal Event Bus (Asynchronous)
  UserModule emits: "user.created" event
  OrderModule listens: "user.created" event
  → No direct dependency between modules
```

---

# Workflow

**Scenario: Customer places an order. System must verify user, check stock, and charge payment.**

```
STEP 1 — HTTP Request
━━━━━━━━━━━━━━━━━━━━━
POST /orders
  ↓
OrderController (Presentation Layer)

STEP 2 — OrderModule Takes Control
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderModule.createOrder(data)
  ↓
Internally calls the OrderService (private)

STEP 3 — OrderModule Calls Other Modules via Public APIs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderService calls → UserModule.getUser(userId)
                   → ProductModule.checkStock(productId)
                   → PaymentModule.charge(amount, card)

Note: OrderModule does NOT:
  - Read from the "users" table directly
  - Import UserRepository
  - Access PaymentModule's internal services

STEP 4 — Each Module Does Its Own Internal Work
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UserModule internally: validates user, returns user data
ProductModule internally: checks stock table, reserves stock
PaymentModule internally: charges via payment gateway

STEP 5 — OrderModule Saves the Order
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderRepository.save(order) → inserts into "orders" table

STEP 6 — Event Published (Optional, Async)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderModule emits: "order.created" event
NotificationModule (listening): sends confirmation email

STEP 7 — Response Returned
━━━━━━━━━━━━━━━━━━━━━━━━━━
201 Created → order data returned to caller
```

---

# Real World Examples

## Example 1: Shopping Mall

```
The Mall (One Building = One Application)
│
├── [Apple Store]     → UserModule (customers/accounts)
│     Staff, inventory, and cash register = private internals
│     "Public API" = the store entrance, products displayed
│
├── [Food Court]      → OrderModule
│     Takes orders from customers, but doesn't manage inventory
│     Calls the kitchen (ProductModule) to check what's available
│
└── [Bank Branch]     → PaymentModule
      Handles all financial transactions
      Other stores don't manage money directly — they go to the bank
```

## Example 2: Shopify

Shopify has publicly described their Ruby on Rails monolith as being organized into components (modules) that have clear ownership and defined interfaces.

```
Shopify Core (Monolith)
├── Storefront Module     → buyer-facing shopping experience
├── Merchant Module       → seller tools and dashboard
├── Order Module          → order lifecycle management
├── Billing Module        → subscription and payment handling
└── Analytics Module      → reporting and insights
```

They enforce component boundaries within the monolith to prevent the "big ball of mud" problem.

## Example 3: A SaaS Product (Before Going Full Microservices)

```
startup-saas/
├── modules/
│   ├── auth/             ← Authentication module
│   ├── workspace/        ← Multi-tenant workspace management
│   ├── billing/          ← Subscription and payments
│   ├── projects/         ← Core project management feature
│   └── notifications/    ← Email/in-app alerts
└── shared/
    ├── database/         ← Single DB connection pool
    └── events/           ← Internal event bus
```

Each module is independently testable. When it's time to extract `billing` into its own service, the boundary is already clearly defined.

---

# Implementation

## Folder Structure (Node.js Example)

```
src/
├── modules/
│   │
│   ├── user/
│   │   ├── index.js          ← PUBLIC: { getUser, createUser }
│   │   ├── userController.js ← HTTP handlers
│   │   ├── userService.js    ← Private business logic
│   │   ├── userRepository.js ← Private DB access
│   │   └── user.model.js     ← Domain object
│   │
│   ├── order/
│   │   ├── index.js          ← PUBLIC: { createOrder, getOrder }
│   │   ├── orderController.js
│   │   ├── orderService.js
│   │   └── orderRepository.js
│   │
│   └── payment/
│       ├── index.js          ← PUBLIC: { charge, refund }
│       ├── paymentService.js
│       └── paymentRepository.js
│
├── shared/
│   ├── database/db.js        ← Shared DB connection
│   ├── events/eventBus.js    ← Internal event system
│   └── middleware/auth.js    ← Shared middleware
│
└── app.js                    ← Single entry point
```

## Public API Boundary (index.js)

```javascript
// modules/user/index.js — The ONLY file other modules import

const userService = require('./userService');

// Only expose what other modules need
module.exports = {
    getUser: (id) => userService.getUser(id),
    getUserProfile: (id) => userService.getProfile(id),
    // userRepository is NOT exported — it's private
};
```

## Cross-Module Call (Correct Pattern)

```javascript
// modules/order/orderService.js

// ✅ Import from the module's public API (index.js)
const UserModule = require('../user');
const ProductModule = require('../product');
const PaymentModule = require('../payment');

async function createOrder(orderData) {
    // Call other modules through their public APIs
    const user = await UserModule.getUser(orderData.userId);
    const product = await ProductModule.getProduct(orderData.productId);

    if (product.stock < orderData.quantity) {
        throw new Error('Insufficient stock');
    }

    await PaymentModule.charge(orderData.paymentMethod, product.price);

    return await orderRepository.save({
        userId: user.id,
        productId: product.id,
        quantity: orderData.quantity,
    });
}
```

## Internal Event Bus (Async Communication)

```javascript
// shared/events/eventBus.js

const EventEmitter = require('events');
const eventBus = new EventEmitter();

module.exports = eventBus;
```

```javascript
// modules/order/orderService.js — Publishing an event
const eventBus = require('../../shared/events/eventBus');

async function createOrder(data) {
    const order = await orderRepository.save(data);
    eventBus.emit('order.created', order); // Publish event
    return order;
}
```

```javascript
// modules/notification/notificationListener.js — Listening
const eventBus = require('../../shared/events/eventBus');

eventBus.on('order.created', async (order) => {
    await sendConfirmationEmail(order);
});
```

---

# Best Practices

## ✅ DO

| Practice | Why |
|---|---|
| **Define a strict public API per module** | Forces you to think about what's truly shared vs. private |
| **One module owns each database table** | Prevents data corruption and conflicting writes |
| **Use an internal event bus for async cross-module communication** | Keeps modules decoupled |
| **Test each module in isolation** | Each module should be independently testable |
| **Name modules after business capabilities** | `user`, `order`, `billing` — not technical layers |
| **Document the public API of each module** | Write a README per module describing what it exposes |
| **Use database schemas per module** | Even in a shared DB, separate schemas (`user.users`, `order.orders`) enforce boundaries |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| **Import private internals from another module** | Violates boundaries, creates hidden dependencies |
| **Share database tables between modules** | One module's migration breaks the other module |
| **Use "shared services" for business logic** | A `SharedService` containing logic for multiple modules is a boundary violation |
| **Make the public API too large** | If a module exposes 30 functions, it has no boundary |
| **Create circular dependencies between modules** | Module A imports Module B which imports Module A — deadlock |

---

# Industry Standards

## Recognized Architectural Patterns

The Modular Monolith is recognized in major architectural literature:

- **"Building Microservices" by Sam Newman** explicitly recommends starting with a modular monolith and extracting services only when you feel the pain.
- **Martin Fowler (ThoughtWorks)** describes the Modular Monolith as the preferred starting architecture for most projects.
- **"Monolith to Microservices"** (Sam Newman, 2020) dedicates chapters to structuring your monolith with clear modules *before* you attempt extraction.

## Real Company Patterns

```
Shopify:
  Their "Component-based Rails" architecture is a Modular Monolith.
  Components have clear ownership and defined interfaces.
  Used at enormous scale.

Basecamp:
  37signals (makers of Basecamp/HEY) explicitly advocate for the
  Modular Monolith as a first-class architecture choice.

Stack Overflow:
  C# monolith with clear namespace/module separation.
  Teams own specific namespaces.
```

## When to Extract a Module to a Service

```
Signal to extract a module:
  ✓ The module needs to scale independently (e.g., image processing)
  ✓ The module needs a different technology (e.g., ML/Python for recommendations)
  ✓ A separate team owns the module end-to-end
  ✓ The module's deployment risk is too high to couple to the main app

The advantage of a Modular Monolith:
  The module boundary is already clean.
  Extraction becomes a matter of:
    1. Move the module's code to a new repo
    2. Replace the direct function call with an HTTP/gRPC call
    3. Deploy independently
```

---

# Common Mistakes

## Mistake 1: Modules That Are Just Folders

```
❌ Wrong — "modules" as folders but with no enforced boundaries:

modules/
├── user/
│   └── userService.js   ← imports from order/orderRepository.js
├── order/
│   └── orderService.js  ← reads directly from user DB table
```

If there are no boundaries enforced, it's just a classic monolith with a different folder structure.

## Mistake 2: Shared "God" Service

```
❌ Wrong:
shared/
└── appService.js   ← contains logic for users, orders, AND products

This is just the business logic from a classic monolith
moved into a "shared" folder. Not modular at all.
```

## Mistake 3: Too Many Modules

```
❌ Wrong:
modules/
├── email-validation/
├── phone-validation/
├── password-hashing/
├── token-generation/
...
30 more modules...

These are utilities, not business capabilities.
They should be in a shared/utils folder, not modules.
```

**A module = a business capability.** If you can't describe your module as a business function ("manage users", "process payments", "track orders"), it's probably not a module.

## Mistake 4: Circular Module Dependencies

```
❌ Wrong:
UserModule imports → OrderModule.getOrderCount(userId)
OrderModule imports → UserModule.getUser(userId)

This creates a circular dependency:
  UserModule can't load without OrderModule
  OrderModule can't load without UserModule

Fix: Use events. OrderModule emits "order.count.requested".
     UserModule listens and responds.
Or: Extract the dependency into a third module.
```

---

# Security & Performance Considerations

## Security

```
Modular Monolith Security
│
├── Advantages over Classic Monolith
│   ├── Payment module's code is clearly isolated
│   ├── Easier to audit who can access what data
│   └── Clear ownership means clearer security responsibility
│
├── Shared Risk (Same as Monolith)
│   ├── Still one process — one exploit can affect all modules
│   └── Shared database connection pool
│
└── Mitigation Strategies
    ├── Database schemas per module (schema-level permissions)
    ├── Module-level authorization checks
    └── Sensitive modules (billing) with extra logging/auditing
```

## Performance

```
Modular Monolith Performance
│
├── Same Advantages as Monolith
│   ├── No network latency between modules
│   ├── In-process function calls are near-instant
│   └── Database transactions work across modules easily
│
├── Potential Issues
│   ├── Large module count → large application memory footprint
│   └── One slow module's operations block other requests
│       (mitigated by async non-blocking code)
│
└── Performance Solutions
    ├── Horizontal scaling (multiple instances behind load balancer)
    ├── Module-specific caching
    └── Async event bus for non-critical cross-module operations
```

---

# Related Technologies

| Technology | Relationship |
|---|---|
| **Monolithic Architecture** | The unstructured predecessor — Modular Monolith is an organized evolution |
| **Microservices** | The distributed evolution — Modular Monolith is the recommended stepping stone |
| **Domain-Driven Design (DDD)** | Modules map to DDD "Bounded Contexts" — the same concept at the domain level |
| **Layered Architecture** | Used *within* each module as the internal structure |
| **Event-Driven Architecture** | Used *between* modules for asynchronous, decoupled communication |
| **Bounded Context** | DDD term for the boundary around a domain concept — maps directly to a module |
| **Ruby on Rails Engines** | Rails mechanism for creating modular components in a Rails monolith |
| **Java Packages** | Namespace-level module separation in Java monoliths |
| **NestJS Modules** | NestJS (Node.js framework) has built-in module system that enforces this pattern |
| **Clean Architecture** | Modular Monolith modules often adopt Clean Architecture internally |

---

# Summary

## What We Learned

- A **Modular Monolith** is a single-deployment application internally divided into **clearly bounded, loosely coupled modules** — each representing a distinct business capability.
- Each module has a **public API** (what it exposes) and **private internals** (what it hides).
- Modules communicate through their public APIs only — never through direct database table access or importing private code.
- Each module **owns its own database tables/schema**.
- Modules can communicate asynchronously via an **internal event bus**.
- It is the recommended **stepping stone** before microservices — boundaries are clean, extraction is straightforward when the time comes.
- Companies like Shopify and Basecamp use this pattern successfully at large scale.

## Key Takeaways

- Think of the Modular Monolith as a **shopping mall**: one building, many independently run stores.
- The golden rule: **a module's private code is private**. Other modules use the public API — nothing else.
- Name modules after **business capabilities**, not technical concerns.
- If a module's boundary is clean, extracting it to a microservice later is straightforward.
- Start here. Migrate to microservices only when you feel the real pain.

---

# Keywords

- Modular Monolith
- Module
- Bounded Context
- Public API
- Private Internals
- Data Ownership
- Event Bus
- Domain-Driven Design
- Module Extraction
- Single Deployment Unit
- Module Boundary
- Loose Coupling
- Internal Event
- Circular Dependency
- Business Capability

---

# Glossary

| Term | Meaning |
|---|---|
| **Modular Monolith** | A single-deployment application with strong internal module boundaries |
| **Module** | A self-contained unit representing one business capability (e.g., UserModule, OrderModule) |
| **Public API** | The defined interface a module exposes to other modules — typically via an index.js or facade class |
| **Private Internals** | The services, repositories, and logic inside a module that other modules cannot access directly |
| **Bounded Context** | A DDD term for the domain boundary around a module — the "context" in which a concept has a specific meaning |
| **Data Ownership** | Each module owns specific database tables and is the only one that can read/write them directly |
| **Event Bus** | An in-process pub/sub system allowing modules to communicate asynchronously without direct dependencies |
| **Module Extraction** | The process of taking a clean module out of the monolith and deploying it as an independent microservice |
| **Circular Dependency** | When Module A depends on Module B which depends on Module A — a design error |
| **Loose Coupling** | Modules depend on each other as little as possible, only through defined public interfaces |
| **Business Capability** | A thing the business does: "manage orders", "process payments" — the correct basis for defining modules |

## Next Recommended Chapters

- [05. Service-Oriented Architecture](./05-Service-Oriented-Architecture.md)
- [06. Microservices Architecture](./06-Microservices-Architecture.md)
- [08. Clean Architecture](./08-Clean-Architecture.md)
