> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Cloud Identity and Access Management (IAM)** is the security framework and service suite used by cloud providers to control access to cloud resources. It manages **Authentication** (verifying *who* a user or application is) and **Authorization** (verifying *what* specific resources they have permission to access, read, or delete).

# Why It Exists
In a cloud platform, all servers, databases, and network drives are hosted on shared global infrastructure accessible over the internet. Without a unified gatekeeper, anyone with a browser could attempt to access your sensitive systems. Furthermore, inside an engineering organization, different employees need separate boundaries: database administrators need to modify tables, developers need to deploy code, and accounting teams need to see invoices. Cloud IAM was created to act as a centralized lock-and-key engine, verifying identities and enforcing precise permission rules across all cloud services.

# Problem It Solves
Cloud IAM solves the problems of unauthorized resource access, shared password leaks, over-privileged application accounts, and lack of security auditing.

### Before Cloud IAM (Shared Credentials Model):
- Teams shared a single master administrator password. If an employee left the company, the password had to be changed on every system, causing service disruption.
- Applications were configured with hardcoded master keys. If a developer accidentally committed these keys to a public repository, attackers could steal them and delete the database.
- Security officers had no way to track who created or deleted a specific server, making security breach investigations impossible.

### After Cloud IAM (Role-Based Access Control):
- Every employee receives a unique user account with Multi-Factor Authentication (MFA) forced on login.
- Virtual servers and container services are assigned temporary, expiring security credentials (IAM Roles) that automatically refresh without passwords in code.
- Every API request is logged by auditing services, recording exactly who made the request, when, and from what IP address.

# Core Concepts
Securing cloud resources requires understanding the components of authorization policy evaluation:

1. **IAM Identity Entities:**
   - **Users:** Individual people (e.g. "Dev-Alice") who log in with a username, password, and MFA code.
   - **Groups:** Collections of users (e.g. "Finance-Team"). Instead of assigning permissions to individuals, policies are attached to the group.
   - **Roles (or Service Accounts):** Temporary identities assumed by applications or cloud resources (e.g., an App Server VM assuming a role that grants access to an S3 database). Roles do not use passwords; they use short-lived token keys that expire in hours.
2. **IAM Policies:** Written rules (typically formatted in JSON) that define access permissions. A policy describes: **Who** (Principal) can perform **What** (Actions) on **Which** (Resources) under specific **Conditions** (e.g. only from the company office IP network).
3. **Principle of Least Privilege:** A fundamental security rule stating that every user and service must only hold the minimum permissions required to perform their specific job, and absolutely nothing more.
4. **Explicit Deny:** In IAM, permissions are denied by default. If a policy has a rule allowing access, but another policy has a rule denying it, the **Deny rule always wins**, blocking access.

# Architecture / Components
The pipeline of IAM policy evaluation when a resource makes a request:

```text
  [ Caller: EC2 App Server VM ]
               │
     (Submits API request: s3:GetObject)
               │
               ▼
  ┌────────────────────────────────────────────────────────┐
  │ Cloud IAM Policy Evaluation Engine                     │
  │                                                        │
  │  1. Check Identity Token validity                      │
  │  2. Locate policies attached to the VM's IAM Role      │
  │  3. Scan policy rules: Is s3:GetObject permitted?     │
  └──────────────────────────┬─────────────────────────────┘
                             │
            (Is there an explicit DENY?)
             ├─── Yes ───► [ Access Blocked ]
             │
             └─── No ───► (Is there an explicit ALLOW?)
                            ├─── No  ───► [ Access Blocked (Default) ]
                            └─── Yes ───► [ Access Granted ]
                                                │
                                                ▼
                                  [ Download file from S3 ]
```

# Workflow
How an application server securely downloads a database configuration file without using hardcoded passwords:

```text
Step 1: Systems engineer creates an IAM Role named `AppServerRole` and attaches a policy allowing `s3:GetObject` on a private config bucket.
                             ↓
Step 2: Engineer assigns `AppServerRole` to a newly launched virtual machine (EC2 instance).
                             ↓
Step 3: The virtual machine boots up. The AWS Security Token Service (STS) automatically injects temporary credentials into the VM.
                             ↓
Step 4: The application running on the VM requests a config file from the private S3 bucket using the temporary credentials.
                             ↓
Step 5: The S3 service queries the IAM engine, which verifies that the VM's role holds the required `GetObject` permission.
                             ↓
Step 6: S3 delivers the file to the VM, and the access request is logged by CloudTrail for security audits.
```

# Real World Examples
Think of Cloud IAM as **a high-security office building badge and key card system**.
- **IAM User (Employee Badge):** An ID card with your name and photo. It lets you walk through the front turnstiles.
- **IAM Group (Department Pass):** All employees in the HR department get badges that open the HR office suite. Instead of programming each badge individually, the building manager programs the door lock: "Let anyone in who belongs to the HR Group."
- **IAM Role (Visitor Escort Badge):** A photocopy machine technician arrives at the lobby. The receptionist does not issue them a permanent employee badge. They hand the technician a temporary visitor badge that automatically deactivates at 5 PM (temporary token) and only opens the copy machine closet doors.
- **IAM Policy (The Lock Instruction Sheet):** A card taped next to the vault door: "Allow entry only if caller holds a Manager Badge AND the current time is between 9 AM and 5 PM."
- **Least Privilege (Restricted Keys):** The building janitor has a key to the utility closets and trash rooms, but their key cannot open the CFO's desk drawers or the server racks.

