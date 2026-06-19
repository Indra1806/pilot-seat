> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Serverless Architecture** (often called **Function-as-a-Service**, or **FaaS**) is a cloud computing design pattern where developers write and deploy individual functions of code without managing any underlying server hardware, operating systems, or virtual machines. The cloud provider (like AWS, Google, or Microsoft) handles all server provisioning, security patches, hardware maintenance, and automatic scaling, charging developers only for the exact milliseconds their code is actively running.

# Why It Exists
In traditional web hosting, developers had to rent a server computer (like an AWS EC2 instance) running 24/7. Even if your website received zero visitors at 3 AM, you still paid the full hourly rate for that server. Furthermore, if your site went viral, you had to manually set up complex load balancers and configure additional servers to share the load. Developers spent half their time patching operating systems, configuring firewalls, and managing server clusters instead of writing features. Engineers created serverless computing in 2014 (starting with AWS Lambda) to offload all infrastructure operations to cloud platforms, allowing developers to focus entirely on writing application logic.

# Problem It Solves
Serverless architecture solves server management overhead, wasted idle server costs, and manual scaling complexities.

### Before Serverless (Traditional Server Hosting):
- You paid a fixed monthly fee for servers even when they sat idle with 0% CPU usage.
- Scaling to handle traffic spikes required setting up complex auto-scaling rules and waiting minutes for new servers to boot up.
- Upgrading operating systems and applying security patches took hours of developer maintenance.

### After Serverless (On-Demand Functions):
- Costs scale to zero: if no one visits your site, you pay absolutely nothing.
- Scaling is instant and infinite: if 10,000 requests arrive at the same second, the cloud provider automatically runs 10,000 isolated instances of your function.
- Zero server maintenance: the cloud provider handles all security updates and hardware.

# Core Concepts
To write serverless applications, you must master the on-demand runtime model:

1. **Ephemeral Functions (FaaS):** Code is written as single-purpose, short-lived functions. They spin up instantly when triggered, run their calculations, and vanish immediately when finished.
2. **Event-Driven Triggers:** Serverless functions are dormant until triggered by an event—such as an HTTP request, a file upload to cloud storage, or a scheduled timer.
3. **Cold Starts:** The latency delay that occurs when a function is triggered after being idle. The cloud provider must locate a physical server, set up a secure container sandbox, load your code, and start the runtime before executing the function, which can add a few seconds of delay to the first request.

# Architecture / Components
The flow of an event triggering a serverless function:

```text
  [ User HTTP Request ] ──> [ API Gateway ] (Routes request to function trigger)
                                  │
                                  ▼
                        [ Serverless VM Sandbox ]  (Spins up in milliseconds)
                                  │
                               (Executes)
                                  ▼
                        [ Lambda / Function ]      (Runs code & queries DB)
                                  │
                                  ▼
                        [ Returns JSON Payload ]
                                  │
                                  ▼
                        [ Sandbox is Destroyed ]   (Scaled back to zero)
```

- **API Gateway:** The front-facing router that intercepts HTTP requests and forwards them as trigger events to specific serverless functions.
- **Serverless Provider:** (e.g. AWS Lambda, Google Cloud Functions) The cloud management system that dynamically provisions execution containers.
- **Cold Start vs. Warm Start:**
  - **Cold Start:** Booting up a new container sandbox after the function has been inactive.
  - **Warm Start:** Reusing an active container sandbox that is still in memory from a very recent request, executing code instantly.

# Workflow
The serverless execution lifecycle:

```text
Step 1: A user uploads an image file `avatar.jpg` to an AWS S3 cloud storage bucket.
                             ↓
Step 2: S3 detects the upload and publishes a `FileCreated` event trigger.
                             ↓
Step 3: The serverless system spins up a new isolated container sandbox (Cold Start).
                             ↓
Step 4: The serverless function downloads the image, resizes it to a thumbnail, and saves it.
                             ↓
Step 5: The function returns success and exits.
                             ↓
Step 6: The system keeps the container warm for 15 minutes, then destroys it if no new uploads occur.
```

# Real World Examples
Think of serverless architecture as **calling an Uber** versus **leasing a company car**.
- Traditional hosting is like leasing a car. You pay a fixed fee every month whether the car is driving on the highway or sitting empty in your garage. You are responsible for oil changes, tire replacement (OS patches), and if 50 employees need to drive at the same time, you have to buy 50 cars (manual scaling).
- Serverless is like calling a ride-share taxi. You don't own the car, you don't do maintenance, and you pay nothing while you sleep. When you need to go somewhere, you call a ride (trigger a function). You pay *only* for the exact minutes you are in the car (execution time). If 50 employees call a ride at the same time, Uber sends 50 separate cars instantly, costing you nothing once the rides end.
- The catch? A **Cold Start** is like waiting 5 minutes for the taxi to arrive at your door when you call it, compared to walking out and driving your pre-warmed leased car immediately.

