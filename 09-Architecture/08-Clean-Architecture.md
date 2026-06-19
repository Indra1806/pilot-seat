> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction

Imagine you're writing a **novel**.

Your story — the plot, the characters, the themes — is the core of your work. It's what truly matters.

Now imagine you've written the entire novel in a specific font, printed on a specific type of paper, sold exclusively through one bookstore chain. Your story is **inseparable** from those delivery details.

One day, the bookstore chain goes bankrupt. The paper manufacturer stops producing that paper. The font becomes unavailable.

Suddenly, your novel — the story itself — is in jeopardy, even though **the story never changed**.

This is what happens to most software. The business logic — the "story" — gets tangled up with the delivery mechanism (framework, database, web server). When the framework changes, the business logic suffers. When the database changes, the application breaks.

**Clean Architecture**, introduced by **Robert C. Martin (Uncle Bob)** in 2012, solves this.

Clean Architecture is a software design philosophy that says:

> **Your business rules — the core of what your application does — should be independent of any framework, database, user interface, or external agent.**

The UI could be swapped from a web app to a mobile app. The database could be swapped from PostgreSQL to MongoDB. The framework could be changed from Express.js to Fastify. And the business logic would be **completely untouched**.

It achieves this through a strict **concentric circle structure** — with the most important, most stable code at the center, and the most changeable, external details at the outer edges.

---

# Why It Exists

Before Clean Architecture, a common pattern emerged in enterprise software:

### The Framework Dependency Problem

```
Typical "Express-based" Application Structure:

app.js (Express) → routes → controllers → services → repositories → database

The Problem:
  → UserService.createUser() imports Express.Request
  → Business logic is sprinkled with database query syntax
  → PaymentService is tightly coupled to Stripe SDK
  → Testing requires a running web server and database

When you change framework → rewrite controllers AND services
When you change database → rewrite repositories AND services
When you test → must spin up the full stack
```

This violated the key software principles:
- **Dependency Rule**: High-level modules (business logic) should not depend on low-level modules (frameworks, databases)
- **Stability**: The core business rules change infrequently. Frameworks and DBs change constantly.
- **Testability**: Business logic should be testable without a web server or database

Robert C. Martin synthesized decades of architectural thinking into one coherent model: **Clean Architecture**.

---

# Problem It Solves

**Problem 1: Business Logic Tied to Framework**

```
❌ Business logic depends on Express:
  function createUser(req: express.Request) {
      // Business logic now needs Express types
      const email = req.body.email;
      if (!isValidEmail(email)) throw new HttpError(400, 'Invalid');
      ...
  }

  → Can't test without an HTTP request
  → Can't reuse in a CLI or background job
  → Framework upgrade requires touching business logic

✅ Clean Architecture solution:
  function createUser(email: string, password: string) {
      // No framework dependency — pure business logic
      if (!isValidEmail(email)) throw new Error('Invalid email');
      ...
  }
  → Testable with a simple function call
  → Usable from any delivery mechanism
```

**Problem 2: Business Logic Tied to Database**

```
❌ Business logic depends on SQL:
  class UserService {
      async registerUser(email, password) {
          const user = await db.query(
              'INSERT INTO users (email, password) VALUES ($1, $2)', [email, password]
          );
          return user.rows[0]; // SQL detail leaking into business logic
      }
  }

✅ Clean Architecture:
  class UserService {
      constructor(userRepository) {  // Dependency injected
          this.userRepository = userRepository;
      }
      async registerUser(email, password) {
          const user = new User(email, password); // Domain object
          return this.userRepository.save(user);  // Doesn't know it's SQL
      }
  }
```

---

# Core Concepts

## 1. The Dependency Rule — The Core Law

The most important rule in Clean Architecture:

```
"Source code dependencies can ONLY point inward."

Outer rings depend on inner rings.
Inner rings NEVER depend on outer rings.

[Outer Layer] → depends on → [Inner Layer]
[Inner Layer] → knows NOTHING about → [Outer Layer]
```

Visually:

