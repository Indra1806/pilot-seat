> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Cloud Compute Services** represent the raw processing power of the cloud. They allow developers to lease CPU, RAM, and graphics processing units (GPUs) to execute application code. Cloud providers deliver compute resources in three primary formats: **Virtual Machines (VMs)**, **Serverless Container Runtimes**, and **Function-as-a-Service (FaaS)**, each offering a different balance between operational control and automated management.

# Why It Exists
Historically, running any piece of software required renting a physical server and keeping it powered on 24/7. However, different software tasks have different execution patterns. A high-traffic web API needs to run continuously with predictable performance, a nightly data cleanup script only needs to run for 10 minutes once a day, and an image resizer only needs to execute for a few seconds when a user uploads a photo. Cloud compute services were designed to offer specialized processing models that match these varying patterns, allowing developers to choose between managing the host operating system or delegating all infrastructure management to the cloud provider.

# Problem It Solves
Cloud compute services solve the problems of resource underutilization, server maintenance overhead, and complex scaling configurations.

### Before Cloud Compute (Single Server Model):
- Running a small, infrequent background script required hosting it on a full virtual machine that was billed 24/7, resulting in high costs for idle resources.
- If application traffic doubled, engineers had to manually configure complex load balancers, create new virtual machines, and synchronize code versions across them.
- Operations teams spent hours updating operating system security patches, adjusting virtual network configurations, and managing server firewall ports.

### After Cloud Compute (Flexible Compute Models):
- Event-driven background tasks run inside Serverless Functions (FaaS) that boot up in milliseconds when triggered, run for seconds, and scale down to zero, costing nothing when inactive.
- Serverless Container runtimes scale the number of running application instances up or down automatically in response to HTTP traffic spikes.
- Teams select Virtual Machines only when they require deep operating system configuration or need to run stateful legacy software.

# Core Concepts
To architect cloud applications, you must match your workload with the appropriate compute model:

1. **Virtual Machines (VMs):** The most flexible compute model (e.g. AWS EC2, GCP Compute Engine, Azure VMs). You lease a slice of a physical server, choose your operating system (Linux or Windows), and configure all software runtimes, ports, and security patches. You pay for the virtual machine continuously as long as it remains in a "Running" state.
2. **Serverless Container Runtimes:** Managed environments designed to run Docker containers (e.g. Google Cloud Run, AWS Fargate, Azure Container Instances). You package your application code and dependencies into a container image and upload it. The provider runs, scales, and manages the underlying servers on your behalf.
3. **Function-as-a-Service (FaaS):** The most granular compute model, often referred to as **Serverless Functions** (e.g. AWS Lambda, Google Cloud Functions, Azure Functions). You upload a single code function (written in Node.js, Python, Java, etc.) and link it to an event trigger. The provider boots a micro-VM, runs the code, and shuts it down instantly.
4. **Cold Start:** The minor boot-up delay (usually 100ms to 2 seconds) that occurs when a serverless container or function spins up from a stopped (zero running instances) state to handle a new incoming request.

# Architecture / Components
The structure of operating system management and control across the three compute models:

```text
  [ Virtual Machine (VM) ]           [ Serverless Container ]         [ Serverless Function (FaaS) ]
  ┌────────────────────────┐         ┌──────────────────────┐         ┌────────────────────────────┐
  │ [ Custom App Code ]    │         │ [ Custom App Code ]  │         │ [ App Code Function ]      │
  │ [ Software Runtimes ]  │         │ [ Container Config ] │         ├────────────────────────────┤
  │ [ Operating System ]   │         ├──────────────────────┤         │ [ Managed Runtimes / OS ]  │
  ├────────────────────────┤         │ [ Managed Runtimes ] │         │ [ Managed Hypervisor ]     │
  │ [ Managed Hypervisor ] │         │ [ Managed OS ]       │         │ [ Managed Hardware ]       │
  │ [ Managed Hardware ]   │         │ [ Managed Hardware ] │         └────────────────────────────┘
  └────────────────────────┘         └──────────────────────┘
  * Gray boxes indicate responsibilities managed automatically by the Cloud Provider.
```

