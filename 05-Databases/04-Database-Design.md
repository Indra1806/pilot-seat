> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Database Design** is the systematic process of planning, structuring, and organizing data elements and their relationships before creating physical database tables. It involves defining what data needs to be stored, categorizing it logically, planning table columns, and mapping how different entities connect to ensure maximum efficiency, speed, and accuracy.

# Why It Exists
In early programming, developers created databases on the fly as they wrote code. If they suddenly needed to track a user's address, they just added a new column to the main table. This unstructured approach led to **Bad Database Design**: databases where identical data was repeated in multiple places, tables grew bloated, queries became slow and complicated, and records broke easily. Engineers created database design methodologies to establish a structured planning phase, using blueprints like Entity-Relationship Diagrams (ERDs) to model data accurately before writing a single line of SQL code.

# Problem It Solves
Database design solves data redundancy (bloat), update anomalies (inconsistencies), and system layout bottlenecks.

### Before Database Design (Unplanned Database):
- Data was duplicated across tables. If a user updated their phone number in one table, it remained outdated in three others, leading to inconsistent records.
- Deleting an item caused unintended data loss (e.g., deleting an order accidentally deleted the entire customer account associated with it).
- Tables had hundreds of columns, making queries slow and difficult to maintain.

### After Database Design (Structured Schema):
- Data is organized into clean, single-purpose tables, minimizing duplicates.
- Relationships are planned mathematically using foreign keys, protecting data integrity.
- Queries are fast and simple because table joins follow a logical, pre-calculated blueprint.

# Core Concepts
To design a database, you must master the three building blocks of data modeling:

1. **Entities (The Nouns):** The primary real-world objects or concepts you want to track (e.g. `User`, `Product`, `Order`, `Invoice`). In the database, each Entity becomes a **Table**.
2. **Attributes (The Details):** The specific properties or characteristics of an Entity (e.g. a `User` entity has the attributes: `Username`, `Email`, and `BirthDate`). In the database, each Attribute becomes a **Column**.
3. **Entity-Relationship Diagram (ERD):** A visual flowchart or diagram that shows the entities, their attributes, and exactly how they link together using relationship lines.

# Architecture / Components
The database design lifecycle goes through three logical modeling layers:

```text
  [ Conceptual Model ] ──> [ Logical Model ] ──> [ Physical Model ]
  (High-level ideas)       (Define tables    )   (Actual SQL schema)
  - Users, Orders, Items   - Columns, Keys   - PostgreSQL types
```

- **Conceptual Model:** The business-focused view outlining what objects need to exist, created in collaboration with stakeholders.
- **Logical Model:** The technical blueprint mapping entities to tables, defining columns, keys, and relationships, independent of any specific database engine.
- **Physical Model:** The final concrete database schema, complete with data types (e.g. `VARCHAR`, `INT`, `NUMERIC`) matching a specific engine (like PostgreSQL).

# Workflow
The steps to design a database schema from scratch:

```text
Step 1: Gather business requirements: list what objects the application must track.
                             ↓
Step 2: Identify **Entities** (nouns) and list their **Attributes** (details).
                             ↓
Step 3: Define **Keys**: choose a unique Primary Key identifier for each entity.
                             ↓
Step 4: Establish **Relationships**: draw lines showing how entities connect.
                             ↓
Step 5: Create the **Entity-Relationship Diagram (ERD)** to visualize the layout.
                             ↓
Step 6: Translate the ERD into a **Physical Schema** using SQL table creation scripts.
```

# Real World Examples
Think of database design as the **architect's blueprint for a house**.
- Before you buy concrete, bricks, and wood (write SQL tables), you must plan the rooms, features, and connections.
- **Entities** are the physical rooms: Kitchen, Bedroom, Bathroom, Living Room.
- **Attributes** are the features inside the rooms: Cabinet dimensions, Stove type, Tub size.
- **Relationships** are the doors and hallways: The hallway connects the Kitchen to the Dining Room.
- If you build without a blueprint, you might put a bathtub in the kitchen, build a bedroom with no doorways, or construct the garage in the middle of the house. An **ERD** is sketching out the blueprint on paper first to make sure every room sits in the right place and connects logically.

