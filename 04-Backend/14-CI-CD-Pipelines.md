> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**CI/CD** stands for **Continuous Integration** and **Continuous Delivery** (or Continuous Deployment). It is an automated software pipeline design that manages how code changes travel from a developer's keyboard to the live production servers. It ensures that every code change is automatically checked for errors, tested, packaged, and deployed without requiring manual human intervention.

# Why It Exists
Before CI/CD, deploying software was a manual, high-stress process. Developers wrote code on their laptops, compiled it manually, and handed it to a system administrator. The administrator had to log in to production servers, upload files using FTP, manually run database scripts, restart services, and hope nothing broke. If a developer forgot to upload a single file, or if there was a syntax typo, the entire website went down, and the team had to spend hours tracing the error. Releases happened only once every few months because they were risky and exhausting. Engineers created CI/CD pipelines to automate this entire workflow, turning deployments into a boring, routine event that happens dozens of times a day.

# Problem It Solves
CI/CD pipelines solve manual release errors, broken production builds, slow feedback loops, and code integration delays.

### Before CI/CD (Manual Releases):
- Code changes sat integrated for months because deploying was difficult and risky, leading to massive conflicts when merging.
- Typos and security vulnerabilities slipped into production frequently because testing relied on manual checklists.
- Finding which code commit broke the server required hours of manual debugging.

### After CI (Continuous Integration):
- Every time a developer pushes code, automated servers (runners) immediately download it, run security audits, and execute unit tests. If any test fails, the code is blocked from merging.

### After CD (Continuous Delivery/Deployment):
- Once the tests pass and code is merged, the pipeline automatically compiles the assets, builds a Docker image, and deploys it to the cloud.

# Core Concepts
To write pipeline configurations, you must master the two stages of the pipeline:

1. **Continuous Integration (CI):** The focus is on **quality control**. Developers merge their code changes back to the main branch frequently. The pipeline automatically runs:
   - **Linters:** Checking code style and syntax rules.
   - **Unit Tests:** Running automated tests to check if code logic works.
   - **Security Scanners:** Checking if external packages contain known bugs.
2. **Continuous Delivery (CD):** The focus is on **automated shipping**. The code is prepared for release. In *Continuous Delivery*, the build is automated, but a human clicks a "Release" button to deploy. In *Continuous Deployment*, the release happens automatically with zero human interaction the millisecond tests pass.
3. **Pipeline Runner:** The background server agent (provided by GitHub Actions, GitLab CI, or Jenkins) that spins up to execute your pipeline script commands.

# Architecture / Components
The flow of code through an automated CI/CD pipeline:

```text
  [ Developer Code Push ] (Git Commit)
            │
            ▼
  ┌────────────────────────────────────────────────────────┐
  │                 CI/CD Pipeline Run                     │
  │                                                        │
  │  [ CI Stage: Test ]                                    │
  │  - Run syntax check (lint)  ──>  - Run unit tests      │
  │                                        │ (Passes)      │
  │                                        ▼               │
  │  [ CD Stage: Deploy ]                                  │
  │  - Build Docker Image       ──>  - Push to AWS Cloud   │
  └────────────────────────────────────────────────────────┘
            │
            ▼
  [ Production Servers Updated ] (Zero downtime)
```

- **Pipeline Configuration File:** A YAML text file (like `.github/workflows/deploy.yml`) defining exactly what steps the pipeline runner must execute.
- **Workflow Triggers:** Events that start the pipeline (e.g. pushing code to the `main` branch, or opening a Pull Request).
- **Environment Secrets:** Encrypted variables (like cloud passwords or database keys) stored securely in the pipeline repository, injected only during build steps.

# Workflow
The lifecycle of an automated deployment:

```text
Step 1: A developer fixes a bug and pushes code to the GitHub `main` branch.
                             ↓
Step 2: GitHub Actions detects the push and starts the pipeline.
                             ↓
Step 3: A pipeline runner spins up, installs Node.js, and runs `npm run test`.
                             ↓
Step 4: If tests fail, the pipeline stops, alerts the developer, and blocks deployment.
                             ↓
Step 5: If tests pass, the runner logs into AWS using encrypted repository secrets.
                             ↓
Step 6: The runner builds a Docker container and deploys it to the live servers automatically.
```

