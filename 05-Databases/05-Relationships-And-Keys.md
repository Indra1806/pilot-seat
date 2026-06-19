> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Relationships and Keys** are the mathematical rules and pointers that connect different tables in a database. They define how individual records in one table relate to records in another table, ensuring that the entire database remains synchronized, logical, and free of orphaned or corrupt information.

# Why It Exists
Before database keys and relationships were invented, if developers wanted to connect two types of data (like a customer and their orders), they had to manually write the customer's name and details directly inside every single order record. If a customer changed their phone number, developers had to search through thousands of order files to update the number in every single record. If they missed even one file, the data became inconsistent. Engineers created unique identifiers called **Keys** to allow tables to point to each other instead of duplicating data, establishing clean, single-point-of-truth relationships.

# Problem It Solves
Relationships and keys solve data duplication, manual record tracing, and orphaned records (records pointing to things that no longer exist).

### Before Relationships and Keys (Isolated Flat Files):
- Customer details (name, email, phone) were copied into every order record, wasting storage space.
- Changing a customer's email required updating hundreds of individual order documents, risking human error.
- Deleting a customer left order records pointing to a non-existent person, creating "ghost" orders with no owner.

### After Relationships and Keys (Linked Tables):
- Customer details are stored exactly once in a `customers` table.
- Each order contains a simple pointer (Foreign Key) linking back to the customer's unique identifier (Primary Key).
- Updating a customer's email takes a single operation in one row; all order records instantly reflect the correct connection.
- Rules (referential integrity) prevent deleting a customer if they have active orders, preventing data corruption.

# Core Concepts
To connect tables, database engines use three types of keys and three primary relationship patterns:

1. **Primary Key (PK):** A column (or set of columns) that uniquely identifies a specific row in a table. No two rows in the same table can share the same Primary Key value, and it can never be empty.
2. **Foreign Key (FK):** A column in one table that stores the Primary Key value of another table, serving as a connector link between the two.
3. **Cardinality (The Mappings):** The rules governing how many records in Table A can connect to records in Table B:
   - **One-to-One (1:1):** Each row in Table A connects to exactly one row in Table B.
   - **One-to-Many (1:N):** Each row in Table A can connect to multiple rows in Table B, but each row in Table B connects to only one row in Table A.
   - **Many-to-Many (N:M):** Multiple rows in Table A can connect to multiple rows in Table B.

# Architecture / Components
In a relational database, relationships are enforced by the database engine's **Referential Integrity constraints**. The engine acts as a border control agent, checking foreign key links every time a write occurs:

```text
  [ Users Table ]                     [ Orders Table ]
  +------------------+                +--------------------+
  | PK: user_id (1)  | <───────────── | PK: order_id (500) |
  |     username     |   (Link)       | FK: user_id (1)    |
  +------------------+                +--------------------+
          ▲                                    │
          │                                    ▼
          │                        [ Database Engine Check ]
          └──────────────────────── Enforces: Does user_id 1 exist?
                                    - If YES: Save order.
                                    - If NO: Reject with constraint error.
```

- **Primary Key Constraint:** Enforces that every row has a unique identifier.
- **Foreign Key Constraint:** Enforces that a foreign key value must match an existing primary key value in the target table.
- **Cascading Rules:** Instructions on what to do when a parent record is deleted or updated (e.g., `ON DELETE CASCADE` automatically deletes all child orders if the parent user is deleted).

# Workflow
How a database establishes and checks relationships during a new transaction:

```text
Step 1: Application requests to place an order for User #42.
                             ↓
Step 2: Database engine intercepts the request and inspects the foreign key column (`user_id = 42`).
                             ↓
Step 3: Engine queries the `users` table: "Does a row exist where `user_id = 42`?"
                             ↓
Step 4: If user exists, the engine inserts the order row and locks the relationship.
                             ↓
Step 5: If user does not exist, the engine blocks the insert and throws a "Foreign Key Constraint Violation" error.
```

# Real World Examples
Think of database relationships and keys as an **office building badge scanning system**.
- **Primary Key:** This is each employee's unique **Badge ID**. No two employees can have the same Badge ID, and every employee must have one.
- **Foreign Key:** This is the **Department Code** printed on the employee's badge. The code points to a list of departments managed in a separate directory.
- **One-to-One Relationship:** Each employee has exactly one designated office locker, and each locker belongs to exactly one employee.
- **One-to-Many Relationship:** One department (e.g., "Engineering") has many employees working in it, but each employee belongs to only one department.
- **Many-to-Many Relationship:** An employee can work on multiple projects, and each project can have multiple employees. To track this, the company uses a **junction sheet** (a signup list) where they write down pairs of `Employee Badge ID` and `Project ID`.

# Implementation
Here is how to create tables with primary keys, foreign keys, and a junction table for many-to-many relationships in SQL:

