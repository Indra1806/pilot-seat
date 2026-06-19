> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Kubernetes** (commonly abbreviated as **K8s**) is an open-source container orchestration platform. It automates the deployment, scaling, network routing, and load balancing of containerized applications across a cluster of multiple physical or virtual server machines.

# Why It Exists
While Docker made containerizing a single application easy, running hundreds of containers across dozens of physical servers in production introduces massive management challenges. If a server hosting 20 containers crashes, an administrator has to manually log in, locate a healthy backup server, and restart the containers. If traffic spikes, scaling up requires manually starting containers and adjusting load balancers. Engineers created Kubernetes to act as an automated system administrator for container clusters, continuously monitoring container health, automating scaling, and managing network routing.

# Problem It Solves
Kubernetes solves manual container deployment management, server-level single-point-of-failure outages, and complex microservice network routing.

### Before Kubernetes (Manual Container Management):
- A host server crash took down all its containers, requiring manual operations intervention to relocate them.
- Scaling up required manually running Docker commands on different servers and updating load balancer configs.
- Internal microservices had to hardcode each other's dynamic IP addresses to communicate.

### After Kubernetes (Orchestrated Cluster):
- If a physical server node crashes, Kubernetes automatically detects the failure and recreates the lost containers on healthy nodes in seconds (Self-Healing).
- Systems automatically scale the number of running containers up or down based on CPU and traffic levels (Autoscaling).
- Containers communicate using stable internal DNS names managed by Kubernetes, which automatically load-balances traffic between them (Service Discovery).

# Core Concepts
To design Kubernetes deployments, you must master pods, deployments, services, and ingress:

1. **Pod:** The smallest deployable unit in Kubernetes. A pod represents a single running process and wraps one or more closely related containers (like an application container and a helper log collector container) that share the same network IP.
2. **Deployment:** A configuration controller that defines the desired state of your application, specifying how many identical copies (**Replicas**) of a Pod should run.
3. **Service:** An abstraction layer that defines a logical set of Pods and a stable network IP address/DNS name to access them, load-balancing traffic between replicas.
4. **Ingress:** An API object that manages external access to the services in a cluster, typically providing HTTP routing, SSL termination, and domain name mapping.

# Architecture / Components
The control plane and worker node architecture of a Kubernetes cluster:

```text
  [ K8s Control Plane (The Brain) ]
  - API Server (Gateway)
  - Scheduler (Decides where containers go)
  - Controller Manager (Enforces replica counts)
             │
             ▼ (Deploys pods over network)
  ┌────────────────────────────────────────────────────────┐
  │                 K8S WORKER NODES (The Muscle)          │
  │                                                            │
  │  [ Node Server 1 ]                  [ Node Server 2 ]      │
  │  ┌─────────────────────────┐        ┌───────────────────┐  │
  │  │ [ Pod: App Replica 1 ]  │        │ [ Pod: Database ] │  │
  │  │ [ Pod: App Replica 2 ]  │        │                   │  │
  │  └─────────────────────────┘        └───────────────────┘  │
  └────────────────────────────────────────────────────────┘
```

- **Control Plane:** The brain of the cluster that schedules workloads, detects node failures, and monitors cluster state.
- **Worker Nodes:** The physical or virtual servers that actually run the containerized Pod processes.
- **Kubelet:** A tiny agent running on each worker node that communicates with the Control Plane and ensures containers are running.

# Workflow
The self-healing workflow of a Kubernetes deployment:

```text
Step 1: Administrator applies a configuration file: `replicas: 3` for Node App.
                             ↓
Step 2: The Control Plane schedules and boots 3 duplicate Pod instances across worker Node 1 and Node 2.
                             ↓
Step 3: Worker Node 1 experiences a hardware power failure and goes offline.
                             ↓
Step 4: The Controller Manager detects the active replicas count has dropped from 3 to 1.
                             ↓
Step 5: The Scheduler identifies healthy worker Node 2 and immediately schedules 2 new Pods on it.
                             ↓
Step 6: Node 2 pulls the Docker image and boots the Pods. The active replicas count returns to 3.
```

# Real World Examples
Think of Kubernetes as a **fleet shipping company coordinator** or **cargo ship captain**.
- **Single Container (Docker):** A single metal shipping container. It is standard and easy to move, but it has no brain.
- **Kubernetes:** The shipping captain and crew running a massive container cargo ship.
  - **Worker Nodes:** The different storage decks of the cargo ship.
  - **Pod:** A wooden crate holding a main television set and its remote control accessories. They travel together in the same box.
  - **Deployment (Playbook):** The captain's shipping order ledger. It says: *"Keep exactly 5 crates of apples on deck at all times. If a crate of apples falls overboard (container crash), immediately go to the cargo hold, grab a new crate, and place it on deck to replace the lost one."* This is **Self-Healing**.
  - **Service (Routing Coordinator):** The ship's internal mail dispatcher. Instead of trying to find the moving mail clerk's exact location, workers drop mail in a designated bin labelled *"Mail Room"*. The dispatcher automatically routes the mail to whichever clerk is currently active.
  - **Ingress:** The harbor pilot standing at the entry deck directing visiting cargo inspectors to the correct department desks.

# Implementation
Here is how developers define a Kubernetes deployment and service using standard YAML configurations:

