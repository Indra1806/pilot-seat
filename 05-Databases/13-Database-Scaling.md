> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Database Scaling** is the collection of architectural strategies used to increase a database's capacity to handle growing volumes of stored data and massive numbers of concurrent read and write operations. Scaling ensures that an application remains fast and responsive as its user base grows from a few hundred visitors to millions of global users.

# Why It Exists
Every database runs on physical computer hardware. A single server has fixed physical limits: a maximum amount of RAM, a set number of CPU cores, and a limited speed at which its SSD can read and write data. As a successful application grows, it eventually reaches a point where its single database server becomes saturated, leading to slow queries, connection timeouts, and system crashes. Database designers invented scaling methodologies to distribute the data processing load across multiple machines, bypassing the physical constraints of a single server.

# Problem It Solves
Database scaling solves database server crashes, disk read/write throughput bottlenecks, and physical hardware capacity limits.

### Before Database Scaling (Single Server Bottleneck):
- High user traffic queued up at the database connection pool, causing application timeouts.
- Storage capacity was capped; if the hard drive filled up, the application could no longer save new user data.
- A single hardware failure (like a power supply blowout) took the entire application offline.

### After Database Scaling (Distributed Architecture):
- Read operations are shared across multiple read-only replica servers, preventing query queueing.
- Massive tables are split across separate physical machines (sharding), providing virtually unlimited storage capacity.
- Secondary servers can immediately take over if the primary server fails (high availability).

# Core Concepts
Database scaling is achieved through two main pathways and three structural division methods:

1. **Vertical Scaling (Scaling Up):** Adding more power (faster CPU, more RAM, faster SSDs) to the existing database server. It is simple to implement but has a hard physical ceiling and becomes exponentially expensive at high tiers.
2. **Horizontal Scaling (Scaling Out):** Adding more database servers to the network and dividing the workload among them. It is highly cost-effective and scalable but introduces database coordination complexity.
3. **Read Replicas:** Copying data from a primary server (Master/Source) to secondary servers (Replicas) to offload read-heavy query traffic.
4. **Database Sharding:** A horizontal scaling technique where a table's rows are split and stored across entirely separate physical database servers based on a specific rule (Sharding Key).
5. **Partitioning:** Splitting a large table into smaller, more manageable sub-tables *on the same physical server* (typically based on date ranges) to improve query speed.

# Architecture / Components
The topology of a horizontally scaled database cluster utilizing a load balancer, read replicas, and sharded write databases:

```text
                               [ Client Requests ]
                                       │
                                       ▼
                            [ Application Server ]
                                       │
               ┌──────────────────────┴──────────────────────┐
               ▼ (Route Writes)                              ▼ (Route Reads)
         [ Primary DB ]                                [ Load Balancer ]
       - Handles all writes                                  │
               │                                ┌────────────┴────────────┐
               ▼ (Replication Link)             ▼                         ▼
      [ Read Replicas ]                  [ Read Replica 1 ]        [ Read Replica 2 ]
    - Syncs changes asynchronously       - Reads only              - Reads only
```

- **Primary Server:** The master server that accepts all data modifications (`INSERT`, `UPDATE`, `DELETE`).
- **Read Replicas:** Multiple duplicate database servers that only accept read queries (`SELECT`) and copy changes from the Primary.
- **Shards:** Separate database instances holding unique subsets of the main dataset.

# Workflow
The routing workflow of a sharded database system when processing a user write:

```text
Step 1: User #1052 registers on the application.
                             ↓
Step 2: The application checks the sharding rule: "User ID % 2".
        - 1052 % 2 = 0 -> Route to Shard A.
        - 1053 % 2 = 1 -> Route to Shard B.
                             ↓
Step 3: The application connects directly to Shard A's physical server.
                             ↓
Step 4: Shard A saves the user record. Shard B never sees or processes the request.
                             ↓
Step 5: When fetching User #1052, the application routes the read request directly to Shard A.
```

# Real World Examples
Think of database scaling as a **successful local bookstore expanding its business**.
- **Vertical Scaling (Scaling Up):** You keep the same bookstore building but hire a taller, stronger manager with a better memory. This works for a while, but eventually, the building is physically packed with customers, and the manager cannot walk any faster. You have hit the physical hardware limit.
- **Horizontal Scaling - Read Replicas:** You print 5 copies of the store's book catalog and hire 5 assistants. When customers want to check if a book is in stock (**Read query**), they can ask any of the assistants. But when a new book arrives (**Write query**), the manager logs it in the master catalog and updates the assistants' copies.
- **Horizontal Scaling - Database Sharding:** Your inventory becomes too large to fit inside a single building. You rent two storefronts. **Store A (Shard A)** stores books starting with letters A-M. **Store B (Shard B)** stores books starting with N-Z. If a customer wants a book starting with "Physics," they go straight to Store B. The stores operate independently.
- **Table Partitioning:** Inside the store, instead of throwing all books onto one massive shelf, you separate them. You put historical archives from 10 years ago on the top shelf, and new arrivals on the bottom shelf. When looking for new releases, you skip scanning the historical archives completely.