# Implementation
Here is how a developer writes a simple AWS Lambda serverless function in JavaScript that handles an API request:

```javascript
// 1. Next-gen serverless code is written as a simple exported handler function
exports.handler = async (event) => {
  console.log("Function triggered by event: ", JSON.stringify(event));

  // 2. Read parameters passed by the trigger (e.g. from the URL query)
  const username = event.queryStringParameters ? event.queryStringParameters.name : "Guest";

  // 3. Perform calculations
  const message = `Hello, ${username}! This response was generated by a serverless function.`;

  // 4. Return an HTTP formatted response payload
  const response = {
    statusCode: 200,
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({ message: message })
  };

  return response;
};
```

# Best Practices
- **Keep Functions Stateless:** Serverless functions are destroyed immediately after execution. Never save files to the local hard drive or store variables in local memory expecting them to be there next time. Always save data to an external database (like PostgreSQL) or cloud storage (like S3).
- **Write Small, Single-Purpose Functions:** Do not pack your entire backend API into a single serverless function. The larger your code file size, the longer the Cold Start delay will be. Keep functions tiny and focused on a single task (e.g. one function for login, one for uploading, one for fetching reports).
- **Control Database Connections:** Traditional databases have connection limits. Since serverless scales instantly to thousands of parallel functions, they can overwhelm your database with connections, crashing it. Use connection pool proxies (like AWS RDS Proxy or Prisma Accelerate) to manage connections.

# Industry Standards
**AWS Lambda** is the pioneer and market leader in serverless FaaS. **Google Cloud Functions** and **Azure Functions** are popular alternatives. Cloudflare also offers **Cloudflare Workers**, which run serverless code on global edge networks inside v8 engine isolates for near-zero cold start times.

# Common Mistakes
- **Using Serverless for Long-Running Tasks:** Serverless functions have execution time limits (e.g., AWS Lambda terminates functions after 15 minutes). Trying to use serverless for rendering 2-hour movies or running 30-minute web scrapers will fail. Use dedicated servers or container tasks for long workloads.
- **Ignoring Cold Start Latency for Public APIs:** Using serverless for high-performance public APIs where users expect sub-100ms response times. If the API is rarely hit, users will experience frequent cold start delays.

# Security & Performance Considerations
- **Function Permissions (IAM):** Since serverless functions are isolated, configure strict IAM access policies for each function. For example, the `SendEmail` function should only have permission to access the email service, not write to the main database.
- **Resource Allocation:** You configure serverless functions by assigning memory limits (e.g. 128MB to 10GB of RAM). The more memory you assign, the more CPU power the cloud provider allocates, speeding up execution but increasing costs.

# Related Technologies
- **AWS Lambda / Cloudflare Workers:** The leading serverless execution platforms.
- **Serverless Framework / SAM:** Command-line developer tools used to package and deploy serverless functions to the cloud easily.
- **Supabase / Firebase:** Backend-as-a-Service platforms that bundle serverless databases and functions out-of-the-box.

# Summary

## What We Learned
- Serverless architecture dynamically runs code functions on-demand without manual infrastructure management.
- Costs scale to zero during idle periods, and execution scales horizontally instantly during traffic spikes.
- Cold start latency occurs when launching dormant containers, requiring compact code payloads.

## Key Takeaways
- Design serverless functions to be completely stateless, offloading persistent data to external storage.
- Avoid using serverless for long-running jobs that exceed runtime execution limits.

# Keywords
- Serverless
- FaaS
- AWS Lambda
- Cold Start
- Event-Driven
- Stateless
- API Gateway
- Autoscaling

# Glossary

| Term | Meaning |
|---|---|
| FaaS | Function-as-a-Service; a cloud computing category allowing execution of code functions in response to events. |
| Cold Start | The boot-up latency delay that occurs when a cloud provider prepares a fresh container sandbox for an inactive function. |
| Statelessness | The design pattern where a software service retains no client data or session memory between requests. |
| Ephemeral | Short-lived; existing or running for only a very brief duration of time before being destroyed. |
| IAM | Identity and Access Management; the security configuration system used to manage access permissions for cloud services. |

## Next Recommended Chapters
- 01-Web-Servers-And-HTTP.md
- 04-Backend/11-Containerization-And-Docker.md
