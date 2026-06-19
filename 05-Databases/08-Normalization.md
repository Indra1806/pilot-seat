> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Database Normalization** is a systematic multi-step design process used to organize columns and tables in a relational database. The goal is to eliminate data redundancy (unnecessary duplicates) and prevent data design anomalies (saving errors) by breaking large, flat tables into smaller, well-defined tables linked by relationships.

# Why It Exists
In early databases, developers often created one giant table to store everything. For example, a single `orders` table might store the customer's name, phone number, address, order date, product name, price, and manufacturer address. If a customer ordered 10 different items, their name, phone number, and address were typed into 10 separate rows. If they moved houses, updating their address required locating and changing every single row. If they had no active orders, deleting their orders deleted the customer's account entirely. Engineers created normalization rules (known as **Normal Forms**) to establish a clean data layout where each piece of information is stored in exactly one place.

# Problem It Solves
Normalization solves data redundancy (wasted space) and the three classic database **Anomalies**:

- **Update Anomaly:** Updating a customer's address in one place but leaving it outdated in another row because the data is duplicated.
- **Insertion Anomaly:** Being unable to add a new customer to the database because they haven't placed an order yet, and the database requires an `order_id` to create a record.
- **Deletion Anomaly:** Accidentally deleting a customer's entire account and contact details from the system when simply deleting their order history.

# Core Concepts
Normalization is achieved by meeting progressive tiers of rules called **Normal Forms**:

1. **First Normal Form (1NF - Atomicity):** Columns must contain only single, indivisible values, and there can be no repeating groups. You cannot store a list of values (like `["red", "blue"]`) in a single table cell.
2. **Second Normal Form (2NF - Full Key Dependency):** The table must already meet 1NF, and all non-key columns must depend on the *entire* Primary Key (this is only an issue when using a composite primary key made of multiple columns).
3. **Third Normal Form (3NF - Direct Dependency):** The table must meet 2NF, and no column can depend on another non-key column. Columns must depend "on the key, the whole key, and nothing but the key."

# Architecture / Components
The flow of database normalization involves splitting a single bloated table into multiple atomic tables linked together:

```text
  [ Bloated Flat Table ] (Customer Details + Order Details + Product Details)
           │
           ▼  (Apply 1NF, 2NF, 3NF Rules)
           │
  ┌────────┴────────────────────────┬────────────────────────┐
  ▼                                 ▼                        ▼
[ Customers Table ]           [ Orders Table ]         [ Products Table ]
- customer_id (PK)            - order_id (PK)          - product_id (PK)
- name, email                 - customer_id (FK)       - name, price
                              - product_id (FK)
```

- **Primary Tables:** Specialized tables containing singular entities (e.g. `customers` or `products`).
- **Linking / Junction Tables:** Tables containing foreign keys designed to reassemble relationships during read queries using `JOIN` statements.

# Workflow
The step-by-step process of normalizing a database design:

```text
Step 1: Inspect raw data tables and check for lists inside cells.
                             ↓
Step 2: Apply 1NF: Split multi-value cells into individual rows or separate columns.
                             ↓
Step 3: Apply 2NF: Identify composite keys and move partial key dependencies to new tables.
                             ↓
Step 4: Apply 3NF: Find columns that depend on other non-key columns and extract them.
                             ↓
Step 5: Define foreign key constraints to link the newly split tables.
```

# Real World Examples
Think of database normalization as **sorting a messy clothes dresser**.
- **Un-normalized Database:** You throw socks, shirts, pants, underwear, and shoes all into one massive drawer. If you need to find a matching pair of socks, you have to dig through the entire pile. If a mud-stained shirt is thrown in, it stains the clean socks nearby.
- **First Normal Form (1NF):** You separate the drawers, but you still clump items. 1NF is like making sure each compartment holds only one single type of item—no packing multiple pairs of shoes into a single sock slot.
- **Second Normal Form (2NF):** You designate drawers based on the main category label. Every item in the "Shirts" drawer must be a shirt. You move non-shirt items to their own drawers.
- **Third Normal Form (3NF):** You remove indirect dependencies. Suppose you have a drawer labelled "Work Outfits." Inside, you store a matching fitness tracker. The fitness tracker doesn't depend on the clothes; it depends on your exercise schedule. You move it to its own drawer and link it with a label. Now, every drawer is perfectly organized, items are easy to find, and washing one item doesn't ruin another.

# Implementation
Here is how a developer refactors a poorly designed, un-normalized table structure into 3NF:

### 1. The Bad, Un-Normalized Table
This table stores orders, customer details, and store addresses. It breaks 1NF because the `tags` column contains list values, and it breaks 3NF because `store_city` depends on `store_zipcode`, not the primary `order_id`.

| order_id (PK) | customer_name | customer_email | store_zipcode | store_city | tags |
|---|---|---|---|---|---|
| 1 | Alice | alice@email.com | 10001 | New York | Tech, Sale |
| 2 | Bob | bob@email.com | 90210 | Beverly Hills | Home |

