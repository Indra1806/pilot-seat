> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
An **ORM** (Object-Relational Mapper) is a software library that lets developers query and manipulate database records using their native programming language (like JavaScript or Python) instead of writing raw database query language (SQL). A **Query Builder** is an intermediate library that helps developers write programmatically constructed queries in code, offering a middle ground between raw SQL and a full ORM.

# Why It Exists
Databases use a table-based data model queried using Structured Query Language (SQL). Application code (like Node.js or Python) uses an Object-Oriented model where data is stored in classes, objects, and memory structures. Historically, matching these two different designs required writing hundreds of lines of tedious helper code (called "boilerplate") to manually map table rows into code objects. Furthermore, writing raw SQL strings inside code files led to frequent syntax errors and left databases vulnerable to hacking. Engineers created ORMs and Query Builders to bridge this "Object-Relational impedance mismatch," allowing developers to write clean, type-safe database queries quickly.

# Problem It Solves
ORMs and Query Builders solve object mapping effort, syntax typos, and SQL injection risks.

### Before ORMs & Query Builders (Raw SQL Strings):
- Developers had to write database queries as plain text strings inside code. If you misspelled a table name or missed a comma, you only found out when the server crashed at runtime.
- Mapping a query result (raw rows) into a nested code object (like a User holding a list of Orders) required writing dozens of lines of manual mapping loops.
- Code was locked to one database type. Swapping from MySQL to PostgreSQL required manually rewritng hundreds of SQL strings.

### After Query Builders:
- Queries are constructed programmatically using clean method chaining (e.g., `.select().from().where()`).
- The builder automatically generates syntactically correct SQL matching the active database engine type.

### After ORMs:
- Database tables are mapped directly to code objects. You fetch a user, and you get a complete, typed class object automatically.
- Database changes (migrations) are managed through code files.

# Core Concepts
To write backend database layers, you must understand the three database access patterns:

1. **Raw SQL:** Writing query strings directly.
   ```sql
   SELECT * FROM users WHERE age > 21;
   ```
2. **Query Builders:** Programmatic SQL generation using method chaining.
   ```javascript
   db('users').select('*').where('age', '>', 21);
   ```
3. **ORMs (Object-Relational Mapping):** Full abstraction. You treat the database like a local collection of objects.
   ```javascript
   await prisma.user.findMany({ where: { age: { gt: 21 } } });
   ```
4. **Migrations:** Version-controlled scripts used to create, update, or alter database table structures safely over time as application requirements change.

# Architecture / Components
The database access layer sits between your service logic and the database engine:

```text
  [ Business Service Logic ] (e.g. Create Order)
              │
              ▼
  [ ORM Layer (e.g. Prisma) ] ──> Translates objects to SQL
              │
              ▼
  [ Database Driver (pg) ]    ──> Manages network connection
              │
              ▼
  [ Database (PostgreSQL) ]   ──> Executes query & returns rows
```

- **Schema Model:** The configuration file (like `schema.prisma`) defining your database tables and their properties in code.
- **Query Engine:** The internal translation core of the ORM that parses code methods and compiles them into optimized SQL queries.
- **Connection Pool:** The manager inside the driver that holds open a group of reusable network connections to the database to speed up queries.

# Workflow
How an ORM executes a query:

```text
Step 1: Code calls `await prisma.user.findUnique({ where: { id: 5 } })`.
                             ↓
Step 2: The ORM reads the code request and validates that the types are correct.
                             ↓
Step 3: The ORM compiles the method into SQL: `SELECT id, name, email FROM users WHERE id = 5 LIMIT 1`.
                             ↓
Step 4: The database driver sends the SQL string to the database engine.
                             ↓
Step 5: The database executes the query and returns raw data rows.
                             ↓
Step 6: The ORM parses the raw data and instantiates a typed User object back to the code.
```

# Real World Examples
Think of database query approaches as **different translation systems**.
- **Raw SQL** is like **speaking a foreign language fluently with no helper**. If you want a coffee, you must speak the language perfectly. If you pronounce one syllable wrong (a syntax typo), the barista doesn't understand you, and you get no coffee. It is powerful and fast, but leaves no room for errors.
- A **Query Builder** is like using a **structured phrasebook**. You look up pre-approved phrases: "Select -> Item -> Coffee." The phrasebook guarantees you speak the correct syntax, preventing typos.
- An **ORM** is like hiring a **full-time personal translator and manager**. You never see or hear the foreign language. You simply tell your translator in your native tongue: *"Please get me a coffee,"* and they handle the conversation, payment, and delivery behind the scenes, handing you the hot cup.

