> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Docker** is an open-source containerization platform that allows developers to package applications, libraries, dependencies, and configurations into a single, isolated execution unit called a **Container**. Docker containers run consistently on any computer, from a developer's local laptop to massive production cloud environments, resolving the classic "works on my machine" deployment issue.

# Why It Exists
Before containerization, deploying applications was complex and error-prone. A developer writing code on a macOS computer had to deploy it to a Linux server. If the server had a slightly different version of Node.js, python, or database libraries installed, the application would crash. To prevent this, teams used **Virtual Machines (VMs)** to bundle applications with their own operating systems. However, VMs are extremely heavy, taking gigabytes of disk space and minutes to boot because they run a full "guest" operating system on top of a hypervisor. Engineers created Docker to isolate applications at the operating system level, sharing the host OS kernel to create lightweight, fast-booting containers.

# Problem It Solves
Docker solves dependency version mismatches, heavy Virtual Machine overhead, and complex local development environment setups.

### Before Docker (Bare Metal or Virtual Machines):
- Developers wasted hours installing matching database and package versions on their local machines.
- Deploying a new service instance took minutes because a full virtual machine had to boot up.
- Different applications on the same server interfered with each other's configurations, leading to dependency conflicts.

### After Docker (Containerized Setup):
- Developers download and run databases or microservices locally in seconds using a single command.
- Containers boot in milliseconds and share the host operating system, allowing a single server to run hundreds of isolated containers.
- Each application runs in its own isolated container bubble, preventing configuration conflicts.

# Core Concepts
To work with Docker, you must master the difference between images and containers, kernel isolation, and layers:

1. **Docker Image:** The read-only, immutable blueprint or template used to create containers. It contains the application code, libraries, and runtime environment settings (similar to a class in object-oriented programming).
2. **Docker Container:** The active, running instance of a Docker Image. It is a runnable process isolated from other containers and the host machine (similar to an object instance).
3. **OS Kernel Sharing (Namespaces & Cgroups):** The Linux kernel technologies Docker uses to isolate containers:
   - **Namespaces:** Restricts what a container can *see* (giving it a private view of the filesystem, network cards, and running processes).
   - **Cgroups (Control Groups):** Restricts what resources a container can *use* (capping its maximum CPU and RAM allocations).
4. **Image Layers:** Docker images are built as a stack of read-only layers. Each instruction in a Dockerfile creates a new layer. Docker caches these layers to speed up subsequent image builds.

# Architecture / Components
The difference in server architecture between Virtual Machines and Docker Containers:

```text
      Virtual Machine Architecture                  Docker Container Architecture
  +------------------------------------+        +------------------------------------+
  | [ App A ] [ App B ]  (Apps)        |        | [ App A ] [ App B ]  (Containerized|
  | [Guest OS][Guest OS] (Heavy OS)    |        | [ Docker Engine ]    (Shared host) |
  | [     Hypervisor     ]             |        | [ Host Operating System (Kernel) ] |
  | [ Host Operating System (Kernel) ] |        | [ Physical Server Hardware ]       |
  | [ Physical Server Hardware ]       |        +------------------------------------+
  +------------------------------------+
```

- **Docker Engine:** The background service (daemon) running on the host machine that manages images, containers, networks, and storage volumes.
- **Docker Hub:** The public registry where developers share pre-built docker images (like official Node.js, PostgreSQL, or Nginx images).

# Workflow
The workflow of writing, building, and running a Docker container:

```text
Step 1: Write a `Dockerfile` defining your application's recipe.
                             ↓
Step 2: Build the read-only blueprint image: `docker build -t my-app:v1 .`
                             ↓
Step 3: Docker executes the Dockerfile line-by-line, creating cached image layers.
                             ↓
Step 4: Start the active container: `docker run -d -p 80:3000 my-app:v1`.
                             ↓
Step 5: The Docker Engine allocates Linux namespaces and cgroups, booting the container process in milliseconds.
                             ↓
Step 6: The application is live and accessible on Port 80.
```

# Real World Examples
Think of Docker as **standard shipping containers** and virtual machines as **leasing entire transport ships**.
- **Without Docker (Direct Install):** You want to ship bananas, furniture, and cars. You pile them all together on the deck of a boat. The bananas get squished by the cars, the furniture gets wet, and the ship's crew doesn't know how to lash them down because every item is shaped differently.
- **Virtual Machines:** To transport a single crate of bananas safely, you buy a whole separate miniature boat, place the crate inside it, and float the small boat inside the cargo hold of the main ship. It is safe, but it wastes massive space and weight.
- **Docker Containers:** You pack the bananas into a standardized metal shipping container (**Docker Container**). The container has standard dimensions and standard locking corner castings. It is completely isolated from the cars in the next container. The shipping crane (**Docker Engine**) doesn't care what is inside the container; it only cares that it matches standard container dimensions, allowing it to load, stack, and move them instantly.
- **Image vs. Container:**
  - **Image:** The factory mold used to stamp out standard metal shipping containers.
  - **Container:** The actual physical metal box loaded with cargo.

