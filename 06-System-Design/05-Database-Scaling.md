> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Database Scaling** in system design refers to the architectural strategies used to expand a system's data tier to handle massive read and write operations, large storage volumes, and high query concurrency. It involves dividing data processing and storage workloads across multiple physical servers using replication, partitioning, and sharding.

# Why It Exists
In a typical web application, the web servers (which execute code) are stateless and easy to scale horizontally. However, the database (which stores the source of truth data) is stateful and hard to scale. If you run 100 web servers, they will all try to read and write to the same single database server. Eventually, the database server's disk I/O, CPU, or network bandwidth becomes saturated, turning the database into the primary system bottleneck. Engineers created database scaling patterns to distribute data and queries across multiple machines, preventing database-induced system failure.

# Problem It Solves
Database scaling solves database query bottlenecks, storage volume saturation, and write throughput limits.

### Before Database Scaling (Single Database Bottleneck):
- Web servers scaled up, but the database froze under the volume of concurrent SQL queries.
- Storing millions of transaction records exceeded the maximum hard drive capacity of a single server.
- A database crash took down the entire website and risked permanent data loss.

### After Database Scaling (Distributed Data Tier):
- Read traffic is distributed across multiple read-only replica servers, keeping fetch queries fast.
- Large tables are sharded across separate physical database nodes, offering unlimited storage.
- Active failover configurations ensure that secondary servers instantly assume control if the primary database fails.

# Core Concepts
To design high-scale data tiers, you must master the three primary data division strategies:

1. **Database Federation (Functional Splitting):** Dividing a single database into separate databases based on business domains (e.g., splitting a single database into a `users_db`, a `products_db`, and a `billing_db`).
2. **Master-Replica Replication:** Directing all writes to a single primary database (Master), which replicates the changes to multiple read-only databases (Replicas).
3. **Database Sharding (Horizontal Partitioning):** Breaking a single large table's rows into smaller subsets and storing them across separate physical database servers based on a sharding key:
   - **Range-Based Sharding:** Splitting data based on alphabetical or numeric ranges (e.g. User IDs 1-1000 on Shard A, 1001-2000 on Shard B).
   - **Hash-Based Sharding:** Applying a mathematical hash function to the ID and modulo-ing it by the shard count to distribute records evenly.
   - **Directory-Based Sharding:** Utilizing a central lookup table that maps where each user's record is located.

# Architecture / Components
The topology of a sharded database system routing queries through a database proxy layer:

```text
                               [ SQL Queries ]
                                      │
                                      ▼
                             [ Database Proxy ] ── (Inspects User ID Shard Key)
                                      │
               ┌──────────────────────┼──────────────────────┐
               ▼ (User ID 104)        ▼ (User ID 205)        ▼ (User ID 309)
         [ Shard Node A ]       [ Shard Node B ]       [ Shard Node C ]
          - Users 0-99           - Users 100-199        - Users 200-299
```

- **Database Proxy / Router:** An intermediary layer that analyzes queries, determines which shard contains the target data, and routes the query to the correct server.
- **Shard Nodes:** Independent physical databases holding a fraction of the total dataset.

# Workflow
The routing workflow of a hash-sharded database write request:

```text
Step 1: Application requests to save user profile details for User ID #4235.
                             ↓
Step 2: The database proxy extracts the shard key (`user_id = 4235`).
                             ↓
Step 3: The proxy calculates the hash: `Hash(4235) = 87921`.
                             ↓
Step 4: The proxy applies the modulo based on the active shard count (e.g., 3 shards): `87921 % 3 = 0`.
                             ↓
Step 5: The proxy identifies Shard 0 as the target location.
                             ↓
Step 6: The query is routed to Shard 0, which writes the user profile to its physical disk.
```

# Real World Examples
Think of database scaling as a **city library catalog system**.
- **Monolith Database:** The library keeps all books and records in a single room. Every librarian must walk into this room to find a book. If 1,000 customers walk in, librarians bump into each other, blocking the door.
- **Database Federation:** You divide the library by subject. The history section gets its own building, the science section gets another building, and the user membership records are moved to the administration office. Librarians no longer crowd the same room.
- **Master-Replica Replication:** You print 10 copies of the catalog and hire 10 assistants. Customers can ask any assistant to check if a book exists (**Read query**). But if the head librarian buys a new book (**Write query**), they write it in the master ledger and update the assistants' catalogs.
- **Database Sharding:**
  - **Range-Based Sharding:** Store 1 holds books with titles starting with A-H. Store 2 holds titles starting with I-P. Store 3 holds titles starting with Q-Z. If a popular author releases a series starting with "S", Store 3 gets packed with traffic while Store 1 sits empty (a Hotspot).
  - **Hash-Based Sharding:** When a book arrives, you assign it a random serial number. You sort books into boxes based on the serial number. This distributes books evenly, but if you need to search for all books by a specific author, you must check every single box (cross-shard query).
  - **Directory-Based Sharding:** You place a master lookup desk at the entrance. Customers ask the clerk: *"Where is the book on Physics?"* The clerk checks a directory sheet and says: *"It's in Box 4."* The customer walks directly to Box 4.

