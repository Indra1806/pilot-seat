> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
A **Load Balancer** is a specialized network component (hardware device or software program) that sits between client users and backend application servers. It intercepts all incoming network requests and distributes them across a pool of duplicate backend servers to prevent overload, optimize server response speeds, and ensure high availability.

# Why It Exists
If an application is scaled horizontally by adding 10 servers, the system needs a way to decide which server handles which user request. Without a load balancer, users would connect directly to a single server's public IP address. If 90% of your users connect to Server A, it will run out of capacity and crash, while Servers B through J sit idle. Engineers created load balancers to act as a traffic director at the entryway of a server cluster, ensuring that request traffic is divided evenly among all available machines.

# Problem It Solves
Load balancing solves server overload, single-point-of-failure downtimes, and network traffic congestion.

### Before Load Balancing (Direct Connection):
- One popular server took the brunt of user requests and crashed, while backup servers went unused.
- If a server crashed, users connected to its IP address received a broken webpage, causing application downtime.
- Users had to know specific server IP addresses to access the application.

### After Load Balancing (Managed Routing):
- Traffic is divided dynamically based on server capacities, keeping response speeds fast for everyone.
- If Server A crashes, the load balancer detects the failure and instantly routes all subsequent traffic to Server B (Failover).
- Users connect to a single unified domain name (e.g. `example.com`) pointed to the load balancer, which hides the internal server IPs.

# Core Concepts
To master load balancing, you must understand routing algorithms and network layers:

1. **Round Robin:** The simplest load balancing algorithm. It routes incoming requests to servers in sequential order (Request 1 to Server A, Request 2 to Server B, Request 3 to Server C, then starts over).
2. **Least Connections:** A dynamic algorithm that routes the next request to whichever server is currently processing the fewest active client connections, making it ideal for long-running request systems.
3. **Layer 4 (L4) Load Balancing:** Routing decisions are made at the Transport network layer (TCP/UDP). The load balancer only inspects the sender's IP address and port number. It is extremely fast because it does not read the actual query content.
4. **Layer 7 (L7) Load Balancing:** Routing decisions are made at the Application layer (HTTP/HTTPS). The load balancer inspects the actual content of the request, such as URL paths, HTTP headers, or browser cookies, enabling intelligent routing.

# Architecture / Components
The network flow of client requests entering a Layer 7 load balancer and being routed to specialized backend microservices:

```text
                               [ User Requests ]
                                       │
                                       ▼ (Hits example.com on Port 80/443)
                               [ Load Balancer ]
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            ▼ (If path is '/users')    ▼ (If path is '/products') ▼ (If path is '/checkout')
     [ User Service ]          [ Catalog Service ]        [ Payment Service ]
     - Server Node A           - Server Node B            - Server Node C
```

- **Reverse Proxy:** The load balancer acts as a reverse proxy, receiving requests from the public internet and forwarding them to internal, private servers.
- **Health Checks:** Periodic ping requests the load balancer sends to backend servers. If a server stops replying, the load balancer marks it "unhealthy" and stops sending it traffic.

# Workflow
How a Layer 7 load balancer processes and routes an incoming user request:

```text
Step 1: A user requests `example.com/checkout`.
                             ↓
Step 2: The load balancer receives the request, decrypts the SSL/TLS encryption, and inspects the HTTP headers.
                             ↓
Step 3: The load balancer identifies the path `/checkout`.
                             ↓
Step 4: The load balancer checks its routing table and selects the "Checkout Service" server pool.
                             ↓
Step 5: The load balancer uses the Least Connections algorithm to select Checkout Server Node 3.
                             ↓
Step 6: The load balancer forwards the request to Server Node 3, receives the response, and returns it to the user.
```

# Real World Examples
Think of a load balancer as an **airport check-in line coordinator**.
- **Without a Coordinator:** Passengers walk into the terminal and run to the first check-in desk they see. Desk A gets a line of 100 passengers, while Desks B through J have zero wait lines. The agent at Desk A is exhausted and making mistakes, while other agents are sitting idle.
- **With a Coordinator (Load Balancer):** A coordinator stands at the front of a single unified queue line. They monitor all check-in desks. 
  - **Round Robin:** The coordinator says: *"Passenger 1 go to Desk A, Passenger 2 go to Desk B, Passenger 3 go to Desk C."*
  - **Least Connections:** The coordinator watches which agent finishes first and says: *"Next passenger, go to Desk F because their line is completely empty."*
  - **Sticky Sessions (IP Hash):** The coordinator says: *"Passenger A, since you started your check-in with Agent B, you must return to Agent B to finish your bag drop so they don't have to start your paperwork over."*
  - **Layer 4 vs. Layer 7:**
    - **Layer 4:** The coordinator only looks at the color of your suitcase (IP address/Port) and directs you to a desk without opening your passport.
    - **Layer 7:** The coordinator stops you, opens your ticket, reads your destination (*"Ah, you are flying to France"*), and directs you to a specialized French-speaking desk.

