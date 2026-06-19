> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Microsoft Azure** is an enterprise-grade cloud computing platform developed by Microsoft. It offers a comprehensive catalog of cloud services, including virtual computing, storage systems, private networks, databases, analytics tools, and machine learning models. Azure holds a dominant position in corporate environments due to its seamless integration with Windows Server, the .NET development framework, and Microsoft Entra ID (formerly Active Directory).

# Why It Exists
Historically, most corporate offices hosted their own infrastructure using Windows Active Directory to log employees into local computer networks and running .NET business applications on physical Windows servers in local server closets. When cloud computing emerged, these companies needed a path to migrate to the cloud without rewriting their entire software catalog or rebuilding their security employee directories. Microsoft created Azure in 2008 to bridge this gap, offering a hybrid cloud platform that integrates directly with existing on-premise Windows infrastructure.

# Problem It Solves
Azure solves the challenges of complex corporate network migrations, user security directory synchronization, legacy Windows Server hosting, and compliance auditing.

### Before Azure (Enterprise On-Premise Silos):
- Syncing user logins between physical corporate offices and external cloud applications was complex, requiring separate passwords for every service.
- Migrating large .NET applications to virtualized systems required completely rewriting database connection layers and host server configurations.
- Corporate compliance audits required manually checking physical hardware logs, disk security stamps, and office room access sheets.

### After Azure (Unified Enterprise Cloud):
- Employees log into cloud systems, databases, and virtual machines using their existing corporate Windows login credentials (via Microsoft Entra ID).
- Legacy .NET applications are deployed to managed cloud runtimes with a single click from Visual Studio, maintaining configuration compatibility.
- Automated security auditors track cloud configurations in real-time, providing immediate compliance stamps matching global standards (ISO, SOC).

# Core Concepts
Azure designs center around management boundaries, enterprise security directories, and specialized resource groups:

1. **Resource Groups:** Logical containers where Azure resources (like virtual machines, database servers, and networks) are deployed and managed as a single unit. If you delete a Resource Group, Azure automatically cleans up and deletes all resources inside it.
2. **Microsoft Entra ID (formerly Azure Active Directory):** A cloud-based identity and access management service. It manages user logins, corporate groups, and application access permissions, serving as the central authentication authority for the entire system.
3. **Managed Identities:** A feature that gives an Azure service (like a virtual machine) an automatically managed identity in Entra ID. This allows the service to authenticate to other secure services (like Azure Key Vault) without developers ever having to write or store passwords in code.
4. **Core Compute Services:**
   - **Azure Virtual Machines:** On-demand Windows and Linux virtual servers.
   - **Azure App Service:** A managed PaaS environment for hosting web applications and APIs, handling scaling and OS patching automatically.
   - **Azure Functions:** Serverless, event-driven code blocks that run when triggered (e.g. database updates, HTTP webhooks).
5. **Storage and Database Services:**
   - **Azure Blob Storage:** Object storage designed to store massive amounts of unstructured file data (images, video streams, backups).
   - **Azure SQL Database:** A fully managed relational database engine based on Microsoft SQL Server.
   - **Azure Cosmos DB:** A globally distributed, multi-model NoSQL database designed for fast response times.
6. **Azure Virtual Network (VNet):** A private, isolated network where Azure resources communicate securely with each other and connect back to physical on-premise datacenters.

# Architecture / Components
A typical enterprise hybrid hosting architecture on Microsoft Azure:

```text
  [ Corporate Office (On-Premise) ]        [ Remote Web Users ]
             │                                     │
     (ExpressRoute VPN Link)                       ▼
             │                        ┌────────────────────────────┐
             ▼                        │ Azure Front Door (Router)  │
  ┌───────────────────────────────────┴────────────────────────────┘
  │ Azure VNet Boundary                                            │
  │                                                                │
  │  Public Access Gateway                                         │
  │   [ Azure App Service (Hosting Web Portal API) ]               │
  │        │                                                       │
  │        ├─── (Authenticates securely via)                       │
  │        ▼                                                       │
  │  Security Vault & Identity Core                                │
  │   [ Azure Key Vault (Keys) ] ◄──► [ Microsoft Entra ID ]       │
  │        │                                                       │
  │        ├─── (Queries relational data)                          │
  │        ▼                                                       │
  │  Private Subnet (No Internet access)                           │
  │   [ Azure SQL Database ] ◄──► [ Blob Storage (Media Files) ]   │
  └────────────────────────────────────────────────────────────────┘
```

# Workflow
How a user authenticates and accesses secure business data on Azure:

```text
Step 1: User visits a corporate web portal, which redirects them to Microsoft Entra ID to enter their credentials.
                             ↓
Step 2: Entra ID validates the login, confirms multi-factor authorization, and issues an access token.
                             ↓
Step 3: The user is routed back to the Azure App Service web frontend with the valid token.
                             ↓
Step 4: The App Service uses a system-assigned Managed Identity to request access keys from Azure Key Vault.
                             ↓
Step 5: Key Vault verifies the Managed Identity and returns the connection string for the database.
                             ↓
Step 6: The App Service queries the Azure SQL Database to fetch the user's company profile.
                             ↓
Step 7: The app retrieves document files from Azure Blob Storage and renders the portal screen.
```

# Real World Examples
Think of Azure components as **a secure corporate office park building**.
- **Resource Groups (Assigned Office Suites):** Think of Resource Groups as individual rented office suites. Suite 101 is the Accounting suite; it contains desks, filing cabinets, and computers. When the lease ends, cleaners wipe the entire Suite 101 clean in one go (deleting the Resource Group deletes all resources inside).
- **Microsoft Entra ID (Receptionist Security Badge Office):** The security desk at the building entrance. Everyone (employees, applications, servers) must show ID and receive a color-coded security badge detailing exactly which floors and offices they are allowed to enter.
- **Managed Identities (Automated Smart Badges):** Instead of giving an automated coffee machine a physical key to the storeroom (passwords in code), you put a smart chip on the machine itself. The door scanner checks the machine's chip and lets it in.
- **Azure VM (Empty cubicle):** You lease an empty office desk. You must install your own computer tower, configure the operating system, and clean the dust.
- **Azure App Service (Serviced cubicle):** You rent a desk that comes with a pre-configured PC, working printer, and daily cleaning staff. You just sit down and start work.
- **Azure Blob Storage (Central document archives):** An infinite filing archive room where workers toss folders, drawings, and logs. You do not care which cabinet they reside in; you fetch them using the tab name on the folder.

# Implementation
Here is how developers use the Azure Command-Line Interface (`az` CLI) to log in, create a Resource Group, spin up a secure Azure Key Vault, and deploy a Virtual Machine:

### Provisioning Resources via Azure CLI
```bash
# 1. Log into your Microsoft Azure account
az login

# 2. Create a Resource Group to organize resources in the East US region
az group create \
  --name my-corporate-rg \
  --location eastus

# 3. Create a secure Azure Key Vault to store database connection strings
az keyvault create \
  --name "my-corp-vault-9029" \
  --resource-group my-corporate-rg \
  --location eastus

# 4. Save a secret password inside the Key Vault
az keyvault secret set \
  --vault-name "my-corp-vault-9029" \
  --name "DbConnectionString" \
  --value "Server=tcp:sqlserver.database.windows.net;Database=DB;"

# 5. Create an Azure Virtual Machine running Linux Ubuntu
az vm create \
  --resource-group my-corporate-rg \
  --name webServerVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys
```

