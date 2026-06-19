> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Relational Database** is a database model that stores and organizes data in structured grids called **Tables** (formally known as *Relations*). Data in different tables is linked—or related—based on shared data values, allowing developers to query complex connections across the database using mathematical logic.

# Why It Exists
Before Edgar F. Codd invented the relational model at IBM in 1970, databases stored data in rigid trees (hierarchical model) or networks. If a programmer wanted to search for a customer's order, they had to write code specifying the exact physical storage path on the hard drive to traverse the network connections. If the database structure changed by even one folder, all application code broke. Codd created the relational model to separate *how data is logically structured* from *how it is physically stored on the disk*, allowing developers to query data using simple math statements without worrying about hardware details.

# Problem It Solves
Relational databases solve data redundancy, structural dependency, and relational integrity problems.

### Before Relational Databases (Network & File Models):
- Code was locked to physical disk paths; changing hardware or file layouts broke all software.
- Data was duplicated repeatedly (e.g. copying a customer's full address inside every order record), wasting disk space and leading to inconsistent data if the customer moved.
- Orphaned data was common (e.g., deleting a customer record but leaving their orders behind in memory with no way to find who owned them).

### After Relational Databases:
- Data is organized logically into independent, normalized tables.
- Tables are connected dynamically using values, avoiding physical path dependencies.
- **Referential Integrity Constraints** prevent orphaned data by automatically blocking actions that would break relationships.

# Core Concepts
To work with relational databases, you must understand the structural components:

1. **Tables (Relations):** A two-dimensional grid representing a specific entity type (like `Customers` or `Products`).
   - **Rows (Tuples):** The horizontal records representing a single unique item (e.g. Customer #15).
   - **Columns (Attributes):** The vertical fields defining properties of the entity (e.g., Email, Phone Number).
2. **Data Constraints (The Guardrails):** Rules applied to columns to ensure data remains correct and complete:
   - `NOT NULL`: Forces a column to hold a value (cannot be blank).
   - `UNIQUE`: Prevents duplicate entries in that column (e.g. no two users can register the same email address).
   - `CHECK`: Validates that data meets custom rules (e.g. `CHECK (age >= 18)`).
3. **Primary and Foreign Keys:** The link bridges. A **Primary Key** is the unique ID column for a table. A **Foreign Key** is a column in a second table that points to the primary key of the first table, establishing the relationship.

# Architecture / Components
The components of a Relational Database Management System (RDBMS):

```text
       [ SQL Query ] ──> [ RDBMS Parser ]
                                │
                                ▼
                       [ Schema Validator ] ──> Verifies columns & constraints
                                │
                                ▼
                       [ Execution Engine ] ──> Runs Relational algebra
                                │
                                ▼
                       [ Storage Manager ]  ──> Reads/writes database files
```

- **RDBMS:** Relational Database Management System; the database software (like PostgreSQL) that manages the tables, logs, and engines.
- **System Catalog:** The internal database where the RDBMS stores metadata explaining the schema (what tables exist, what columns they have, and what rules are active).

# Workflow
How a relational database enforces integrity rules during an insert:

```text
Step 1: A client attempts to insert a new order: `INSERT INTO orders (user_id, price) VALUES (99, -50)`.
                             ↓
Step 2: The RDBMS intercepts the request and reads the Schema Validator catalog rules.
                             ↓
Step 3: Check 1: The database checks if the user ID `99` actually exists in the `Users` table (Foreign Key constraint).
                             ↓
Step 4: Check 2: The database checks if the price is a positive number (CHECK constraint: `price > 0`).
                             ↓
Step 5: Since the price is `-50` (violating the CHECK rule), the transaction fails immediately.
                             ↓
Step 6: The RDBMS returns a constraint violation error and discards the insert, keeping data clean.
```

# Real World Examples
Think of a relational database as a **structured corporate ledger book**.
- The ledger book contains multiple pages, where each page is a **Table** (e.g. Page 1: "Employees", Page 2: "Laptops").
- Every page is a grid. The columns are pre-printed header labels (Name, ID, Hire Date) that define what data is allowed in that column. You cannot write a name in the "Hire Date" column.
- Every row is a single employee record.
- To prevent confusion, every employee has a unique Badge Number (**Primary Key**).
- If you assign John Smith a laptop, you do not write John's address, department, and phone number in the laptop ledger. You simply write John's Badge Number (**Foreign Key**) next to the laptop's serial number. If you need John's details later, you look up his badge number on the "Employees" page.

# Implementation
Here is how a developer defines a relational table structure with strict constraints using SQL:

```sql
-- 1. Create the parent Users table
CREATE TABLE users (
  id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Unique ID (Primary Key)
  username VARCHAR(50) NOT NULL UNIQUE,            -- Text must be unique, cannot be blank
  age INT CHECK (age >= 13)                         -- Number constraint check
);

-- 2. Create the child Orders table with relationships
CREATE TABLE orders (
  order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  user_id INT,
  total_amount NUMERIC(10, 2) NOT NULL,
  
  -- Foreign Key Constraint linking the orders table to the users table
  CONSTRAINT fk_user
    FOREIGN KEY(user_id) 
	  REFERENCES users(id)
	  ON DELETE CASCADE -- If a user is deleted, automatically delete all their orders
);
```

# Best Practices
- **Define Keys Properly:** Every table must have a primary key. Never create a table without one, as this makes rows hard to target and query.
- **Enforce Constraints in the Database:** Do not rely solely on your Node.js or Python backend code to check for duplicate emails or negative prices. Enforce these constraints directly in the database schema. Code can have bugs or bypass checks, but database constraints are absolute.
- **Choose the Right Cascade Actions:** When defining foreign keys, specify what should happen if a parent record is deleted. Use `ON DELETE RESTRICT` if you want to block the delete if the user has orders, or `ON DELETE CASCADE` to delete child orders automatically.

# Industry Standards
The standard language used to interact with relational databases is **SQL (Structured Query Language)**. The most popular open-source relational database management systems in the software industry are **PostgreSQL** (highly advanced, feature-rich) and **MySQL** (simple, fast, widely deployed).

# Common Mistakes
- **Orphaned Rows:** Deleting a user but forgetting to delete their orders because you did not set up a Foreign Key constraint. This leaves order rows in your database pointing to a non-existent user ID, causing frontend crashes.
- **Storing Lists in a Single Column:** Saving a list of tags or user hobbies as a comma-separated text string (like `"reading,chess,hiking"`) inside a single table cell. This violates relational normalization rules and makes searching for users who like "chess" extremely slow. Use separate tables.

# Security & Performance Considerations
- **Constraint Costs:** Enforcing complex rules (like unique constraints) adds a minor write delay because the database must check the entire table for duplicates before saving a new row. Keep check constraints simple.
- **Relational Integrity Protection:** Constraints act as a final, automated security barrier, preventing bugs in your application code from saving corrupted, invalid, or illegal data values.

# Related Technologies
- **PostgreSQL / MySQL:** The standard relational database engines.
- **Relational Algebra:** The branch of mathematics that defines how tables are filtered, projected, and joined.

# Summary

## What We Learned
- Relational databases structure data in grids called tables containing rows and columns.
- Table columns use data constraints (NOT NULL, UNIQUE, CHECK) to enforce schema integrity.
- Primary and Foreign keys establish mathematical relationship bridges between tables, preventing orphaned records.

## Key Takeaways
- Enforce business logic rules directly in the database using constraints.
- Link tables mathematically using foreign keys with defined delete cascade rules.

# Keywords
- Relational Database
- Table
- Row
- Column
- Constraint
- Primary Key
- Foreign Key
- Schema Integrity

# Glossary

| Term | Meaning |
|---|---|
| RDBMS | Relational Database Management System; the database software engine that stores and manages relational tables. |
| Constraint | A rule applied to a database column that restricts what values can be inserted into it. |
| Tuple | The formal mathematical term for a single horizontal row in a relational table. |
| Attribute | The formal mathematical term for a vertical column field in a relational table. |
| Cascade Delete | A constraint setting that automatically deletes child records if the corresponding parent record is deleted. |

## Next Recommended Chapters
- 01-Introduction-to-Databases.md
- 03-SQL-Fundamentals.md