# Workflow
How a cloud provider routes and executes traffic differently across compute models:

### Virtual Machine Workflow (Continuous Execution)
```text
Step 1: User request hits the Load Balancer -> Forwarded to a running VM instance.
Step 2: The VM processes the request and returns the response. The VM remains active and billed 24/7.
```

### Serverless Container Workflow (Scale-to-Zero Deployment)
```text
Step 1: User request hits the Load Balancer. Cloud Run checks if any container instances are running.
Step 2: If 0 instances are active, Cloud Run boots a container image (Cold Start: 1 second delay).
Step 3: The container processes the request, returns the response, and remains active for a 15-minute idle timeout.
Step 4: If no more traffic arrives, Cloud Run shuts down the container, halting all billing.
```

### Serverless Function Workflow (Event-Driven Execution)
```text
Step 1: User uploads an image file to an S3 bucket, triggering an ObjectCreated event.
Step 2: AWS Lambda detects the event, boots a lightweight micro-VM, and executes the resizing function code.
Step 3: The function resizes the image, writes it to another bucket, and exits. The micro-VM is destroyed.
```

# Real World Examples
Think of cloud compute models as **different transportation options**.
- **Virtual Machines (Leasing a car):** You lease a car by the month. You pay the bill whether the car sits in your driveway or drives on the highway. You must buy fuel, change the oil, pay for insurance, and drive it. You have total control over the radio, the speed, and the route.
- **Serverless Containers (Calling a ride-sharing taxi):** You do not own a vehicle, check the tire pressure, or pay insurance. When you need to go to the grocery store, you request a ride on your phone. A car arrives, takes you to the store, and departs. You only pay for the exact minutes and miles of your trip.
- **Serverless Functions / FaaS (Renting a public scooter):** You need to travel three blocks down the street. You find a scooter parked on the sidewalk, scan the code, ride it for 90 seconds, drop it on the curb, and pay a small fee. You would not use it to haul furniture or travel across the country, but it is perfect for a quick, single-purpose trip.

# Implementation
Here is how developers write and structure code for different compute services. Example A shows a serverless Node.js function (AWS Lambda), and Example B shows a standard Docker configuration file used to run a web app on a Serverless Container platform (Google Cloud Run):

### Example A: Serverless Function Code (AWS Lambda in Node.js)
```javascript
// index.js
// This function triggers automatically when a user requests an API endpoint
exports.handler = async (event) => {
    // 1. Extract query data from the triggering event
    const username = event.queryStringParameters?.name || "Guest";
    
    // 2. Execute simple business logic
    const greeting = `Hello, ${username}! Processed securely in a serverless micro-VM.`;
    
    // 3. Return a standard HTTP response structure
    return {
        statusCode: 200,
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ message: greeting }),
    };
};
```

### Example B: Serverless Container Configuration (Dockerfile for Cloud Run)
```dockerfile
# 1. Use the official Node.js lightweight image as the base
FROM node:18-alpine

# 2. Set the working directory inside the container
WORKDIR /usr/src/app

# 3. Copy dependency files and install production packages
COPY package*.json ./
RUN npm ci --only=production

# 4. Copy the application source code
COPY . .

# 5. Expose the port that Google Cloud Run will direct traffic to
EXPOSE 8080

# 6. Define the command to start the web server process
CMD [ "node", "server.js" ]
```

# Best Practices
- **Design Serverless Applications to be Stateless:** Serverless containers and functions boot up and shut down constantly. Never save session files or user data locally to the container's hard drive. Always store persistent state in a central database or cache (like Redis).
- **Consolidate Low-Traffic Workloads:** Do not run multiple tiny applications on separate, underutilized Virtual Machines. Combine them into a single VM using container isolation, or migrate them to serverless containers to scale down to zero when inactive.
- **Optimize Cold Start Boot Times:** In serverless functions, avoid importing heavy, unnecessary third-party libraries. Keep dependency sizes small and initialize database connections outside the primary handler function so they can be reused across warm requests.

