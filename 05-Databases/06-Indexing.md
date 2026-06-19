> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Database Index** is a separate, highly organized data structure that a database engine builds and maintains alongside a table. It acts as a search map that allows the database engine to locate and retrieve specific rows instantly, without having to read through every single record in the table.

# Why It Exists
When a database table is created, rows are stored on the hard drive in the order they are inserted. If a table grows to 10 million rows, searching for a user with the email `alex@example.com` forces the database engine to read all 10 million rows from first to last to find the match. This is called a **Full Table Scan** and it is incredibly slow, taking seconds or even minutes. Engineers invented indexes to keep data sorted in a separate quick-lookup structure, reducing search times from minutes to microseconds.

# Problem It Solves
Database indexing solves slow search queries, high server CPU usage, and disc read bottlenecks.

### Before Indexing (Full Table Scan):
- The database engine had to load entire tables from the hard drive into memory to find a single row.
- Search speed decreased linearly as the table grew (searching 10 million rows took 10 times longer than 1 million rows).
- Heavy search queries locked tables and caused application lag.

### After Indexing (Indexed Lookups):
- The database engine reads a tiny index map first, obtaining the exact disk coordinate of the matching row.
- Search speeds remain virtually instantaneous, even as table sizes grow from millions to billions of rows.
- CPU usage drops significantly because the database engine does not need to scan irrelevant records.

# Core Concepts
To master indexing, you must understand the underlying lookup mechanics:

1. **Full Table Scan (Sequential Scan):** The default search method where the database engine reads a table row-by-row from start to finish.
2. **B-Tree (Balanced Tree):** The default data structure used for database indexes. It organizes data in a hierarchical tree of nodes. By starting at the root node and following "greater than" or "less than" pointers down through branches, the engine can find any value in a massive table in just 3 or 4 quick decisions.
3. **Index Overhead:** The trade-off of indexing. While indexes make reading data extremely fast, they slow down write operations (inserting, updating, deleting) because the database engine must update both the table *and* the index map every time a change is made.

# Architecture / Components
An index organizes table pointers into a balanced lookup tree:

```text
                           [ Root Node ]
                            /         \
                           /           \
                 [ Branch: A-M ]   [ Branch: N-Z ]
                     /       \         /       \
                    /         \       /         \
                 [ A-F ]     [ G-M ] [ N-S ]   [ T-Z ]  <-- Leaf Nodes
                   │           │       │         │
                   ▼           ▼       ▼         ▼
                 [ Row ]     [ Row ] [ Row ]   [ Row ]  <-- Physical Table Rows on Disk
```

- **Root Node:** The starting point of the index tree search.
- **Branch Nodes:** Intermediate layers that route the search query to the correct subsection based on sorted ranges.
- **Leaf Nodes:** The final level of the index tree containing the actual sorted key values paired with direct disk pointers to the corresponding rows in the physical table.

# Workflow
The step-by-step lookup process of an indexed query:

```text
Step 1: Application runs: SELECT * FROM users WHERE email = 'bob@email.com';
                             ↓
Step 2: The Query Planner checks the schema and detects an index on the `email` column.
                             ↓
Step 3: The engine traverses the B-Tree index:
        - Root: "Does 'bob' start with A-M?" -> Yes.
        - Branch: "Does 'bob' start with A-F?" -> Yes.
        - Leaf: Locate 'bob@email.com' and extract its physical disk address (e.g. Block 504, Line 12).
                             ↓
Step 4: The engine jumps directly to Block 504 on the disk, retrieves the row, and returns it.
```

# Real World Examples
Think of a database index as the **index at the back of a thick textbook**.
- If you want to find where the textbook explains "caching," and there is no index, you have to read the book page-by-page, line-by-line, from Page 1 to the end. This is a **Full Table Scan**.
- If the textbook has an index at the back, you flip to the "C" section, find "caching", and see: `Page 124, 215, 340`. You flip directly to Page 124. This is an **Indexed Lookup**.
- The index pages take up extra paper at the back of the book (Storage overhead).
- If the author decides to add a new chapter to the book, they have to rewrite pages and then spend extra time updating the index references at the back to match the new page numbers (Write overhead).

# Implementation
Here is how to create, view, and analyze database indexes in SQL:

### 1. Creating a Table with Indexes
```sql
CREATE TABLE users (
  user_id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY, -- Automatically indexed!
  email VARCHAR(100) NOT NULL UNIQUE,                  -- Automatically indexed because it is UNIQUE!
  first_name VARCHAR(50) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

-- Manually create an index on a frequently searched column
CREATE INDEX idx_users_created_at ON users (created_at);
```