```
        [Frameworks & Drivers]   ← can depend on
              ↓
       [Interface Adapters]      ← can depend on
              ↓
          [Use Cases]            ← can depend on
              ↓
            [Entities]           ← depends on NOTHING external
```

## 2. The Four Rings

```
         ┌─────────────────────────────────────────┐
         │         Frameworks & Drivers             │  Outermost
         │  ┌───────────────────────────────────┐   │
         │  │       Interface Adapters           │   │
         │  │  ┌─────────────────────────────┐  │   │
         │  │  │       Use Cases             │  │   │
         │  │  │  ┌─────────────────────┐   │  │   │
         │  │  │  │     Entities        │   │  │   │  Innermost
         │  │  │  └─────────────────────┘   │  │   │
         │  │  └─────────────────────────────┘  │   │
         │  └───────────────────────────────────┘   │
         └─────────────────────────────────────────┘
```

## 3. The Four Rings Explained

### Ring 1: Entities (Innermost — Most Stable)
```
What it is: Pure business objects — the fundamental rules of your domain.

Examples:
  User entity:     has email, password, roles. Can validate itself.
  Order entity:    has items, total, status. Can calculate totals.
  Product entity:  has name, price, stock.

Rules:
  → No framework imports
  → No database imports
  → No HTTP imports
  → Pure business rules and domain objects

Change frequency: VERY RARELY — only when core business fundamentals change
```

### Ring 2: Use Cases (Application Business Rules)
```
What it is: The specific actions your application performs.
            Orchestrate entities to fulfill a specific user goal.

Examples:
  CreateUserUseCase       → validates, creates User entity, saves via repo
  PlaceOrderUseCase       → checks stock, creates Order, charges payment
  ResetPasswordUseCase    → validates token, creates new password hash

Rules:
  → Knows about entities
  → Does NOT know about frameworks, databases, or HTTP
  → Receives and returns simple data objects (not HTTP request/response)
  → Defines interfaces (ports) that outer layers must implement

Change frequency: SOMETIMES — when application features change
```

### Ring 3: Interface Adapters
```
What it is: Converts data between the format of use cases and the format
            of external systems (UI, database, APIs).

Examples:
  Controllers       → convert HTTP request → use case input
  Presenters        → convert use case output → HTTP response format
  Repositories      → convert domain objects → database rows
  Gateways          → convert external API responses → domain objects

Rules:
  → Knows about use cases and entities
  → Knows about the external format (HTTP, SQL, etc.)
  → The "translation layer"

Change frequency: SOMETIMES — when external interfaces change
```

### Ring 4: Frameworks & Drivers (Outermost — Most Volatile)
```
What it is: The actual frameworks, databases, and tools.

Examples:
  Express.js / Fastify  → web framework
  PostgreSQL / MongoDB  → database
  Stripe SDK            → payment provider
  SendGrid              → email service
  Kafka client          → message broker

Rules:
  → This is "glue code"
  → Should be thin wrappers
  → Can be completely replaced without affecting any inner ring

Change frequency: FREQUENTLY — technologies change, vendors change
```

---

# Architecture / Components

## Clean Architecture Data Flow

```
Request coming in (e.g., HTTP POST /users):

OUTER → INNER (request):
[HTTP Framework]
  ↓ calls
[Controller (Interface Adapter)]
  ↓ creates InputDTO, calls
[CreateUserUseCase (Use Case)]
  ↓ creates
[User Entity (Entity)]
  ↓ calls
[UserRepository Interface (Use Case defines this)]
  ↓ implemented by
[PostgresUserRepository (Interface Adapter)]
  ↓ uses
[PostgreSQL (Framework & Driver)]

INNER → OUTER (response):
[PostgreSQL] → returns raw data
[PostgresUserRepository] → maps to User entity
[CreateUserUseCase] → returns OutputDTO
[Presenter (Interface Adapter)] → formats as HTTP response JSON
[HTTP Framework] → sends response
```

## Dependency Inversion in Practice

The key trick that enables Clean Architecture:

```
The Use Case defines an INTERFACE (port):
  interface UserRepository {
      save(user: User): Promise<User>;
      findByEmail(email: string): Promise<User | null>;
  }

The Use Case depends on the INTERFACE (not the implementation):
  class CreateUserUseCase {
      constructor(private userRepo: UserRepository) {}
      // Doesn't know if it's PostgreSQL, MongoDB, or an in-memory mock
  }

The outer layer IMPLEMENTS the interface:
  class PostgresUserRepository implements UserRepository {
      async save(user: User): Promise<User> {
          const row = await db.query('INSERT INTO users...');
          return mapRowToUser(row);
      }
  }

At startup, inject the concrete implementation:
  const useCase = new CreateUserUseCase(new PostgresUserRepository());
  // Or for tests:
  const useCase = new CreateUserUseCase(new InMemoryUserRepository());
```

## Folder Structure

```
src/
│
├── domain/                   ← Ring 1: Entities
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   └── value-objects/
│       ├── Email.ts          ← Email validation logic lives here
│       └── Money.ts
│
├── application/              ← Ring 2: Use Cases
│   ├── use-cases/
│   │   ├── CreateUser.ts
│   │   ├── PlaceOrder.ts
│   │   └── ResetPassword.ts
│   ├── interfaces/           ← Ports: interfaces the outer layers implement
│   │   ├── IUserRepository.ts
│   │   └── IEmailService.ts
│   └── dto/                  ← Data transfer objects (input/output contracts)
│       ├── CreateUserInput.ts
│       └── CreateUserOutput.ts
│
├── infrastructure/           ← Ring 3+4: Adapters + Frameworks
│   ├── persistence/
│   │   └── PostgresUserRepository.ts  ← implements IUserRepository
│   ├── web/
│   │   ├── controllers/
│   │   │   └── UserController.ts
│   │   └── presenters/
│   │       └── UserPresenter.ts
│   └── services/
│       └── SendGridEmailService.ts    ← implements IEmailService
│
└── main.ts                   ← Composition root: wire everything together
```

---

# Workflow

**Scenario: Register a new user via REST API.**

```
STEP 1 — HTTP Request Arrives
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /users { "email": "jane@example.com", "password": "secret123" }

STEP 2 — Controller (Interface Adapter)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
UserController.register(req, res):
  → Extracts email and password from req.body
  → Creates CreateUserInput DTO (just plain data, no Express types)
  → Calls createUserUseCase.execute(input)

STEP 3 — CreateUserUseCase (Use Case)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CreateUserUseCase.execute({ email, password }):
  → Validates: email format, password strength
  → Checks: userRepository.findByEmail(email) → must not exist
  → Creates: new User entity (domain object)
  → Hashes password (domain rule)
  → Calls: userRepository.save(user)
  → Publishes: domain event "UserRegistered" (optional)
  → Returns: CreateUserOutput DTO { userId, email, createdAt }

STEP 4 — Repository (Interface Adapter)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PostgresUserRepository.save(user):
  → Maps User entity to database columns
  → Executes: INSERT INTO users (email, password_hash) VALUES (...)
  → Maps returned row back to User entity

STEP 5 — Back to Controller
━━━━━━━━━━━━━━━━━━━━━━━━━━━
UserController receives CreateUserOutput from use case
  → Presenter formats it as HTTP JSON response
  → res.status(201).json({ id: ..., email: ... })
```

---

# Real World Examples

## Example 1: The Novel Writing Analogy

```
Entities = The Story (plot, characters, themes)
  → Always true regardless of format

Use Cases = The Chapters (structured narrative)
  → How the story is told

Interface Adapters = The Editor / Publisher
  → Formats the story for hardcover, e-book, audio

Frameworks & Drivers = The Printing Press / Kindle App
  → The delivery mechanism — can be swapped
```

## Example 2: The ATM Machine

```
Entities:
  Account (balance, owner, account number)
  Transaction (amount, type, timestamp)

Use Cases:
  WithdrawCashUseCase (validates balance, creates transaction)
  CheckBalanceUseCase (returns current balance)
  TransferFundsUseCase (debit one, credit another)

Interface Adapters:
  ATM Screen Controller (reads button presses, shows results)
  Bank API Gateway (adapts use case output to bank network format)

Frameworks & Drivers:
  ATM hardware (card reader, cash dispenser, screen)
  Bank network protocol
```

