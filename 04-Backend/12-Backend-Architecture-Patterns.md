> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Backend Architecture Patterns** are structural blueprints used to organize an application's code files, layers, and classes. They define how data flows between different sections of the code, how business logic is separated from technical implementation details (like databases and APIs), and how to keep code modular, testable, and maintainable over years of development.

# Why It Exists
When developers write their first backend app, they often write all their code in a single file: receiving HTTP requests, validating inputs, executing business rules, querying the database, and returning the response. This is called a **Spaghetti Code** monolith. As the project grows, this single file becomes thousands of lines long. If you want to change how you query the database, you risk breaking how you authenticate users because the code is tightly coupled. If you want to run automated tests to check if your discount calculations work, you have to spin up a live web server and database connection, which is slow and fragile. Engineers created architectural patterns to divide code into isolated layers with strict boundaries, ensuring changes in one layer do not ripple to break others.

# Problem It Solves
Backend architecture patterns solve spaghetti code chaos, tight database coupling, and untestable business logic.

### Before Architecture Patterns (Spaghetti Code):
- A change in the database schema forced developers to rewrite code inside HTTP controllers and API formatting layers.
- Business logic (like calculating taxes or interest rates) was scattered across multiple files, leading to inconsistent rules.
- Writing unit tests was nearly impossible because business code was mixed with database connections and web frameworks.

### After Layered Architecture (MVC / Three-Tier):
- Code is divided by responsibility: receiving requests, executing business logic, and querying databases are kept isolated.

### After Clean / Hexagonal Architecture:
- Core business rules (the Domain) sit at the center, completely independent of frameworks, databases, or transport layers. You can swap your SQL database for a NoSQL database, or change your Express API server for a gRPC server, without changing a single line of your core business logic code.

# Core Concepts
To write modular backend code, you must master two architectural designs:

1. **Layered (Controller-Service-Repository) Architecture:**
   - **Controller (Transport Layer):** Receives the incoming HTTP request, validates the input format, and routes it to the appropriate service.
   - **Service (Business Logic Layer):** The brain of the app. It holds business calculations, rules, and workflows. It does not know or care about database queries or HTTP headers.
   - **Repository (Data Access Layer):** The database manager. It runs SQL queries or ORM calls to retrieve and save raw records, handing them up to the Service layer.
2. **Clean / Hexagonal Architecture (Ports and Adapters):** An advanced pattern where dependency directions point strictly inward. The core business rules (the Domain) are wrapped in abstract interfaces (Ports). External systems (like database drivers, web frameworks, or third-party email APIs) implement these interfaces (Adapters).
3. **Dependency Injection (DI):** Passing a class its dependencies (like handing a Service its Repository) rather than having the class create them internally. This makes testing easy because you can inject a fake database connector (a mock) during tests.

# Architecture / Components
Comparing the dependency directions of Layered and Hexagonal architectures:

### Layered Architecture (Top-Down Flow)
```text
  [ Controller (HTTP) ] ──> [ Service (Business) ] ──> [ Repository (SQL) ] ──> [ Database ]
```

### Hexagonal Architecture (Inward Dependencies)
```text
      [ Express Adapter ] ──┐               ┌── [ Postgres Adapter ]
                            ▼               ▼
                      ┌───────────────────────────┐
                      │   [ Domain Core / Rules ] │  <- No external dependencies
                      │   (Interfaces / Ports)    │
                      └───────────────────────────┘
```

- **Domain Core:** The central, pure business logic classes and entities.
- **Ports (Interfaces):** Abstract contracts defining what functions the core needs (e.g. *"I need a way to save a user, I don't care how"*).
- **Adapters:** The concrete implementation code (e.g. *"I will use PostgreSQL to save the user"*).

# Workflow
How a layered architecture handles creating a new user:

```text
Step 1: The browser sends `POST /users` containing registration data.
                             ↓
Step 2: The Controller receives the request, validates the input, and calls `UserService.register(data)`.
                             ↓
Step 3: The Service runs business checks (e.g., checks if the user is over 18, hashes the password).
                             ↓
Step 4: The Service calls the repository: `UserRepository.save(newUser)`.
                             ↓
Step 5: The Repository runs the SQL query `INSERT INTO users...` to save the record to the database.
                             ↓
Step 6: The Repository returns success up to the Service, which returns it to the Controller, which responds `201 Created`.
```

# Real World Examples
Think of backend architectures as a **professional restaurant kitchen division of labor**.
- A spaghetti codebase is like a tiny food cart where one owner takes your order, runs back to chop vegetables, cooks the meat, washes the plates, and processes payment. If 100 customers arrive, the owner collapses.
- **Layered Architecture** is a formal restaurant team:
  - The **Host/Waiter (Controller)** handles front-of-house requests, greets you, checks your reservation, and takes your order. They do not cook.
  - The **Chef (Service)** sits in the back kitchen. They know the secret recipes (Business Logic) and cook the food. They never see the customer.
  - The **Pantry Manager (Repository)** manages raw food storage. When the chef asks for steak, the pantry manager goes to the freezer (Database), retrieves the raw meat, and hands it to the chef.