# Best Practices
- **Organize with Resource Groups:** Never deploy resources at random. Group all resources related to a specific project into a single Resource Group. Use tags (e.g. `environment: production`, `owner: finance`) to calculate billing costs easily.
- **Avoid Storing Keys in Code (Use Key Vault):** Never write API credentials or database strings inside your application settings. Store them inside Azure Key Vault and fetch them dynamically using Azure Managed Identity.
- **Check Azure Advisor Weekly:** Regularly review Azure Advisor in the portal dashboard. It scans your running infrastructure and outputs recommendations for improving security, reliability, performance, and cost savings.

# Industry Standards
Large enterprise architectures follow the **Azure Enterprise-Scale Landing Zones**. This standard provides blueprint templates to automatically set up subscriptions, active directory syncs, shared virtual networks (VNets), and corporate firewall rules in a hub-spoke model. This ensures that any new application sub-project launched by a development team instantly complies with the company's security policies.

# Common Mistakes
- **Orphaned Resources Generating Charges:** Deleting a virtual machine but forgetting to delete its attached Managed Disks or Virtual Network Interface cards. These separate storage units continue to generate bills. Always delete the entire Resource Group to clean up everything.
- **Exposing Remote Desktop Protocol (RDP) Ports:** Leaving Windows server virtual machines open with port `3389` accessible to the public internet. Attackers scan for open RDP ports continuously, attempting brute-force logins. Protect VMs using Azure Bastion.
- **Ignoring Subscription Billing Limits:** Storing development resources, testing databases, and production apps under a single subscription without configured budget alerts, leading to runaway usage costs.

# Security & Performance Considerations
- **Network Security Groups (NSGs):** Protect resources by placing them behind Network Security Groups. NSGs act as virtual firewalls at the network interface level, allowing you to define precise inbound and outbound IP address filters.
- **Azure ExpressRoute:** For companies requiring maximum security and speed, do not connect to Azure over the public internet. Use Azure ExpressRoute to establish a private, dedicated physical fiber connection between your corporate offices and Azure datacenters.

# Related Technologies
- **Bicep:** Microsoft's modern, domain-specific language used to write declarative Infrastructure as Code files for Azure, replacing verbose JSON ARM templates.
- **Azure DevOps:** A complete DevOps platform providing boards, Git code repositories, pipeline builds, and artifact registries.
- **Microsoft Entra ID Domain Services:** Provides managed domain services like group policies and LDAP authentication.

# Summary

## What We Learned
- Microsoft Azure focuses heavily on enterprise systems, hybrid environments, and Microsoft software integrations.
- The platform uses Resource Groups to manage the lifecycle of related resources collectively.
- Entra ID serves as the central security and authentication directory, while Managed Identities remove the need for hardcoded credentials.
- Core building blocks cover Virtual Machines, managed web runtimes (App Service), and scalable relational databases (Azure SQL).

## Key Takeaways
- Delete entire Resource Groups when cleaning up test environments to avoid paying for orphaned disks and networks.
- Store sensitive configuration keys inside Azure Key Vault, retrieving them dynamically via Managed Identity.
- Utilize Azure Advisor to audit cloud environments for security configurations and cost optimization.

# Keywords
- Microsoft Azure
- Resource Group
- Microsoft Entra ID
- Managed Identity
- Azure App Service
- Azure Virtual Machine
- Azure SQL Database
- Azure Key Vault
- VNet
- Bicep

# Glossary

| Term | Meaning |
|---|---|
| Resource Group | A logical folder container in Azure that holds related resources for an application. |
| Microsoft Entra ID | A cloud-based identity service that handles authentication and resource permissions. |
| Managed Identity | An identity created in Entra ID assigned to a resource, allowing secure access to other tools without passwords. |
| Azure SQL Database | A fully managed SQL Server database hosted on Azure cloud. |
| Azure Key Vault | A secure cloud vault used to encrypt, store, and manage keys, passwords, and certificates. |
| Azure Bastion | A managed service providing secure, web-based RDP and SSH access to VMs without public IP exposure. |

## Next Recommended Chapters
- 05-Oracle-Cloud-Platform.md