### 1. One-to-Many Example (Users and Orders)
```sql
-- Parent Table
CREATE TABLE users (
  user_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Primary Key
  username VARCHAR(50) NOT NULL
);

-- Child Table
CREATE TABLE orders (
  order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Primary Key
  user_id INT NOT NULL,                                  -- Foreign Key Column
  order_date DATE NOT NULL,

  -- Define the relationship constraint
  CONSTRAINT fk_user
    FOREIGN KEY (user_id) 
    REFERENCES users(user_id)
    ON DELETE CASCADE -- If a user is deleted, delete their orders too
);
```

### 2. Many-to-Many Example (Students and Courses) using a Junction Table
```sql
CREATE TABLE students (
  student_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(100) NOT NULL
);

CREATE TABLE courses (
  course_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  title VARCHAR(100) NOT NULL
);

-- Junction Table linking Students and Courses
CREATE TABLE student_courses (
  student_id INT NOT NULL,
  course_id INT NOT NULL,

  -- The composite Primary Key makes sure a student can't register for the same class twice
  PRIMARY KEY (student_id, course_id),

  -- Link back to students
  CONSTRAINT fk_student
    FOREIGN KEY (student_id) 
    REFERENCES students(student_id) 
    ON DELETE CASCADE,

  -- Link back to courses
  CONSTRAINT fk_course
    FOREIGN KEY (course_id) 
    REFERENCES courses(course_id) 
    ON DELETE CASCADE
);
```

# Best Practices
- **Always Define a Primary Key:** Never create a database table without a primary key. It guarantees that every single record can be target-accessed, edited, or deleted.
- **Use Simple Autoincrementing Numbers or UUIDs:** For primary keys, use either autoincrementing integers (`BIGINT`) or universally unique strings (UUIDs). Avoid using real-world data like email addresses or phone numbers as primary keys, as these can change.
- **Index Your Foreign Keys:** Relational databases frequently perform joins using foreign keys. Creating an index on foreign key columns ensures that queries linking tables together remain lightning-fast.

# Industry Standards
For high-scale global applications (like Netflix or Uber), database designs often use **UUIDs (Universally Unique Identifiers)** instead of sequential numbers (1, 2, 3...) for primary keys. Because UUIDs are generated randomly and are guaranteed to be unique worldwide, different database servers across the globe can create records simultaneously without ever generating duplicate keys.

# Common Mistakes
- **Orphaned Rows (Missing Foreign Keys):** Creating columns that reference other tables without declaring them as foreign keys in the SQL schema. Without the database enforcing the constraint, deleting a parent row leaves broken pointer values (orphans) in child tables.
- **Creating Too Many Many-to-Many Tables:** Designing tables that could easily be represented as a simple one-to-many relationship, leading to unnecessary junction tables and slow, overly complex queries.
- **Ignoring Cascading Deletes:** Deleting parent records and leaving children to cause SQL errors, or forgetting to specify `ON DELETE RESTRICT` when you want to block the deletion of parent items that have children.

# Security & Performance Considerations
- **Preventing Enumeration Attacks:** If you use sequential integers for primary keys (e.g. `yourdomain.com/user/1`), attackers can easily guess other user IDs by simply changing the number to `2`, `3`, or `4`. Using random UUIDs prevents this type of vulnerability.
- **Join Performance Overload:** Querying relationships using SQL `JOIN` requires the database to match foreign keys under the hood. As tables grow to millions of rows, joins on unindexed foreign keys will cause severe database slow-downs.

# Related Technologies
- **UUID Gen v4 / v7:** Algorithms for creating globally unique, non-sequential database primary keys.
- **Prisma / Sequelize:** Modern Object-Relational Mappers (ORMs) that allow developers to define relationships using Javascript/Typescript objects instead of raw SQL code.

# Summary

## What We Learned
- Keys are unique identifiers that allow tables to point to each other, preventing data duplication.
- Primary keys identify a single row; Foreign keys store a primary key to connect two rows.
- Cardinality defines whether tables are linked One-to-One, One-to-Many, or Many-to-Many.
- Junction tables are necessary to connect Many-to-Many relationships.

## Key Takeaways
- Enforce foreign keys directly in the SQL schema to guarantee referential integrity and avoid orphaned records.
- Index foreign key columns to ensure relationships resolve quickly in read queries.
- Use UUIDs instead of sequential IDs in public-facing tables to block enumeration attacks.

# Keywords
- Primary Key
- Foreign Key
- One-to-One
- One-to-Many
- Many-to-Many
- Junction Table
- Referential Integrity
- UUID
- Cascade

# Glossary

| Term | Meaning |
|---|---|
| Primary Key | A column that uniquely identifies every single row in a table. |
| Foreign Key | A column that references the primary key of another table to link records together. |
| Cardinality | The quantitative relationship (1:1, 1:N, N:M) between linked tables. |
| Junction Table | A middle table containing foreign keys that links two main tables in a Many-to-Many relationship. |
| Referential Integrity | A database state where all foreign keys point to valid, existing primary keys. |
| Cascade | A database rule that automatically deletes or updates child records when their parent record is deleted or updated. |

## Next Recommended Chapters
- 06-Indexing.md
