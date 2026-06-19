> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**GitHub Actions** is a cloud-native continuous integration and continuous delivery (CI/CD) automation platform built directly into GitHub. It allows developers to define automated workflows that trigger on specific Git events—such as code pushes, pull request creations, or project releases—running build and test scripts on virtual machine containers called **Runners**.

# Why It Exists
Historically, setting up a CI/CD pipeline required developers to host, maintain, and pay for separate, dedicated automation servers (like Jenkins). Developers had to manually connect webhooks between GitHub and the external CI server, manage server OS updates, and configure virtual build environments from scratch. GitHub created GitHub Actions to embed automation directly into repositories, allowing developers to configure pipelines using simple YAML text files stored alongside their project code, with GitHub managing the underlying runner infrastructure.

# Problem It Solves
GitHub Actions solves complex external pipeline integrations, manual runner server maintenance, and unstandardized build environment setups.

### Before GitHub Actions (External Pipelines):
- Teams had to host, secure, and pay for dedicated virtual servers to run CI builds.
- Webhook connection drops between GitHub and external servers caused silent build failures.
- Sharing build scripts across different team projects required copy-pasting complex configuration files.

### After GitHub Actions (Native Workflows):
- Pipelines are defined as code inside the `.github/workflows/` directory of the project.
- GitHub provides free, pre-configured virtual machines (Ubuntu, Windows, macOS) to run tasks.
- Reusable, community-approved plugins (**Actions**) are imported with a single line of config, replacing complex scripting.

# Core Concepts
To write GitHub Actions, you must understand workflows, triggers, jobs, and steps:

1. **Workflow:** The complete, automated process playbook defined in a `.yml` file.
2. **Event (Trigger):** The specific GitHub activity that starts the workflow (e.g., `push`, `pull_request`, or `release`).
3. **Job:** A set of steps executed sequentially on the same Runner. By default, multiple jobs in a workflow run concurrently in parallel unless configured to wait for each other.
4. **Step:** An individual task within a job. A step can run shell commands (like `npm test`) or run a pre-made **Action**.
5. **Action:** A reusable plugin package that automates a common task (e.g. checking out the code repository, setting up Node.js, or uploading files to AWS).

# Architecture / Components
The flow of an event triggering a GitHub Actions workflow:

```text
                  [ Developer pushes code to GitHub ]
                                  │
                                  ▼ (Triggers Event)
                      [ GitHub Workflow Engine ]
                                  │
                                  ▼ (Spins up Runner)
                      [ Virtual Runner Machine ]
                                  │
          ┌───────────────────────┴───────────────────────┐
          ▼ (Job 1: Build & Test)                         ▼ (Job 2: Lint Code)
     [ Step 1: Checkout Code ]                       [ Step 1: Checkout Code ]
     [ Step 2: Set up Node.js ]                      [ Step 2: Run Linter ]
     [ Step 3: Run npm test ]
```

- **Runners:** Virtual machines managed by GitHub (hosted) or managed by your team (self-hosted) that execute the job steps.
- **GitHub Marketplace:** The central registry where developers share reusable actions.

# Workflow
The execution loop of a pull request validation workflow:

```text
Step 1: A developer opens a Pull Request on GitHub.
                             ↓
Step 2: The GitHub Action engine detects the `pull_request` event and reads `.github/workflows/validate.yml`.
                             ↓
Step 3: The engine requests a clean Ubuntu virtual machine (Runner) from the GitHub cloud pool.
                             ↓
Step 4: The Runner downloads the code (`actions/checkout`), installs Node.js (`actions/setup-node`), and runs `npm run test`.
                             ↓
Step 5: The Runner reports the execution status (Success or Failure) back to the Pull Request page.
                             ↓
Step 6: GitHub displays a green checkmark next to the PR, allowing the lead engineer to merge the code.
```

# Real World Examples
Think of GitHub Actions as **writing a playbook instructions list for automated factory robots**.
- **The Event (Trigger):** The factory sensor detects that a raw box has arrived on the conveyor belt (**Code Push**).
- **The Workflow (Playbook):** The instruction sheet clipped to the workstation wall. It says: *"When a box arrives, inspect it, label it, and ship it."*
- **The Runner:** The robot arm assigned to read the sheet and execute the work. GitHub manages the robots; you just write the instruction sheet.
- **Jobs:** Workstations. Workstation 1 inspects the box for holes. Workstation 2 tapes it shut. They can operate at the same time.
- **Steps:** The sequential instructions for the robot:
  - Step 1: Pick up tape dispenser (**Checkout code action**).
  - Step 2: Apply tape to center seam (**Run test command**).
  - Step 3: Place box on delivery cart (**Deploy action**).
- **Actions:** Pre-made tools. Instead of building a tape dispenser from scratch, the robot goes to the toolbox and grabs a pre-built dispenser labelled `actions/tape-dispenser-v2`.

# Implementation
Here is how to write a fully functional GitHub Actions workflow file for a Node.js application, including build, test, and security secrets configuration:

### GitHub Actions Workflow File (`.github/workflows/ci.yml`)
Store this file directly inside your repository at `.github/workflows/ci.yml`:

```yaml
name: Node.js CI/CD Pipeline

# 1. Define when this workflow will trigger
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # 2. Define a job named 'test_and_build'
  test_and_build:
    # Run on a clean, GitHub-hosted Ubuntu virtual machine
    runs-on: ubuntu-latest

    steps:
      # Step 1: Use a pre-made action to download the code repository onto the runner
      - name: Checkout Code Repository
        uses: actions/checkout@v4

      # Step 2: Use an action to install and configure Node.js
      - name: Setup Node.js Environment
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm' # Auto-caches node_modules to speed up subsequent builds

      # Step 3: Install project dependencies
      - name: Install Dependencies
        run: npm ci

      # Step 4: Run the test suite
      - name: Run Test Suite
        run: npm test

      # Step 5: Securely deploy using secrets (Only runs on push to main)
      - name: Deploy to Server
        if: github.event_name == 'push'
        env:
          DEPLOY_KEY: ${{ secrets.PROD_SERVER_SSH_KEY }} # Secure secret injection
        run: |
          echo "Connecting to production server..."
          # Command to deploy code using DEPLOY_KEY...
```

# Best Practices
- **Pin Action Versions:** When importing community actions, always specify the major version tag or exact commit hash (e.g. `uses: actions/checkout@v4`). This prevents your pipeline from breaking if the author releases a breaking update.
- **Enable Cache Dependencies:** Speed up build times by enabling package caching (e.g., using the `cache: 'npm'` option in `setup-node`). This prevents the runner from downloading hundreds of npm packages from scratch on every run.
- **Use Matrix Builds for Cross-Platform Testing:** If you write a library, test it across multiple OS and runtime versions simultaneously using a `matrix` configuration (e.g., testing Node 18, 20, and 22 on Ubuntu and Windows with a single job declaration).

# Industry Standards
Almost all modern GitHub-based open-source and corporate projects use GitHub Actions as their primary CI/CD platform. They restrict merging into the `main` branch unless the GitHub Actions check status returns "success" (Branch Protection).

# Common Mistakes
- **Exposing Secrets in logs:** Writing commands like `echo "${{ secrets.API_KEY }}"` inside build scripts. Although GitHub attempts to mask secrets with asterisks (`***`), printing secrets to standard outputs makes them vulnerable to exposure in build logs.
- **Infinite Loop Triggers:** Writing a workflow that triggers on `push`, and having the workflow step commit changes back to the same branch. This triggers the workflow again, creating an infinite loop of virtual machines running and inflating your cloud bill.
- **Forgetting to specify the checkout action:** Writing a step to compile code without running `uses: actions/checkout@v4` first. The runner starts in an empty directory and will throw "File not found" errors because it hasn't downloaded your repository yet.

# Security & Performance Considerations
- **Third-Party Action Supply Chain Attacks:** Using unverified actions from untrusted developers in the GitHub Marketplace. A malicious actor can update their action code to steal your environment secrets and upload them to their server. Only use actions from verified organizations (like GitHub, Docker, or AWS).
- **Runner Billing Optimization:** GitHub charges for runner execution minutes. To optimize costs, set a timeout limit on your jobs (e.g., `timeout-minutes: 10`) so a hanging loop or stuck test doesn't consume hours of paid runner time.

# Related Technologies
- **GitHub Secrets:** The secure encrypted repository vault used to store API keys and passwords.
- **act:** An open-source tool that allows developers to run their GitHub Actions workflows locally on their own machines using Docker.

# Summary

## What We Learned
- GitHub Actions is a native CI/CD automation engine built directly inside GitHub.
- Workflows are defined in YAML files, triggered by events, and executed on virtual machine Runners.
- Jobs segment workflows into workstations, while Actions are reusable tools that automate common tasks.

## Key Takeaways
- Always store production credentials in GitHub Secrets and inject them dynamically into runner environments.
- Pin third-party actions to specific versions to prevent unexpected pipeline breakages.
- Configure job timeouts and package caches to optimize pipeline speed and running costs.

# Keywords
- GitHub Actions
- Workflow
- Event Trigger
- Runner
- Job
- Step
- GitHub Secrets
- YAML Configuration
- Cache
- Matrix Build

# Glossary

| Term | Meaning |
|---|---|
| Runner | A virtual machine managed by GitHub that executes workflow jobs. |
| Workflow | A configurable automated process defined as a YAML file in `.github/workflows/`. |
| Action | A reusable task application imported to perform common jobs (e.g. download code). |
| YAML | A human-readable data serialization language used to write configuration files. |
| GitHub Secrets | Encrypted storage inside a GitHub repository used to save private keys and passwords. |
| Matrix Build | A configuration that runs the same job across multiple combinations of OS and runtime versions simultaneously. |

## Next Recommended Chapters
- 05-Docker.md
