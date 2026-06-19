> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Database** is a software program running on a server that is designed to store, organize, query, and retrieve data efficiently at scale. Databases are broadly divided into two main categories: **SQL** (Structured Query Language, or Relational) databases and **NoSQL** (Not Only SQL, or Non-Relational) databases, which differ in how they store data, organize schemas, and scale to handle traffic.

# Why It Exists
Before database software existed, early computer applications saved data in simple, raw text files on a computer's hard drive. If you wanted to search for a customer's order, your program had to read the entire text file from top to bottom, which was extremely slow. If two users tried to save updates at the exact same millisecond, the file would become corrupted. Engineers created relational SQL databases in the 1970s to establish a secure, organized table-based storage system that guaranteed data safety. Decades later, as internet services grew to handle millions of active users and massive streams of dynamic data, engineers created NoSQL databases to handle flexible data shapes and easily scale storage across multiple computers.

# Problem It Solves
SQL and NoSQL solve slow data retrieval, data corruption (ACID compliance), and horizontal scaling problems.

### Before Databases (Raw Files):
- Searching data was extremely slow, requiring linear scans of text files.
- Simultaneous updates corrupted data files (Concurrency conflicts).
- Relating data (linking a customer to an order) was manual and error-prone.

### After SQL (Relational Tables):
- Data is organized in structured tables with strict columns.
- Transactions are guaranteed to be safe (using ACID principles): either the entire transaction succeeds (like transferring money), or it rolls back completely, preventing half-finished data errors.

### After NoSQL (Flexible Documents):
- Developers store data with changing, unstructured shapes without writing complex table migration scripts.
- Storage scales easily by distributing folders across dozens of server computers (Horizontal scaling).

# Core Concepts
To choose the right database for your application, you must understand the two main models:

1. **Relational (SQL):** Data is stored in **Tables** containing **Rows** (records) and **Columns** (properties). Tables are linked together using **Keys** (Primary and Foreign keys) to establish relationships (like linking a user ID in the `Orders` table to the `Users` table). Relational databases enforce a strict **Schema** (blueprint) that defines exactly what data type each column must hold.
2. **Non-Relational (NoSQL):** Data is commonly stored in **Documents** (usually JSON-like structures) grouped into **Collections**. Each document can contain completely different fields, allowing for a flexible, dynamic schema.
3. **ACID vs. BASE:**
   - **ACID (SQL Standard):** Guarantees absolute data consistency and safety (critical for banking or inventory).
   - **BASE (NoSQL Standard):** Prioritizes availability and speed over absolute consistency. The data might be temporarily out-of-sync across servers but will become consistent eventually (eventual consistency, great for social media likes or chat messages).

# Architecture / Components
Comparing the storage layouts of SQL and NoSQL databases:

### SQL Architecture (Relational Tables)
```text
  [ Users Table ]                     [ Orders Table ]
  +----+-------+                      +----+---------+---------+
  | ID | Name  | <── (Relationship) ──| ID | User_ID | Total   |
  +----+-------+                      +----+---------+---------+
  | 1  | Alice |                      | 99 | 1       | $50.00  |
  +----+-------+                      +----+---------+---------+
```

### NoSQL Architecture (Flexible Document Collections)
```json
  // Document 1 (User Profile)
  { "id": 1, "name": "Alice", "hobbies": ["reading", "cycling"] }
  
  // Document 2 (User Profile with different fields)
  { "id": 2, "name": "Bob", "company": "Tech Corp", "skills": { "js": 5 } }
```

- **Primary Key:** A unique identifier column (like a user ID) that uniquely identifies a row in a SQL table.
- **Foreign Key:** A column in one SQL table that references the Primary Key of another table, creating a relationship.
- **Document (NoSQL):** A self-contained JSON object containing key-value pairs representing a single record.

# Workflow
How queries retrieve related data:

### SQL Joins (Stitching Tables together)
```text
Step 1: Client sends query: "SELECT orders and customer names."
                             ↓
Step 2: Database engine matches the Foreign Key `user_id` in the `Orders` table to the Primary Key `id` in the `Users` table.
                             ↓
Step 3: Database dynamically merges the rows and returns a clean, temporary table of results.
```

### NoSQL Retrieval (Self-contained documents)
```text
Step 1: Client sends query: "Get User 1 document."
                             ↓
Step 2: Database engine fetches the single User 1 JSON document.
                             ↓
Step 3: Database returns the document immediately (no merging required, since all hobbies and details are already nested inside the user object).
```

