> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Cloud Networking and VPCs** (Virtual Private Clouds) form the software-defined networking infrastructure of the cloud. They allow developers to create private, isolated network partitions within a cloud account, define IP address ranges, divide networks into public and private subnets, configure routing paths, and apply network-level firewall security rules.

# Why It Exists
In early public cloud computing, every virtual machine was launched with a public IP address and sat directly on the public internet. This meant backend code servers and database systems were exposed to anyone scanning the web for open ports. To secure systems, engineers had to configure complex software firewalls on every individual machine. Virtual Private Clouds were invented to bring the security of physical, fenced-off corporate datacenters into virtual software. By enclosing resources inside a VPC, companies can hide database servers from the internet, control exactly which paths data can travel, and restrict public access to entry points like load balancers.

# Problem It Solves
Cloud networking and VPCs solve the problems of public server exposure, database hacking risks, uncontrolled resource-to-resource traffic, and complex networking hardware maintenance.

### Before VPCs (Public Internet Sharing Model):
- Databases sat on public IP addresses. Attackers attempted to brute-force database passwords continuously over the internet.
- Setting up a secure network required purchasing physical routers, wiring hardware switches, and configuring firewalls manually.
- Connecting remote office networks to cloud servers required routing traffic over the insecure public internet, exposing data to interception.

### After VPCs (Isolated Private Networks):
- Databases are placed in private subnets with no routing pathways to or from the internet, making them invisible to external scanners.
- Networks are constructed virtually in seconds using code (IaC), deploying subnets and load balancers automatically.
- Dedicated encryption tunnels (VPNs) link on-premise offices directly to the private cloud network, securing corporate data transit.

# Core Concepts
Designing cloud networks requires mastering subnets, routing rules, and security gateways:

1. **Virtual Private Cloud (VPC / VCN):** A private network boundary dedicated to your cloud account. It uses a **CIDR Block** (Classless Inter-Domain Routing, e.g. `10.0.0.0/16`) to define the range of private IP addresses available inside the network.
2. **Subnets (Public vs. Private):** Subnets are divisions of a VPC's IP address range:
   - **Public Subnet:** Connected to an **Internet Gateway (IGW)**, giving resources public IP addresses. Used for load balancers and public web servers.
   - **Private Subnet:** Isolated from the internet, using private IP addresses (e.g. `10.0.2.0/24`). Used for databases and backend microservices.
3. **NAT Gateway (Network Address Translation):** A device placed in the public subnet. It allows servers inside a private subnet to connect outbound to the internet (to download software packages or pull updates) while blocking outsiders from initiating connections back to those servers.
4. **Route Tables:** The virtual routing maps of your VPC. They contain rules (routes) that determine where network packets are directed based on their destination IP address.
5. **Security Groups vs. Network ACLs (NACLs):**
   - **Security Groups:** Stateful firewalls attached to individual servers (e.g. EC2 instances). If you allow an incoming request, the response is allowed out automatically.
   - **Network ACLs:** Stateless firewalls guarding the boundary of an entire subnet. You must write explicit rules for both incoming and outgoing traffic.

# Architecture / Components
The flow of network traffic through public and private subnets inside a VPC:

```text
  [ User Traffic on the Internet ]
                 │
                 ▼
  ┌────────────────────────────────────────────────────────┐
  │ VPC boundary (10.0.0.0/16)                             │
  │                                                        │
  │  Internet Gateway (IGW) - Entrance gate                │
  │              │                                         │
  │              ▼                                         │
  │  Public Subnet (10.0.1.0/24)                           │
  │   [ Public Load Balancer ] ◄──┐                        │
  │   [ NAT Gateway ] ◄───────────┼───────────┐            │
  │              │                │           │            │
  │              ▼                │           │            │
  │  Private Subnet (10.0.2.0/24) │           │            │
  │   [ App Server VM ] ──────────┘           │            │
  │     (Allows inbound from Load Balancer)   │            │
  │     (Sends outbound patches to NAT) ──────┘            │
  │              │                                         │
  │              ▼                                         │
  │  Database Subnet (10.0.3.0/24)                         │
  │   [ Database Server (Private IP: 10.0.3.15) ]          │
  │     (Allows inbound only from App Server Group)        │
  └────────────────────────────────────────────────────────┘
```

# Workflow
How a network packet travels securely from a user's browser to a private database and back:

```text
Step 1: User sends an HTTP request. The packet arrives at the VPC's Internet Gateway.
                             ↓
Step 2: The public Route Table directs the packet to the public subnet containing the Load Balancer.
                             ↓
Step 3: The Load Balancer checks its rules and forwards the packet into the private subnet, targeting the App Server VM.
                             ↓
Step 4: The subnet boundary's Network ACL allows the packet, and the App Server's Security Group verifies it is on port 8080.
                             ↓
Step 5: The App Server processes the query and connects to the Database Server in the Database Subnet on port 5432.
                             ↓
Step 6: The Database Security Group allows the request because it originates from the App Server's Security Group profile.
                             ↓
Step 7: If the Database needs an OS update, it sends a packet to the NAT Gateway, which routes it out to the internet.
```