# Implementation
Here is how developers define security permissions using JSON policies. This example shows an IAM Policy that allows read-only access to a specific storage bucket, blocking all delete operations:

### JSON IAM Policy: Read-Only Bucket Access
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListAndReadAssets",
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-assets-bucket",
        "arn:aws:s3:::my-company-assets-bucket/*"
      ]
    },
    {
      "Sid": "BlockAllDeletesExplicitly",
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "s3:DeleteObject",
        "s3:DeleteBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-company-assets-bucket",
        "arn:aws:s3:::my-company-assets-bucket/*"
      ]
    }
  ]
}
```

# Best Practices
- **Lock Down the Root Account:** The root account is the master email identity used to create the cloud account. Never use it for daily tasks. Secure it with a complex password, enable hardware MFA, delete all API access keys, and create restricted administrator IAM users for daily operations.
- **Attach Policies to Groups, Never Users:** Individual employee roles change frequently. If you attach policies directly to individual users, you will eventually lose track of who has access. Always create Groups (e.g. `SysAdmins`, `Developers`), attach policies to the groups, and slide users in and out.
- **Enforce Single Sign-On (SSO):** Integrate your cloud account with your company's central identity directory (like Okta or Microsoft Entra ID). This ensures that when an employee is terminated and deactivated in the central directory, their cloud access is automatically revoked instantly.

# Industry Standards
Modern security compliance standards enforce **Credential Rotation** and **MFA Enrollment Policies**. Automated scanners monitor cloud directories to flag and disable any human accounts that do not have multi-factor authentication active, or any API access keys that have been active for more than 90 days, reducing the risk of compromised accounts.

# Common Mistakes
- **Sharing API Access Keys:** Creating a single "developer" user, generating an Access Key ID and Secret Key, and emailing them to the entire team. If one laptop is compromised, the entire cloud infrastructure is exposed. Every user must have their own unique account.
- **Using Wildcard Permissions in Production:** Using `"Action": "*"` (allow everything) in application policies during initial development because it is convenient, and forgetting to lock it down before production release. Always replace wildcards with narrow, specific actions.
- **Hardcoding Keys in Source Code:** Writing AWS or GCP credentials directly inside Javascript or Python code files. Attackers run automated web index crawlers that scan public code repositories, stealing hardcoded keys in seconds.

# Security & Performance Considerations
- **Metadata Service (IMDS):** Virtual servers query local metadata IP addresses (e.g. `169.254.169.254` on AWS) to fetch their temporary IAM Role credentials at runtime. Systems engineers must secure this endpoint using IMDSv2 to prevent Server-Side Request Forgery (SSRF) attacks from stealing server tokens.
- **Policy Limits:** Cloud providers set size limits on individual policy documents (e.g., 6KB). To avoid hitting these limits, organize resources using wildcard prefixes (like `s3:::my-bucket/logs-*`) rather than listing hundreds of individual file resources.

# Related Technologies
- **AWS STS (Security Token Service):** The underlying engine that issues temporary, expiring credentials to roles and services.
- **Okta:** A popular external identity provider used to manage single sign-on credentials for corporate employees.
- **HashiCorp Vault:** A secure secret vault commonly used to manage and rotate cloud credentials dynamically.

# Summary

## What We Learned
- Cloud IAM manages Authentication (identity verification) and Authorization (access rule enforcement).
- The resource hierarchy separates permissions for Users, Groups, and temporary Roles.
- IAM policy evaluation defaults to Deny, allowing access only when an explicit Allow is present, and Deny rules always override allows.
- The Principle of Least Privilege protects systems by restricting access to only the necessary resources required to run.

## Key Takeaways
- Deactivate access keys on root accounts and enforce Multi-Factor Authentication on all human accounts.
- Assign permissions to logical Groups rather than individual User accounts to maintain audit clarity.
- Utilize temporary IAM Roles or Service Accounts to authorize applications, preventing key leaks from hardcoded files.

# Keywords
- IAM
- Authentication
- Authorization
- Least Privilege
- IAM Role
- Service Account
- IAM Policy
- Explicit Deny
- STS Token
- Single Sign-On (SSO)

# Glossary

| Term | Meaning |
|---|---|
| Authentication | The process of verifying the identity of a user or system attempting to connect to resources. |
| Authorization | The process of verifying what specific actions an authenticated identity is permitted to perform. |
| IAM Role | An identity with temporary security credentials that can be assumed by users, applications, or services. |
| Service Account | A non-human IAM identity used by applications and containers to authenticate with cloud APIs. |
| Least Privilege | The security practice of restricting user and application permissions to only what is necessary. |
| SSO | Single Sign-On; a session service letting users enter one set of credentials to access multiple apps. |

## Next Recommended Chapters
- 10-Cloud-Networking-and-VPCs.md
