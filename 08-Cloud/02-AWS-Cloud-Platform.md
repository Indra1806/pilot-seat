> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Amazon Web Services (AWS)** is the world’s most comprehensive and widely adopted cloud computing platform. It provides on-demand infrastructure resources—such as virtual servers, data storage, networking channels, databases, security keys, and artificial intelligence tools—over the internet under a metered pay-as-you-go pricing model.

# Why It Exists
Historically, building a web application required companies to buy physical server hardware, wait weeks for it to ship, install it in a local server room, configure routers and network switches, and hire dedicated operations personnel to manage hardware issues and power supplies. AWS was created in 2006 to turn this capital-intensive process into a remote software utility. By renting virtual infrastructure instantly, developers and companies could focus their time and resources entirely on writing code and building applications.

# Problem It Solves
AWS solves the issues of high starting costs, slow scaling pipelines, hardware replacement chores, and single-point-of-failure hosting risks.

### Before AWS (Traditional Server Ownership):
- Launching a simple web app required spending thousands of dollars upfront to purchase server hardware that might never be fully utilized.
- If a server hard drive failed, an engineer had to travel to the datacenter to unplug the broken drive and slide in a new one.
- Handling traffic spikes during shopping sales required buying extra servers that sat idle and wasted for the rest of the year.

### After AWS (On-Demand Cloud Resources):
- Developers run applications on virtual servers in minutes, paying only for the exact seconds the server remains active.
- AWS manages physical infrastructure maintenance, fiber optic networks, backup generators, and server hardware repairs globally.
- Auto-scaling rules automatically deploy new virtual servers during traffic surges and destroy them when traffic subsides to save money.

# Core Concepts
To design applications in AWS, you must understand its core regional structure and primary service categories:

1. **Regions and Availability Zones (AZs):** AWS operates globally in geographic locations called **Regions** (e.g. US East or Ireland). Each Region contains multiple, isolated datacenters called **Availability Zones (AZs)**. AZs are connected via low-latency fiber cables. Running servers in multiple AZs ensures that if one datacenter loses power, your application stays online in another.
2. **Compute Services:** Runtimes for running application code:
   - **Amazon EC2 (Elastic Compute Cloud):** Virtual servers that give you full operating system control.
   - **AWS Lambda (Serverless Compute):** Runs code snippets in response to triggers (like file uploads or API requests) without configuring any servers, scaling down to zero when idle.
3. **Storage Services:** Environments for storing files and data:
   - **Amazon S3 (Simple Storage Service):** Unlimited object storage for files (images, backups, static websites) accessible via web URLs.
   - **Amazon EBS (Elastic Block Store):** High-performance virtual hard drives attached directly to individual EC2 servers.
4. **Networking Services:** Tools to connect and secure resources:
   - **Amazon VPC (Virtual Private Cloud):** An isolated, private network boundary you define to keep databases hidden from the public internet.
   - **Amazon Route 53:** A highly available Domain Name System (DNS) web service that translates web addresses (like `example.com`) into computer IP addresses.
   - **Elastic Load Balancing (ELB):** A device that distributes incoming user traffic across multiple backend virtual servers to prevent overloading.
5. **Database Services:** Managed storage for structured data:
   - **Amazon RDS (Relational Database Service):** Automatically configures, backs up, and patches SQL databases (PostgreSQL, MySQL).
   - **Amazon DynamoDB:** A highly scalable, serverless NoSQL database designed for fast, millisecond responses.
6. **AWS Identity and Access Management (IAM):** The security gatekeeper that controls which users and services have permission to create, access, or delete AWS resources.

# Architecture / Components
A typical production architecture for hosting a high-availability web application on AWS:

```text
  [ Users on the Internet ]
             │
             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Amazon Route 53 (DNS lookup)                           │
  └──────────────────────────┬─────────────────────────────┘
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Amazon CloudFront (CDN Caching)                        │
  └──────────────────────────┬─────────────────────────────┘
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ VPC boundary                                           │
  │                                                        │
  │  Public Subnets (Facing Internet)                      │
  │   [ Application Load Balancer ]                        │
  │        │                                               │
  │        ├─── (Distributes traffic)                      │
  │        ▼                                               │
  │  Private Subnets (Secure Zone)                         │
  │    Availability Zone A        Availability Zone B      │
  │    [ Amazon EC2 Instance ]    [ Amazon EC2 Instance ]  │
  │        │                          │                    │
  │        └─── (Reads/Writes) ───────┼───┐                │
  │                                   │   │                │
  │    Database Subnet                ▼   ▼                │
  │    [ Amazon RDS Primary ] ◄──► [ Amazon RDS Standby ]  │
  └────────────────────────────────────────────────────────┘
```

# Workflow
How a user request traverses AWS components to fetch data from a web application:

```text
Step 1: User types `my-app.com` in their browser, prompting Route 53 to return the public IP address of the load balancer.
                             ↓
Step 2: Traffic hits CloudFront (CDN) to check if static assets (like images) can be served from local edge caches.
                             ↓
Step 3: If not cached, the request goes to the Application Load Balancer (ALB) inside the public subnet of the VPC.
                             ↓
Step 4: The ALB forwards the traffic to an EC2 instance running in a secure, private subnet in Availability Zone A.
                             ↓
Step 5: The EC2 instance processes the request, authenticating via an IAM Instance Profile role to query database records.
                             ↓
Step 6: The EC2 instance retrieves data from the Amazon RDS database and writes a log file to an S3 backup bucket.
                             ↓
Step 7: The EC2 instance returns the response to the user via the Load Balancer, and the browser displays the web page.
```

# Real World Examples
Think of AWS components as **a massive global amusement park infrastructure**.
- **Regions (Theme Park Locations):** AWS has theme parks worldwide (Orlando, Paris, Tokyo). You choose the closest location to your visitors so they do not have to travel far.
- **Availability Zones (Themed Lands):** Within the Orlando park, you have separate lands (Fantasyland, Adventureland) running on separate power grids. If a power outage hits Fantasyland (AZ failure), the rides in Adventureland remain fully open.
- **EC2 (Renting a food truck):** You pay a daily lease for a food truck. You must bring your own chefs, cooking fuel, and ingredients (operating system, software code, dependencies), but you can drive it anywhere you want.
- **S3 (Renting an infinite storage locker):** You drop boxes, bags, and items of any size into a giant warehouse. You do not worry about how they fit; the staff hands you a ticket number for each box, and you use that ticket to retrieve it instantly.
- **RDS (Hiring a database manager):** Instead of sweeping the floor and backing up the inventory sheets yourself, you hire a manager who automatically organizes your ingredients, cleans the kitchen, and saves backup copies in a safe every night.
- **VPC (Park Security Fence):** A fence separating the public visitor walkways (Public Subnet) from the employee-only kitchens and cash vaults (Private Subnet).
- **IAM (Staff ID Badge System):** Each employee receives a badge. The ticket collector cannot open the cash vault, and the accountant cannot drive the food truck. Everyone has access only to what they need to do their job (Least Privilege).

# Implementation
Here is how developers use the AWS Command-Line Interface (CLI) to configure their credentials, create a secure S3 storage bucket, and deploy a virtual machine (EC2):

### Interacting with AWS via CLI
```bash
# 1. Configure the CLI with your IAM Access Keys and default Region
aws configure
# Inputs prompted:
# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: us-east-1
# Default output format [None]: json

# 2. Create a globally unique Amazon S3 bucket to store image files
aws s3 mb s3://my-unique-application-assets-bucket-9039

# 3. Upload a file to the S3 bucket
aws s3 cp profile-picture.jpg s3://my-unique-application-assets-bucket-9039/profile-picture.jpg

# 4. Launch a single EC2 virtual machine instance running Ubuntu Server
aws ec2 run-instances \
  --image-id ami-0c7217cdde317cfec \
  --count 1 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-08e1e721 \
  --subnet-id subnet-091a104f
```