### Kubernetes Manifest File (`deployment.yml`)
Create this file to define a 3-replica Node.js application and expose it via a load-balanced service:

```yaml
# Part 1: Define the Deployment (Desired State of Pods)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app-deployment
  labels:
    app: node-web-app
spec:
  replicas: 3 # Keep exactly 3 identical copies running
  selector:
    matchLabels:
      app: node-web-app
  template:
    metadata:
      labels:
        app: node-web-app
    spec:
      containers:
      - name: web-container
        image: my-registry.com/node-web-app:v1.2.0
        ports:
        - containerPort: 3000
        resources:
          limits:
            cpu: "500m" # Cap CPU usage at 0.5 cores (cgroups)
            memory: "512Mi" # Cap RAM usage at 512 Megabytes
---
# Part 2: Define the Service (Stable Network IP and Load Balancer)
apiVersion: v1
kind: Service
metadata:
  name: node-app-service
spec:
  selector:
    app: node-web-app # Route traffic to pods matching this label
  ports:
    - protocol: TCP
      port: 80 # Exposed port on the service
      targetPort: 3000 # Target port on the pod container
  type: LoadBalancer # Automatically provisions a cloud load balancer
```

# Best Practices
- **Define Resource Requests and Limits:** Always specify CPU and memory limits for your containers (using `resources.limits`). If a container has a memory leak, specifying a limit allows Kubernetes to terminate it before it consumes all the memory on the host server, protecting neighboring containers.
- **Implement Liveness and Readiness Probes:** Configure Kubernetes to query health check endpoints. A **Liveness Probe** checks if a container has frozen and needs a reboot. A **Readiness Probe** checks if a container is finished booting (e.g. database connection established) and is ready to accept user traffic.
- **Store Configuration Separately:** Use **ConfigMaps** (for public configurations) and **Secrets** (for private keys) to inject configurations into containers dynamically at runtime, keeping container images environment-neutral.

# Industry Standards
Almost all modern cloud architectures standardize on managed Kubernetes services, such as AWS **EKS (Elastic Kubernetes Service)** or Google Cloud **GKE (Google Kubernetes Engine)**. These services automate the complex installation, scaling, and security patches of the Kubernetes Control Plane.

# Common Mistakes
- **Using Local Disk Storage inside Pods:** Saving persistent data (like user uploads or databases) inside a Pod directory without configuring a **PersistentVolume (PV)**. Pods are completely disposable; if a Pod is restarted or moved, all local data is deleted permanently.
- **Forgetting Readiness Probes on Deployments:** Deploying updates without readiness checks. During a new code rollout, Kubernetes will immediately route user traffic to the new containers before they have finished booting, resulting in visitors seeing HTTP 502 Bad Gateway errors.
- **Hardcoding Dynamic IP Addresses:** Configuring microservices to query other services using IP addresses. Pod IPs in Kubernetes are ephemeral (short-lived) and change on every reboot. Always use Kubernetes Service DNS names (e.g. `http://auth-service`).

# Security & Performance Considerations
- **Namespace Isolation Limitations:** By default, all Pods in a Kubernetes cluster can communicate with each other over the internal network. Administrators must configure **Network Policies** (firewall rules inside the cluster) to block unauthorized services (like a public blog frontend) from querying the database pods directly.
- **Cold Start Latency:** When scaling up, if your container images are large (e.g., 1GB), the worker node will take several minutes to download (pull) the image from the registry before booting, causing lag in auto-scaling. Keep container images small (under 100MB).

# Related Technologies
- **Helm:** The package manager for Kubernetes, used to install pre-configured applications (like databases or dashboards) using a single command.
- **kubectl:** The official command-line interface tool used to query and manage Kubernetes clusters.
- **Minikube:** A tool that runs a single-node Kubernetes cluster locally inside a virtual machine on your laptop for testing.

# Summary

## What We Learned
- Kubernetes is an automation engine that schedules, scales, heals, and routes network traffic for containerized applications.
- Pods are the basic container wrappers; Deployments enforce desired replica counts; Services provide stable load-balanced IPs.
- Control Planes coordinate cluster logic, while Worker Nodes run the physical container processes.

## Key Takeaways
- Always configure CPU/RAM limits to prevent single container leaks from crashing the host server node.
- Utilize Readiness and Liveness probes to guarantee zero-downtime rolling updates and automated reboots.
- Never hardcode IP addresses; leverage Kubernetes internal service DNS routing instead.

# Keywords
- Kubernetes
- Container Orchestration
- Pod
- Deployment
- Service
- Ingress
- Control Plane
- Worker Node
- Self-Healing
- Readiness Probe

# Glossary

| Term | Meaning |
|---|---|
| Pod | The smallest deployable unit in Kubernetes, hosting one or more co-located containers. |
| Deployment | A controller configuration that maintains a specified number of identical running Pod replicas. |
| Service | An internal load balancer providing a stable IP address and DNS name for a set of Pods. |
| Ingress | A router object that configures external HTTP access routes into internal Kubernetes services. |
| Self-Healing | The automated process where Kubernetes restarts failed containers or reschedules pods when servers crash. |
| Kubelet | The primary node agent that runs on worker nodes to monitor and execute pods. |

## Next Recommended Chapters
- 08-Terraform.md