## Example 3: Netflix — Recommendation Engine

```
Entities:
  User (preferences, watch history, subscription tier)
  Content (genre, rating, duration)
  Recommendation (score, reason)

Use Cases:
  GenerateRecommendationsUseCase
  → applies recommendation algorithm
  → uses User entity and Content entity
  → doesn't know if the result goes to iOS or Android or web

Interface Adapters:
  RecommendationAPIController (formats for the mobile app API response)
  ContentRepository (fetches content metadata from database)
  UserProfileRepository (fetches user data)

Frameworks & Drivers:
  Python ML framework (TensorFlow)
  Cassandra database
  REST API layer (Flask/FastAPI)
```

---

# Implementation

## Entity (Pure Business Object)

```typescript
// domain/entities/User.ts

export class User {
    private readonly id: string;
    private email: Email;  // Value object
    private passwordHash: string;
    private readonly createdAt: Date;

    constructor(email: string, passwordHash: string) {
        this.email = new Email(email); // Validates on creation
        this.passwordHash = passwordHash;
        this.createdAt = new Date();
        this.id = generateId();
    }

    getEmail(): string { return this.email.getValue(); }
    getId(): string { return this.id; }
    getCreatedAt(): Date { return this.createdAt; }

    // Business rule: can this user access premium features?
    canAccessPremium(): boolean {
        const daysSinceCreation = daysBetween(this.createdAt, new Date());
        return daysSinceCreation >= 30;
    }

    // No framework imports. No database imports. Just business logic.
}
```

## Use Case (Application Business Rule)

```typescript
// application/use-cases/CreateUser.ts

import { User } from '../../domain/entities/User';
import { IUserRepository } from '../interfaces/IUserRepository';
import { IEmailService } from '../interfaces/IEmailService';

interface CreateUserInput {
    email: string;
    password: string;
}

interface CreateUserOutput {
    userId: string;
    email: string;
    createdAt: Date;
}

export class CreateUserUseCase {
    // Use case depends on INTERFACES, not concrete implementations
    constructor(
        private userRepository: IUserRepository,
        private emailService: IEmailService,
    ) {}

    async execute(input: CreateUserInput): Promise<CreateUserOutput> {
        // Business Rule: email must be unique
        const existing = await this.userRepository.findByEmail(input.email);
        if (existing) throw new Error('Email already registered');

        // Business Rule: password must be strong
        if (input.password.length < 8) throw new Error('Password too short');

        const passwordHash = await hashPassword(input.password);
        const user = new User(input.email, passwordHash);

        await this.userRepository.save(user);
        await this.emailService.sendWelcomeEmail(user.getEmail());

        return {
            userId: user.getId(),
            email: user.getEmail(),
            createdAt: user.getCreatedAt(),
        };
    }
}
```

## Repository Interface (Port)

```typescript
// application/interfaces/IUserRepository.ts

import { User } from '../../domain/entities/User';

// The use case DEFINES this interface.
// The outer layer IMPLEMENTS it.
export interface IUserRepository {
    save(user: User): Promise<User>;
    findById(id: string): Promise<User | null>;
    findByEmail(email: string): Promise<User | null>;
    delete(id: string): Promise<void>;
}
```

## Concrete Repository (Interface Adapter)

```typescript
// infrastructure/persistence/PostgresUserRepository.ts

import { IUserRepository } from '../../application/interfaces/IUserRepository';
import { User } from '../../domain/entities/User';
import { db } from '../database/connection';

export class PostgresUserRepository implements IUserRepository {
    async save(user: User): Promise<User> {
        await db.query(
            'INSERT INTO users (id, email, password_hash, created_at) VALUES ($1, $2, $3, $4)',
            [user.getId(), user.getEmail(), user.getPasswordHash(), user.getCreatedAt()]
        );
        return user;
    }

    async findByEmail(email: string): Promise<User | null> {
        const result = await db.query('SELECT * FROM users WHERE email = $1', [email]);
        if (!result.rows.length) return null;
        return this.mapRowToUser(result.rows[0]);
    }

    private mapRowToUser(row: any): User {
        // Map database row to domain entity
        return User.reconstitute(row.id, row.email, row.password_hash, row.created_at);
    }
}
```