# Implementation
Here is how to set up table partitioning by range (like dates) in SQL:

### Table Partitioning by Date Range (PostgreSQL syntax)
Instead of searching a single table containing 10 years of logs, we partition it by year so queries only scan the relevant year's storage sector:

```sql
-- 1. Create the parent partition-enabled table structure
CREATE TABLE traffic_logs (
  log_id INT GENERATED ALWAYS AS IDENTITY,
  log_date DATE NOT NULL,
  ip_address VARCHAR(45) NOT NULL,
  request_path TEXT NOT NULL
) PARTITION BY RANGE (log_date); -- Define partitioning key

-- 2. Create the child partition tables that will actually store the data on disk
CREATE TABLE logs_2025 PARTITION OF traffic_logs
  FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');

CREATE TABLE logs_2026 PARTITION OF traffic_logs
  FOR VALUES FROM ('2026-01-01') TO ('2027-01-01');

-- 3. Writing data (PostgreSQL automatically routes this row to the logs_2026 table)
INSERT INTO traffic_logs (log_date, ip_address, request_path)
VALUES ('2026-05-12', '192.168.1.1', '/homepage');

-- 4. Querying data (Engine ignores logs_2025 entirely, scanning only logs_2026)
SELECT * FROM traffic_logs WHERE log_date = '2026-05-12';
```

# Best Practices
- **Implement Caching Before Scaling:** Never shard your database to fix read slowness before implementing an in-memory cache (like Redis). Caching is far cheaper and easier to maintain than sharded database clusters.
- **Choose a Balanced Sharding Key:** When sharding, choose a sharding key (like `user_id` or `tenant_id`) that distributes data evenly across all shard servers. If you shard by `country_code` and 90% of your users live in the USA, your USA shard will crash while other shards sit empty. This is called a **Hotspot**.
- **Optimize Replication Lag Tolerance:** Build your application logic to expect replication lag. For instance, after a user updates their profile settings, serve their subsequent edit screens directly from the Primary write database for a few seconds before switching back to Replica reads.

# Industry Standards
Massive tech platforms use automated scaling frameworks. Vitess (used by YouTube and Slack) is an open-source database clustering system that sits in front of MySQL, automatically handles sharding routing, and presents the database to developers as if it were a single, non-sharded database server.

# Common Mistakes
- **Premature Sharding:** Sharding a database during the first week of a startup. Sharding makes transactions across shards extremely difficult, prevents joins between sharded tables, and adds massive operational complexity. Only shard when physical hardware limits are actually reached.
- **Ignoring Join Limitations in Shards:** Attempting to join a `users` table sharded on `user_id` with a `products` table sharded on `product_id`. Relational databases cannot run joins across different physical servers easily. Developers must either duplicate tables or run separate queries and join data inside application code.
- **Forgetting to Monitor Replication Lag:** Allowing replicas to fall minutes behind the primary database. If replication lag is high, visitors will see highly outdated data (like buying a item that the master already marked as sold out).

# Security & Performance Considerations
- **Split-Brain Scenario:** If a network failure cuts off communication between the primary server and the replicas, the replicas might assume the primary is dead and promote one of themselves to be the new primary. When the network recovers, you have two primary write servers writing different data, leading to catastrophic database desynchronization. Teams use quorum algorithms (like Raft) to prevent this.
- **Distributed Query Latency:** If you run a query that needs to collect data from all shards simultaneously (a cross-shard scatter-gather query), the query speed will be limited by the slowest network response among all the shard servers.

# Related Technologies
- **Vitess:** A database clustering system for horizontal scaling of MySQL.
- **Citus:** A PostgreSQL extension that turns Postgres into a distributed sharded database engine.
- **Raft / Paxos:** Consensus protocols used by distributed database engines to coordinate leadership and prevent split-brain errors.

# Summary

## What We Learned
- Scaling is the process of expanding database capacity to bypass physical hardware limitations.
- Vertical scaling adds power to one machine; horizontal scaling distributes data across multiple machines.
- Replicas offload read queries, while sharding splits table rows across distinct physical servers to handle writes and storage bloat.

## Key Takeaways
- Maximize vertical scaling and in-memory caching before attempting to shard your database.
- Select a sharding key that ensures an even distribution of data to avoid server hotspots.
- Ensure your application can tolerate minor replication lag when reading from secondary replicas.

# Keywords
- Scaling
- Vertical Scaling
- Horizontal Scaling
- Read Replica
- Sharding
- Table Partitioning
- Sharding Key
- Hotspot
- Replication Lag
- Split-Brain

# Glossary

| Term | Meaning |
|---|---|
| Vertical Scaling | Increasing database capacity by adding faster CPUs, more RAM, or larger SSDs to a single server. |
| Horizontal Scaling | Increasing database capacity by adding more servers to a network cluster. |
| Read Replica | A copy of a primary database server dedicated strictly to serving read queries. |
| Sharding | The process of splitting table rows across multiple physically independent database servers. |
| Partitioning | Dividing a large table into smaller storage segments on the same physical disk. |
| Sharding Key | The specific table column used to determine which database shard a row will be stored on. |

## Next Recommended Chapters
- 14-Database-Security.md
