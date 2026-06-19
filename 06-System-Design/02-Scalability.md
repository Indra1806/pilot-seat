> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Scalability** is the architectural capability of a software system to handle growing amounts of work, data storage, or user request traffic by gracefully expanding its processing resources. A scalable system maintains high speed, reliability, and cost-efficiency even as its usage metrics grow by orders of magnitude.

# Why It Exists
Every computer hardware unit has physical performance ceilings. If a website becomes popular and goes from 10 visitors a day to 10 million visitors, running everything on a single server will exhaust its CPU processing power, run out of memory (RAM), and saturate the network card. This results in the server freezing and the application going offline. Engineers created scaling patterns to allow software architectures to expand dynamically, bypassing the physical constraints of individual computer hardware.

# Problem It Solves
Scalability solves server crashes due to CPU/RAM overload, slow query speeds under high concurrent request volumes, and physical disk storage exhaustion.

### Before Scalability (Unscaled Server):
- Sudden traffic spikes (like viral news posts or ticket launches) completely crashed backend servers.
- Adding more features made the server progressively slower for all users.
- Storing millions of user uploads (photos/videos) filled up the local hard drive, causing the system to lock up.

### After Scalability (Scaled Infrastructure):
- The system automatically provisions new server instances to handle traffic spikes, then shuts them down when traffic drops (Autoscaling).
- Heavy workloads are distributed across multiple worker nodes, keeping the user experience fast.
- Uploaded files are stored in central cloud storage systems that expand automatically, providing infinite storage capacity.

# Core Concepts
To build scalable systems, you must understand scaling directions, state management, and estimation techniques:

1. **Vertical Scaling (Scaling Up):** Adding more resources (more RAM, more CPU cores, faster disks) to a single existing server. It is simple to configure but has a hard physical ceiling and becomes exponentially expensive at high tiers.
2. **Horizontal Scaling (Scaling Out):** Adding more servers to a network cluster and distributing the workload among them. It is highly cost-effective and provides virtually unlimited scale, but increases software configuration complexity.
3. **Stateless Architecture:** An design style where application servers do not store any user session data (like login state or shopping cart items) on their local hard drives. This allows any incoming user request to be handled by any server in the cluster.
4. **Capacity Estimation (Back-of-the-Envelope Calculations):** High-level mathematical estimates used to plan resource requirements (like CPU, RAM, disk space, and network bandwidth) before building a system.

# Architecture / Components
The difference between a stateful architecture (hard to scale) and a stateless architecture (easy to scale):

```text
  Stateful Architecture (Hard to Scale)         Stateless Architecture (Easy to Scale)
  
      [ User A ]      [ User B ]                    [ User A ]      [ User B ]
          │               │                             │               │
          ▼               ▼                             ▼               ▼
    [ Server 1 ]    [ Server 2 ]                 [ Load Balancer (Routes requests) ]
   (Stores User A  (Stores User B                       │               │
    session data)   session data)                       ▼               ▼
                                                  [ Server 1 ]    [ Server 2 ]
                                                (No local data)  (No local data)
  - If Server 1 crashes, User A is logged out!          │               │
  - If traffic grows, we cannot easily route            ▼               ▼
    User A's queries to Server 2.                [ Central Session Cache (Redis) ]
                                                 - Shared state accessible by all
```

- **Stateless App Servers:** Processing nodes that execute code but hold no persistent data locally.
- **Shared State Storage:** Central databases or caches (like Redis) where user session data is consolidated.

# Workflow
The calculation workflow for back-of-the-envelope capacity planning:

```text
Step 1: Identify key business traffic metrics (e.g. 10 million Daily Active Users - DAU).
                             ↓
Step 2: Estimate the query volume: "If each user visits 10 times a day, that is 100 million requests/day."
                             ↓
Step 3: Convert to Requests Per Second (RPS): 100 million / 86,400 seconds = ~1,200 RPS on average.
                             ↓
Step 4: Estimate storage requirements: "If each write saves a 10KB JSON, storage grows by 100 million * 10KB = 1TB/day."
                             ↓
Step 5: Calculate network bandwidth: 1,200 requests/second * 10KB payload = 12 Megabytes/second (MB/s).
```

# Real World Examples
Think of database scaling as a **busy neighborhood hamburger restaurant**.
- **Vertical Scaling (Scaling Up):** You keep the same kitchen building and the same chef, but you buy a faster grill, a sharper knife, and send the chef to culinary school. This speeds up cooking, but eventually, the chef is moving as fast as physically possible, and you cannot fit a larger grill in the small kitchen. You have hit the physical hardware limit.
- **Horizontal Scaling (Scaling Out):** Instead of upgrading the kitchen, you lease 3 additional identical storefronts next door and hire 3 more chefs. You place a greeter at the door (Load Balancer) to direct incoming hungry customers to the kitchen with the shortest line. If your business doubles again, you just rent 4 more storefronts.
- **Stateful Problem:** A customer walks into Kitchen 1, sits at Table 3, and orders a drink. They then walk to the restroom. When they return, the host seats them in Kitchen 2. Because Kitchen 2 has no idea they ordered a drink, the customer is frustrated. The order was stored "statefully" inside Kitchen 1.
- **Stateless Solution:** The waiter writes the order on a digital tablet that sends it to a central screen in the office (**Shared Database Cache**). It doesn't matter which kitchen the customer sits in; the chef looks at the central screen and prepares their drink. The kitchens are stateless.

