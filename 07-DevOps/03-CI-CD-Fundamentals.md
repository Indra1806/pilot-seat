> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**CI/CD Fundamentals** (Continuous Integration and Continuous Delivery/Deployment) is the study of automated software release pipelines. It covers the methodologies and automation flows that compile code, execute tests, package applications, and deploy updates to production environments, replacing manual developer operations with consistent, automated scripts.

# Why It Exists
Traditionally, releasing software was a manual, high-risk process. Developers wrote code on their local machines for months, then handed a folder of files to operations teams. Operations had to compile the code, configure servers manually, and copy files. Often, the code wouldn't build because of configuration differences ("works on my machine" syndrome). If a bug escaped into production, rolling back the release took hours of manual work. Engineers created CI/CD pipelines to establish automated, standardized release pipelines that test and deploy code updates continuously, reducing release cycles from months to minutes.

# Problem It Solves
CI/CD pipelines solve integration delays, manual configuration errors, and high-risk manual deployment downtimes.

### Before CI/CD (Manual Releases):
- Code changes were merged only at the end of a sprint, leading to massive, painful merge conflicts ("merge hell").
- Testing was done manually by QA teams over weeks, delaying bug discovery.
- Deploying involved manually copying files over FTP, which risked missing dependencies or corrupting active servers.

### After CI/CD (Automated Pipelines):
- Code is merged and validated multiple times a day, keeping the master branch healthy.
- Automated tests run on every single code commit, catching bugs seconds after they are written.
- Deployments are standardized as code-driven pipelines, ensuring identical, repeatable setups on staging and production servers.

# Core Concepts
To design automation pipelines, you must understand the distinction between Integration, Delivery, and Deployment:

1. **Continuous Integration (CI):** The practice of automating the merging, building, and testing of code changes. The moment a developer pushes code to a Git repository, the CI server automatically downloads the code, builds it, and executes test suites.
2. **Continuous Delivery (CD):** An extension of CI where the pipeline automatically packages the code into a runnable bundle (an **Artifact**, such as a Docker image or zip file) and prepares it for deployment. Releasing to production requires a manual confirmation click.
3. **Continuous Deployment (CD):** A fully automated version of Continuous Delivery. Any code change that passes all automated test stages is deployed directly to production servers automatically, with zero human intervention.
4. **Build Artifact:** The final, compiled, and packaged version of the application code that is ready to run on a server.

# Architecture / Components
The sequential stages of a standard CI/CD software delivery pipeline:

```text
  [ Developer Push ] ──> [ Git Repository ]
                               │
                               ▼ (Trigger Pipeline webhook)
  ┌────────────────────────────────────────────────────────┐
  │                   CI/CD PIPELINE ENGINE                │
  │                                                            │
  │  [ CI Stage: Build ] ──> [ CI Stage: Test ] ───────────────┐
  │  - Compile code          - Run unit tests                  │
  │  - Install dependencies  - Run code linter                 │
  │                                                            │
  │  ┌─────────────────────────────────────────────────────────┘
  │  ▼
  │  [ CD Stage: Package ] ──> [ CD Stage: Deploy ]
  │  - Create zip / Docker     - Deploy to Staging (Auto)
  │  - Upload Artifact         - Deploy to Production (Manual/Auto)
  └────────────────────────────────────────────────────────┘
```

- **Pipeline Runner:** The virtual machine or container server that runs the build, test, and deploy shell commands.
- **Artifact Registry:** A secure storage server where compiled build artifacts are archived.

# Workflow
The automation loop of a Continuous Deployment pipeline:

```text
Step 1: Developer pushes a bug fix branch to GitHub.
                             ↓
Step 2: GitHub triggers the pipeline engine via a webhook.
                             ↓
Step 3: The pipeline runner starts: installs Node.js, runs `npm install`, and compiles the codebase.
                             ↓
Step 4: The runner executes tests: `npm test`.
        - If tests fail: Stop pipeline and notify developer.
                             ↓
Step 5: If tests pass: The runner packages the code into a Docker container and uploads it to an Artifact Registry.
                             ↓
Step 6: The runner connects to the production server and pulls the new Docker image, completing the deploy with zero downtime.
```

# Real World Examples
Think of a CI/CD pipeline as an **automated car manufacturing assembly line**.
- **Without CI/CD (Manual Assembly):** Workers build car parts in different workshops without checking if they fit together. At the end of the month, they bring all parts to the main floor. The doors don't fit the frame, the engine mounts are too small, and the steering wheel is missing bolts. They spend weeks manually filing and hammering parts to fit (**Integration hell**).
- **With CI/CD:**
  - **Continuous Integration (CI):** The moment a worker stamps a new car door (commits code), a robot picks it up, measures it to the fraction of a millimeter, and fits it onto a test frame (**Automated Build**). The robot then bangs the door 1,000 times to verify durability (**Automated Test**). If it bends, an alarm sounds, and the door is recycled. The main assembly line never stops.
  - **Continuous Delivery (Manual Gate CD):** Once the car passes all tests, the robot drives it to the loading dock, details it, boxes it, and puts the keys in a drawer. The car sits ready to go. The sales manager walks up, signs the release paper, and clicks a button to ship it to the dealership.
  - **Continuous Deployment (Automatic CD):** The moment the car rolls off the test track, the factory gates automatically open, and the car self-drives straight to the dealership lot, ready for customers. There is no middleman signing papers.
  - **Artifact:** The fully boxed, key-included car ready for the road.