### 2. Refactoring to Third Normal Form (3NF)
We split this data into four separate tables, ensuring each table has a single, clear purpose:

```sql
-- Table 1: Customers (Holds customer information only)
CREATE TABLE customers (
  customer_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(100) NOT NULL UNIQUE
);

-- Table 2: Postal Codes (Resolves store city transitive dependency)
CREATE TABLE postal_locations (
  zipcode VARCHAR(10) PRIMARY KEY,
  city VARCHAR(100) NOT NULL
);

-- Table 3: Orders (Links customer and location via Foreign Keys)
CREATE TABLE orders (
  order_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  customer_id INT NOT NULL,
  store_zipcode VARCHAR(10) NOT NULL,
  order_date DATE NOT NULL,

  CONSTRAINT fk_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  CONSTRAINT fk_zipcode FOREIGN KEY (store_zipcode) REFERENCES postal_locations(zipcode)
);

-- Table 4: Tags Junction Table (Resolves 1NF repeating list values)
CREATE TABLE order_tags (
  order_id INT NOT NULL,
  tag_name VARCHAR(50) NOT NULL,
  PRIMARY KEY (order_id, tag_name),
  CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);
```

# Best Practices
- **Aim for 3NF by Default:** When designing transactional backend systems (like e-commerce checkouts or banking ledgers), always normalize schemas to 3NF to prevent data corruption.
- **Understand the Cost of Joins:** Remember that highly normalized databases require more table joins to pull a complete record, which can increase query execution times.
- **Normalize First, Denormalize Later:** Always design your database in a fully normalized state first. Only copy data across tables (denormalize) later if performance testing reveals that specific table joins are too slow.

# Industry Standards
While transactional databases (OLTP) are normalized to 3NF to handle millions of quick writes, analytical databases (OLAP / Data Warehouses used by data scientists to run reports) are often completely denormalized. They combine data into enormous flat tables so that business reports can read billions of records without performing slow joins.

# Common Mistakes
- **Applying 2NF/3NF to NoSQL databases:** Trying to normalize a MongoDB database. NoSQL databases are designed around nested documents. Normalizing them defeats their purpose and causes massive performance drops.
- **Over-Normalizing:** Splitting tables into dozens of tiny, single-column tables. This makes writing SQL queries an absolute nightmare because developers have to join 10 tables just to load a simple profile page.
- **Forgetting Foreign Key Indexes:** Normalizing tables splits data, requiring more queries to run SQL `JOIN` statements. If you forget to index the foreign keys connecting these tables, join operations will run incredibly slowly.

# Security & Performance Considerations
- **Join Explosion:** If a database is over-normalized, reading data requires complex multi-join queries. If multiple users execute these queries simultaneously, the database server CPU will hit 100% utilization, causing system-wide timeouts.
- **Logical Data Segregation:** Normalizing sensitive tables (e.g. storing passwords, recovery tokens, and credit cards in separate tables with 1:1 links) makes it easier to lock down access. You can grant access to the `users` profile table while completely blocking access to the sensitive `credentials` table.

# Related Technologies
- **OLTP (Online Transaction Processing):** Highly normalized database systems designed for fast writes (e.g., PostgreSQL).
- **OLAP (Online Analytical Processing):** Denormalized, column-oriented database systems designed for fast reading and big data analysis (e.g., Snowflake, BigQuery).

# Summary

## What We Learned
- Normalization is the design discipline of organizing tables to eliminate data redundancy and save anomalies.
- The three normal forms progress from making values atomic (1NF), to verifying full primary key dependency (2NF), to removing indirect column dependencies (3NF).
- Transactional databases are highly normalized, whereas analytical databases are denormalized for speed.

## Key Takeaways
- Normalize your transactional database schemas to 3NF during the design phase to keep data consistent.
- Utilize foreign keys and constraints to reassemble normalized tables safely.
- Avoid over-normalizing schemas past 3NF, as it adds unnecessary complexity to simple SQL queries.

# Keywords
- Normalization
- Redundancy
- Anomalies
- First Normal Form (1NF)
- Second Normal Form (2NF)
- Third Normal Form (3NF)
- Transitive Dependency
- Denormalization
- OLTP
- OLAP

# Glossary

| Term | Meaning |
|---|---|
| Redundancy | Storing the exact same piece of data in multiple places within a database. |
| Insertion Anomaly | A database design flaw where you cannot save a new record because it requires other, unrelated data to exist first. |
| Deletion Anomaly | A design flaw where deleting a child record accidentally wipes out unrelated parent data. |
| Atomic Value | A value that cannot be divided into smaller parts (e.g., a single email address vs a list of addresses). |
| Transitive Dependency | A relationship where a column's value depends on a non-key column, which in turn depends on the primary key. |
| Denormalization | The intentional duplication of data in database tables to improve read speed at the cost of storage. |

## Next Recommended Chapters
- 09-PostgreSQL.md
