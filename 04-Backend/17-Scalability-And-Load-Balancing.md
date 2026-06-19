> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Scalability** is the capacity of a backend system to handle growing amounts of work (traffic, database writes, or file processing) by dynamically adding computing resources. **Load Balancing** is the network practice of distributing incoming user traffic across a pool of multiple backend servers, ensuring no single server gets overloaded, thereby maximizing website availability and uptime.

# Why It Exists
Every physical server computer has hardware limits: a maximum amount of RAM, a fixed number of CPU cores, and a limited network card bandwidth. If your application runs on a single server, and your traffic grows from 100 users to 1 million users, that server's CPU will hit 100% usage, memory will run out, and the website will crash. Historically, developers tried to solve this by buying larger, incredibly expensive mainframe computers. However, even the largest computers have physical limits. Engineers created load balancers and horizontal scaling patterns to let applications run across dozens of cheap, standard servers coordinated as a single network.

# Problem It Solves
Scalability and load balancing solve single server hardware bottlenecks, single points of failure (server crashes), and unequal traffic distribution.

### Before Scaling & Load Balancing (Single Server):
- When traffic spiked, the server CPU maxed out, and the website went down for everyone.
- If the server hardware failed or lost power, the entire business went offline (single point of failure).
- Upgrading the server required shutting it down, causing planned downtime.

### After Horizontal Scaling & Load Balancing:
- User traffic is divided evenly among 5 or 10 separate server nodes.
- If Server A crashes, the load balancer instantly detects the failure and routes all subsequent traffic to Servers B, C, and D, keeping the site online (fault tolerance).
- Upgrades are seamless: you update Server A while others handle traffic, then rotate through the pool (rolling updates).

# Core Concepts
To scale backend systems, you must master the differences in scaling dimensions and routing rules:

1. **Vertical Scaling vs. Horizontal Scaling:**
   - **Vertical Scaling (Scale Up):** Adding more power (CPU, RAM) to your existing server. This has a physical ceiling and is highly expensive.
   - **Horizontal Scaling (Scale Out):** Adding more servers of the same size to your pool. This has no ceiling and is cheap, but requires your application code to be stateless.
2. **Stateless Servers:** To scale horizontally, your servers must not store user session files or uploaded images on their local hard drives. If a user logs in on Server A, and their next click is routed to Server B, Server B must be able to recognize them by reading from a shared database (like Redis) rather than its local disk.
3. **Load Balancing Algorithms:** How the load balancer decides which server gets the next request:
   - **Round Robin:** Routing requests sequentially (Request 1 -> Server A, Request 2 -> Server B, Request 3 -> Server C).
   - **Least Connections:** Routing the next request to the server currently handling the fewest active connections.
   - **IP Hash:** Routing requests from a specific user IP address to the exact same server every time (useful for sticky sessions).

# Architecture / Components
The flow of user traffic through a Load Balancer to a stateless server pool and replicated database:

```text
                           [ User Traffic ]
                                  │
                                  ▼
                         [ Load Balancer ] (Nginx / AWS ALB)
                         ┌────────┼────────┐
                         ▼        ▼        ▼
                    [ Server A ] [ Server B ] [ Server C ]  (Stateless Node Pool)
                         │        │        │
                         └────────┼────────┘
                                  ▼
                         [ Shared Session Cache ] (Redis RAM)
                                  │
                                  ▼
                        [ Primary Database ]  (Handles Writes)
                         │        │
                         ▼        ▼
                    [ Replica ] [ Replica ]   (Handles Reads)
```

- **Load Balancer:** The entry router (e.g. Nginx, HAProxy, or AWS Application Load Balancer) that distributes traffic.
- **Read Replicas:** Database clones that mirror the primary database in real-time. The primary handles writes (inserts/updates), while replicas handle read queries (selects), splitting the load.
- **Database Sharding:** Splitting a massive database table horizontally across multiple separate database servers (e.g., storing users A-M on Database 1, and N-Z on Database 2).

# Workflow
How a load-balanced, stateless system handles a user request:

```text
Step 1: A user requests `GET /catalog` in their browser.
                             ↓
Step 2: The request hits the Load Balancer on Port 443.
                             ↓
Step 3: The Load Balancer runs the "Least Connections" algorithm and selects Server B.
                             ↓
Step 4: Server B receives the request, fetches the catalog data from a Database Read Replica.
                             ↓
Step 5: Server B reads the user's session state from a shared Redis database.
                             ↓
Step 6: Server B builds the response and sends it back through the Load Balancer to the user.
```

# Real World Examples
Think of scalability as a **supermarket checkout area**.
- A single-server application is like a grocery store with **one cashier working one register**.
- **Vertical Scaling** is like training that cashier to scan items super-fast and buying them a high-speed scanner. It runs faster, but one human still has a physical speed limit. If the store grows into a massive Walmart, one lane will choke, and if the cashier gets sick, the store closes.
- **Horizontal Scaling** is opening **10 separate checkout lanes**, each with a standard cashier.
- The **Load Balancer** is the **store manager standing at the entry to the checkout area**, directing arriving shoppers:
  - *Round Robin* is sending Shopper 1 to Lane 1, Shopper 2 to Lane 2, etc.
  - *Least Connections* is looking at which lane has the shortest line and sending you there.
