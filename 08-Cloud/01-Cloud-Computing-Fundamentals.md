> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Cloud Computing** is the delivery of computing services—including servers, storage, databases, networking, software, and analytics—over the internet ("the cloud") on a pay-as-you-go pricing model. Instead of buying and operating physical computers, users rent virtual computing resources from cloud providers (like Amazon, Google, or Microsoft).

# Why It Exists
Historically, running software required purchasing expensive server racks, renting space in physical datacenters, installing backup generators and cooling systems, and hiring networking specialists to run cables. This process took months, required huge upfront capital investment, and was difficult to scale if user traffic fluctuated. Cloud computing was created to turn physical computer hardware into a utility service, allowing businesses to lease virtual computing infrastructure instantly over the internet, similar to how homes lease electricity and water from municipal grids.

# Problem It Solves
Cloud computing solves high upfront hardware costs, slow infrastructure deployment times, maintenance overhead, and resource wastage.

### Before Cloud Computing (On-Premise Infrastructure):
- Startups spent months raising money just to buy hardware servers before testing their software product.
- If a server's hard drive crashed, database administrators had to drive to the datacenter to physically replace it.
- Companies purchased extra servers to handle peak holiday traffic, leaving those servers sitting idle and wasted during the rest of the year.

### After Cloud Computing (On-Demand Infrastructure):
- Anyone can spin up ten virtual servers globally in under a minute for pennies, paying only for the minutes they are active.
- Hardware repairs, power backup systems, and physical security are managed entirely by the cloud provider.
- Applications scale up automatically during traffic spikes and scale down when traffic drops to minimize hosting bills.

# Core Concepts
Understanding the cloud requires mastering virtualization, service models, and responsibility sharing:

1. **Virtualization:** The foundational technology of cloud computing. It uses software called a **Hypervisor** to split a single physical computer (the host) into multiple separate virtual computers (guests). Each guest virtual machine (VM) runs its own operating system and behaves like independent hardware.
2. **Shared Responsibility Model:** The division of security and operational tasks between the provider and the customer. The provider is responsible for **security "of" the cloud** (protecting datacenters, physical wires, and hypervisors). The customer is responsible for **security "in" the cloud** (configuring firewalls, writing secure code, patching their OS, and protecting data).
3. **Cloud Service Models:**
   - **Infrastructure as a Service (IaaS):** Leasing raw hardware infrastructure (virtual machines, disks, networks). You install and maintain the OS, libraries, and application.
   - **Platform as a Service (PaaS):** Leasing a fully managed environment. The provider handles the OS, networking, database setup, and runtime updates. You only upload your application code.
   - **Software as a Service (SaaS):** Accessing a complete software application over the web. You do not manage any code or server; you simply use the product.
4. **Cloud Deployment Models:**
   - **Public Cloud:** Resources are owned by a provider and shared with other customers (multi-tenancy) over the public internet.
   - **Private Cloud:** Computing resources are used exclusively by one organization, hosted either on-premise or by a dedicated provider.
   - **Hybrid Cloud:** An architecture combining public cloud resources with private clouds or on-premise systems, allowing data and applications to move between them.

# Architecture / Components
The layers of virtualized hardware isolation inside a cloud datacenter:

```text
  ┌────────────────────────────────────────────────────────┐
  │ Customer Web Application Code & Runtimes               │
  ├────────────────────────────────────────────────────────┤
  │ Guest Operating System (e.g. Ubuntu, Windows Server)   │
  ├────────────────────────────────────────────────────────┤
  │ Virtual Machine Layer (Virtual CPU, RAM, Disk)         │
  └──────────────────────────┬─────────────────────────────┘
                             │
                  (Isolated VM Boundaries)
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Hypervisor Software (Splits physical resources)        │
  ├────────────────────────────────────────────────────────┤
  │ Physical Host Server (Datacenter rack CPU, RAM, Disk)  │
  ├────────────────────────────────────────────────────────┤
  │ Cloud Datacenter Infrastructure (Power, Cooling, Net)  │
  └────────────────────────────────────────────────────────┘
```

# Workflow
How a cloud provider provisions a virtual server instance for a developer:

```text
Step 1: Developer requests a virtual machine (e.g. 2 vCPUs, 8GB RAM, Ubuntu OS) via the CLI.
                             ↓
Step 2: The cloud provider's API endpoint validates the request and checks billing limits.
                             ↓
Step 3: The cloud controller finds a physical host server in the target region with spare capacity.
                             ↓
Step 4: The controller instructs the physical host's Hypervisor to allocate 2 CPU cores and 8GB RAM.
                             ↓
Step 5: The Hypervisor clones the requested OS image onto the allocated virtual storage disk.
                             ↓
Step 6: The Virtual Machine boots up, receives a virtual network card and IP address, and opens SSH access.
                             ↓
Step 7: The developer receives the connection details and deploys their application within seconds.
```

# Real World Examples
Think of cloud service models as **different housing arrangements**.
- **On-Premise (Building a house from scratch):** You buy the land, pour the concrete, run the copper wires, install the pipes, and paint the rooms. If the roof leaks, you have to climb up and fix it. You own everything, but it is slow and expensive.
- **IaaS (Renting an unfurnished apartment):** The landlord maintains the building structure, roof, and plumbing (security OF the cloud). But inside, you must buy your own furniture, set up the TV, paint the walls, and clean the floors (security IN the cloud).
- **PaaS (Staying in a fully furnished hotel room):** The hotel provides the bed, TV, light bulbs, electricity, and clean towels. You just bring your luggage, unpack, and sleep. You cannot knock down walls or change the wallpaper, but it is ready instantly.
- **SaaS (Booking an all-inclusive cruise ship cabin):** You do not manage any furniture, cooking, cleaning, or navigation. You just walk aboard and enjoy the trip. Everything is operated and serviced for you.

