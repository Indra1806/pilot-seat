> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Distributed System** is a collection of physically separate computer servers (referred to as **Nodes**) connected over a network, coordinating their actions and sharing data in a way that appears to the end-user as a single, unified computer system.

# Why It Exists
A single computer server, no matter how powerful or expensive, is a single point of failure. If the server loses power, suffers a hardware malfunction, or is disconnected from the network, the entire application goes offline. Furthermore, a single server cannot scale past the physical limits of current chip technology. Engineers created distributed systems to achieve high availability, fault tolerance, and near-infinite scaling by combining multiple standard, cheap servers into a unified network.

# Problem It Solves
Distributed systems solve single-point-of-failure downtimes, geographical network delays, and vertical hardware limit ceilings.

### Before Distributed Systems (Single Server Setup):
- A single component failure (like a power supply or disk crash) caused system-wide downtime.
- Users located far from the server experienced slow response times.
- Scaling database capacity required buying million-dollar mainframes.

### After Distributed Systems (Distributed Node Setup):
- If Node A crashes, Nodes B and C automatically take over the workload with zero downtime.
- Nodes are deployed in different global regions to process traffic close to local users.
- Computing capacity is increased by simply adding cheap, standard servers to the cluster.

# Core Concepts
To design distributed systems, you must master node coordination, time management, and network failure states:

1. **Nodes and Networks:** A distributed system consists of independent processing units (Nodes) that share state by sending messages over network cables, which are inherently unreliable and prone to lag.
2. **Clock Drift (Physical Time Limits):** Computer internal clocks rely on quartz crystals, which drift by milliseconds over time. Because physical clocks cannot be perfectly synchronized across servers, distributed systems cannot rely on physical timestamps to determine the exact order of events.
3. **Consensus (Agreement):** The process by which independent nodes in a cluster agree on a single data value or system state (e.g. electing a leader node or agreeing if a transaction committed).
4. **Network Partition:** A communication failure where a network cable or switch drops packets, splitting a cluster of nodes into two or more isolated groups that can no longer talk to each other.

# Architecture / Components
The topology of a distributed cluster using consensus voting to elect a Leader node and sync Follower nodes:

```text
       [ Follower Node A ]             [ Leader Node ] (Handles writes)
               │                             │
               │ (Heartbeat / Sync)          │ (Heartbeat / Sync)
               ▼                             ▼
       [ Network Partition ] ────────> [ Follower Node B ] (Disconnected!)
       - A and Leader can talk.
       - Node B is isolated.
       - Consensus Check: Do we have a majority (2 out of 3)? Yes. Cluster remains active.
```

- **Leader Node:** The coordinator node elected by the cluster to manage writes and coordinate updates.
- **Follower Nodes:** Standby nodes that accept read queries and replicate state from the Leader.
- **Consensus Protocol:** The voting algorithm (like Raft) that manages node elections and data commits.

# Workflow
The consensus voting process (using Raft protocol) to write data across a distributed cluster:

```text
Step 1: The client sends a write request to the Leader node: "Set counter = 5".
                             ↓
Step 2: The Leader records the entry in its local draft log and sends copy requests to all Follower nodes.
                             ↓
Step 3: Followers receive the copy request, write it to their local draft logs, and reply "OK" to the Leader.
                             ↓
Step 4: Once the Leader receives "OK" replies from a majority of nodes (e.g., 2 out of 3 nodes), the Leader commits the write.
                             ↓
Step 5: The Leader sends a success response to the client.
                             ↓
Step 6: The Leader instructs the Followers to commit the entry, finalizing the cluster state.
```

# Real World Examples
Think of a distributed system as a **global shipping company with offices in New York, London, and Tokyo**.
- **Without a Distributed System:** The company keeps all shipping ledgers in the New York office. If a clerk in Tokyo wants to ship a package, they must call New York to verify the customer account details. If New York has a power outage, the London and Tokyo offices cannot operate.
- **With a Distributed System:** Each office keeps a copy of the ledger (**Nodes**). They sync updates over international phone lines (**Network**).
- **Network Partition:** The undersea phone line between London and New York breaks. London and Tokyo can talk, but New York is isolated. Do the London and Tokyo offices keep shipping packages? Yes, because they represent the majority (2 out of 3 offices) and can vote to keep operating. New York realizes it is isolated and locks its operations to prevent writing conflicting records.
- **Clock Drift:** Each office has a clock on the wall. Over time, the London clock runs 3 seconds fast. A Tokyo clerk logs a package shipment at 12:00:02, and a London clerk logs a shipment at 12:00:01. Because of the drift, they can't agree on who shipped first based on wall time. They must use **Logical Ordering** (e.g. stamping sequence numbers like *Order #101* and *Order #102*) to keep the ledger correct.
- **Consensus (Leader Election):** The company needs a boss to make final decisions. The managers vote. As long as a manager gets a majority of votes (2 out of 3), they become the Leader. If the Leader goes offline, the remaining managers hold a new vote immediately.