# Implementation
Here is a conceptual look at a pipeline script configuration (using standard YAML structure) that defines build, test, and deploy stages:

### Conceptual Pipeline Definition (`pipeline.yml`)
```yaml
name: Application CI/CD Pipeline

# Trigger the pipeline on any push to the main branch
on:
  push:
    branches:
      - main

stages:
  # 1. Build Stage
  - name: Build Code
    run:
      - npm install
      - npm run build

  # 2. Test Stage
  - name: Run Tests
    run:
      - npm run lint
      - npm test

  # 3. Packaging Stage (Only runs if tests pass)
  - name: Package Artifact
    run:
      - tar -czf release-artifact.tar.gz ./dist/
      - upload-to-registry release-artifact.tar.gz

  # 4. Deploy Stage (Deploys the packaged artifact)
  - name: Deploy to Production
    run:
      - ssh prod-server "pull-and-extract-artifact release-artifact.tar.gz"
      - ssh prod-server "restart-application"
```

# Best Practices
- **Run the exact same build once:** Do not compile your code multiple times for different environments. Compile your build artifact once in the build stage, and deploy that *exact same* artifact to staging, testing, and production. This ensures that compiler differences don't introduce bugs in production.
- **Fail Fast:** Place fast-running checks (like code linters and syntax checks) at the very beginning of the pipeline. If a developer forgot a semicolon, the pipeline should fail in 5 seconds, rather than waiting 20 minutes for heavy integration tests to run.
- **Keep Pipelines Fast:** Optimize your pipeline run times by caching dependencies (like `node_modules` folders) between runs. A pipeline that takes 30 minutes to complete discourages developers from pushing small, frequent code updates.

# Industry Standards
Modern engineering organizations prioritize the **Four Key DORA Metrics** to measure pipeline efficiency:
- **Deployment Frequency:** How often code is successfully released to production.
- **Lead Time for Changes:** The time it takes for a commit to go from code push to running in production.
- **Change Failure Rate:** The percentage of deployments that cause a failure in production.
- **Mean Time to Restore (MTTR):** The time it takes to recover from a production outage.

# Common Mistakes
- **Skipping Automated Testing:** Setting up a CI/CD pipeline that builds and deploys code but runs zero tests. This simply automates the process of shipping bugs to production faster.
- **Hardcoding Server Secrets in Pipeline Code:** Saving production SSH keys or database passwords directly inside the `pipeline.yml` file. Hackers can read these files easily. Always use the pipeline platform's encrypted **Secrets Vault**.
- **Allowing Flaky Tests to Pass:** Tolerating "flaky tests"—tests that fail 20% of the time due to timing issues, but pass on retry. Developers will lose trust in the pipeline alarms and eventually approve a real bug.

# Security & Performance Considerations
- **Pipeline Runner Isolation:** Pipeline runners execute raw terminal commands defined in project files. If a project folder is compromised, an attacker can modify the pipeline file to print out environment secrets or run cryptocurrency miners on your runners. Always run pipelines inside isolated, throwaway containers.
- **Artifact Version Poisoning:** If your artifact registry does not use strict version pinning (e.g. tagging every release with a commit hash like `v1.2.0-a8d2e`), and instead overwrites a generic tag like `latest`, a failed build can overwrite a good build, resulting in rolling out broken code during server reboots.

# Related Technologies
- **Jenkins:** An open-source, highly extensible automation server for custom pipelines.
- **GitHub Actions:** GitHub's managed, YAML-based pipeline runner service.
- **GitLab CI:** A built-in, YAML-driven CI/CD runner native to GitLab.

# Summary

## What We Learned
- CI/CD pipelines automate the testing, building, packaging, and deployment of software to ensure consistency.
- Continuous Integration focuses on automated building and testing; Continuous Delivery prepares deployable artifacts; Continuous Deployment releases them automatically.
- DORA metrics are used by modern teams to measure delivery speed, frequency, and stability.

## Key Takeaways
- Compile the build artifact once and promote the exact same bundle through all testing environments.
- Protect production servers by keeping credentials stored securely in encrypted pipeline secrets.
- Optimize build runner speeds by caching packages and placing fast syntax checks first.

# Keywords
- Continuous Integration (CI)
- Continuous Delivery
- Continuous Deployment
- Build Artifact
- Pipeline Runner
- Artifact Registry
- DORA Metrics
- Webhook
- Flaky Test
- Secrets Vault

# Glossary

| Term | Meaning |
|---|---|
| Pipeline | An automated sequence of stages (build, test, deploy) that code passes through to be released. |
| Artifact | The compiled and packaged version of an application (like a zip or Docker image) ready for execution. |
| Webhook | An HTTP notification sent by a server (like GitHub) to trigger an external action (like a build). |
| Flaky Test | A test query that exhibits both passing and failing results without any changes to the code. |
| DORA Metrics | Industry standard metrics (Deployment Frequency, Lead Time, MTTR, Failure Rate) used to measure DevOps efficiency. |
| Secrets Vault | A secure, encrypted database managed by pipeline providers to store deployment passwords and keys. |

## Next Recommended Chapters
- 04-GitHub-Actions.md
