> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Infrastructure as Code (IaC)** is the operational methodology of managing, provisioning, and configuring computing infrastructure—including servers, storage, network topologies, and load balancers—through machine-readable configuration files, rather than through manual hardware setups or graphical dashboard clicks.

# Why It Exists
Before IaC, setting up infrastructure was a manual task performed by system administrators who racked physical servers and ran manual install scripts. As virtualization and cloud computing emerged, creating virtual servers became fast, but administrators still configured them manually. Over time, servers in a cluster accumulated minor, undocumented software updates and configuration changes. This is known as **Configuration Drift**—servers that are supposed to be identical become slightly different, leading to mysterious bugs that are impossible to replicate. Engineers created the IaC methodology to ensure that server setups are 100% reproducible, version-controlled, and automated.

# Problem It Solves
IaC solves configuration drift, slow manual server deployments, and undocumented infrastructure setups.

### Before Infrastructure as Code (Manual Management):
- Setting up staging and production servers resulted in slight environment mismatches.
- Administrators had no history of why a server setting was changed, as changes were made manually.
- If a datacenter suffered an outage, rebuilding the infrastructure required days of manual configuration.

### After Infrastructure as Code (Code-Driven Management):
- Staging and production environments are created using the exact same code files, guaranteeing consistency.
- All infrastructure edits are versioned in Git repositories, establishing a clear audit trail.
- Rebuilding an entire global cloud network takes minutes, automated by IaC pipelines.

# Core Concepts
To design IaC pipelines, you must master the trade-offs of provisioning styles and mutability models:

1. **Declarative vs. Imperative:**
   - **Declarative (What):** You define the desired end state (e.g. "I want 5 servers"). The IaC tool analyzes the current state and handles the API steps to make it match your declaration.
   - **Imperative (How):** You define the exact sequence of commands to execute (e.g. "Step 1: Create server. Step 2: Install Node.js. Step 3: Open port 80").
2. **Mutable vs. Immutable Infrastructure:**
   - **Mutable (Changeable):** Running software updates directly on active, running servers. This is simple but eventually leads to configuration drift.
   - **Immutable (Unchangeable):** Never editing active servers. If you need to update a configuration or deploy new code, you destroy the old servers and deploy brand-new, pre-configured servers in their place.
3. **Configuration Drift:** The phenomenon where the real-world configuration of a server diverges from its code blueprint due to manual edits or background system updates.

# Architecture / Components
The GitOps workflow where infrastructure code modifications are reviewed and deployed automatically:

```text
  [ Developer edits main.tf ] ──> [ Pushes to GitHub ]
                                          │
                                          ▼ (Opens Pull Request)
                               [ GitHub Actions Pipeline ]
                               - Runs 'terraform plan'
                               - Prints plan on PR for team review
                                          │
                                          ▼ (PR merged to main)
                               [ GitOps Runner (Apply) ]
                               - Executes cloud API calls
                               - Rebuilds infrastructure to match repo
```

- **Infrastructure Source of Truth:** A Git repository containing the configuration files.
- **GitOps Controller:** An automation pipeline (like GitHub Actions or ArgoCD) that automatically syncs cloud infrastructure with the repository code.

# Workflow
The workflow of resolving a Configuration Drift anomaly:

```text
Step 1: An engineer manually logs into AWS and changes a firewall port from closed to open to debug an issue.
                             ↓
Step 2: The engineer forgets to close the port or record the change in the Terraform code (Drift occurs).
                             ↓
Step 3: The automated GitOps pipeline runs its daily sync check.
                             ↓
Step 4: The IaC engine compares the real-world AWS settings with the HCL code in the Git repository.
                             ↓
Step 5: The engine detects the mismatch: Code says "Port 80 only", but AWS says "Port 80 and Port 22 open".
                             ↓
Step 6: The engine automatically overwrites the manual edit, closing Port 22 to restore the desired state.
```

# Real World Examples
Think of Infrastructure as Code as **building prefabricated modular houses vs. manual on-site construction**.
- **Mutable (On-Site Construction):** You build a brick house. Five years later, you want a new window, so you hire workers to hammer a hole in the wall and install one. Later, you want a chimney, so you cut a hole in the ceiling. Over time, your house is full of patches, and no one has an accurate blueprint of how the pipes and wiring look today. If the house burns down, you cannot rebuild it exactly as it was.
- **Immutable (Prefabricated IaC):** You design a house blueprint in a computer program (**HCL Code**). If you want a new window, you don't hammer the walls. You edit the blueprint code to add the window, send it to the factory (**IaC Engine**), build a brand-new prefabricated house, tear down the old house, and drop the new one in its place.
  - **No Drift:** You are guaranteed that the physical house matches the blueprint exactly.
  - **Disaster Recovery:** If the house is destroyed, you print a new one from the blueprint in 1 day.
- **Declarative vs. Imperative:**
  - **Imperative:** Telling a butler: *"Walk to the kitchen, open the cabinet, grab a cup, fill it with water, and bring it to me."* If the cabinet is locked, the butler freezes and crashes.
  - **Declarative:** Telling the butler: *"I want a cup of water."* The butler figures out how to get it, handling any locked cabinets themselves.