# Implementation
Here is a text-based ERD layout and its translation into physical SQL table code:

### 1. The Logical ERD Model
```text
  +-------------------+          1 : N          +-------------------+
  |      CUSTOMERS    | ──────────────────────< |       ORDERS      |
  +-------------------+                         +-------------------+
  | PK | customer_id  |                         | PK | order_id     |
  |    | name         |                         | FK | customer_id  |
  |    | email        |                         |    | order_date   |
  +-------------------+                         |    | total_amount |
                                                +-------------------+
```

### 2. The Physical SQL Implementation
```sql
-- Create the Customers table
CREATE TABLE customers (
  customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Primary Key
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE
);

-- Create the Orders table
CREATE TABLE orders (
  order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  customer_id INT NOT NULL, -- Foreign Key referencing customers
  order_date DATE NOT NULL,
  total_amount NUMERIC(10, 2) NOT NULL,

  -- Enforce relationship link constraint
  CONSTRAINT fk_customer
    FOREIGN KEY(customer_id) 
      REFERENCES customers(customer_id)
);
```

# Best Practices
- **Use Meaningful, Plural Table Names:** Name your tables using plural nouns (e.g. `users`, `products`, `orders`). Keep column names singular and consistent (e.g. `user_id`, `product_id`).
- **Plan for Scalability:** Think ahead when choosing data types. If your app will grow to hold billions of records, use `BIGINT` for your primary keys instead of a standard `INT` (which maxes out at ~2.1 billion).
- **Draw ERDs First:** Never start coding a backend before drawing your database ERD. Use simple visual tools (like Lucidchart or dbdiagram.io) to design and review your schema with your team.

# Industry Standards
Professional teams use standard notation styles like **Crow's Foot notation** to draw ERDs. These diagrams show entities as boxes and use special line endpoints (like a trident shape for "Many" and a single tick mark for "One") to represent relationship boundaries.

# Common Mistakes
- **Creating a Single "Everything" Table:** Trying to store users, user orders, and product details inside one massive table. This is called a "flat table" and leads to massive data duplication and slow processing.
- **Inconsistent Naming Conventions:** Mixing naming conventions (e.g., naming one primary key `userID`, another `customer_id`, and a third `id`). This makes writing SQL queries highly confusing. Pick one standard (like snake_case) and stick to it.

# Security & Performance Considerations
- **Logical Segmentation:** Designing your database structure to isolate sensitive tables (like credit card tokens or login credentials) from public tables (like product reviews). This makes it easy to apply strict security controls to the sensitive tables only.
- **Normalization vs. Performance:** Normalizing a database (splitting data into separate tables) keeps data clean, but requires more CPU-intensive SQL `JOIN` operations to read. Sometimes, database designers intentionally duplicate a field (denormalization) to avoid slow joins in high-traffic sections.

# Related Technologies
- **dbdiagram.io:** A popular code-based tool used to design and draw ERDs easily.
- **Storybook / DB Docs:** Tools used to document database schemas automatically for team reference.

# Summary

## What We Learned
- Database design is the architectural blueprint phase that prevents data redundancy and corruption.
- Data structures are modeled as Entities (tables), Attributes (columns), and relationships.
- ERDs visually map out logical schemas before translating them into physical SQL code databases.

## Key Takeaways
- Always draw an ERD and get layout approval before writing database tables.
- Establish consistent snake_case naming conventions for all tables and keys.

# Keywords
- Database Design
- Schema
- Entity
- Attribute
- ERD
- Conceptual Model
- Physical Model
- Data Modeling

# Glossary

| Term | Meaning |
|---|---|
| Entity | A distinct real-world object or concept (like a Customer) that is represented as a table in a database. |
| Attribute | A property or detail of an entity (like Customer Email) that is represented as a column in a database. |
| ERD | Entity-Relationship Diagram; a flowchart showing how database tables link together. |
| Crow's Foot Notation | The standard visual styling system of lines and endpoints used to draw relationships on ERDs. |
| Denormalization | The practice of intentionally copying data across tables to speed up read queries, at the cost of duplicate storage. |

## Next Recommended Chapters
- 02-Relational-Databases.md
- 05-Relationships-And-Keys.md
