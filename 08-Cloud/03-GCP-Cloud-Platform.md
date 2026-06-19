> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Google Cloud Platform (GCP)** is a suite of cloud computing services offered by Google. It runs on the same physical infrastructure that Google uses internally for its consumer products, such as Google Search, YouTube, and Gmail. GCP is particularly renowned for its advanced data analytics engines, container orchestration systems (like Kubernetes), and machine learning frameworks (Vertex AI).

# Why It Exists
Google built massive global network infrastructure, specialized storage units, and distributed database systems to run its own high-traffic search engine and video streaming platforms. GCP was created to open this proprietary, high-speed infrastructure to external developers and businesses. By hosting applications on Google's network, companies can leverage Google's innovations in database design, data processing, and artificial intelligence as pay-as-you-go utility services.

# Problem It Solves
GCP solves the challenges of hosting massive datasets, running highly containerized applications, scaling services dynamically without boot latency, and training complex AI models.

### Before GCP (Traditional Infrastructure Limitations):
- Running big data analytics required purchasing clusters of physical servers, writing complex sorting systems, and waiting hours or days for queries to finish.
- Managing containerized microservices required manual setup of clustering software, load balancers, and container registry servers.
- Launching AI applications required buying expensive graphic card (GPU) processors and manually installing machine learning libraries.

### After GCP (Google-Scale Cloud Utilities):
- Serverless data warehouses scan petabytes of data in seconds, billing only for the bytes read during the query execution.
- Managed Kubernetes clusters boot up automatically, handling networking, container upgrades, and system scaling via Google's control plane.
- AI developers lease powerful machine learning hardware (TPUs and GPUs) instantly, utilizing pre-configured AI pipelines to deploy models in minutes.

# Core Concepts
GCP divides its offerings into project boundaries, global regions, and specialized service clusters:

1. **Resource Hierarchy:** GCP organizes resources using a structured folder directory: **Organization** (the company) -> **Folders** (departments) -> **Projects** (specific app environments, like Development or Production) -> **Resources** (VMs, databases). All billing and access permissions are tied directly to the Project.
2. **Regions and Zones:** A Region is a geographic area (e.g. `us-central1` in Iowa). Each Region contains multiple, separate datacenters called **Zones** (e.g. `us-central1-a`, `us-central1-b`). Splitting applications across Zones protects them from localized datacenter failures.
3. **Compute and Orchestration Services:**
   - **Compute Engine:** On-demand virtual machines with custom CPU and memory sizes.
   - **Cloud Run:** A serverless hosting runtime that runs container images (Docker) automatically, scaling from zero to thousands of instances based on traffic.
   - **Google Kubernetes Engine (GKE):** A managed service that coordinates containerized applications across multiple servers using Kubernetes.
4. **Storage and Databases:**
   - **Cloud Storage:** Unlimited, secure object storage for raw files, divided into storage classes based on access frequency.
   - **Cloud SQL:** Managed relational database hosting for PostgreSQL, MySQL, and SQL Server.
   - **Firestore:** A serverless NoSQL document database designed for real-time mobile and web apps.
   - **BigQuery:** A serverless data warehouse used to run fast analytical queries on massive datasets.
5. **Identity and Access Management (IAM):** A security system that uses **Service Accounts** (virtual identities assigned to applications) and IAM Roles to define exactly what actions services can perform.

# Architecture / Components
A production-ready GCP layout for a containerized microservice application with data analytics:

```text
  [ Users on the Internet ]
             │
             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Cloud DNS (Maps domain to IP)                          │
  └──────────────────────────┬─────────────────────────────┘
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Cloud Load Balancing (Global HTTP(S) Load Balancer)     │
  └──────────────────────────┬─────────────────────────────┘
                             │
                             ▼
  ┌────────────────────────────────────────────────────────┐
  │ Google VPC boundary                                    │
  │                                                        │
  │  Public Access Routing Layer                           │
  │   [ Cloud Run Service (Web Frontend Container) ]       │
  │        │                                               │
  │        ├─── (Queries database)                         │
  │        ▼                                               │
  │  Private Network Zone (Shared VPC)                     │
  │   [ Cloud SQL Relational Database (PostgreSQL) ]       │
  │        │                                               │
  │        └─── (Exports analytics logs)                   │
  │             │                                          │
  │             ▼                                          │
  │  Analytical Data Warehouse (No public routing)         │
  │   [ BigQuery Analytics Engine ]                        │
  └────────────────────────────────────────────────────────┘
```

# Workflow
How GCP handles a containerized API request and processes analytics logs:

```text
Step 1: A user clicks a button on a mobile app, prompting Cloud DNS to route the request to the Global Load Balancer.
                             ↓
Step 2: The Load Balancer directs the HTTP request to the Cloud Run serverless container service.
                             ↓
Step 3: If no containers are active, Cloud Run boots a container instance in under 1 second (a fast warm start).
                             ↓
Step 4: The container runs its logic, using an attached Service Account to read secure database keys from Secret Manager.
                             ↓
Step 5: The container queries the Cloud SQL database to retrieve user records and return them to the client.
                             ↓
Step 6: The application exports transaction log entries into a Cloud Storage bucket.
                             ↓
Step 7: BigQuery continuously indexes the log files from Cloud Storage, allowing business analysts to query trends.
```

# Real World Examples
Think of GCP components as **a city-sized logistics warehouse system**.
- **Resource Hierarchy (Corporate Directory):** Think of the Organization as the corporate headquarters, Folders as departments (Logistics, Finance), Projects as individual filing cabinets, and Resources as files. If you lock a filing cabinet (Project), everything inside is secure.
- **Compute Engine (Leasing an empty cargo truck):** You lease a truck by the hour. You must drive it, load it, check the oil, and repair the engine yourself. It gives you maximum control but high work.
- **Cloud Run (Hiring a courier delivery service):** You pack your goods into a standard shipping box (Docker container). You hand the box to the courier service. They deliver it automatically using their own vehicles, and you only pay for the exact minutes the package was in transit.
- **Google Kubernetes Engine (Fleet Dispatch Manager):** You operate a massive fleet of 200 cargo trucks. You hire a dispatch manager (GKE) who coordinates the schedules, switches drivers when they are tired, and routes trucks around traffic delays.
- **BigQuery (Supercharged library catalog scanner):** Imagine a library with millions of books. BigQuery is a scanner that can search every page of every book in under 3 seconds to find every instance of a word, showing you the exact count instantly.

# Implementation
Here is how developers use the Google Cloud SDK (`gcloud` CLI) to authenticate, create a storage bucket, and deploy a containerized application to Cloud Run:

### Deploying to GCP via Command-Line
```bash
# 1. Authenticate with your Google account
gcloud auth login

# 2. Configure the CLI to point to your target Project ID
gcloud config set project my-online-store-project-203

# 3. Create a Cloud Storage bucket in the multi-region US area
gcloud storage buckets create gs://my-store-assets-bucket-203 \
  --location=us \
  --uniform-bucket-level-access

# 4. Push and build your application container to Google Artifact Registry
gcloud builds submit --tag gcr.io/my-online-store-project-203/web-app:1.0.0

# 5. Deploy the container image to Cloud Run
# Configures the runtime, allocates memory, and opens the service to the internet
gcloud run deploy web-app-service \
  --image gcr.io/my-online-store-project-203/web-app:1.0.0 \
  --region us-central1 \
  --platform managed \
  --allow-unauthenticated \
  --max-instances 5
```

# Best Practices
- **Restrict Default Service Accounts:** When you launch a Compute Engine VM, GCP automatically attaches a default Compute Engine Service Account. This account holds broad "Editor" permissions by default. Always deactivate these default settings and attach custom service accounts with restricted permissions.
- **Enforce Project Isolation:** Never mix development, staging, and production environments inside the same GCP Project. Create completely separate Projects for each environment so a mistake in development cannot impact production data.
- **Utilize Cloud Storage Uniform Bucket Access:** Always enable Uniform Bucket-Level Access on your Cloud Storage buckets. This forces all file permissions to be managed via IAM roles rather than individual file access control lists (ACLs), preventing accidental public leaks of private files.

