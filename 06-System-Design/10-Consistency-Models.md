> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Consistency Models** are the architectural rules that define how and when data updates made on one server (node) in a distributed database cluster propagate and become visible to read queries executed on other servers across the network. They establish the guarantees of data correctness versus system speed.

# Why It Exists
To handle high traffic, databases are replicated across multiple physical servers around the globe. If a user in New York updates their profile name, writing that change to the New York database node takes milliseconds. However, copying that change to replica servers in London and Tokyo over network cables takes time. If a reader in Tokyo queries the local replica database a millisecond later, they will read the old, outdated profile name. Engineers established consistency models to define how databases manage this synchronization delay and to coordinate how applications handle temporarily out-of-sync records.

# Problem It Solves
Consistency Models solve stale data representation, distributed data divergence, and transaction write latency bottlenecks.

### Before Consistency Models (Undefined Sync Rules):
- Databases returned random, unpredictable data states depending on which physical replica node the user's connection hit.
- Concurrent updates in different regions clashed, resulting in corrupted, split records that could not be reconciled.
- Applications froze because they forced every single read query to wait for global synchronization before returning.

### After Consistency Models (Defined Sync Guarantees):
- Financial ledgers enforce **Strong Consistency** to guarantee that bank balances are 100% accurate worldwide, accepting slower speeds.
- Social media feeds use **Eventual Consistency** to update likes counts quickly, accepting that some users will see minor delays.
- Profile edit pages enforce **Read-Your-Own-Writes** consistency, ensuring a user always sees their own updates immediately.

# Core Concepts
To design distributed data stores, you must master the three primary consistency classifications:

1. **Strong Consistency (Linearizability):** The database guarantees that once a write is completed, any subsequent read query executed anywhere on the network will instantly retrieve the new value. It is highly correct but slows down writes because the system must wait for network sync.
2. **Eventual Consistency:** The database guarantees that if no new writes occur, all replica nodes on the network will eventually catch up and synchronize to the same state. It is incredibly fast but allows readers to temporarily retrieve stale data.
3. **Read-Your-Own-Writes Consistency:** A specialized consistency model guaranteeing that a specific user who updates a record will always see their update on subsequent reads, even if other global users still see the old cached data for a few seconds.

# Architecture / Components
The difference in write propagation speed between Strong Consistency and Eventual Consistency:

```text
  Strong Consistency (Blocking network wait)
  [ Client ] ──> [ Primary DB ] ─── (Sync Copy) ───> [ Replica DB ] (Acknowledge)
       │                                                   │
       ▼ (Returns Success only after replica syncs)        │
  [ Complete! ] <──────────────────────────────────────────┘

  Eventual Consistency (Non-blocking, Asynchronous)
  [ Client ] ──> [ Primary DB ] (Returns Success immediately!)
       │              │
       ▼              ▼ (Asynchronous background synchronization)
  [ Complete! ]  [ Replica DB ] (Catches up after 2 seconds...)
```

- **Replication Sync Link:** The network channel where updates are replicated across nodes.
- **Client Session Pin:** A routing mechanism used to direct a specific user to the same node they wrote to, ensuring Read-Your-Own-Writes consistency.

# Workflow
The workflow of an Eventual Consistency conflict resolution using the **Last-Write-Wins (LWW)** strategy:

```text
Step 1: User A in New York edits post #101 to title "Hello" at 12:00:01.002.
                             ↓
Step 2: User B in London edits post #101 to title "World" at 12:00:01.005.
                             ↓
Step 3: The New York database writes "Hello" locally and queues an asynchronous replication message.
                             ↓
Step 4: The London database writes "World" locally and queues an asynchronous replication message.
                             ↓
Step 5: The replication messages arrive at opposite servers.
                             ↓
Step 6: The engines inspect the timestamps: London's write (12:00:01.005) is newer than New York's (12:00:01.002). Both databases update to "World". Eventual consistency is resolved.
```

# Real World Examples
Think of consistency models as **updating a shared spreadsheet in a global company**.
- **Strong Consistency:** You have a master inventory spreadsheet. If you want to change the stock of laptops from 10 to 9, you write the change. The system locks the spreadsheet globally. No employee in London or Tokyo can open or view the catalog until the change is verified on their screens. Everyone sees 9 laptops, but work is temporarily paused during the update.
- **Eventual Consistency:** You write the laptop change. The system updates your screen to 9 instantly. In the background, an email is sent to Tokyo and London notifying them of the change. For the next 30 seconds, a Tokyo clerk looks at their local copy and still sees 10 laptops. If they try to sell a laptop, they might make a mistake. But after a minute, all emails are processed, and everyone's copy eventually says 9.
- **Read-Your-Own-Writes:** You edit your name in the company profile system from "Alice" to "Al". The system updates your screen immediately so you see "Al" and know it worked. However, if your colleague in Paris searches the employee directory a second later, they still see "Alice" because the update is still traveling over the network. You see your change; they see it eventually.

