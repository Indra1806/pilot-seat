> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**MongoDB** is the leading NoSQL (non-relational) document database engine. Instead of storing data in rigid tables with rows and columns, MongoDB stores data in flexible, self-contained documents using a JSON-like format. It is designed to handle rapidly changing data structures, high volumes of unstructured data, and simple horizontal scaling across multiple servers.

# Why It Exists
In the early 2000s, as web applications grew to handle millions of users, developers grew frustrated with the rigidity of relational SQL databases. If a developer wanted to add a new feature that required a new user profile field, they had to run database migrations to alter tables containing millions of rows, which could take hours and lock the database. Furthermore, matching complex nested objects (like a user profile with multiple phone numbers and addresses) required writing complex `JOIN` queries. Engineers created MongoDB to provide a schema-free database where nested data could be stored in a single document, allowing developers to change data structures instantly without running migrations.

# Problem It Solves
MongoDB solves rigid schema restrictions, database migration downtime, and complex relational join bottlenecks.

### Before MongoDB (Relational SQL Database):
- Adding a new field to a database table required running migrations, which risked table locks and application downtime.
- Storing nested lists (like a user's purchase history) required creating multiple tables linked by foreign keys, making data retrieval slow due to complex joins.
- Scaling database capacity required buying larger, more expensive hardware (vertical scaling).

### After MongoDB (NoSQL Document Database):
- The database is schema-flexible: different documents in the same collection can have completely different fields.
- Nested structures are saved directly inside a single document as arrays or sub-documents, allowing the backend to retrieve all related data in a single quick read.
- Data is easily distributed across multiple cheap servers (horizontal scaling) using built-in clustering.

# Core Concepts
MongoDB replaces relational concepts with document-focused structures:

1. **Document:** The basic unit of data in MongoDB, equivalent to a row in a relational database. Documents are stored in **BSON** (Binary JSON) format, which allows them to hold nested arrays, dates, and sub-documents.
2. **Collection:** A grouping of MongoDB documents, equivalent to a table in a relational database. Unlike tables, collections do not enforce a fixed schema on the documents within them.
3. **BSON (Binary JSON):** A binary-serialized optimization of JSON that MongoDB uses to store documents. BSON extends JSON by adding support for specific data types like dates, 64-bit integers, and raw binary data, while allowing fast binary lookups.

# Architecture / Components
The document model comparison between a relational table and a MongoDB collection:

```text
  Relational Table (Rigid Rows)            MongoDB Collection (Flexible Documents)
  +----+-------+-------+                  +----------------------------------------+
  | id | name  | phone |                  |  Drawer: [ Users Collection ]          |
  +----+-------+-------+                  |                                        |
  | 1  | Alice | 123   |                  |  {                                     |
  | 2  | Bob   | NULL  |  ─────────────>  |    "_id": 1,                           |
  +----+-------+-------+                  |    "name": "Alice",                    |
                                          |    "phones": ["123-456", "789-012"]    |
                                          |  },                                    |
                                          |  {                                     |
                                          |    "_id": 2,                           |
                                          |    "name": "Bob",                      |
                                          |    "address": { "city": "Seattle" }    |
                                          |  }                                     |
                                          +----------------------------------------+
```

- **Documents (Folders):** Self-contained data structures containing key-value pairs.
- **Collections (Drawers):** Folders grouped together without strict schema enforcement.
- **BSON Storage Engine:** Compresses documents and indexes fields for high-speed retrieval.

# Workflow
The lifecycle of inserting and reading a document in MongoDB:

```text
Step 1: The application creates a Javascript object containing nested user data.
                             ↓
Step 2: The MongoDB client converts the Javascript object into binary **BSON** format.
                             ↓
Step 3: The client transmits the BSON data to the MongoDB server.
                             ↓
Step 4: The server assigns a unique `_id` (ObjectId) key and writes the document to the collection.
                             ↓
Step 5: When queried, the server reads the BSON document from disk and returns it as JSON.
```

# Real World Examples
Think of MongoDB as a **manila folder filing cabinet** and SQL as a **giant grid spreadsheet**.
- A relational database is like a spreadsheet where every row must have the exact same columns. If you add a "Twitter handle" column for one user, you must add it for all 1 million users, even if 99% of them don't have Twitter accounts (leaving columns filled with empty `NULL` values).
- MongoDB is like a filing cabinet drawer (**Collection**). Each user has their own manila folder (**Document**). 
- If Alice has two phone numbers, you write them on a list and drop the sheet into her folder. If Bob has no phone number but has a Twitter handle, you write `Twitter: @bob` on his sheet and drop it in. You don't have to rewrite everyone else's folders just because Bob has different details. Each folder stands on its own.

# Implementation
Here is how to connect, insert, and query documents in MongoDB using Node.js:

### 1. Connecting and Inserting Documents
```javascript
const { MongoClient } = require('mongodb');
const uri = "mongodb://localhost:27017";
const client = new MongoClient(uri);

async function runMongoDB() {
  try {
    await client.connect();
    const database = client.db('store');
    const customers = database.collection('customers');

    // Insert a flexible document with nested objects and arrays
    const newCustomer = {
      name: "Alice Smith",
      email: "alice@email.com",
      contacts: ["123-4567", "987-6543"], // Nested Array
      address: {                          // Nested Sub-document
        city: "Seattle",
        zip: "98101"
      },
      preferences: { theme: "dark" }      // Dynamic fields
    };

    const result = await customers.insertOne(newCustomer);
    console.log(`Created customer document with id: ${result.insertedId}`);
  } finally {
    await client.close();
  }
}
runMongoDB();
```

### 2. Querying Documents in MongoDB (No SQL required)
MongoDB uses object-based query filters instead of SQL text queries:

```javascript
// Query equivalent to: SELECT * FROM customers WHERE address.city = 'Seattle'
const query = { "address.city": "Seattle" };

const customerCursor = customers.find(query);
await customerCursor.forEach(customer => {
  console.log(`Found Customer: ${customer.name} using theme ${customer.preferences.theme}`);
});
```

# Best Practices
- **Design for Your Access Patterns:** Since MongoDB does not perform joins well, structure your documents so that all the information your app needs to load a page is stored together inside a single document.
- **Always Index Query Filters:** Just like SQL, MongoDB needs indexes. If you query nested properties (like `address.city`), make sure to build a single or compound index on that nested key to prevent slow collection scans.
- **Do Not Grow Documents Indefinitely:** MongoDB has a hard limit of **16MB** per single BSON document. Do not push growing lists (like a history of all page views) inside a single document; split them into separate documents instead.

# Industry Standards
MongoDB is the core database choice for the popular **MERN stack** (MongoDB, Express, React, Node.js). Companies like EA, eBay, and Forbes use MongoDB to store catalog items, player profiles, and content management articles where data structures change rapidly.

# Common Mistakes
- **Normalizing MongoDB Schemas:** Splitting data across multiple collections and trying to rebuild them using `$lookup` (MongoDB's equivalent of a JOIN). This is incredibly slow and indicates that a relational database should have been used instead.
- **Ignoring Document Size Limits:** Attempting to store large binary files (like images or PDF reports) directly inside a document. For files larger than 16MB, developers must use MongoDB's **GridFS** storage system or save the file to an external cloud bucket and store only the URL in MongoDB.
- **Assuming NoSQL means "No Schema at All":** Allowing the application to write random, corrupted data into collections without validation. Developers should use application-level schemas (via libraries like **Mongoose**) to enforce basic data sanity rules.

# Security & Performance Considerations
- **NoSQL Injection:** While MongoDB does not use SQL, it is still vulnerable to injection attacks if query filters are built using raw user input. Attackers can pass operator objects (like `{"$gt": ""}`) to bypass login screens. Developers must sanitize inputs or use schema libraries.
- **Memory-Heavy Workloads:** MongoDB relies heavily on caching indexes in system memory (RAM). If your active indexes grow larger than the server's available RAM, MongoDB performance will drop off a cliff as it is forced to read indexes from the hard drive.

# Related Technologies
- **Mongoose:** A popular Object Data Modeling (ODM) library for Node.js that adds structure, validation, and schema definitions to MongoDB.
- **GridFS:** A MongoDB system for storing and retrieving files that exceed the 16MB document size limit.

# Summary

## What We Learned
- MongoDB is a non-relational document database storing data in flexible, nested BSON structures.
- It eliminates the need for strict schemas and complex table joins by nesting arrays and sub-documents directly within a single record.
- It scales horizontally by default, distributing collections across cheap servers.

## Key Takeaways
- Use MongoDB when data structures are dynamic, rapidly changing, or nested.
- Embed related data (arrays, objects) directly inside documents to avoid slow, multi-collection lookups.
- Enforce basic schema boundaries in application code using Mongoose to protect data integrity.

# Keywords
- MongoDB
- NoSQL
- BSON
- Document
- Collection
- ObjectId
- Embedding
- Mongoose
- Horizontal Scaling
- NoSQL Injection

# Glossary

| Term | Meaning |
|---|---|
| Document | A single record in MongoDB, represented in BSON format containing key-value pairs. |
| Collection | A grouping of documents inside MongoDB, equivalent to a relational table. |
| BSON | Binary JSON; the binary serialization format used by MongoDB to store data. |
| Embedding | The practice of nesting related data (like a list of addresses) directly inside a parent document. |
| ODM | Object Data Modeling; a library (like Mongoose) that translates database documents into code objects with schema validation. |
| ObjectId | A 12-byte unique identifier automatically generated by MongoDB to act as a document's primary key. |

## Next Recommended Chapters
- 12-Redis.md