- **Hexagonal Architecture** is making sure the chef's recipes are written in standardized units (like grams and Celsius) completely independent of what model of stove is currently installed. If you replace a gas stove with an electric induction stove tomorrow (swap adapters), the chef's recipe booklet (the Domain) does not change at all.

# Implementation
Here is how you write a decoupled, testable layered architecture in Node.js using Dependency Injection:

### 1. The Repository Layer (`UserRepository.js`)
```javascript
class UserRepository {
  async save(user) {
    // Runs actual database query
    console.log(`Saving user ${user.name} to PostgreSQL database...`);
    return { id: 101, ...user };
  }
}
module.exports = UserRepository;
```

### 2. The Service Layer with Dependency Injection (`UserService.js`)
```javascript
class UserService {
  // Pass the repository dependency through the constructor
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async registerUser(name, email, age) {
    // Business Rule check
    if (age < 18) {
      throw new Error("User must be 18 or older to register.");
    }
    
    const newUser = { name, email, age };
    // Call the injected database repository
    return await this.userRepository.save(newUser);
  }
}
module.exports = UserService;
```

### 3. The Controller Layer (`UserController.js`)
```javascript
const express = require('express');
const UserRepository = require('./UserRepository');
const UserService = require('./UserService');

const app = express();
app.use(express.json());

// Initialize and inject dependencies
const userRepo = new UserRepository();
const userService = new UserService(userRepo);

app.post('/users', async (req, res) => {
  try {
    const { name, email, age } = req.body;
    // Route request to the Service layer
    const result = await userService.registerUser(name, email, age);
    res.status(201).json(result);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.listen(3000);
```

# Best Practices
- **Never Put Business Logic in Controllers:** Controllers should only handle transport concerns: routing, HTTP status codes, and input validation format checks. If you are checking password requirements or calculating product discounts inside a controller, move it to a Service class.
- **Dependencies Must Point Downwards:** In layered architecture, Controllers know about Services, and Services know about Repositories. Repositories should never know about Services, and Services should never import Controller logic.
- **Use Dependency Injection for Testing:** Always inject dependencies into your classes. This allows you to write isolated unit tests for your business logic by passing a fake repository (mock) that simulates database queries without hitting a real database, keeping tests incredibly fast.

# Industry Standards
Almost all enterprise frameworks enforce layered architectures out-of-the-box: **Spring Boot** (Java), **NestJS** (TypeScript), and **Django** (Python). Large scale, long-term platforms (like banking, core e-commerce transactions, or medical systems) implement **Hexagonal (Clean) Architecture** to insulate their core intellectual property (business rules) from changing cloud framework trends.

# Common Mistakes
- **Leaking DB Structures to Controllers:** Returning raw database database objects directly from the repository, through the service, and out of the controller to the client. If you rename a database column, your frontend API structure breaks. Use Data Transfer Objects (DTOs) or mapped models to separate data structures.
- **Creating Circular Dependencies:** Class A imports Class B, which imports Class A, creating an import loop that crashes the compiler runtime.

# Security & Performance Considerations
- **Isolated Auditing:** Since business rules are consolidated inside the Service layer, auditing code for security compliance is highly efficient because you only need to inspect one directory, rather than tracing logic across database and route files.
- **Instantiating Overhead:** Injecting dependencies manually in large apps can create massive, messy startup scripts. Use Dependency Injection containers (DI frameworks) to handle creating and injecting classes automatically.

# Related Technologies
- **NestJS:** A TypeScript framework built entirely around Controller-Service-Repository architecture and Dependency Injection.
- **Prisma DTOs:** Data Transfer Objects used to map database records into clean client-side formats.

# Summary

## What We Learned
- Architectural patterns prevent spaghetti code by organizing code files by responsibility.
- Layered MVC architecture isolates transport routing (Controllers), business rules (Services), and database interactions (Repositories).
- Dependency Injection enables testability by passing dependent objects at runtime.

## Key Takeaways
- Enforce strict layer boundaries; do not write business logic or database queries inside HTTP routing files.
- Inject repository dependencies into services to support fast, mock-based unit testing.

# Keywords
- Architecture
- Layered Architecture
- Clean Architecture
- Hexagonal
- Controller
- Service
- Repository
- Dependency Injection

# Glossary

| Term | Meaning |
|---|---|
| Spaghetti Code | An unorganized codebase containing tightly coupled modules with no clear separation of concerns. |
| Dependency Injection | The design pattern where an object receives its dependencies from an external source rather than creating them itself. |
| Domain | The core business logic, entities, and rules of a software application, completely isolated from technical infrastructure. |
| Interface | An abstract code contract defining method signatures that concrete classes must implement. |
| Mock | A fake object used in unit testing to simulate the behavior of real dependencies (like a database) to keep tests fast and isolated. |

## Next Recommended Chapters
- 06-ORMs-And-Query-Builders.md
- 04-Backend/13-Microservices-Architecture.md