# Implementation
Here is how backend applications dynamically route read and write queries to different database servers using code configuration:

### Master-Replica Query Routing in Node.js
```javascript
const { Pool } = require('pg');

// 1. Configure separate connection pools for Master (Write) and Replica (Read)
const masterPool = new Pool({
  host: 'master-db-server',
  database: 'my_app',
  user: 'writer'
});

const replicaPool = new Pool({
  host: 'replica-db-server-1',
  database: 'my_app',
  user: 'reader'
});

// 2. Query wrapper that routes queries based on operation type
async function executeQuery(sql, params = []) {
  // If the query starts with SELECT, route to the read-only replica pool
  const isReadQuery = sql.trim().toUpperCase().startsWith('SELECT');
  
  if (isReadQuery) {
    console.log("Routing read query to Replica Database...");
    return replicaPool.query(sql, params);
  } else {
    console.log("Routing write query to Master Database...");
    return masterPool.query(sql, params);
  }
}

// Example Usage
async function run() {
  // Saved to Master
  await executeQuery('INSERT INTO products (name, price) VALUES ($1, $2)', ['Smartphone', 499]);
  
  // Read from Replica
  const result = await executeQuery('SELECT * FROM products WHERE price < $1', [500]);
  console.log(`Found ${result.rows.length} products.`);
}
run();
```

# Best Practices
- **Implement Master-Replica Scaling First:** Before attempting to shard, scale your reads by setting up read replicas. Read replicas are much simpler to manage than sharding.
- **Select an Immutable Shard Key:** Choose a sharding key (like `user_id`) that will never change. If you shard by `country` and a user moves countries, moving their entire database history from one physical server to another is a complex, error-prone operation.
- **Avoid Cross-Shard Joins:** Design your shard schema so that tables that need to be joined frequently are stored on the same shard node. Relational databases cannot run joins across different physical servers.

# Industry Standards
High-scale platforms (like Uber or Facebook) use massive sharded database clusters. They utilize middleware layers (like **Vitess** for MySQL or **Citus** for PostgreSQL) that sit in front of the database shards, automatically translating standard SQL queries into sharded routes so developers don't have to write custom routing logic in their backend application code.

# Common Mistakes
- **Premature Sharding:** Sharding a database when the dataset size is only a few gigabytes. Sharding introduces massive engineering overhead, blocks multi-table joins, and makes transactions complex. Only shard when you have reached physical hardware capacity limits.
- **Creating Hotspots:** Sharding by a key that distributes data unevenly (e.g. sharding by `status` where 95% of rows are marked as `active`). One shard server will run out of disk space and crash while the others remain empty.
- **Ignoring Replication Lag during immediate reads:** Writing a user profile update to the master database, and immediately querying a read replica to display the next page. Replicas take milliseconds to sync, meaning the user may see their old data.

# Security & Performance Considerations
- **Two-Phase Commit (2PC) Latency:** When running transactions that span multiple shards (e.g., transferring points from User A on Shard 1 to User B on Shard 2), the database must use a **Two-Phase Commit** protocol to guarantee ACID safety. This protocol requires network handshakes between all shards, making transactions very slow.
- **Replication Link Security:** Database replication streams raw data updates across the network. If the connection between the master and replicas is not encrypted via TLS, attackers on the network can sniff all data updates in plain text.

# Related Technologies
- **Vitess:** An open-source database middleware that scales MySQL clusters.
- **Citus:** A PostgreSQL extension that transforms Postgres into a distributed sharded database.
- **ProxySQL:** A high-performance SQL proxy used to route queries between masters and replicas.

# Summary

## What We Learned
- Database scaling divides read and write workloads across multiple servers to bypass single-server disk and network limits.
- Read replicas offload fetch queries from the primary write database.
- Sharding splits table rows across physically separate database instances based on a sharding key.

## Key Takeaways
- Always exhaust caching and read replica options before attempting to shard.
- Keep table joins local to a single shard to avoid slow cross-network queries.
- Structure backend code to direct all writes to the master database and all reads to replicas.

# Keywords
- Database Scaling
- Federation
- Master-Replica
- Sharding
- Shard Key
- Range-Based Sharding
- Hash-Based Sharding
- Directory-Based Sharding
- Database Proxy
- Hotspot

# Glossary

| Term | Meaning |
|---|---|
| Federation | Splitting a single database into separate databases based on business features (e.g., user database vs billing database). |
| Read Replica | A copy of the primary database server used only to resolve read queries. |
| Sharding | The horizontal partitioning technique of splitting a table's rows across different physical servers. |
| Shard Key | The table column value used to determine which database shard a row belongs to. |
| Modulo Sharding | A hash-based sharding method where the hash of an ID is divided by the shard count to determine the target server. |
| Cross-Shard Query | A query that must scan multiple database shard servers to collect data, resulting in slow speeds. |

## Next Recommended Chapters
- 06-CDN.md
