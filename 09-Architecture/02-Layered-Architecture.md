> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine a **restaurant**.

When you walk in and order a burger, you don't go into the kitchen, grab the ingredients, cook them yourself, and then wash your own plate.

Instead, the restaurant is divided into **layers of responsibility**:

```
You (Customer)
    ↓
Waiter (Takes your order, talks to kitchen)
    ↓
Chef (Cooks the food using recipes and rules)
    ↓
Storage Room (Has all the raw ingredients)
```

Every person only does **their job**. The waiter doesn't cook. The chef doesn't talk to customers. The storage room doesn't take orders.

This is **Layered Architecture** — a software design pattern where an application is divided into distinct **horizontal layers**, each with a **single, clear responsibility**, and each communicating only with the layer directly above or below it.

It is also called:
- **N-Tier Architecture** (where N = the number of layers)
- **Multi-Tier Architecture**
- **Tiered Architecture**

It is one of the **oldest and most widely used** architectural patterns in software engineering.

---

# Why It Exists

In the early days of software, everything was written in one giant file.

Code that handled the user interface, the business logic, and the database was all **mixed together**.

```
[ One Giant Mess ]

User clicks button
→ Code runs SQL query
→ Code validates the input
→ Code renders the result
→ All in one function
```

This created massive problems:

- **Impossible to maintain:** Changing the UI broke the database code.
- **Impossible to test:** You couldn't test business logic without a running UI.
- **Impossible to scale:** You couldn't add more servers for just one part.
- **Impossible to reuse:** You couldn't reuse database code in another project without copying everything.
- **Impossible to understand:** A new developer had no idea where anything lived.

Engineers needed a way to **separate responsibilities** clearly.

The solution was to stack the application into clean, organized **horizontal layers** — like floors in a building.

---

# Problem It Solves

**Before Layered Architecture:**

```
┌─────────────────────────────────────────────────┐
│              One Tangled Application             │
│                                                  │
│  HTML + SQL + Business Rules + Validation        │
│  + UI Logic + Database Calls = All Mixed         │
└─────────────────────────────────────────────────┘
```

Problems:
- Change the button color → risk breaking the database query
- Test the pricing rule → must launch the whole UI first
- Scale the app → must scale everything, not just the busy part

**After Layered Architecture:**

```
┌─────────────────────────────────────────────────┐
│            Presentation Layer (UI)               │
│        React, HTML, Mobile App, REST API         │
├─────────────────────────────────────────────────┤
│            Business Logic Layer                  │
│     Rules, Validation, Pricing, Workflow         │
├─────────────────────────────────────────────────┤
│            Data Access Layer                     │
│     SQL Queries, ORM, Repository Pattern         │
├─────────────────────────────────────────────────┤
│                 Database                         │
│        PostgreSQL, MySQL, MongoDB                │
└─────────────────────────────────────────────────┘
```

Benefits:
- Change the UI → business logic is untouched
- Test pricing rules → inject fake data, no UI needed
- Scale the backend → leave the database alone

---

# Core Concepts

## 1. Layers Are Stacked Horizontally

Each layer sits on top of another. Information flows **downward** (requests) and **upward** (responses).

```
User Action
    ↓   (request flows down)
Presentation Layer
    ↓
Business Layer
    ↓
Data Access Layer
    ↓
Database

Database Result
    ↑   (response flows up)
Data Access Layer
    ↑
Business Layer
    ↑
Presentation Layer
    ↑
User Sees Result
```

## 2. Each Layer Has One Job

| Layer               | Its One Job                                    |
|---------------------|------------------------------------------------|
| Presentation        | Talk to the user. Show data. Collect input.    |
| Business Logic      | Apply rules. Validate. Calculate. Decide.      |
| Data Access         | Read and write to the database. Nothing else.  |
| Database            | Store data permanently.                        |

## 3. Strict Layer Rules

The golden rule of layered architecture:

> **Each layer can ONLY talk to the layer directly below it.**