# Implementation
Here is how developers write a `Dockerfile` to containerize a Node.js application, and the CLI commands used to run it:

### 1. Writing the Dockerfile (`Dockerfile`)
Create this file in the root of your Node.js application:

```dockerfile
# Step 1: Start from the official lightweight Node.js base image
FROM node:20-alpine

# Step 2: Set the working directory inside the container
WORKDIR /usr/src/app

# Step 3: Copy package files first to leverage Docker layer caching
COPY package*.json ./

# Step 4: Install dependencies
RUN npm ci --only=production

# Step 5: Copy the rest of the application source code files
COPY . .

# Step 6: Expose the port the app listens on
EXPOSE 3000

# Step 7: Define the command to start the application process
CMD ["node", "server.js"]
```

### 2. Building and Operating the Container via CLI
```bash
# Build the Docker image from the Dockerfile in the current directory (.)
docker build -t my-web-app:1.0 .

# List all local Docker images stored on your machine
docker images

# Run the container in detached background mode (-d), mapping port 80 on your host to port 3000 in the container
docker run -d -p 80:3000 --name active-web-app my-web-app:1.0

# View all running Docker containers
docker ps

# Stream application logs from the running container
docker logs active-web-app

# Stop and delete the running container
docker stop active-web-app
docker rm active-web-app
```

# Best Practices
- **Utilize Layer Caching Efficiently:** Always copy `package.json` files and run dependency installs (`RUN npm install`) *before* copying the rest of your application code (`COPY . .`). Because source code changes frequently, copying it first invalidates the cache, forcing Docker to re-download all dependencies on every single build.
- **Use `.dockerignore`:** Create a `.dockerignore` file in your project root to prevent copying local `node_modules`, log files, or secret `.env` files into the Docker image, keeping your built image sizes small.
- **Cater to Single Responsibility:** Design each container to run one single process (e.g. run your Node app in one container, your database in a second container, and Redis in a third), rather than trying to squeeze multiple services into a single container.

# Industry Standards
Kubernetes and modern cloud deployment platforms require applications to be packaged as container images. The Open Container Initiative (OCI) defines standard specifications for container formats and runtimes to guarantee compatibility across different cloud vendors.

# Common Mistakes
- **Confusing Container Storage with Persistent Storage:** Storing user files or database data directly inside a running container's local directory without configuring a **Docker Volume**. When a container is stopped or deleted, its local storage is destroyed, resulting in permanent data loss.
- **Creating Gigabyte-Sized Images:** Using heavy default base images (like `FROM ubuntu` or `FROM node:latest`). These contain unnecessary tools, increasing image sizes to 1GB. Always use lightweight Alpine Linux base images (like `node:20-alpine`) which keep image sizes under 150MB.
- **Running Containers as Root:** Leaving the default root user active inside the container. If an attacker exploits a vulnerability in your code, they gain root access inside the container and can attempt to break out to hack the host server. Always configure a non-root user.

# Security & Performance Considerations
- **Container Breakouts:** Although containers are isolated, they share the host OS kernel. If the host kernel has a vulnerability, a privileged container can exploit it to escape isolation and gain control of the host machine. Never run containers with the `--privileged` flag in production.
- **Port Mapping Collisions:** Attempting to bind two different containers to the same port on the host machine (e.g. running two containers mapped to `-p 80:3000`). The Docker Engine will throw a network conflict error.

# Related Technologies
- **Docker Compose:** A tool used to run and coordinate multi-container Docker applications using a single YAML file.
- **Podman:** An alternative, daemonless container engine that runs OCI containers without requiring a background root service.

# Summary

## What We Learned
- Docker packages software and dependencies into isolated container units that run identically on any machine.
- Containers share the host OS kernel using namespaces and cgroups, making them lighter and faster than Virtual Machines.
- Dockerfiles define step-by-step image blueprints made of cached layers, which are instantiated as active containers.

## Key Takeaways
- Structure your Dockerfiles to copy package files and install dependencies before copying source code to maximize caching speeds.
- Configure Docker Volumes to store persistent data (like database folders) safely outside the container lifecycle.
- Use alpine-based base images and `.dockerignore` files to keep built container sizes minimal.

# Keywords
- Docker
- Containerization
- Docker Image
- Docker Container
- Dockerfile
- Namespaces
- Control Groups (Cgroups)
- Layer Caching
- Volume
- Docker Daemon

# Glossary

| Term | Meaning |
|---|---|
| Image | A read-only template containing the code, environment, and libraries needed to run a container. |
| Container | A running, isolated instance of a Docker image operating as a process on the host machine. |
| Dockerfile | A text file containing sequential instructions used to build a Docker image. |
| Volume | A storage directory mapped from the host machine into the container to persist data. |
| Namespace | A Linux kernel feature that isolates system resources (networks, file structures, processes) for a container. |
| Cgroup | Control Group; a kernel feature that limits CPU, memory, and disk usage limits for a container process. |

## Next Recommended Chapters
- 06-Containerization.md