- For this to work, the cashiers must be **Stateless**: they cannot store your cart list in their private pockets. If you step from Lane 1 to Lane 2, the cashier must access a shared network database to see your scanned items.
- **Database Read Replicas** are like placing **3 printed price catalogs at the registers** so cashiers can quickly look up prices (Reads) without running to the master accounting ledger desk in the back office (Write Master) every time.

# Implementation
Here is how you configure a reverse proxy load balancer in **Nginx** to distribute web traffic across three backend application servers:

### The Nginx Configuration File (`nginx.conf`)
```nginx
# 1. Define the pool of backend application servers (upstream)
upstream my_backend_app {
    # Nginx uses Round Robin by default to distribute traffic
    server 10.0.0.10:3000; # Private IP of Server A
    server 10.0.0.11:3000; # Private IP of Server B
    server 10.0.0.12:3000; # Private IP of Server C
}

server {
    listen 80;
    server_name mysite.com;

    # 2. Route all public HTTP requests to our upstream pool
    location / {
        proxy_pass http://my_backend_app;
        
        # Pass standard headers so backend servers know the client's original details
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

# Best Practices
- **Design Applications to be Stateless from Day One:** Never save files, uploaded avatars, or session variables directly to the server's local hard drive or local node memory. Always use external cloud storage (like S3) and shared cache databases (like Redis) so your servers can be shut down, replaced, or scaled horizontally at any moment.
- **Implement Health Checks:** Configure your load balancer to ping a specific route on your servers (like `/healthz`) every 5 seconds. If Server A fails to respond (due to a crash or network error), the load balancer should automatically mark it as unhealthy and stop sending traffic to it, keeping your site online.
- **Automate Autoscaling:** Use cloud features (like AWS Auto Scaling groups) to monitor your server pool's average CPU usage. Configure rules to spin up new servers automatically if CPU usage exceeds 75%, and delete them when usage drops, saving costs.

# Industry Standards
Modern cloud applications use **Nginx** or **HAProxy** for software load balancing, or native cloud routing layers like **AWS Application Load Balancers (ALB)** or **Cloudflare Tunnels**. The orchestrator **Kubernetes** is the industry standard used to automatically scale and load balance dockerized application containers across huge server fleets.

# Common Mistakes
- **Using Sticky Sessions as a Permanent Crutch:** Configuring the load balancer to route a user to the exact same server because your code stores their session in local memory. If that specific server crashes, the user's session is wiped, and they are logged out. Build stateless sessions instead.
- **Neglecting Database Scaling:** Scaling your application servers to 50 nodes, but keeping one single SQL database. The database will choke under the flood of connection requests from your scaled servers, creating a major performance bottleneck. Scale your database using read replicas.

# Security & Performance Considerations
- **SSL Termination:** Performing the heavy cryptographic calculations required for HTTPS encryption at the load balancer level (SSL Termination). The load balancer decodes the secure HTTPS request and forwards it as standard, fast HTTP to the backend servers inside your private, secure network, offloading CPU work from your application servers.
- **DDoS Protection:** Load balancers act as the first line of defense against Distributed Denial of Service (DDoS) attacks. You can configure them to rate limit IP addresses or drop suspicious malformed requests before they hit your application servers.

# Related Technologies
- **HAProxy:** A popular, high-performance TCP/HTTP load balancer software.
- **Kubernetes Ingress:** The API resource that manages external access and load balancing to services inside a Kubernetes cluster.
- **Amazon Route 53:** A DNS service used to route traffic across different geographical cloud regions (Global Server Load Balancing).

# Summary

## What We Learned
- Scalability expands system capacity, and load balancing distributes traffic across a server pool.
- Horizontal scaling requires applications to be stateless, offloading session data to shared memory stores.
- Database scaling splits write loads on primary servers from read loads on replica servers.

## Key Takeaways
- Ensure all servers are stateless to enable seamless, automated horizontal scaling.
- Configure automated health checks on the load balancer to bypass crashed server nodes.

# Keywords
- Scalability
- Load Balancing
- Horizontal Scaling
- Stateless
- Round Robin
- Nginx
- Read Replica
- Health Check

# Glossary

| Term | Meaning |
|---|---|
| Vertical Scaling | The process of adding more resources (CPU, RAM) to a single existing server computer. |
| Horizontal Scaling | The process of adding more server computers to a pool to share traffic load. |
| Stateless | An application design where servers do not store client session data or state in local files. |
| Upstream | The backend pool of application servers positioned behind a load balancer reverse proxy. |
| SSL Termination | Decrypting secure HTTPS traffic at the load balancer level before forwarding it to backend servers as plain HTTP. |

## Next Recommended Chapters
- 11-Containerization-And-Docker.md
- 13-Microservices-Architecture.md