# Implementation
Here is how backend developers write stateless application logic by leveraging external caches (like Redis) for user sessions rather than storing them in server memory:

### Stateful vs. Stateless Session Logic in Node.js
```javascript
const express = require('express');
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const { createClient } = require('redis');

const app = express();

// 1. STATEFUL APPROACH (Bad for scaling): Stores session data in local server memory
/*
app.use(session({
  secret: 'my_secret',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: false } // Session is stored in RAM of this specific server
}));
*/

// 2. STATELESS APPROACH (Good for scaling): Session data is saved to a shared Redis server
const redisClient = createClient({ url: 'redis://session-cache-server:6379' });
redisClient.connect().catch(console.error);

app.use(session({
  store: new RedisStore({ client: redisClient }), // External shared store
  secret: 'my_secret_key',
  resave: false,
  saveUninitialized: false
}));

app.get('/dashboard', (req, res) => {
  if (req.session.userId) {
    res.send(`Welcome back User ${req.session.userId}`);
  } else {
    res.status(401).send("Unauthorized");
  }
});
```

# Best Practices
- **Prioritize Statelessness:** Build your backend servers to be completely stateless. Never store temporary user files or application logs on the local server disk; send files to cloud storage (like AWS S3) and logs to a central collector (like Elasticsearch).
- **Scale Horizontally by Default:** Assume your application will succeed and require multiple servers. Design your network, database connections, and session storage around a horizontal model from day one.
- **Always Calculate Peak Load, Not Average:** When performing capacity estimations, don't just plan for average traffic. A system that averages 100 requests per second might spike to 5,000 requests per second during a marketing event. Plan hardware capacity to handle these peak spikes.

# Industry Standards
Startups and tech enterprises leverage cloud providers (like Amazon Web Services or Google Cloud) to implement **Autoscaling Groups**. These groups monitor database CPU usage and automatically spin up new virtual servers during high traffic spikes, and terminate them when traffic decreases to save hosting costs.

# Common Mistakes
- **Storing User Uploads Locally:** Saving profile pictures to the local `public/uploads/` folder of a backend server. In a horizontally scaled system, if a user uploads a photo to Server A, and their next query hits Server B, the photo will appear broken because Server B's local disk doesn't have it.
- **Running Out of IP Addresses:** Failing to design network subnets large enough to accommodate horizontal server expansion. If your subnet only has 25 available IP addresses, your autoscaling group cannot expand past 25 servers.
- **Ignoring Database Limits:** Scaling your stateless application web servers to 100 instances without checking if your database can handle 100 times more concurrent connections, leading to database failure.

# Security & Performance Considerations
- **Session Hijacking on Shared Stores:** In a stateless architecture, session tokens must travel over network cables between the application server and the central session store (Redis). Developers must encrypt these connections to prevent attackers from sniffing session tokens.
- **Over-Scaling Cost Explosion:** Autoscaling groups can scale up infinitely if attacked (e.g. a Distributed Denial of Service - DDoS attack). If not configured with strict limits, the system will spin up thousands of servers to process the attack traffic, resulting in a massive, unexpected cloud billing invoice.

# Related Technologies
- **Kubernetes HPA (Horizontal Pod Autoscaler):** A system that automatically scales the number of running application containers based on CPU utilization.
- **AWS S3 / Google Cloud Storage:** Cloud object storage systems designed to store uploaded files statelessly at infinite scale.

# Summary

## What We Learned
- Scalability is the architectural ability to handle growing traffic and storage requirements by expanding computing resources.
- Vertical scaling adds power to a single server; horizontal scaling adds duplicate servers to a network.
- Keeping application servers stateless by moving data to shared caches is the key to simple horizontal scaling.

## Key Takeaways
- Never save persistent files or user session state on the local drives of your backend servers.
- Use back-of-the-envelope calculations to plan server count, network bandwidth, and database size needs.
- Set strict maximum limits on autoscaling groups to prevent runaway cloud hosting bills during traffic spikes or attacks.

# Keywords
- Scalability
- Vertical Scaling
- Horizontal Scaling
- Stateless
- Stateful
- Capacity Planning
- Back-of-the-Envelope
- Autoscaling
- Shared State
- Session Store

# Glossary

| Term | Meaning |
|---|---|
| Vertical Scaling | Upgrading a single server with more CPU cores, RAM, or storage capacity. |
| Horizontal Scaling | Adding more separate servers to a network cluster to share processing loads. |
| Stateless | An architecture design where servers do not store user data locally, making them interchangeable. |
| Autoscaling | A cloud system that automatically increases or decreases active server counts based on real-time traffic levels. |
| RPS | Requests Per Second; the measure of how many network requests a system processes every second. |
| Capacity Estimation | High-level math calculations used to estimate CPU, memory, storage, and bandwidth needs for a system. |

## Next Recommended Chapters
- 03-Load-Balancing.md