# Implementation
Here is a comparison of how the same server configuration task is represented in Imperative scripting (Bash) vs. Declarative IaC (Terraform):

### 1. Imperative Style (Bash Script - "How to build")
This script executes sequential commands. If the script fails halfway through, re-running it will cause errors because it attempts to recreate resources that already exist.

```bash
#!/bin/bash
# Step 1: Create a server (using AWS CLI)
INSTANCE_ID=$(aws ec2 run-instances --image-id ami-0c55b159cbfafe1f0 --count 1 --instance-type t2.micro --query 'Instances[0].InstanceId' --output text)

# Step 2: Wait for server to boot
aws ec2 wait instance-status-ok --instance-ids $INSTANCE_ID

# Step 3: Install Nginx
ssh -i key.pem ec2-user@instance-ip "sudo yum install -y nginx && sudo systemctl start nginx"
```

### 2. Declarative Style (Terraform - "What to build")
This HCL code defines only the end state. Re-running this code has no effect if the server already exists, preventing duplicate resource creation.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # User data script executes automatically on boot
  user_data = <<-EOF
              #!/bin/bash
              sudo yum install -y nginx
              sudo systemctl start nginx
              EOF
}
```

# Best Practices
- **Prioritize Immutable Infrastructure:** Build your systems around the immutable model. Instead of SSH-ing into running production servers to run updates, build a new container image and deploy it in place of the old one.
- **Use Git as the Sole Source of Truth (GitOps):** Block developers from accessing the cloud console to make manual changes. All infrastructure changes must be proposed via Git pull requests and applied automatically by pipeline runners.
- **Implement Drift Detection:** Configure your CI/CD pipelines to run scheduled plan checks (e.g. once a day) to detect and alert you if the real-world cloud resources have drifted from the repository code.

# Industry Standards
Modern enterprises adopt **GitOps** principles. They use continuous delivery tools (like ArgoCD for Kubernetes or Terraform Cloud) to monitor Git repositories and automatically reconcile any differences between the code and the live production environment.

# Common Mistakes
- **Mixing Mutable and Immutable Practices:** Managing servers with Terraform but using manual SSH logins to install software packages. This defeats the purpose of IaC, resulting in configuration drift and untracked dependencies.
- **Lacking State Locking:** Running Terraform applies from multiple developers' laptops simultaneously without remote state locking. This corrupts the `tfstate` file, causing Terraform to lose track of resources and create duplicate servers.
- **Forgetting to Version IaC Code:** Keeping Terraform files on local folders without committing them to Git. Infrastructure modifications must be tracked in version control just like application code.

# Security & Performance Considerations
- **Credential Least Privilege:** The pipeline runner that executes IaC scripts needs high-level administrative API access to create and delete cloud resources. Teams must restrict these runner credentials using restricted IAM roles to prevent attackers from compromising the entire cloud account.
- **Plan Phase Execution Time:** In massive environments with thousands of resources, running `terraform plan` to compare code to the cloud takes significant time. Segregate your infrastructure into separate directories (e.g., separating network VPC configs from database configs) to keep plans fast.

# Related Technologies
- **ArgoCD:** A declarative, GitOps continuous delivery tool for Kubernetes.
- **Ansible:** An open-source IT automation tool used for imperative software configuration management.
- **AWS CloudFormation:** Amazon's native declarative Infrastructure as Code service.

# Summary

## What We Learned
- Infrastructure as Code manages networks, servers, and storage through version-controlled text files rather than manual clicking.
- Imperative scripting defines step-by-step commands; declarative configurations declare only the desired end state.
- Immutable infrastructure replaces old servers completely during updates, preventing configuration drift.
- GitOps automates infrastructure deployment by treating Git repositories as the single source of truth.

## Key Takeaways
- Enforce strict branch protection and automated GitOps pipelines for all infrastructure code updates.
- Choose declarative IaC frameworks over imperative bash scripts to manage complex cloud resources safely.
- Never make manual changes directly inside the cloud provider's console once the resource is managed by IaC.

# Keywords
- Infrastructure as Code (IaC)
- Configuration Drift
- Mutable Infrastructure
- Immutable Infrastructure
- Declarative
- Imperative
- GitOps
- State Locking
- Reconcile
- AWS CloudFormation

# Glossary

| Term | Meaning |
|---|---|
| Configuration Drift | The divergence between the active configuration of cloud resources and the state declared in code. |
| GitOps | A practice that uses Git repositories as the single source of truth for defining and deploying infrastructure. |
| Immutable Infrastructure | An infrastructure strategy where servers are replaced completely rather than modified in place during updates. |
| Imperative Programming | A style of programming where you specify the explicit sequential steps a system must take. |
| Declarative Programming | A style of programming where you declare only the desired outcome, leaving the system to execute the steps. |
| Reconcile | The automated process where a GitOps engine corrects live resources to match the repository configuration. |

## Next Recommended Chapters
- 10-Monitoring-And-Observability.md