## Controller (Interface Adapter)

```typescript
// infrastructure/web/controllers/UserController.ts

import { CreateUserUseCase } from '../../../application/use-cases/CreateUser';

export class UserController {
    constructor(private createUserUseCase: CreateUserUseCase) {}

    async register(req: Request, res: Response) {
        // Convert HTTP request → use case input
        const input = {
            email: req.body.email,
            password: req.body.password,
        };

        const output = await this.createUserUseCase.execute(input);

        // Convert use case output → HTTP response
        res.status(201).json({
            id: output.userId,
            email: output.email,
            createdAt: output.createdAt,
        });
    }
}
```

## Composition Root (Wiring Everything Together)

```typescript
// main.ts — The ONLY place where concrete dependencies are assembled

import { PostgresUserRepository } from './infrastructure/persistence/PostgresUserRepository';
import { SendGridEmailService } from './infrastructure/services/SendGridEmailService';
import { CreateUserUseCase } from './application/use-cases/CreateUser';
import { UserController } from './infrastructure/web/controllers/UserController';

// Compose the dependency graph (dependency injection)
const userRepository = new PostgresUserRepository();
const emailService = new SendGridEmailService();
const createUserUseCase = new CreateUserUseCase(userRepository, emailService);
const userController = new UserController(createUserUseCase);

// Register routes
app.post('/users', (req, res) => userController.register(req, res));
```

---

# Best Practices

## ✅ DO

| Practice | Why |
|---|---|
| **Enforce the Dependency Rule strictly** | Inner rings must never import from outer rings |
| **Define interfaces (ports) in the Use Case layer** | Use cases define WHAT they need; outer layers decide HOW |
| **Use DTOs for use case input/output** | Don't pass HTTP request or database rows into use cases |
| **Put ALL business validation in entities and use cases** | Never in controllers or repositories |
| **Make use cases independently testable** | Inject mock repositories → test without database or HTTP |
| **Name use cases after user goals** | CreateUser, PlaceOrder, ResetPassword — not CRUDService |
| **Keep entities free of framework and database code** | An entity should be usable in any context |

## ❌ DON'T

| Mistake | What Goes Wrong |
|---|---|
| **Import Express.Request in a use case** | Use case now depends on the web framework |
| **Put SQL in a use case** | Use case now depends on the database |
| **Make repositories return database row objects** | Leaks database schema into business logic |
| **Skip the interface layer (use concrete classes directly)** | Can't test, can't swap implementations |
| **Create an "anemic" entity** | Entity with only getters/setters and no behavior is not an entity — it's a data bag |
| **Put business logic in controllers** | Can't test or reuse without an HTTP layer |

---

# Industry Standards

## Clean Architecture Influences

Clean Architecture synthesizes several earlier architectural patterns:

```
Predecessors of Clean Architecture:
  ├── Hexagonal Architecture (Alistair Cockburn, 2005)
  │     → "Ports and Adapters" — same concept, different vocabulary
  │
  ├── Onion Architecture (Jeffrey Palermo, 2008)
  │     → Same concentric circle idea
  │
  ├── DCI Architecture (Trygve Reenskaug, 2009)
  │     → Data, Context, Interaction separation
  │
  └── Clean Architecture (Robert C. Martin, 2012)
        → Synthesizes all of the above into one coherent model
```

## In Production

```
Companies using Clean Architecture principles:
  → Google: Internal services follow similar port/adapter patterns
  → Netflix: Domain-driven isolation between recommendation core and delivery
  → Banking systems: Regulatory requirements force business logic isolation
  → Healthcare systems: HIPAA compliance requires testable, auditable core

Frameworks supporting Clean Architecture:
  → NestJS (Node.js): Built-in module and dependency injection system
  → Spring Boot (Java): @Service, @Repository, interface injection
  → Django REST Framework (Python): Serializers as adapters, domain in models
  → Android/iOS: Clean Architecture is the official Google/Apple recommendation
```