# Real World Examples
Think of a CI/CD pipeline as a **book publishing automated assembly line**.
- In early publishing, printing a book was manual: a writer printed pages, mailed them to a proofreader, who manually marked edits, sent it to a printing press, and someone packed it onto a truck. If the writer misspelled a name, they only found out after thousands of books were printed.
- A **CI/CD pipeline** is an automated assembly line. When the writer finishes a chapter and hits submit (commits code):
  - **CI** is the automated proofreader. The system instantly runs the chapter through spell-checkers (linters) and cross-reference verifiers (unit tests). If a single name is misspelled, the chapter is rejected and sent back to the writer instantly.
  - **CD** is the printing press and delivery service. Once the automated checkers approve the chapter, the system automatically prints it, binds it into a cover (Docker build), loads it onto delivery drones, and places it on bookstore shelves globally (deploys to production servers) with zero human intervention.

# Implementation
Here is how you write a basic CI/CD pipeline configuration file for **GitHub Actions** (`.github/workflows/ci.yml`) to test and lint a Node.js project:

```yaml
# 1. Define the name of the workflow
name: Node.js CI Pipeline

# 2. Define the trigger events
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# 3. Define the list of jobs to run
jobs:
  build-and-test:
    # Use a standard Linux runner agent
    runs-on: ubuntu-latest

    steps:
    # A. Download the repository code onto the runner
    - name: Checkout Code
      uses: actions/checkout@v3

    # B. Install Node.js on the runner environment
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18
        cache: 'npm'

    # C. Install the project dependency packages
    - name: Install Dependencies
      run: npm ci

    # D. Run the code linter (syntax checks)
    - name: Run Linter
      run: npm run lint

    # E. Execute all unit tests
    - name: Run Automated Tests
      run: npm test
```

# Best Practices
- **Write Fast Tests:** If your test suite takes 45 minutes to run, developers will avoid pushing code, defeating the purpose of "Continuous" integration. Optimize your tests so they run in under 5 minutes.
- **Never Hardcode Secrets in Pipeline YAML:** Never write cloud passwords, database strings, or private keys directly in your pipeline YAML file. Use GitHub repository secrets and reference them using variables (e.g. `${{ secrets.AWS_ACCESS_KEY }}`).
- **Keep Production Deploys Idempotent:** Ensure that running the deployment step multiple times does not cause downtime or break the server database structure.

# Industry Standards
**GitHub Actions** and **GitLab CI** are the leading modern cloud CI/CD systems due to their direct integration with code repositories. **Jenkins** is the classic open-source self-hosted pipeline tool, still widely used in legacy enterprise systems. Modern deployments use **ArgoCD** or **Terraform** to manage cloud infrastructure dynamically.

# Common Mistakes
- **Deploying Without Testing:** Writing a pipeline that builds and deploys code but runs no automated tests. This simply automates the delivery of bugs to your users.
- **Treating the Pipeline as a Trash Can:** Merging broken code and assuming you will fix it in the pipeline later. Keep the main branch sacred; only merge code that passes tests locally.

# Security & Performance Considerations
- **Supply Chain Injection:** Attackers target pipeline actions or plugins. If you use a third-party pipeline action, specify a strict commit hash (like `@v3.0.1`) rather than a floating tag (like `@latest`) to prevent running injected malicious runner code.
- **Cache Dependencies:** Configure caching for package managers (like `npm` or `pip`) inside your workflow file. This prevents the runner from downloading identical libraries from scratch on every run, cutting build times in half.

# Related Technologies
- **GitHub Actions / GitLab CI / Jenkins:** The standard workflow pipeline orchestration tools.
- **Docker:** The tool used to package application builds inside pipeline steps before cloud deployment.
- **Terraform:** Infrastructure-as-code software used to build and configure cloud servers automatically inside pipelines.

# Summary

## What We Learned
- CI/CD pipelines automate the testing, compilation, packaging, and deployment of backend applications.
- Continuous Integration focuses on automated validation; Continuous Deployment handles hands-free cloud releases.
- Using repository environment secrets secures database and API keys during build workflows.

## Key Takeaways
- Optimize test scripts to maintain fast, efficient build cycles.
- Isolate configuration pipelines from raw passwords by utilizing secure vault keys.

# Keywords
- CI/CD
- Pipeline
- Continuous Integration
- Continuous Deployment
- Runner
- Linter
- Automated Testing
- Secrets

# Glossary

| Term | Meaning |
|---|---|
| Continuous Integration | The practice of automatically testing and merging developer code changes into a shared main repository branch. |
| Continuous Deployment | The practice of automatically building and deploying code changes directly to production servers once they pass tests. |
| Runner | The virtual or physical server agent that executes the list of tasks defined in a pipeline configuration file. |
| Linter | A static analysis tool that inspects code files for syntax formatting errors and stylistic issues. |
| Pipeline | The complete automated chain of steps (lint -> test -> build -> deploy) that code goes through to reach production. |

## Next Recommended Chapters
- 11-Containerization-And-Docker.md
- 04-Backend/15-Observability.md