# Implementation
Here is how you query a database using **Prisma** (a modern, type-safe ORM for Node.js):

### 1. Define the Schema (`schema.prisma`)
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int
  author   User   @relation(fields: [authorId], references: [id])
}
```

### 2. Fetch data in your JavaScript code
```javascript
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

async function getAuthorDetails(postId) {
  // Fetch the post AND automatically join the author data in one step!
  const post = await prisma.post.findUnique({
    where: { id: postId },
    include: { author: true } // Tells the ORM to run a SQL JOIN automatically
  });

  console.log(`Post: ${post.title}`);
  console.log(`Author Name: ${post.author.name}`);
}
```

# Best Practices
- **Use ORMs for Standard CRUD:** Use an ORM for 95% of your daily database work (creating records, reading profiles, updating settings). It saves massive development time.
- **Use Raw SQL for Complex Analytics:** If you need to write a massive, highly complex query to calculate company financial reports across millions of rows, write a raw SQL query. ORMs are notoriously bad at compiling highly complex, nested analytical queries efficiently.
- **Never Query in Loops:** Avoid fetching a list of items and then running an ORM query inside a loop to fetch details for each item (the N+1 query problem). Always use the ORM's relational inclusion methods (like `include` or `join`) to fetch all data in a single query.

# Industry Standards
In the JavaScript/TypeScript ecosystem, **Prisma** and **TypeORM** are the leading ORM libraries, while **Knex.js** is the standard Query Builder. In Python, **SQLAlchemy** and the built-in **Django ORM** are the standard, while Java developers use **Hibernate**.

# Common Mistakes
- **Assuming ORM is Magic:** Forgetting that there is still a database running behind the scenes. Writing code like `const users = await prisma.user.findMany()` on a table containing 10 million records will try to download gigabytes of data into your server's RAM, instantly crashing your application.
- **Neglecting Indexes:** Thinking the ORM automatically makes queries fast. You still need to manually configure indexes in your database schema files.

# Security & Performance Considerations
- **Parameterized Queries:** Both ORMs and Query Builders protect you from SQL injection attacks by default because they compile queries using **Parameterized Parameters** (replacing user inputs with placeholders like `$1`), ensuring the database treats inputs strictly as text rather than executable commands.
- **The "Abstraction Tax":** Because ORMs translate your code into SQL, they introduce a minor performance delay (overhead). If you are building a high-speed stock trading engine where microseconds matter, bypass ORMs and use raw SQL.

# Related Technologies
- **Prisma / Sequelize:** Standard JavaScript/TypeScript ORM tools.
- **Knex.js:** A popular Javascript Query Builder.
- **pg / mysql2:** The low-level database drivers that actually establish network sockets to databases.

# Summary

## What We Learned
- ORMs translate database tables into native programming code classes and objects.
- Query Builders programmatically construct SQL statements, offering code-safety without full object mapping.
- Database migrations act as version control scripts to modify database structures safely.

## Key Takeaways
- Use ORMs to accelerate development speeds and guarantee SQL injection security.
- Avoid N+1 queries by leveraging relational preloading techniques in your ORM client.

# Keywords
- ORM
- Query Builder
- Prisma
- SQL
- Migration
- Schema
- Data Mapping
- N+1 Problem

# Glossary

| Term | Meaning |
|---|---|
| ORM | Object-Relational Mapping; a library that maps database tables to code structures. |
| Query Builder | A programmatic code translator that converts method chains into clean SQL syntax. |
| Migration | A version-controlled database schema script used to alter table structures. |
| Boilerplate | Tedious, repetitive code that is required in multiple places with little or no modification. |
| Prepared Statement | A database feature where a SQL query is compiled with placeholders before inserting user inputs, preventing SQL injection. |

## Next Recommended Chapters
- 05-Databases-SQL-vs-NoSQL.md
- 04-Backend/07-Caching-with-Redis.md