---

# Common Mistakes

## Mistake 1: The Anemic Domain Model

```
❌ Wrong — Entity with no behavior:
  class User {
      id: string;
      email: string;
      password: string;
      // No methods. Just data storage.
  }

All logic is then in UserService.checkPassword(), UserService.validateEmail()...
The entity has no meaning — it's just a struct.

✅ Right — Entity with behavior:
  class User {
      private email: Email;   // Value object enforces valid email
      
      canResetPassword(): boolean { return daysSince(this.lastReset) > 24 * 60; }
      isEmailVerified(): boolean { return this.emailVerifiedAt !== null; }
      canAccessAdmin(): boolean { return this.roles.includes('ADMIN'); }
  }
```

## Mistake 2: Framework Types in Use Cases

```
❌ Wrong:
  class CreateUserUseCase {
      async execute(req: express.Request) {  // ← framework dependency!
          const email = req.body.email;
          ...
      }
  }

✅ Right:
  class CreateUserUseCase {
      async execute(input: { email: string; password: string }) {
          // Pure data — no framework
      }
  }
```

## Mistake 3: Database Queries in Use Cases

```
❌ Wrong:
  class PlaceOrderUseCase {
      async execute(orderData: any) {
          const user = await db.query('SELECT * FROM users WHERE id = $1', [...]);
          // SQL directly in use case!
      }
  }

✅ Right:
  class PlaceOrderUseCase {
      constructor(private userRepo: IUserRepository) {}
      async execute(orderData: OrderInput) {
          const user = await this.userRepo.findById(orderData.userId);
          // Use case doesn't know it's SQL
      }
  }
```

## Mistake 4: Too Many Layers for Simple Operations

```
Clean Architecture is powerful but adds overhead.
For simple CRUD applications or small projects, it may be overkill.

Use Clean Architecture when:
  ✓ Complex business rules that must be tested in isolation
  ✓ Multiple delivery mechanisms (REST, CLI, background jobs)
  ✓ Likely to change frameworks or databases
  ✓ Large codebase with multiple teams

Consider skipping when:
  ✗ Simple CRUD API with no real business logic
  ✗ Tiny codebase
  ✗ Prototype or early-stage validation
```

---

# Security & Performance Considerations

## Security

```
Clean Architecture Security Advantages
│
├── Clear Security Boundaries
│   → Authentication/authorization checks live in specific layers
│   → Use cases can enforce access rules cleanly:
│       if (!currentUser.canPlaceOrder()) throw UnauthorizedError;
│
├── No Framework Vulnerabilities in Core
│   → Business logic doesn't depend on framework
│   → Framework vulnerability doesn't directly affect business rules
│
└── Testable Security Rules
    → Business authorization rules live in pure use cases
    → Can write unit tests that verify "user without admin role cannot delete"
    → Without needing a running server
```

## Performance

```
Clean Architecture Performance Considerations
│
├── Overhead of Abstraction Layers
│   → Function calls through multiple layers add minimal overhead
│   → Each layer call is nanoseconds — negligible
│
├── Mapping Overhead
│   → Converting database rows to entities to DTOs adds CPU
│   → For high-volume read endpoints, consider lightweight read models
│   → CQRS: use Clean Architecture for writes, simple queries for reads
│
└── Dependency Injection Container
    → Frameworks like Spring/NestJS manage DI automatically
    → Manual DI (as in our examples) is simple for medium-sized apps
```

---

# Related Technologies