# Best Practices
- **Enforce IAM Least Privilege:** Never share root access keys or assign full admin permissions to services. Create narrow, specific IAM policies that only allow the exact actions needed (e.g. allowing an EC2 server to read from S3 but not delete anything).
- **Keep Databases in Private Subnets:** Never assign public IP addresses to your Amazon RDS or DynamoDB databases. Keep them isolated in private VPC subnets and only allow network access from your application servers via Security Groups.
- **Enable CloudTrail and Billing Alarms:** Always enable AWS CloudTrail (which records every single API call made inside your account for audit logs) and set up AWS Budgets to alert you before costs cross your monthly limit.

# Industry Standards
Modern architectures follow the **AWS Well-Architected Framework**. It organizes best design practices across six operational pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and Sustainability. Organizations run automated scans (using tools like AWS Security Hub) to check if their infrastructure matches these standards.

# Common Mistakes
- **Committing IAM Keys to GitHub:** Accidentally hardcoding IAM user access keys into configuration files and committing them to public Git repositories. Automated bots scan GitHub continuously to steal these keys within seconds, launching expensive crypto-mining servers on your bill. Always use **IAM Roles** for servers instead of hardcoded keys.
- **Using default VPC subnets for production:** Deploying all applications in the default public VPC subnets, exposing databases and system controls directly to internet attacks.
- **Ignoring unused EBS volumes:** Deleting an EC2 virtual machine but forgetting to delete its attached EBS storage disks. The unused storage volumes continue to generate charges indefinitely.

# Security & Performance Considerations
- **Data Encryption Management:** Always check the encryption boxes when creating S3 buckets or EBS volumes. AWS uses the **Key Management Service (KMS)** to handle encryption keys, protecting your stored data from unauthorized physical access inside the datacenter.
- **VPC Security Groups vs Network ACLs:** Security Groups act as a firewall for individual servers (stateful: if you allow incoming traffic, outgoing response traffic is allowed automatically). Network ACLs act as firewalls for entire subnets (stateless: you must define both incoming and outgoing rules).

# Related Technologies
- **Terraform:** Infrastructure as Code utility commonly used to deploy AWS resources.
- **LocalStack:** An open-source emulator that allows developers to run mock AWS services (like S3 and Lambda) locally on their own computers without spending money.
- **MinIO:** A self-hosted object storage server compatible with the AWS S3 API.

# Summary

## What We Learned
- AWS provides on-demand virtual computing resources, removing the need for physical datacenter investments.
- Global deployments are structured around Regions (geographic zones) and Availability Zones (physically isolated datacenters).
- Compute (EC2, Lambda), Storage (S3, EBS), Databases (RDS), and Networking (VPC) form the core building blocks of cloud systems.
- IAM controls resource permissions, ensuring all services operate under restricted authorization profiles.

## Key Takeaways
- Isolate database layers inside private subnets and restrict connection routes using Security Groups.
- Use IAM roles and instance profiles rather than hardcoding static access keys inside code files.
- Design for infrastructure failures by replicating applications across multiple Availability Zones.

# Keywords
- AWS
- Region
- Availability Zone
- Amazon EC2
- Amazon S3
- Amazon RDS
- Amazon VPC
- AWS IAM
- Security Group
- AWS Lambda

# Glossary

| Term | Meaning |
|---|---|
| Region | A geographic area containing multiple isolated Availability Zones connected by high-speed fiber. |
| Availability Zone (AZ) | One or more discrete physical datacenters with redundant power, networking, and cooling in a Region. |
| Amazon EC2 | Virtual server instances used to run application runtimes in the cloud. |
| Amazon S3 | Simple Storage Service; a scalable, web-based object storage service for files and assets. |
| Amazon VPC | A private, isolated network space created within your AWS account. |
| IAM Role | A temporary credential container used to grant AWS service access without static access keys. |

## Next Recommended Chapters
- 03-GCP-Cloud-Platform.md