# Implementation
Here is how developers ensure Read-Your-Own-Writes consistency when building applications on top of eventually consistent database replicas:

### Ensuring Read-Your-Own-Writes (Node.js)
Instead of querying a random read replica (which might be stale), developers route reads for the active editor directly to the primary database:

```javascript
const { Pool } = require('pg');

const primaryDB = new Pool({ host: 'primary-write-db' });
const replicaDB = new Pool({ host: 'replica-read-db-1' });

async function getUserProfile(userId, activeSessionUserId) {
  // If the user requesting the profile is the owner who just edited it,
  // query the primary database directly to ensure they see their own edits
  const isOwnerRequest = (userId === activeSessionUserId);
  
  if (isOwnerRequest) {
    console.log("Routing query to Primary DB to guarantee Read-Your-Own-Writes...");
    const res = await primaryDB.query('SELECT * FROM users WHERE id = $1', [userId]);
    return res.rows[0];
  } else {
    // Other users query the eventually consistent read replica
    console.log("Routing query to Read Replica...");
    const res = await replicaDB.query('SELECT * FROM users WHERE id = $1', [userId]);
    return res.rows[0];
  }
}
```

# Best Practices
- **Use Eventual Consistency by Default:** Keep your systems fast by using eventual consistency for non-critical features (like social likes, view counters, or profile comments).
- **Reserve Strong Consistency for Financial Transactions:** Only use strong consistency when data correctness is critical (like checking out a shopping cart or changing account password credentials).
- **Pin Active Sessions to Nodes:** When a user completes a write query, set a cookie that pins their subsequent read requests to the primary write node for a few seconds. This guarantees Read-Your-Own-Writes consistency without overloading the primary database with all global reads.

# Industry Standards
Cloud-native NoSQL databases (like Amazon DynamoDB or Cassandra) are designed around tunable consistency. Developers can specify on a query-by-query basis whether they want a "Strongly Consistent Read" (which queries a majority of nodes for speed cost) or an "Eventually Consistent Read" (which reads from the nearest replica for maximum speed).

# Common Mistakes
- **Assuming Eventual Consistency is Instant:** Writing backend integration code that inserts a user record and immediately runs an API call that expects a replica node to have that record, resulting in "Record not found" errors.
- **Last-Write-Wins Data Loss:** Resolving conflicts in eventually consistent databases by simply overwriting old records based on server clocks. If clocks are out of sync, a slower server's outdated write can overwrite a newer write, causing data loss. Use **Vector Clocks** to track version history instead.
- **Forgetting Network Cost of Strong Reads:** Forcing all database select queries to be strongly consistent. This disables the scaling benefits of your read replicas, routing all traffic back to your primary node and crashing the system.

# Security & Performance Considerations
- **Stale Auth Checks Security Vulnerability:** If a user is banned, the ban is written to the primary database. If authorization checks run on eventually consistent replicas, the banned user can continue accessing the application for several seconds or minutes until the ban status propagates.
- **Replication Lag Monitoring:** Database administrators must monitor replication lag metrics. If lag grows to minutes, eventual consistency is compromised, and the system is serving unacceptably stale data to visitors.

# Related Technologies
- **Amazon DynamoDB:** A NoSQL database offering tunable read consistency options.
- **Cassandra:** A highly scalable NoSQL database utilizing eventual consistency models.
- **Vector Clocks:** An algorithm used to generate logical version histories to resolve data conflicts.

# Summary

## What We Learned
- Consistency models define the trade-off rules between database write speeds and read correctness.
- Strong consistency guarantees immediate global correctness at the cost of write speed.
- Eventual consistency enables massive scaling by syncing replicas in the background, accepting minor sync delays.
- Read-Your-Own-Writes pins the writer's reads to the primary database to prevent user confusion.

## Key Takeaways
- Use strong consistency exclusively for critical business logic (billing, accounts).
- Use eventual consistency for high-frequency, non-critical updates (analytics, social streams).
- Route owner-specific queries directly to the primary database to maintain Read-Your-Own-Writes guarantees.

# Keywords
- Consistency Model
- Strong Consistency
- Eventual Consistency
- Linearizability
- Read-Your-Own-Writes
- Last-Write-Wins (LWW)
- Replication Lag
- Vector Clock
- Conflict Resolution
- Tunable Consistency

# Glossary

| Term | Meaning |
|---|---|
| Strong Consistency | A database guarantee that all readers will instantly see the latest committed write. |
| Eventual Consistency | A database model where data updates propagate to all nodes asynchronously in the background. |
| Replication Lag | The time delay it takes for a write operation to copy from a primary server to replica nodes. |
| Last-Write-Wins | A conflict resolution strategy where the record with the newest physical timestamp overwrites older records. |
| Linearizability | A strict consistency guarantee where operations appear to take effect instantaneously at a specific point in time. |
| Vector Clock | A logical version tracking system used to detect and resolve concurrent write conflicts in distributed databases. |

## Next Recommended Chapters
- 11-CAP-Theorem.md