- ✅ Presentation → Business Layer (allowed)
- ✅ Business Layer → Data Access Layer (allowed)
- ❌ Presentation → Database directly (forbidden)
- ❌ Business Layer → Presentation (forbidden — layers don't talk upward)

This is called **layer isolation** or **layer enforcement**.

---

# Architecture / Components

## The Classic 3-Tier Model

The most common form of layered architecture:

```
┌───────────────────────────────────────────────────┐
│                TIER 1: PRESENTATION               │
│                                                   │
│   What the user sees and interacts with.          │
│   → Web browser, mobile app, desktop app          │
│   → Sends requests to the Business Layer          │
│   → Renders the response for the user             │
└───────────────────────┬───────────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────────┐
│              TIER 2: BUSINESS LOGIC               │
│                                                   │
│   Where the "brain" of the application lives.     │
│   → Validates input (e.g., "email must be valid") │
│   → Applies rules (e.g., "discount if > $100")    │
│   → Orchestrates what needs to happen             │
│   → Talks to Data Access when it needs data       │
└───────────────────────┬───────────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────────┐
│              TIER 3: DATA ACCESS                  │
│                                                   │
│   The only layer that touches the database.       │
│   → Runs SQL queries or ORM calls                 │
│   → Handles connection pooling                    │
│   → Returns raw data back to Business Layer       │
└───────────────────────┬───────────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────────┐
│                   DATABASE                        │
│                                                   │
│   Where data is stored permanently.               │
│   → PostgreSQL, MySQL, MongoDB, Redis             │
└───────────────────────────────────────────────────┘
```

## The Extended 4-Tier Model

Some systems add a **Service Layer** (also called an Application Layer):

```
┌─────────────────────────────────────┐
│        Presentation Layer           │  ← UI, API controllers
├─────────────────────────────────────┤
│         Service Layer               │  ← Orchestration, use cases
├─────────────────────────────────────┤
│        Business Logic Layer         │  ← Core domain rules
├─────────────────────────────────────┤
│         Data Access Layer           │  ← Repositories, queries
├─────────────────────────────────────┤
│            Database                 │  ← PostgreSQL, MongoDB
└─────────────────────────────────────┘
```

## What Each Layer Contains (Concrete Examples)

### Presentation Layer
```
Controllers       → Handle HTTP requests (e.g., GET /users)
View Templates    → HTML, JSX, Vue components
API Responses     → Format JSON/XML to send back
Input Handling    → Read form data, query params, headers
```

### Business Logic Layer
```
Services          → UserService, OrderService, PaymentService
Validators        → Check email format, age > 18, price > 0
Rules             → "Apply 10% discount if order > $100"
Workflows         → Order placed → Reserve stock → Charge card
```

### Data Access Layer
```
Repositories      → UserRepository, ProductRepository
ORM Models        → Sequelize, TypeORM, SQLAlchemy models
Query Builders    → Raw SQL wrappers
Connection Pool   → Manages DB connections efficiently
```

---

# Workflow

**Scenario: A user logs into an e-commerce site and views their order history.**

```
STEP 1 — User Request
━━━━━━━━━━━━━━━━━━━━━
User opens browser → types "myshop.com/orders"
Browser sends: GET /orders (with auth cookie)

STEP 2 — Presentation Layer Receives Request
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderController.getOrders() is called
  → Extracts user ID from the auth cookie
  → Calls Business Layer: orderService.getOrdersForUser(userId)

STEP 3 — Business Layer Applies Rules
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderService.getOrdersForUser(userId)
  → Validates: Is userId valid? Is the user authenticated?
  → Applies rules: "Only show last 12 months of orders"
  → Calls Data Layer: orderRepository.findByUserId(userId, last12months)

STEP 4 — Data Access Layer Fetches Data
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderRepository.findByUserId(userId, dateRange)
  → Builds SQL query:
      SELECT * FROM orders
      WHERE user_id = 123
      AND created_at > '2024-06-01'
  → Executes against the database
  → Returns raw order records

STEP 5 — Data Flows Back Up
━━━━━━━━━━━━━━━━━━━━━━━━━━━
Database → Data Layer → Business Layer → Presentation Layer

STEP 6 — Presentation Layer Formats and Responds
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OrderController formats the data as JSON
  → Sends HTTP 200 response with orders array
Browser renders the order history page for the user
```

---

# Real World Examples

## Example 1: Restaurant (The Classic Analogy)

```
┌──────────────────────────────────────────┐
│  CUSTOMER (Presentation Layer)           │
│  "I want a cheeseburger, please."        │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  WAITER (Service / Business Layer)       │
│  "Got it — checks if item is on menu,    │
│   checks if kitchen can handle it,       │
│   sends the order to the chef."          │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  CHEF (Business Logic Layer)             │
│  "Applies cooking recipe (the rules),   │
│   checks inventory for ingredients."    │
└──────────────┬───────────────────────────┘
               │
               ▼
┌──────────────────────────────────────────┐
│  STORAGE ROOM (Data Access Layer)        │
│  "Retrieves raw ingredients from         │
│   the freezer and pantry (database)."   │
└──────────────────────────────────────────┘
```

## Example 2: ATM Machine

```
ATM Screen       → Presentation Layer (shows UI, reads card)
ATM Software     → Business Layer (validates PIN, checks balance rules)
Bank API         → Data Access Layer (talks to the core banking database)
Core Database    → Stores all accounts and transactions
```

## Example 3: Netflix (Simplified)

```
Netflix App (React/Mobile)  → Presentation Layer
Recommendation Engine       → Business Layer (what to show you)
Content API Service         → Data Access Layer (fetches videos, metadata)
Content Database + S3       → Stores the actual movies and show data
```

---

# Implementation

## Folder Structure (Node.js / Express Example)

```
src/
├── controllers/          ← Presentation Layer
│   ├── userController.js
│   └── orderController.js
│
├── services/             ← Business Logic Layer
│   ├── userService.js
│   └── orderService.js
│
├── repositories/         ← Data Access Layer
│   ├── userRepository.js
│   └── orderRepository.js
│
├── models/               ← Database Models (shared with Data Layer)
│   ├── User.js
│   └── Order.js
│
└── database/             ← DB connection setup
    └── db.js
```

## Code Flow Example (JavaScript / Express)

### Presentation Layer — Controller
```javascript
// controllers/orderController.js

const orderService = require('../services/orderService');

async function getOrders(req, res) {
    const userId = req.user.id;               // Get user from auth token
    const orders = await orderService.getOrdersForUser(userId);
    res.json({ success: true, data: orders }); // Format and return response
}

module.exports = { getOrders };
```

### Business Logic Layer — Service
```javascript
// services/orderService.js

const orderRepository = require('../repositories/orderRepository');

async function getOrdersForUser(userId) {
    // Rule: only return orders from the last 12 months
    const twelveMonthsAgo = new Date();
    twelveMonthsAgo.setFullYear(twelveMonthsAgo.getFullYear() - 1);

    // Rule: user ID must be valid
    if (!userId || typeof userId !== 'number') {
        throw new Error('Invalid user ID');
    }

    return await orderRepository.findByUserId(userId, twelveMonthsAgo);
}

module.exports = { getOrdersForUser };
```

### Data Access Layer — Repository
```javascript
// repositories/orderRepository.js

const db = require('../database/db');

async function findByUserId(userId, since) {
    // Only this layer knows about SQL
    const result = await db.query(
        `SELECT * FROM orders
         WHERE user_id = $1 AND created_at > $2
         ORDER BY created_at DESC`,
        [userId, since]
    );
    return result.rows;
}

module.exports = { findByUserId };
```

---

# Best Practices

## ✅ DO

| Practice | Why It Matters |
|---|---|
| **Never skip layers** | Presentation should NEVER call the database directly |
| **Keep layers thin** | Controllers shouldn't contain business rules |
| **Use interfaces/contracts** | Define what each layer expects from the one below |
| **Put validation in the Business Layer** | Don't validate in the controller or the DB layer |
| **Name layers consistently** | controller, service, repository — pick a convention and stick to it |
| **Keep the Data Layer generic** | Repositories should not know about business rules |
| **Use Dependency Injection** | Don't hardcode layer dependencies — inject them |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| Database calls in controllers | Controller becomes a god object; impossible to test |
| Business logic in repositories | Data layer becomes polluted with application rules |
| Layers calling each other in circles | Creates spaghetti flow and infinite loops |
| Skipping the business layer for "simple" operations | "Simple" operations grow over time and become complex |
| Exposing database models directly in API responses | Leaks internal schema; breaks when DB changes |

---

# Industry Standards

## How Big Companies Use Layered Architecture

### Amazon (E-Commerce Core)
```
Web / Mobile App           → Presentation Layer
Order Processing Engine    → Business Logic Layer
Order Data Service         → Data Access Layer
DynamoDB / Aurora          → Database
```

### Google (Search — Simplified)
```
Search UI (search.google.com) → Presentation
Ranking Algorithm              → Business Logic
Index Retrieval Service        → Data Access
Web Index Database             → Storage
```

### Banking Systems (Wells Fargo, HSBC)
- Core banking systems almost universally use layered architecture
- Strict layer separation is often **regulatory required** for audit trails
- The Data Access Layer is heavily secured — nothing reaches the database directly

### Enterprise Java (Spring Framework)
```java
@RestController          // Presentation Layer
@Service                 // Business Logic Layer
@Repository              // Data Access Layer
```
Spring's annotations literally map to the three layers of N-Tier architecture.

---

# Common Mistakes

## Mistake 1: Fat Controllers (Too Much Logic in Presentation)

```javascript
// ❌ WRONG — Controller doing business logic
async function createOrder(req, res) {
    const { userId, items } = req.body;

    // This is business logic — it doesn't belong here!
    if (items.length === 0) throw new Error('Cart is empty');
    const total = items.reduce((sum, item) => sum + item.price, 0);
    if (total > 10000) throw new Error('Order too large');

    // This is database access — it doesn't belong here either!
    await db.query('INSERT INTO orders ...');

    res.json({ success: true });
}
```

```javascript
// ✅ CORRECT — Controller only orchestrates
async function createOrder(req, res) {
    const order = await orderService.createOrder(req.body);
    res.json({ success: true, data: order });
}
```

## Mistake 2: Repositories Containing Business Rules

```javascript
// ❌ WRONG — Business logic inside a repository
async function findEligibleOrders(userId) {
    // This is a business rule — it doesn't belong here!
    const minOrderValue = 50;
    return await db.query(
        `SELECT * FROM orders WHERE user_id = $1 AND total > $2`,
        [userId, minOrderValue]
    );
}
```

```javascript
// ✅ CORRECT — Repository only knows SQL
async function findByUserId(userId) {
    return await db.query(
        `SELECT * FROM orders WHERE user_id = $1`,
        [userId]
    );
}

// Business rule lives in the service layer
async function getEligibleOrders(userId) {
    const orders = await orderRepository.findByUserId(userId);
    return orders.filter(order => order.total > 50); // Rule here
}
```

## Mistake 3: Bypassing Layers (Layer Jumping)

```
❌ Presentation Layer → Database (Direct Call)

This is like the customer walking into the kitchen
and cooking their own food. Chaos.
```

## Mistake 4: Anemic Domain Model

Making the Business Layer so thin that it just passes data through without applying any real logic. If your service is just:

```javascript
// ❌ Meaningless business layer
async function getUser(id) {
    return userRepository.findById(id); // No rules, no logic at all
}
```

The Business Layer has no purpose here — this is a warning sign you haven't identified your real business rules yet.

---

# Security & Performance Considerations

## Security

```
Security Concerns in Layered Architecture
│
├── Presentation Layer Risks
│   ├── SQL Injection (if queries are built in the controller)
│   ├── XSS (Cross-Site Scripting) from unescaped output
│   └── Exposed internal errors (stack traces in API responses)
│
├── Business Layer Risks
│   ├── Missing authorization checks (who can do what?)
│   └── Business rule bypass (skipping validation)
│
└── Data Access Layer Risks
    ├── Exposed DB credentials in code
    ├── Over-fetching (returning all columns when you need two)
    └── No query parameterization → SQL Injection
```

**Best Defense Strategy:**
- Validate input in the **Presentation Layer** (format check)
- Apply authorization in the **Business Layer** (can this user do this?)
- Use parameterized queries in the **Data Access Layer** (prevent SQL injection)
- Never let the database layer be reachable from outside the server

## Performance

```
Performance Bottlenecks in Layered Architecture
│
├── N+1 Query Problem
│   → Business layer calls repository in a loop
│   → Results in 1 + N database queries instead of 1
│   → Fix: Use JOIN queries or batch loading
│
├── Over-fetching
│   → Selecting all columns (SELECT *) when only 2 are needed
│   → Fix: Select only needed columns in repositories
│
├── Too Many Layers = Latency
│   → Each extra layer adds a function call and data copy
│   → Fix: Avoid adding layers that add no real value
│
└── Missing Cache Layer
    → Every request hits the database
    → Fix: Add a caching layer (Redis) between Business and Data layers
```

**Performance Enhancement — Adding Cache:**

```
Presentation Layer
      ↓
Business Layer
      ↓
   Cache (Redis)   ← Check here first
      ↓
Data Access Layer
      ↓
Database
```

---

# Related Technologies

| Technology | Relationship to Layered Architecture |
|---|---|
| **MVC (Model-View-Controller)** | MVC is a design pattern that maps to layered architecture: View=Presentation, Controller=Orchestration, Model=Data |
| **Spring Framework (Java)** | Uses `@Controller`, `@Service`, `@Repository` annotations for each layer |
| **Express.js (Node.js)** | Commonly structured as routes → services → repositories |
| **Django (Python)** | Views = Presentation, Logic in views/services, ORM = Data Layer |
| **Clean Architecture** | An evolution of layered architecture with stricter dependency rules |
| **Hexagonal Architecture** | A more flexible form with ports and adapters instead of strict layers |
| **Repository Pattern** | A pattern specifically for the Data Access Layer |
| **Service Layer Pattern** | A pattern for organizing the Business Logic Layer |
| **ORM (TypeORM, Sequelize)** | Tools commonly used in the Data Access Layer |

---

# Summary

## What We Learned

- **Layered Architecture** divides an application into horizontal layers, each with a single responsibility.
- The classic model has **3 tiers**: Presentation → Business Logic → Data Access → Database.
- Layers only communicate with the **layer directly below** — no skipping, no jumping.
- **Presentation Layer** handles what the user sees and sends.
- **Business Logic Layer** applies rules, validation, and orchestrates workflows.
- **Data Access Layer** is the only layer that reads and writes from the database.
- Adding a **Cache Layer** between Business and Data Access dramatically improves performance.
- Fat controllers, thin services, and layer-jumping are the most common mistakes.

## Key Takeaways

- Think of it like a restaurant: Customer → Waiter → Chef → Storage Room.
- Never write SQL in a controller. Never write business rules in a repository.
- The moment you skip a layer for "simplicity," you're borrowing technical debt.
- Test each layer independently: business rules need no UI, repositories need no business logic.
- Layered Architecture is the gateway concept to Clean Architecture and Hexagonal Architecture.

---

# Keywords

- Layered Architecture
- N-Tier Architecture
- 3-Tier Architecture
- Presentation Layer
- Business Logic Layer
- Data Access Layer
- Repository Pattern
- Service Layer
- Layer Isolation
- Separation of Concerns
- Fat Controller
- Anemic Domain Model
- Layer Jumping
- N+1 Query Problem
- Dependency Injection

---

# Glossary

| Term | Meaning |
|---|---|
| **Layered Architecture** | A software pattern where the application is split into horizontal layers, each with one responsibility |
| **N-Tier Architecture** | Same as layered architecture; "N" refers to the number of tiers (usually 3 or 4) |
| **Presentation Layer** | The top layer — handles user input and display (UI, API controllers) |
| **Business Logic Layer** | The middle layer — applies rules, validation, and workflows |
| **Data Access Layer** | The bottom layer — the only layer that talks to the database |
| **Repository Pattern** | A design pattern that wraps all database operations in a dedicated class/module |
| **Service Layer** | A variation of the Business Layer that focuses on orchestrating use cases |
| **Layer Isolation** | The rule that each layer can only talk to the one directly below it |
| **Fat Controller** | An anti-pattern where the controller contains too much business logic |
| **Anemic Domain Model** | An anti-pattern where the business layer has no real logic — just passes data through |
| **N+1 Query Problem** | A performance bug where calling a repository in a loop creates N extra queries |
| **Dependency Injection** | A technique to pass layer dependencies from outside rather than hardcoding them |
| **Tight Coupling** | When two layers depend so heavily on each other that changing one breaks the other |
| **Separation of Concerns** | The principle that each part of the system should only handle one concern |

## Next Recommended Chapters

- [03. Monolithic Architecture](./03-Monolithic-Architecture.md)
- [08. Clean Architecture](./08-Clean-Architecture.md)
- [09. Hexagonal Architecture](./09-Hexagonal-Architecture.md)