# Implementation
Here is a conceptual look at how developers implement consensus checking in code using a Raft-based distributed key-value store (like **etcd**) to write and read synchronized configurations:

### Accessing a Distributed Consensus Store (Node.js)
```javascript
const { Etcd3 } = require('etcd3');
// Connect to a cluster of 3 distributed etcd nodes
const client = new Etcd3({ hosts: ['10.0.0.1:2379', '10.0.0.2:2379', '10.0.0.3:2379'] });

async function manageDistributedConfig() {
  const configKey = 'maintenance_mode';

  // 1. Write a value: etcd coordinates consensus across the cluster before returning
  await client.put(configKey).value('true');
  console.log("Configuration updated across the distributed cluster successfully.");

  // 2. Read the value: etcd guarantees strong consistency
  const val = await client.get(configKey).string();
  console.log(`Current cluster configuration state: ${val}`);
}
manageDistributedConfig();
```

# Best Practices
- **Design for Network Failures:** Always assume the network will drop packets, experience latency, or partition. Your code must handle timeouts, retries, and duplicate requests safely.
- **Use Logical Clocks for Ordering:** Never trust physical timestamps (system time) to order transactions in a distributed system. Use logical clocks (like **Lamport Timestamps** or sequential vector clocks) that order events based on cause-and-effect relationships.
- **Avoid Custom Consensus Code:** Never write your own consensus or cluster election code. Implementing Paxos or Raft correctly is incredibly difficult. Leverage mature, battle-tested tools (like ZooKeeper, etcd, or Consul) to manage cluster state.

# Industry Standards
Modern cloud infrastructures depend on consensus tools to manage state. **etcd** serves as the primary consensus store for Kubernetes, keeping cluster states synchronized. Database engines like CockroachDB use Raft internally to manage database sharding consensus.

# Common Mistakes
- **Assuming Infinite Network Reliability:** Writing code that hangs indefinitely if a network query fails to reply. Always specify strict request timeouts and circuit-breaker thresholds.
- **Relying on NTP for Database Timestamps:** Assuming Network Time Protocol (NTP) keeps physical clocks perfectly synchronized. NTP can drift or jump backward during clock corrections, causing database record ordering bugs.
- **Ignoring the Split-Brain Trap:** Setting up a 2-node cluster. If Node A and Node B lose connection, neither can obtain a majority vote (which requires > 50%, or 2 nodes). Both will lock up or both will try to become the leader, corrupting data. Clusters should always have an odd number of nodes (3, 5, 7...).

# Security & Performance Considerations
- **Consensus Write Overhead:** Consensus requires multiple network roundtrips to get a majority vote. Therefore, distributed consensus databases (like etcd) are relatively slow to write to and should only be used for critical metadata (like config settings), not high-frequency user data.
- **Network Partition Exposure:** When a partition occurs, nodes must choose between returning outdated data (stale reads) or throwing errors to protect consistency. Systems must be configured according to their business requirements.

# Related Technologies
- **etcd:** A distributed, consistent key-value store used for shared configuration.
- **ZooKeeper:** A centralized service for maintaining configuration information and hierarchical naming.
- **Raft:** A consensus algorithm designed to be easy to understand and implement, used to manage replicated logs.

# Summary

## What We Learned
- A distributed system combines independent computer nodes over a network to act as a single, resilient system.
- Physical clocks cannot be trusted to order events due to clock drift; logical ordering sequences must be used instead.
- Consensus protocols (like Raft) require a majority vote (>50%) to elect leaders and commit changes safely.

## Key Takeaways
- Always deploy distributed clusters with an odd number of nodes (minimum 3) to allow majority voting.
- Utilize established tools like etcd or ZooKeeper to manage cluster configurations rather than writing custom consensus code.
- Enforce strict query timeouts to prevent network partitions from locking up your backend applications.

# Keywords
- Distributed System
- Node
- Consensus
- Clock Drift
- Network Partition
- Raft
- Paxos
- etcd
- Split-Brain
- Logical Clock

# Glossary

| Term | Meaning |
|---|---|
| Node | A single physical server or virtual machine running as part of a distributed cluster. |
| Network Partition | A communication blockage that splits a cluster of nodes into isolated sub-groups. |
| Consensus | The process of getting multiple independent nodes in a cluster to agree on a state or value. |
| Clock Drift | The phenomenon where physical computer clocks fall out of synchronization over time. |
| Lamport Timestamp | A logical clock mechanism used to determine the sequential order of events based on causality. |
| Split-Brain | A failure state where a partitioned cluster divides, and both sides elect a leader and write conflicting data. |

## Next Recommended Chapters
- 10-Consistency-Models.md