# Industry Standards
Modern GCP architectures leverage **Shared VPCs**. In large enterprises, a centralized networking team manages a single host project containing the primary VPC networks, subnets, and firewalls. Developer teams deploy their applications in separate "service projects" that connect directly to the shared VPC. This ensures security compliance while allowing developers to deploy applications autonomously.

# Common Mistakes
- **Running Analytical Queries in Transactional Databases:** Querying millions of historical rows inside Cloud SQL, which locks tables and crashes active web applications. Use Cloud Dataflow to export historical data to BigQuery, reserving Cloud SQL for quick daily updates.
- **Configuring Unlimited Auto-Scaling on Cloud Run:** Failing to set a `--max-instances` cap on Cloud Run services. If your site is hit by a massive surge of spam traffic or a DDoS attack, Cloud Run will scale up to thousands of instances to handle the load, resulting in a large billing invoice.
- **Failing to Configure Object Lifecycle Rules:** Storing temporary database backup files in Cloud Storage forever. Configure Lifecycle Management rules to automatically delete backup files after 30 days.

# Security & Performance Considerations
- **VPC Service Controls:** To prevent data theft, GCP allows organizations to configure VPC Service Controls. This creates a secure boundary around sensitive services (like BigQuery or Cloud Storage), blocking users from copying data outside the project network, even if they have valid credentials.
- **GKE Auto-Repair and Auto-Upgrade:** Enable auto-repair and auto-upgrade features on Google Kubernetes Engine node pools. Google's control plane monitors node health and automatically replaces failing servers, keeping nodes patched with the latest security updates.

# Related Technologies
- **Kubernetes:** The open-source container orchestrator originally designed and open-sourced by Google.
- **TensorFlow:** Google's open-source machine learning framework, heavily optimized to run on GCP hardware.
- **Apache Beam / Cloud Dataflow:** Batch and stream data processing engines.

# Summary

## What We Learned
- Google Cloud Platform runs on Google's internal global network, emphasizing containerization, data analytics, and artificial intelligence.
- The GCP hierarchy organizes resources under Projects, which are contained within Folders and Organizations.
- Core compute ranges from virtual machines (Compute Engine) to container orchestration (GKE) and serverless runtimes (Cloud Run).
- Service Accounts serve as application identities, managed via IAM roles to enforce resource access control.

## Key Takeaways
- Always deploy custom, restricted Service Accounts to virtual machines and containers rather than using the default broad Editor accounts.
- Use BigQuery for processing analytical queries and large datasets, preserving Cloud SQL for active transactional databases.
- Restrict Cloud Run auto-scaling by configuring maximum instance limits to prevent unexpected billing invoices.

# Keywords
- GCP
- Google Cloud Platform
- Project ID
- Compute Engine
- Cloud Run
- Google Kubernetes Engine (GKE)
- BigQuery
- Cloud Storage
- Service Account
- Vertex AI

# Glossary

| Term | Meaning |
|---|---|
| Project | The core organizing entity in GCP; holds resources, handles billing, and sets security boundaries. |
| Service Account | A special Google account that represents a non-human user, used by applications to authenticate and make API calls. |
| Cloud Run | A managed serverless compute platform that runs containerized applications and scales them automatically. |
| GKE | Google Kubernetes Engine; a managed Kubernetes service used to deploy and orchestrate containerized applications. |
| BigQuery | A serverless, highly scalable cloud data warehouse designed for fast SQL queries on massive datasets. |
| Vertex AI | A centralized machine learning platform used to train, deploy, and monitor AI models. |

## Next Recommended Chapters
- 04-Azure-Cloud-Platform.md