### 2. Querying with and without Indexes (Explanation using EXPLAIN)
To see if a database is actually using an index, developers prefix their SQL query with `EXPLAIN ANALYZE`:

```sql
-- Searching by an unindexed column (first_name)
EXPLAIN ANALYZE SELECT * FROM users WHERE first_name = 'Alice';
-- Output will show: "Seq Scan on users... (cost=0.00..154.00 rows=5 width=244)"
-- This means it did a sequential / full table scan.

-- Searching by an indexed column (created_at)
EXPLAIN ANALYZE SELECT * FROM users WHERE created_at > '2026-01-01';
-- Output will show: "Index Scan using idx_users_created_at on users... (cost=0.15..8.20 rows=1 width=244)"
-- This means it skipped scanning and read the index directly.
```

# Best Practices
- **Index Only Frequently Queried Columns:** Create indexes on columns used in `WHERE` filters, `JOIN` conditions, or `ORDER BY` sorting statements.
- **Do Not Over-Index:** Don't index every column in a table. Each index consumes disk space and degrades the performance of `INSERT`, `UPDATE`, and `DELETE` queries.
- **Index High-Cardinality Columns:** An index works best on columns containing many unique values (like `email` or `user_id`). Indexing low-cardinality columns (like `gender` or `status_active` which contain only a couple of distinct options) provides little to no speed benefit.

# Industry Standards
Most relational databases (like PostgreSQL and MySQL) automatically create a hidden index on any column marked as a `PRIMARY KEY` or `UNIQUE` constraint. Software developers do not need to create these indexes manually.

# Common Mistakes
- **Indexing Small Tables:** Creating indexes on tables with only a few hundred rows. Sequential scanning on tiny tables is faster than loading and traversing an index file.
- **Forgetting Composite Indexes for Multi-Column Queries:** Creating separate indexes on `first_name` and `last_name` when queries always search for `first_name AND last_name` together. In this case, a single **Composite Index** (an index spanning both columns) should be used.
- **Using Functions on Indexed Columns:** Writing queries like `WHERE LOWER(email) = 'alice@email.com'`. Using the `LOWER()` function bypasses the standard index on `email`, forcing a slow sequential scan. Developers must create a specialized **Expression Index** to handle this.

# Security & Performance Considerations
- **Index Bloat:** Over time, as records are deleted and updated, indexes can collect "dead" pointer references, becoming fragmented and bloated. Administrators must periodically rebuild indexes (`REINDEX`) to reclaim disk space and restore peak speed.
- **Write Amplification:** On high-frequency write tables (like logging systems), indexing too many columns will starve server resources because every single write requires the system to update multiple indexes on disk.

# Related Technologies
- **Explain / Explain Analyze:** Built-in SQL tools used to read query plans and measure index usage.
- **PgHero:** A database dashboard tool that automatically detects missing indexes or highlights unused indexes that are wasting space.

# Summary

## What We Learned
- Database indexes are quick-lookup maps that prevent slow, disk-heavy sequential scans.
- B-Tree structures allow databases to locate records in microseconds using a hierarchical decision tree.
- Indexes require a trade-off: they speed up read operations but slow down write operations and consume extra disk storage.

## Key Takeaways
- Use `EXPLAIN` to confirm that the database planner is actually using your index.
- Keep index counts low; target only columns used regularly in filters, joins, and sorts.
- Keep track of write overhead when designing tables that receive high-throughput write traffic.

# Keywords
- Index
- Full Table Scan
- B-Tree
- Root Node
- Leaf Node
- Query Planner
- Write Overhead
- Explain Analyze
- Composite Index

# Glossary

| Term | Meaning |
|---|---|
| Full Table Scan | A search operation where the database reads every single row of a table from disk. |
| B-Tree | Balanced Tree; a self-balancing sorted tree structure that databases use to search index keys in logarithmic time. |
| Leaf Node | The bottom-most nodes in an index tree that contain the actual sorted keys and physical disk pointers. |
| Query Planner | The software component in a database engine that decides the most efficient query execution strategy (such as using an index vs a table scan). |
| Cardinality | In the context of indexing, this refers to the uniqueness of data values in a column. |
| Composite Index | A single index built on two or more columns simultaneously. |

## Next Recommended Chapters
- 07-Transactions-And-ACID.md