# Industry Standards
Modern cloud deployments utilize **Spot Instances** (also known as Preemptible VMs) to optimize costs. Cloud providers sell their unused physical server capacity at discounts of up to 90%. The trade-off is that the provider can reclaim the server with short notice (usually 30 seconds to 2 minutes) when other customers need it. Organizations run stateless batch-processing workers on Spot Instances to save money.

# Common Mistakes
- **Running Long-Running Tasks in FaaS:** Attempting to run a database backup or video rendering script that takes 30 minutes to complete inside a serverless function. AWS Lambda has a strict 15-minute execution limit. The function will be forcefully terminated mid-execution. Use VMs or container tasks for long-running jobs.
- **Assuming Serverless Means No Cost:** Deploying a serverless function that receives millions of requests per second continuously. While FaaS is cheap for low, bursty traffic, a high-volume, constant workload is often cheaper when hosted on auto-scaled Virtual Machines.
- **Hardcoding Local Paths:** Writing files to `/var/tmp/` on a serverless container and expecting them to be there when the next user visits. These temporary local files are destroyed when the container instance scales down or updates.

# Security & Performance Considerations
- **Configure Timeout Limits:** Always set a strict timeout limit on serverless functions (e.g. 10 seconds). If your function connects to a hanging database and has no timeout configured, it will run until it hits the provider's maximum limit, inflating your billing costs.
- **FaaS Memory to CPU Allocation:** In serverless functions, you select the memory allocation size (e.g. 512MB or 2GB). Cloud providers automatically allocate proportional CPU power based on this memory setting. If your function is performing heavy calculations, increasing the memory allocation will speed up execution by giving it more CPU cores.

# Related Technologies
- **Firecracker:** An open-source virtualization technology developed by Amazon to run ultra-fast, secure micro-VMs that power services like AWS Lambda.
- **Knative:** A Kubernetes-based platform used to deploy serverless workloads and scale containers down to zero inside private Kubernetes clusters.
- **AWS Fargate:** A serverless compute engine for containers that works with both Amazon ECS and EKS.

# Summary

## What We Learned
- Cloud compute provides processing resources in three main categories: Virtual Machines (VMs), Serverless Containers, and Function-as-a-Service (FaaS).
- VMs offer complete operating system control and run continuously; serverless containers run packaged images and scale dynamically; FaaS runs event-driven code fragments and scales to zero.
- Cold Starts represent the latency delay when a serverless system provisions a new instance from a stopped state.

## Key Takeaways
- Select Virtual Machines for stateful, legacy applications or continuous workloads with constant CPU utilization.
- Use Serverless Containers for stateless web APIs to benefit from auto-scaling without server management chores.
- Ensure all serverless application components remain stateless by outsourcing data storage to database engines or caches.

# Keywords
- Compute Services
- Virtual Machine
- Serverless Container
- FaaS
- Cold Start
- AWS Lambda
- Cloud Run
- Spot Instance
- Stateless Compute
- Hypervisor

# Glossary

| Term | Meaning |
|---|---|
| Virtual Machine | A software-based simulation of a physical computer running its own guest operating system. |
| Serverless Container | A cloud service model that runs containerized code without requiring the user to manage virtual servers. |
| FaaS | Function-as-a-Service; a serverless compute model that executes user code blocks in response to system events. |
| Cold Start | The time required for a serverless platform to download, boot, and initialize a container or code runtime. |
| Spot Instance | Discovered, unused cloud server capacity offered at a discount, which can be reclaimed by the provider at any time. |
| Stateful App | An application that saves data locally to its own system memory or hard drive between user requests. |

## Next Recommended Chapters
- 07-Cloud-Storage-Architectures.md