# Real World Examples
Think of Cloud VPC components as **a secure corporate office headquarters**.
- **VPC (The office perimeter wall):** A wall built around the entire company property. It keeps outsiders from walking onto the lawn.
- **Public Subnet (The building lobby):** The reception area. Anyone can walk through the front door from the street, register, and talk to the receptionist. The lobby has windows and a clear entry gate.
- **Private Subnet (The security vaults and boardrooms):** The inner hallways behind heavy badge-locked doors. There are no windows facing the street, and visitors cannot walk in without an escort badge.
- **Internet Gateway (The front revolving door):** The door letting people walk into and out of the lobby.
- **NAT Gateway (The mail chute):** A clerk in the cash vault needs to mail a letter out. They drop it in a one-way mail slot. The mail goes out, and responses are returned through the slot, but a thief on the street cannot crawl up the slot to get inside the vault.
- **Security Group (The office door guard):** A guard standing at the door of the server room. They check IDs and only let in employees who work in the IT department.

# Implementation
Here is how developers use Terraform (Infrastructure as Code) to write configurations for a secure VPC network containing one public subnet, one private subnet, and an Internet Gateway:

### Defining a VPC in Terraform (Declarative IaC)
```hcl
# 1. Define the main VPC network box
resource "aws_vpc" "main_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags = {
    Name = "production-vpc"
  }
}

# 2. Create the Internet Gateway to allow public traffic
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id
  tags = {
    Name = "main-igw"
  }
}

# 3. Define the Public Subnet (Lobby zone)
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true # Gives booted servers public IPs
  tags = {
    Name = "public-subnet-1a"
  }
}

# 4. Define the Private Subnet (Vault zone)
resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.main_vpc.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "private-subnet-1a"
  }
}

# 5. Create a Route Table directing public subnet traffic to the Internet Gateway
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id
  route {
    cidr_block = "0.0.0.0/0" # Match all internet traffic
    gateway_id = aws_internet_gateway.igw.id
  }
}
```

# Best Practices
- **Plan IP Address CIDR Ranges Carefully:** Choose CIDR ranges (like `10.0.0.0/16`) that do not overlap with your corporate office network or other cloud networks. If you choose overlapping IP ranges, you will not be able to peer the networks together later.
- **Never Put Databases in Public Subnets:** Keep your database engines and private servers isolated. Put them in private subnets with no route to the internet gateway. Allow database access only from your application servers using security group rules.
- **Use Security Groups over Network ACLs:** Security Groups are stateful and easier to manage because you only need to define inbound rules. Reserve Network ACLs for block-level protection (e.g. blocking traffic from a compromised IP range at the subnet boundary).

# Industry Standards
Modern multi-application cloud setups utilize **Hub-and-Spoke Topologies**. A central VPC (the Hub) hosts shared services like internet firewalls, VPN connections, and active directory systems. Individual application networks (the Spokes) peer directly with the Hub but are isolated from each other. A central **Transit Gateway** coordinates routing among all networks, enforcing enterprise security policies.

# Common Mistakes
- **Exposing SSH Ports to the World:** Setting a Security Group rule that allows SSH port `22` access from `0.0.0.0/0` (any IP address). Attackers run automated scripts to scan the web for open SSH ports, attempting brute-force logins continuously. Limit SSH access to your company's office IP address range.
- **Overlooking NAT Gateway Costs:** Deploying multiple NAT Gateways across test environments. NAT Gateways are billed hourly by cloud providers in addition to data processing fees. Shut down NAT Gateways in testing environments when they are not actively needed.
- **overlapping IP allocations:** Allocating the same subnet range in separate zones, causing configuration conflicts when setting up server clusters.

# Security & Performance Considerations
- **VPC Flow Logs:** Enable VPC Flow Logs to record metadata about all IP traffic entering and leaving your network interfaces. This data is critical for debugging connection failures and identifying network attacks.
- **NAT Gateway Bandwidth Limits:** A single NAT Gateway scales automatically up to 45 Gbps of throughput. If your private servers upload massive database logs or datasets continuously, they can saturate the NAT Gateway, causing other servers to suffer connection delays. Use VPC Endpoints to connect privately to storage without using NAT resources.

# Related Technologies
- **VPN Connection (Virtual Private Network):** An encrypted tunnel used to link local computer networks to cloud networks over the internet.
- **Transit Gateway:** An AWS service that acts as a central cloud router connecting multiple VPCs and on-premise networks.
- **VPC Endpoint:** A private network link that connects resources to cloud services (like S3) using Google or Amazon's private network instead of routing traffic over the public internet.

# Summary

## What We Learned
- VPCs create private software-defined networks, isolating cloud resources from public internet hazards.
- Public subnets host internet-facing resources (load balancers), while private subnets isolate backend servers and databases.
- NAT Gateways let private servers connect outbound to the internet while blocking incoming connection attempts.
- Route tables direct data packets, while stateful Security Groups act as firewalls at the server level.

## Key Takeaways
- Keep databases in private subnets and restrict connection routes using Security Groups.
- Plan non-overlapping CIDR IP blocks to prevent routing issues when peering networks.
- Turn on VPC Flow Logs to monitor and audit network traffic for security compliance.

# Keywords
- VPC
- Subnet
- Internet Gateway
- NAT Gateway
- Route Table
- Security Group
- CIDR Block
- VPC Peering
- Transit Gateway
- Flow Logs

# Glossary

| Term | Meaning |
|---|---|
| VPC | Virtual Private Cloud; an isolated, private virtual network created within a cloud provider account. |
| CIDR Block | Classless Inter-Domain Routing; a method of defining IP address allocation ranges. |
| Public Subnet | A network division with a direct route to an Internet Gateway, allowing public IP assignments. |
| Private Subnet | A network division isolated from the internet, containing resources using private IP addresses. |
| NAT Gateway | A network device letting private resources send data outbound while blocking inbound connections. |
| Security Group | A stateful virtual firewall that controls incoming and outgoing traffic for individual server instances. |

## Next Recommended Chapters
- 11-Cloud-Security-and-Compliance.md