# Real World Examples
Think of database architectures as **different filing systems**.
- **SQL** is like a **strict library card catalog cabinet**. Every book card must fit exactly the same pre-printed template: Title, Author, Year, and Publisher ID. If a book doesn't have an author, you must leave the slot empty, but you cannot delete the card slot itself. To know the publisher's address, you check the Publisher ID, walk over to the "Publishers" drawer, and find the card matching that ID (a JOIN). It keeps data incredibly clean and prevents repeating the publisher's address on 500 different book cards.
- **NoSQL** is like a **folder cabinet**. Each record is a folder containing a sheet of paper. Folder 1 contains a book sheet with a Title and Author. Folder 2 contains a magazine sheet with a Title, Issue Number, and a list of articles. Folder 3 holds a newspaper. You can toss any paper shape into the cabinet without asking permission (Flexible Schema). However, if you want to find links between folders, you have to read them manually since there are no pre-drawn bridges connecting the sheets.

# Implementation
Here is how you query databases using SQL language and MongoDB NoSQL language:

### 1. SQL Querying (Relational Database - e.g. PostgreSQL)
```sql
-- 1. Create a table with strict columns
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) UNIQUE
);

-- 2. Query data matching conditions
SELECT name, email 
FROM users 
WHERE id = 12;
```

### 2. NoSQL Querying (Document Database - e.g. MongoDB)
```javascript
// 1. Insert a document (no table creation or schema setup required!)
db.users.insertOne({
  name: "Alice",
  email: "alice@email.com",
  hobbies: ["chess", "swimming"] // Array nested directly inside the record
});

// 2. Query data matching conditions
db.users.find({ name: "Alice" });
```

# Best Practices
- **Use SQL by Default:** For 90% of business applications, SQL (like PostgreSQL) is the correct choice. It keeps data structured, prevents orphaned records (like an order pointing to a deleted customer), and handles complex analytical reporting easily.
- **Use NoSQL for High-Volume, Simple Shapes:** Use NoSQL (like MongoDB or DynamoDB) if you are handling massive streams of unstructured data (like logging sensor data, user analytics clicks, or chat feeds) where speed and write-capacity are more important than complex table relations.
- **Always Add Indexes:** Database tables search sequentially by default. If you have 1 million rows, searching for a user by email takes time. Add an **Index** to the `email` column to build a fast search directory, speeding up queries from seconds to milliseconds.

# Industry Standards
**PostgreSQL** and **MySQL** are the industry standards for relational SQL databases. **MongoDB** is the leading document NoSQL database, while **Redis** is the standard key-value NoSQL database used for high-speed caching.

# Common Mistakes
- **Normalizing Everything in NoSQL:** Trying to build relationships in MongoDB by creating "User IDs" in order documents and manually running multiple code queries to link them. If your data is highly relational, use a SQL database.
- **Ignoring Database Migrations in SQL:** Modifying a SQL database schema manually in production. Always write version-controlled migration files (scripts) to update table structures safely.

# Security & Performance Considerations
- **SQL Injection:** A critical security vulnerability where an attacker injects SQL commands into input fields (e.g. typing `' OR 1=1; --` in a username box) to bypass authentication and read the database. Always use parameterized queries (prepared statements) to sanitize inputs.
- **N+1 Query Problem:** A common performance bottleneck where backend code fetches a list of 100 posts, and then runs 100 separate database queries to fetch the author details for each post. Use SQL `JOIN` statements to fetch all data in a single query.

# Related Technologies
- **PostgreSQL / MySQL:** The standard relational SQL databases.
- **MongoDB / DynamoDB:** The standard non-relational document databases.
- **ORM (Object-Relational Mapping):** Libraries (like Prisma or Sequelize) that let developers write database queries using standard programming code instead of raw SQL language.

# Summary

## What We Learned
- SQL databases are structured, relational, use strict table schemas, and enforce ACID transaction safety.
- NoSQL databases offer flexible schemas, store documents or key-value pairs, and scale horizontally across servers easily.
- Database indexes are critical to speed up query execution times in production.

## Key Takeaways
- Choose SQL (PostgreSQL) when data consistency and relationships are paramount (finance, user accounts).
- Choose NoSQL (MongoDB) when data shapes are dynamic and write speeds must scale horizontally.

# Keywords
- Database
- SQL
- NoSQL
- Schema
- Relationship
- Primary Key
- ACID
- Index

# Glossary

| Term | Meaning |
|---|---|
| Schema | The formal structural blueprint defining tables, columns, and data types in a database. |
| JOIN | A SQL operation used to dynamically combine rows from two or more tables based on a related column. |
| ACID | Atomicity, Consistency, Isolation, Durability; a set of database properties ensuring safe, reliable transaction processing. |
| Horizontal Scaling | Adding more server computers to share the load of a database, rather than making a single computer more powerful. |
| Index | A special lookup table built by the database engine to speed up search queries on specific columns. |

## Next Recommended Chapters
- 04-Authorization-And-RBAC.md
- 04-Backend/06-ORMs-And-Query-Builders.md
