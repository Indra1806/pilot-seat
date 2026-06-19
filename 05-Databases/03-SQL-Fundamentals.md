> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**SQL**, short for **Structured Query Language**, is the universal programming language used to communicate with, manage, and query relational database management systems. SQL acts as the standard set of instructions that you send to a database engine telling it exactly what data to retrieve, insert, edit, or delete from its tables.

# Why It Exists
Before SQL was developed by IBM researchers in the 1970s, database engines had no common language. To query a database, developers had to write custom assembly-style code indicating exactly how the computer read hard drive segments. If you switched from one database supplier to another, you had to throw away all your code and rewrite it from scratch. Engineers created SQL to establish a unified, readable language modeled on simple English sentences (like SELECT, INSERT, FROM, and WHERE), allowing developers to write database queries that run compatibly across different database products.

# Problem It Solves
SQL solves database command fragmentation, data retrieval complexity, and structured data filtering.

### Before SQL:
- Querying data required writing complex, low-level loops to traverse physical disk addresses.
- Code was locked to specific database hardware vendors.
- Joining data across different folders required writing custom sorting and filtering algorithms in application memory.

### After SQL:
- Developers write declarative, human-readable queries.
- A single SQL query can filter, sort, group, and merge data from multiple tables in microseconds.
- SQL runs on PostgreSQL, MySQL, SQLite, Oracle, and Microsoft SQL Server with minimal syntax changes.

# Core Concepts
To write database queries, you must understand the two main categories of SQL statements:

1. **DDL (Data Definition Language):** Statements used to define or modify the structure of the database itself (like creating or deleting tables).
   - `CREATE TABLE`: Build a new table grid.
   - `ALTER TABLE`: Add or remove columns.
2. **DML (Data Manipulation Language):** Statements used to manage the actual data records inside the tables (CRUD operations).
   - `SELECT`: Read records.
   - `INSERT`: Create records.
   - `UPDATE`: Modify records.
   - `DELETE`: Remove records.
3. **The WHERE Clause (Filtering):** The filter gate that specifies exactly which rows should be affected by a query (e.g. `WHERE age > 18`).
4. **Table JOINs (Merging):** Sticking two tables together on the fly based on a shared column (usually matching a Foreign Key to a Primary Key).

# Architecture / Components
The SQL execution process inside a database engine:

```text
  [ SQL Query Text ] ──> [ Parser ] ──> [ Optimizer (Query Planner) ]
                                                    │
                                                    ▼
                                          [ Execution Engine ]
                                                    │
                                                    ▼
                                        [ Returns Mapped Data ]
```

- **Parser:** The syntax check system that reads your SQL text and verifies that the spelling and grammar are correct, turning it into a query tree.
- **Optimizer (Query Planner):** The calculator engine that decides the fastest way to run your query (e.g. deciding whether to use an index or search the whole table, and what order to merge tables in).
- **Execution Engine:** The core system that retrieves data bytes from the disk and outputs the results.

# Workflow
How a JOIN query operates to fetch orders with user names:

```text
Step 1: Client sends: `SELECT orders.id, users.name FROM orders JOIN users ON orders.user_id = users.id;`
                             ↓
Step 2: The database Parser verifies the syntax and checks if both tables exist.
                             ↓
Step 3: The Query Planner decides whether to scan the `orders` or `users` table first.
                             ↓
Step 4: The database merges the tables by matching the `user_id` values in `orders` to `id` values in `users`.
                             ↓
Step 5: The database filters out columns, keeping only the order ID and user name.
                             ↓
Step 6: The engine returns the mapped rows to the application.
```

# Real World Examples
Think of SQL as writing **structured request slips** to a **library archive supervisor**.
- The database tables are the shelves of archives.
- **SELECT** is like writing: *"Show me the Titles of all Books in Aisle 4"* (`SELECT title FROM books`).
- **WHERE** is the filter: *"...but only if they were published after 2010"* (`WHERE published_year > 2010`).
- **INSERT** is handing the supervisor a new card saying: *"Add this book to the shelves"* (`INSERT INTO books...`).
- **UPDATE** is: *"Find Book Card #12 and change its location to Room B"* (`UPDATE books SET location = 'Room B' WHERE id = 12`).
- **JOIN** is asking: *"Show me all books in Aisle 4, and fetch the author's biography details from the Author drawer by matching the Author IDs on both cards."*

# Implementation
Here are the essential SQL commands developers use to manage records:

### 1. Insert records (Create)
```sql
INSERT INTO employees (name, department, salary) 
VALUES ('Alice', 'Engineering', 95000.00);
```

### 2. Query and Filter records (Read)
```sql
SELECT name, salary 
FROM employees 
WHERE department = 'Engineering' AND salary > 80000 
ORDER BY salary DESC;
```

### 3. Join Tables (Merging related records)
```sql
-- Fetch orders along with the customer's name by joining on the user_id foreign key
SELECT orders.id, orders.total_amount, users.name 
FROM orders
INNER JOIN users ON orders.user_id = users.id;
```

### 4. Group and Aggregate data (Calculations)
```sql
-- Count how many employees are in each department and calculate their average salary
SELECT department, COUNT(*) as employee_count, AVG(salary) as average_salary
FROM employees
GROUP BY department;
```

# Best Practices
- **Never Use SELECT \* in Production:** Avoid writing `SELECT * FROM users`. This downloads every single column, including long text bios or password hashes you don't need, wasting network bandwidth. Explicitly list the columns you need: `SELECT id, email FROM users`.
- **Always Use WHERE with UPDATE and DELETE:** If you write `UPDATE users SET status = 'suspended'`, you will suspend *every single user* in the entire database because you forgot the filter. Always write: `UPDATE users SET status = 'suspended' WHERE id = 12`.
- **Write SQL Keywords in UPPERCASE:** Write SQL commands in uppercase (like `SELECT`, `FROM`, `WHERE`, `JOIN`) and table/column names in lowercase. This makes queries easy to scan and read.

# Industry Standards
While SQL is standardized by ANSI, every database engine implements its own minor dialect changes (e.g. PostgreSQL uses `LIMIT`, while Microsoft SQL Server uses `TOP` to restrict row counts). Standard **ANSI SQL** syntax remains compatible across PostgreSQL, MySQL, SQLite, Oracle, and MS SQL.

# Common Mistakes
- **Incorrect JOIN Types:** Using an `INNER JOIN` when you should use a `LEFT JOIN`.
  - An `INNER JOIN` only returns rows where a match exists in *both* tables. If you join `users` and `orders`, users who have never placed an order will be completely excluded from the results.
  - A `LEFT JOIN` returns *all* users from the left table, and simply fills in blank fields (nulls) for their orders if they have never purchased anything.
- **Forgetting Group By Columns:** Trying to run a query like `SELECT department, name, AVG(salary) GROUP BY department`. You cannot select the individual `name` column when grouping by `department` because the database doesn't know which employee name to show for the grouped department average.

# Security & Performance Considerations
- **SQL Injection Prevention:** Never write queries by stitching strings together: `query = "SELECT * FROM users WHERE name = '" + input + "'"`. Attackers can input code syntax (like `' OR 1=1; --`) to bypass authentication and delete tables. Always use **Parameterized Queries** (prepared statements) where inputs are treated strictly as data.
- **Index Optimization:** When writing `WHERE` filters or `JOIN` matches on columns (like filtering by `created_at` date), ensure those columns are indexed, otherwise the database engine must run a slow full-table scan.

# Related Technologies
- **SQLite:** A lightweight, single-file relational database engine popular for mobile apps and local testing.
- **Postgres / MySQL:** The leading production-grade SQL databases.
- **ORM / Query Builders:** Coding libraries used to generate SQL queries programmatically.

# Summary

## What We Learned
- SQL is the standard declarative language used to manage and query relational databases.
- DDL manages table structures; DML manages database records (CRUD operations).
- JOIN clauses dynamically merge separate tables, and GROUP BY consolidates records for statistical math calculations.

## Key Takeaways
- Always specify target filters (WHERE) when executing updates or deletes.
- Write explicit column names rather than using wildcard selections (`*`) to preserve network bandwidth.

# Keywords
- SQL
- DDL
- DML
- SELECT
- JOIN
- WHERE
- GROUP BY
- Aggregation

# Glossary

| Term | Meaning |
|---|---|
| DDL | Data Definition Language; SQL commands used to build and alter database structures (like tables). |
| DML | Data Manipulation Language; SQL commands used to manage data records (insert, read, update, delete). |
| INNER JOIN | A join type that returns records only when there is a matching key value in both tables. |
| LEFT JOIN | A join type that returns all records from the first (left) table, and matching records from the second (right) table. |
| Aggregation | Mathematical calculations performed on a group of rows, returning a single value (e.g. SUM, AVG, COUNT). |

## Next Recommended Chapters
- 02-Relational-Databases.md
- 05-Relationships-And-Keys.md