# Implementation
Here is how developers use command-line interface (CLI) utilities to deploy a basic virtual machine in the cloud instantly. The examples show both AWS and Google Cloud:

### Spinning Up a Virtual Machine (CLI)
```bash
# Example A: Creating an AWS EC2 Virtual Machine
aws ec2 run-instances \
  --image-id ami-0c7217cdde317cfec \
  --count 1 \
  --instance-type t2.micro \
  --key-name my-ssh-key \
  --security-group-ids sg-90392102 \
  --region us-east-1

# Example B: Creating a Google Cloud Compute Engine VM
gcloud compute instances create my-app-server \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --image-project=ubuntu-os-cloud \
  --image-family=ubuntu-2204-lts \
  --metadata=startup-script="echo 'Hello Cloud World' > /index.html"
```

# Best Practices
- **Never Ignore the Shared Responsibility Model:** Never assume the cloud provider secures your databases or updates your operating systems. Check the agreement matrix to confirm which tasks fall under your responsibility.
- **Configure Strict Billing Alerts:** Because cloud platforms scale automatically, a runaway code loop or developer test could create resource spikes, resulting in massive credit card charges. Configure notifications to alert you the moment project costs cross a set limit.
- **Deploy Across Multiple Zones:** Always run application servers in at least two separate Availability Zones (physically separate datacenters). This ensures your application stays online if one datacenter loses power or internet connectivity.

# Industry Standards
Modern enterprises operate on **Multi-Cloud** or **Hybrid Cloud** strategies. Rather than relying on a single vendor, they distribute workloads across AWS, GCP, and Azure. This prevents cloud provider lock-in, increases negotiating power, and ensures that a massive global outage at one provider cannot take down their entire service.

# Common Mistakes
- **Treating Cloud Servers Like Pets:** Naming individual virtual servers, manually modifying their files, and logging in to fix errors. Cloud servers should be treated like "cattle"—generic, replaceable, and managed via automated code. If a server behaves strangely, delete it and spin up a new one automatically.
- **Leaving Unused Resources Running:** Leaving high-capacity test databases or idle servers running over the weekend. Because cloud billing is metered, idle resources continue to consume budget. Always delete or shut down test environments when they are not in use.
- **Storing Secrets in Cloud Images:** Creating custom machine disk templates (images) containing hardcoded passwords or API keys. If these machine images are shared publicly or compromised, attackers can steal your credentials. Use runtime secrets injection instead.

# Security & Performance Considerations
- **Noisy Neighbors:** Because physical host servers are shared with other customers, a virtual machine owned by another company running high CPU workloads could impact the performance of your VM (noisy neighbor effect). Choose dedicated instances or specialized machine classes if performance consistency is critical.
- **Hypervisor Vulnerabilities:** Though extremely rare, bugs in hypervisor software can allow a malicious user to break out of their virtual machine boundaries and access the physical host server or other virtual machines (VM Escape attacks). Providers patch these host systems continuously to mitigate risks.

# Related Technologies
- **KVM (Kernel-based Virtual Machine):** An open-source virtualization technology built into the Linux kernel, powering many cloud environments.
- **VMware ESXi:** A popular enterprise hypervisor software used to build private cloud environments on-premise.
- **Terraform:** Infrastructure as Code utility used to automate the provisioning of cloud VMs and networks.

# Summary

## What We Learned
- Cloud computing rents virtualized infrastructure over the internet, converting physical capital expenses into variable operating costs.
- Virtualization uses hypervisor software to slice single physical hosts into isolated virtual machines.
- The Shared Responsibility Model defines what security aspects the provider manages vs what the customer must secure.
- Service models range from raw infrastructure (IaaS) and managed platforms (PaaS) to complete software suites (SaaS).

## Key Takeaways
- Select the appropriate service model: IaaS for control, PaaS for speed of development, and SaaS for ready-made tools.
- Configure billing warnings on day one of setting up any cloud account to prevent unexpected charges.
- Treat virtual servers as temporary, replaceable resources managed via code, never as permanent physical machines.

# Keywords
- Cloud Computing
- Virtualization
- Hypervisor
- IaaS
- PaaS
- SaaS
- Shared Responsibility Model
- Virtual Machine
- Availability Zone
- Billing Alerts

# Glossary

| Term | Meaning |
|---|---|
| Virtualization | Software that simulates hardware functionality to create multiple virtual systems from a single physical machine. |
| Hypervisor | The controller software that creates, runs, and isolates virtual machines on a physical host server. |
| Shared Responsibility Model | A security framework outlining the separate safety obligations of the cloud provider and the customer. |
| Host Machine | The underlying physical computer server hardware that resources are sliced from. |
| Guest Machine | The virtual machine running on a host server, operating on its own virtualized resource partition. |
| Public Cloud | A cloud environment where infrastructure resources are shared dynamically across multiple separate customer accounts. |

## Next Recommended Chapters
- 02-AWS-Cloud-Platform.md