# Implementation
Here is a basic configuration for Nginx, the industry-standard software load balancer, showing how to define a backend server pool and route HTTP requests:

### Nginx Load Balancer Configuration (`nginx.conf`)
```nginx
# 1. Define the backend pool of application servers
upstream backend_app_servers {
    # Round Robin is used by default
    server 192.168.1.101:8080 max_fails=3 fail_timeout=10s;
    server 192.168.1.102:8080 max_fails=3 fail_timeout=10s;
    server 192.168.1.103:8080 max_fails=3 fail_timeout=10s;
    
    # Or use Least Connections by adding:
    # least_conn;
}

server {
    listen 80;
    server_name mywebsite.com;

    # 2. Intercept traffic and forward to the upstream backend pool
    location / {
        proxy_pass http://backend_app_servers;
        
        # Pass client IP details to the backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

# Best Practices
- **Configure Strict Health Checks:** Set up the load balancer to query a lightweight health check path (like `/healthz`) on your servers every few seconds. If a server freezes or runs out of database connections, the load balancer should remove it from the pool immediately.
- **Enable SSL Termination:** Let the load balancer handle the CPU-heavy work of decrypting SSL/TLS certificates. It can then forward decrypted HTTP queries to your backend servers over a secure private network, freeing up backend server CPU.
- **Implement Redundant Load Balancers:** A load balancer is a single point of failure. If your only load balancer crashes, your entire site is offline. Set up a backup load balancer in a "Active-Passive" configuration using a floating IP address (VRRP) to automatically take over if the active load balancer fails.

# Industry Standards
Massive cloud-scale systems leverage cloud-native services like AWS **ALB (Application Load Balancer)** for Layer 7 routing and **NLB (Network Load Balancer)** for ultra-fast Layer 4 routing, which can handle millions of requests per second.

# Common Mistakes
- **Failing to Pass Client IP Address:** By default, when a load balancer forwards a request, the backend server sees the request as coming from the load balancer's IP address. If developers don't pass the `X-Forwarded-For` header, security firewalls and user analytics will think all website traffic is coming from a single internal machine.
- **Using Sticky Sessions on Stateless Apps:** Enabling sticky sessions (forcing a user to stay on the same server node) when it isn't necessary. This prevents true load balancing, as some servers will end up holding more active users than others.
- **Failing to Tune Connection Limits:** Forgetting to increase Nginx system connection limits (file descriptors). Under heavy traffic, Nginx will throw "Too many open files" errors and reject incoming users because it hit system-level connection ceilings.

# Security & Performance Considerations
- **DDoS Mitigation at the Gateway:** The load balancer is your first line of defense. By configuring rate-limiting rules directly on the load balancer, you can block automated hacking scripts and DDoS attacks before they reach your backend application code.
- **SSL Overhead latency:** While SSL termination speeds up backend processing, decrypting HTTPS requests at the load balancer increases the load balancer's memory and CPU usage. Teams must size load balancer servers larger than standard web nodes.

# Related Technologies
- **Nginx:** A high-performance reverse proxy and software load balancer.
- **HAProxy:** A dedicated, extremely fast software load balancer.
- **AWS ELB (Elastic Load Balancing):** Amazon's managed cloud load balancing service.

# Summary

## What We Learned
- A load balancer acts as a traffic director, distributing client queries across a pool of backend servers to optimize speed and availability.
- Round Robin routes traffic sequentially, while Least Connections routes traffic to the quietest server node.
- Layer 4 operates at the TCP layer for speed; Layer 7 operates at the HTTP level for intelligent content-based routing.

## Key Takeaways
- Always configure active health checks to detect and isolate crashed servers automatically.
- Terminate SSL/TLS certificates at the load balancer to save backend application CPU cycles.
- Run redundant load balancers using high-availability setups to prevent a single point of failure.

# Keywords
- Load Balancer
- Round Robin
- Least Connections
- Layer 4 (L4)
- Layer 7 (L7)
- Reverse Proxy
- Health Check
- SSL Termination
- Active-Passive
- X-Forwarded-For

# Glossary

| Term | Meaning |
|---|---|
| Load Balancer | A device or software that distributes network traffic across multiple servers. |
| Reverse Proxy | A server that accepts requests on behalf of internal servers, protecting the internal network structure. |
| Health Check | A test query run by a load balancer to verify if a backend server is still running. |
| SSL Termination | The process of decrypting HTTPS requests at the load balancer so the backend servers receive decrypted HTTP data. |
| Sticky Session | A configuration where a client is consistently routed to the same physical backend server for the duration of their visit. |
| Layer 7 | The application layer of the OSI network model, handling HTTP/HTTPS protocol details. |

## Next Recommended Chapters
- 04-Caching.md
