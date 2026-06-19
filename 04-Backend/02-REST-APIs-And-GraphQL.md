> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
An **API** (Application Programming Interface) is the bridge that allows different software applications—like a frontend web browser and a backend server—to communicate and exchange data. **REST** (Representational State Transfer) and **GraphQL** are two different architectural designs used to structure APIs, defining how clients request data and how servers pack and deliver it.

# Why It Exists
Before standard API designs like REST, backend applications communicated using complex, hardware-specific protocols (like SOAP or RPC). Developers had to write custom translation wrappers for every service. If a developer wanted to connect a mobile app to a website's server, they had to study pages of custom network protocols. Engineers created REST in 2000 to establish a simple, universal standard using HTTP. As mobile networks grew, REST's rigid data formats sometimes proved too slow, leading engineers at Facebook to create GraphQL in 2012 to let clients request only the specific data they need.

# Problem It Solves
REST and GraphQL solve complex cross-system communication, bandwidth wastage, and rigid data structure problems.

### Before Standardized APIs:
- Connecting a frontend browser to a backend database required writing low-level socket communication.
- Every software system had custom messaging formats, making integrations fragile and slow.

### After REST:
- Systems represent data as "Resources" at specific URLs, using standard HTTP methods (GET, POST).
- Communication is stateless and highly cacheable.

### After GraphQL:
- Clients avoid downloading unnecessary data fields (Over-fetching) or making multiple sequential network calls to get related data (Under-fetching).

# Core Concepts
To design modern web services, you must understand the two main API patterns:

1. **REST (Resource-Oriented):** Data is modeled as nouns called **Resources** (like `/users` or `/orders`). You interact with these resources using standard HTTP verbs:
   - `GET /users`: Fetch all users.
   - `POST /users`: Create a new user.
2. **GraphQL (Query-Oriented):** Instead of many URLs, GraphQL uses a **single endpoint** (usually `/graphql`). The client sends a text schema called a **Query** describing the exact fields they want, and the server returns a JSON response matching that shape.
3. **Over-fetching vs. Under-fetching:**
   - **Over-fetching:** Downloading more data than you need (e.g. fetching a full user profile when you only need their name).
   - **Under-fetching:** Downloading too little data, forcing you to make multiple back-to-back API calls (e.g. fetching an order, then fetching the user details, then fetching the product details).

# Architecture / Components
The difference in endpoint routing between REST and GraphQL architectures:

### REST Architecture (Multiple Endpoints)
```text
  [ Client ] ─── GET /users/1 ───────────> [ User Endpoint ] ──> Returns ID, Name, Address, Phone
  [ Client ] ─── GET /users/1/orders ────> [ Order Endpoint ] ──> Returns Order List
```

### GraphQL Architecture (Single Endpoint)
```text
  [ Client ] ─── POST /graphql ──────────> [ GraphQL Engine ] ──> Returns only name and order titles
                 (Sends precise Query)     (Parses Schema)
```

- **REST Endpoint:** A specific URL path representing a data resource.
- **GraphQL Schema:** The master blueprint defining all data types, queries (reads), and mutations (writes).
- **GraphQL Resolver:** The backend function responsible for fetching the data for a specific field in a query.

# Workflow
Comparing the data-fetching workflows:

### REST Workflow
```text
Step 1: Client sends `GET /users/42` -> Receives full user profile.
                             ↓
Step 2: Client extracts user ID and sends `GET /users/42/posts` -> Receives post list.
                             ↓
Step 3: Client extracts post IDs and sends `GET /posts/101/comments` -> Receives comments.
```

### GraphQL Workflow
```text
Step 1: Client sends a single POST request to `/graphql` with a nested query:
        "Give me User 42's name, their posts, and comment titles."
                             ↓
Step 2: The GraphQL engine resolves all fields in parallel on the server.
                             ↓
Step 3: Client receives a single, custom-shaped JSON response containing all requested data.
```

# Real World Examples
Think of APIs as **restaurant menus**.
- **REST** is like a **fixed à la carte menu**. If you order "Plate #5: The Cheeseburger" (`GET /burger`), you get the bun, the beef patty, the lettuce, the cheese, and the fries. If you only want to eat the beef patty, you still have to buy the whole plate and throw the rest away (Over-fetching). If you also want a drink, you have to place a separate order for "Plate #12: Soda" (`GET /soda`) (Under-fetching).
- **GraphQL** is like a **custom buffet or personal chef**. Instead of choosing from pre-set plates, you hand the chef a slip of paper listing exactly what you want: *"Please give me a plate with 1 beef patty, 2 slices of cheese, and 1 soda."* The chef returns a single plate containing exactly those items, with no wasted food.