| Technology | Relationship |
|---|---|
| **Hexagonal Architecture** | The direct predecessor — "Ports and Adapters" maps exactly to Clean Architecture's rings |
| **Onion Architecture** | Another predecessor — concentric rings with same dependency direction |
| **Domain-Driven Design (DDD)** | DDD Entities and Value Objects map directly to Clean Architecture's Entity ring |
| **CQRS** | Often combined with Clean Architecture — separate use cases for commands vs. queries |
| **Repository Pattern** | The Repository interface in Clean Architecture is the classic repository pattern |
| **Dependency Injection** | The mechanism that makes Clean Architecture work — inject implementations at the composition root |
| **NestJS (Node.js)** | Built-in DI container and module system aligns naturally with Clean Architecture |
| **Spring Boot (Java)** | `@Service`, `@Repository`, `@Component` annotations support Clean Architecture structure |
| **Layered Architecture** | A simpler version — Clean Architecture is an evolution with stricter dependency rules |
| **Microservices** | Each microservice can be internally structured as a Clean Architecture |

---

# Summary

## What We Learned

- **Clean Architecture** is a design philosophy where **business rules are independent** of frameworks, databases, UIs, and external agents.
- It uses **concentric rings**: Entities (innermost) → Use Cases → Interface Adapters → Frameworks & Drivers (outermost).
- The **Dependency Rule**: source code dependencies point **only inward** — outer rings know about inner rings, never the reverse.
- **Entities** are pure business objects with behavior — no framework or database imports.
- **Use Cases** orchestrate entities to fulfil specific application goals — no HTTP, no SQL.
- **Interface Adapters** translate between use case data formats and external system formats (controllers, presenters, repositories).
- **Frameworks & Drivers** are the actual tools (Express, PostgreSQL) — thin wrappers at the outermost ring.
- The key mechanism is **Dependency Inversion**: Use cases define *interfaces*, outer layers *implement* them.
- This makes the system **testable without a running server or database**.

## Key Takeaways

- Think of it like writing a novel: the story (business logic) should be independent of the printing press (framework) or bookstore (database).
- The innermost ring changes least. The outermost ring changes most. Design accordingly.
- If your use case imports `express.Request` or runs SQL directly — it's not Clean Architecture.
- The composition root (`main.ts`) is the ONLY place where concrete implementations are wired together.
- Clean Architecture is powerful for complex domains — for simple CRUDs, it may be overkill.

---

# Keywords

- Clean Architecture
- Dependency Rule
- Entities
- Use Cases
- Interface Adapters
- Frameworks and Drivers
- Dependency Inversion
- Ports and Adapters
- Repository Pattern
- Composition Root
- DTO (Data Transfer Object)
- Anemic Domain Model
- Value Object
- Domain Event
- Uncle Bob (Robert C. Martin)

---

# Glossary

| Term | Meaning |
|---|---|
| **Clean Architecture** | An architectural style by Robert C. Martin where business rules are independent of delivery mechanisms |
| **Dependency Rule** | Inner rings may not depend on outer rings — dependencies only point inward |
| **Entity** | The innermost ring — pure business objects with domain behavior and no framework dependencies |
| **Use Case** | The application business rules — orchestrates entities to accomplish user goals |
| **Interface Adapter** | The translation ring — converts data between use case format and external formats |
| **Frameworks & Drivers** | The outermost ring — actual tools (Express, PostgreSQL) as thin wrappers |
| **Port** | An interface defined by the use case layer — specifying what it needs from the outside world |
| **Adapter** | A concrete implementation of a port — the actual database code, email service, etc. |
| **DTO** | Data Transfer Object — a simple data structure for carrying input/output between layers |
| **Composition Root** | The single place (main.ts) where all dependencies are assembled and injected |
| **Dependency Inversion** | High-level modules define what they need (interface); low-level modules implement it |
| **Anemic Domain Model** | An anti-pattern: domain objects with no behavior — just data holders |
| **Value Object** | An immutable domain object defined by its value, not its identity (e.g., Email, Money) |
| **Domain Event** | A record of something significant that happened in the domain — used in event-driven Clean Architecture |

## Next Recommended Chapters

- [09. Hexagonal Architecture](./09-Hexagonal-Architecture.md)
- [12. Architecture Patterns](./12-Architecture-Patterns.md)
- [07. Event-Driven Architecture](./07-Event-Driven-Architecture.md)