# Implementation
Here is how developers write endpoints for both styles:

### 1. Implementing REST in Node.js (using Express)
```javascript
const express = require('express');
const app = express();

// REST Resource: Users
app.get('/api/users/:id', (req, res) => {
  const userId = req.params.id;
  // Fetches full user from database
  const user = { id: userId, name: "Alice", email: "alice@email.com", age: 30 };
  res.json(user); // Returns full object
});

app.listen(3000);
```

### 2. Implementing GraphQL in Node.js (using Schema definition)
```javascript
const { graphqlHTTP } = require('express-graphql');
const { buildSchema } = require('graphql');

// Define the GraphQL schema shape
const schema = buildSchema(`
  type User {
    id: ID!
    name: String!
    email: String
  }
  type Query {
    user(id: ID!): User
  }
`);

// Define how to fetch the user (Resolver)
const root = {
  user: ({ id }) => {
    return { id, name: "Alice", email: "alice@email.com" };
  }
};

app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true // Enables interactive browser test playground
}));
```

# Best Practices
- **REST: Use Plural Nouns for Resources:** Always name your routes using plural nouns (e.g. `/api/items` or `/api/customers`), never verbs (avoid `/api/getAllItems` or `/api/deleteCustomer`). Let HTTP methods declare the action.
- **GraphQL: Avoid Deeply Nested Queries:** Hackers can send extremely deep, recursive queries (e.g., fetch users, who have friends, who have friends, etc.) that crash your database. Implement query depth limiting on the server.
- **Versioning REST APIs:** When making breaking changes to your REST API, version the URL path (e.g. `/api/v1/users` and `/api/v2/users`) so you do not break active frontend applications.

# Industry Standards
Almost all public web APIs (like Stripe, Twilio, or GitHub) are built using **REST** because it is simple, uses standard caching, and is widely understood. However, modern internal client-server communication (such as inside Facebook, Twitter, and Netflix) is commonly built using **GraphQL** or **gRPC** to optimize mobile performance.

# Common Mistakes
- **Using GET to Modify Data:** Writing a route like `GET /api/delete-user?id=5`. Web search crawlers or browser pre-loaders can scan these links and accidentally delete your database records. Always use `DELETE` or `POST` for modifications.
- **Ignoring HTTP Status Codes in REST:** Returning `200 OK` for every single request, even when database queries fail, and placing the error message inside the JSON body. This confuses frontend client libraries.

# Security & Performance Considerations
- **Endpoint Protection:** Implement strict rate limiting on your API endpoints to prevent script bots from scraping your database or launching Denial of Service (DoS) attacks.
- **REST Caching:** REST is naturally easy to cache. Because each resource has a unique URL (e.g. `/api/products/12`), intermediate routers and CDNs can cache the response. GraphQL is difficult to cache because all requests use `POST` to the same `/graphql` endpoint.

# Related Technologies
- **JSON:** The universal text format used to serialize and transmit API payloads.
- **Postman:** A popular development tool used to test and document REST and GraphQL APIs.
- **gRPC:** A high-performance RPC framework developed by Google, commonly used for internal microservice communication.

# Summary

## What We Learned
- APIs enable different applications to communicate using structured formats.
- REST structures APIs around resources (nouns) using standard HTTP methods.
- GraphQL uses a single endpoint and custom query schemas to prevent over-fetching and under-fetching.

## Key Takeaways
- Use REST for public integrations and highly cacheable data endpoints.
- Use GraphQL to streamline data requirements for complex frontend layouts.

# Keywords
- API
- REST
- GraphQL
- Over-fetching
- Under-fetching
- Schema
- Resolver
- Endpoint

# Glossary

| Term | Meaning |
|---|---|
| API | Application Programming Interface; a set of protocols enabling applications to share data. |
| Resource | Any data object exposed by an API (e.g. a User, a Product, an Order). |
| Over-fetching | A network performance problem where an API returns more fields than the client actually needs. |
| Under-fetching | A performance problem where an API returns too little data, requiring multiple sequential requests. |
| Schema | A formal document outlining all data types and queries supported by a GraphQL API. |
| Resolver | The backend script that retrieves data for a specific field in a GraphQL query. |

## Next Recommended Chapters
- 01-Web-Servers-And-HTTP.md
- 04-Backend/03-Authentication-And-Sessions.md
