> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**DevSecOps** (Development, Security, and Operations) is the practice of integrating security checks directly into every stage of the software development and deployment lifecycle. Instead of treating security as a final review before launching an application, DevSecOps builds security automated testing directly into the continuous integration and deployment (CI/CD) pipelines.

# Why It Exists
Historically, software development followed a model where developers wrote code, operators deployed it, and a separate security team audited the application right before it went live. If the security team found a vulnerability, the release had to be canceled, forcing developers to rewrite core parts of the application under intense deadline pressure. If the release couldn't be delayed, the application was launched with known security flaws. DevSecOps was created to automate security checks so that code is scanned for vulnerabilities continuously, preventing security issues from reaching production in the first place.

# Problem It Solves
DevSecOps solves the problems of late-stage security bottlenecks, human error in security audits, and undetected vulnerabilities in external code libraries.

### Before DevSecOps (Traditional Security Audits):
- Security audits were conducted manually at the very end of the project, delaying launches by days or weeks.
- Developers did not discover they were using vulnerable third-party packages until their code was ready for production.
- API keys and passwords were occasionally committed to public repositories by mistake, exposing company data to hackers.

### After DevSecOps (Continuous Security Integration):
- Security scans run automatically on every code change, giving developers immediate feedback within minutes.
- Automated dependency scanners check third-party libraries for newly discovered security threats before they are merged.
- Pipeline check gates scan code for accidentally hardcoded secrets and block commits before they leave the developer's computer.

# Core Concepts
DevSecOps relies on automated scanning and credential security:

1. **Shift Left:** The practice of moving security testing earlier in the software development lifecycle (to the "left" of the timeline, where code is being written) rather than at the end.
2. **Static Application Security Testing (SAST):** Automated tools that inspect source code files line-by-line looking for insecure coding patterns (like SQL injection risks or unsafe text inputs) without running the application.
3. **Dynamic Application Security Testing (DAST):** Automated tools that scan a running instance of the application from the outside, simulating hacker behavior (like injecting malicious scripts into input forms) to find active entry points.
4. **Software Composition Analysis (SCA):** Automated tools that catalog every third-party package and dependency used by the application and cross-reference them against global vulnerability databases (like CVE databases).
5. **Secrets Management:** The practice of storing application passwords, database credentials, and API keys inside a secure, centralized vault instead of writing them directly in application configuration files.

# Architecture / Components
The flow of security scanning through a DevSecOps CI/CD pipeline:

```text
  [ Developer Commits Code ]
              │
              ▼
  ┌────────────────────────────────────────────────────────┐
  │ CI/CD Security Pipeline Check                          │
  │                                                        │
  │  1. Secrets Scanner: Check for hardcoded API keys      │
  │  2. SCA Scanner: Check third-party packages for CVEs   │
  │  3. SAST Scanner: Check source code for syntax flaws   │
  └────────────────────────────────────────────────────────┘
              │
     (Passes scan gates?)
       ├─── No ───► [ Block Merge & Alert Developer ]
       │
       └─── Yes ──► [ Deploy to Test Environment ]
                          │
                          ▼
             ┌──────────────────────────┐
             │ DAST Scanner: Run tests  │
             │ against running app APIs │
             └──────────────────────────┘
                          │
                  (Passes checks?)
                    ├─── No ───► [ Rollback & Alert ]
                    │
                    └─── Yes ──► [ Deploy to Production ]
```

# Workflow
The operations of a DevSecOps pipeline during a typical code update:

```text
Step 1: A developer updates a library package and writes new code to handle database records.
                             ↓
Step 2: The developer pushes the code branch to GitHub.
                             ↓
Step 3: GitHub Actions triggers the security pipeline. An SCA scanner checks the updated library and finds a known exploit.
                             ↓
Step 4: The pipeline fails the build, blocks the pull request from being merged, and alerts the developer.
                             ↓
Step 5: The developer updates the library to a patched version and pushes the fix.
                             ↓
Step 6: The pipeline runs again, passes all SAST and SCA checks, and builds the container image.
                             ↓
Step 7: The image is launched in a staging environment where a DAST tool probes the API endpoints for weaknesses.
                             ↓
Step 8: Once the DAST tool reports clean results, the application is deployed to production.
```

# Real World Examples
Think of DevSecOps as **an automated airport security screening system**.
- **Traditional Security (Manual Audit):** Passengers board the plane, find their seats, and strap in. Right before the doors close, security officers board the plane to search every suitcase and passenger by hand. If they find a banned item, the flight is delayed, bags must be unloaded, and everyone is frustrated.
- **DevSecOps (Continuous Scanning):** Security is integrated into the entire airport journey:
  - **SCA / Check-In Gate:** The airline check-in system checks passenger IDs against international safety databases before they can even print a boarding pass.
  - **SAST / Carry-On X-Ray:** Carry-on bags are placed on a conveyor belt and scanned by an automated X-ray machine. It looks inside the closed bag (without opening it) to search for forbidden shapes (like liquid bottles or metal blades).
  - **DAST / Metal Detector:** Passengers walk through a metal detector and body scanner. This is a dynamic test on the active passenger (the running system) to see if they set off alarms.
  - **Secrets Vault:** Instead of carrying cash and high-value keys in their pockets, travelers keep their valuables locked in a secure safe box at the hotel, using temporary plastic key cards that expire when they check out.

# Implementation
Here is how developers configure a GitHub Actions workflow to run **Trivy** (a popular open-source SCA and container scanner) to audit code and dependencies:

### Running Dependency and Security Scans in GitHub Actions
```yaml
name: DevSecOps Security Scan

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  security-audit:
    name: Run Vulnerability Scanners
    runs-on: ubuntu-latest

    steps:
      # 1. Check out the application source code
      - name: Checkout Code
        uses: actions/checkout@v4

      # 2. Audit Node.js dependency files for known vulnerabilities
      - name: Run NPM Audit
        run: |
          npm ci
          npm audit --audit-level=high # Fail the build if high-risk bugs are found

      # 3. Run Trivy to scan the source code repository for misconfigurations and exposed secrets
      - name: Run Trivy FS Scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          exit-code: '1' # Tells GitHub Actions to fail the job if bugs are found
          severity: 'CRITICAL,HIGH'

      # 4. Inject secure API tokens at runtime instead of hardcoding them
      - name: Deploy to Cloud
        env:
          # Pull values from GitHub Encrypted Secrets store rather than source files
          CLOUD_API_TOKEN: ${{ secrets.CLOUD_API_TOKEN }}
        run: |
          echo "Deploying application safely..."
```

# Best Practices
- **Define Strict Failure Thresholds:** Configure security scanners to ignore minor warnings but immediately fail builds on `HIGH` or `CRITICAL` issues. If scanners block code for every tiny warning, developers will bypass the checks entirely.
- **Automate Secret Protection:** Use tools like `git-secrets` or Git pre-commit hooks to scan files on developers' local machines. This stops API keys from ever being written into a local commit record.
- **Audit Third-Party Packages Continuously:** Run scheduled security pipeline builds daily, even if no new code has been written. A library that was secure yesterday might have a new vulnerability discovered and cataloged today.

# Industry Standards
Modern engineering organizations implement **Software Bills of Materials (SBOMs)**. An SBOM is an automated inventory list of every single software component, package, license, and utility used to build an application. If a major open-source vulnerability is announced (like the Log4j incident), security teams query their SBOM inventory directory to instantly identify which applications need to be patched.

# Common Mistakes
- **Treating Scanning as a Checklist Item:** Turning on security scanners but allowing developers to bypass alerts with a simple override without code review.
- **Hardcoding "Test" Credentials:** Leaving real database keys inside testing configurations or mock folders, assuming they are safe because they are not "production" credentials.
- **Ignoring Secrets in Git History:** Deleting an API key from a file in a new commit, but leaving it inside the repository's historical git commit records. Attackers scan public Git history to steal credentials. Leaked keys must be immediately revoked and replaced.

# Security & Performance Considerations
- **Pipeline Execution Latency:** Complex SAST and container image scanning can take 15–20 minutes to complete. To keep pipelines fast, run quick scans (like secret detection and dependency audits) on every commit, and reserve heavy scans (like deep container audits and DAST) for nightly builds.
- **False Positive Tuning:** Security tools run on pattern matching and frequently flag safe code as unsafe. Regularly review and tune scanner configuration rule files to keep false alerts low.

# Related Technologies
- **Trivy:** An open-source security scanner that checks codebases, filesystems, and container images.
- **Snyk:** A cloud platform that monitors application dependencies and code for security threats.
- **HashiCorp Vault:** A secure system used to store, manage, and distribute sensitive API tokens and credentials.

# Summary

## What We Learned
- DevSecOps embeds security checks directly into the CI/CD pipeline, shifting security validation to the early development stages.
- SAST scans raw code patterns, SCA monitors third-party libraries for known vulnerabilities, and DAST scans live, running applications.
- Credential keys should never be committed to code repositories; instead, use environment variable injection and secure vaults.

## Key Takeaways
- Prevent credential leaks by implementing local git pre-commit secrets scanning tools.
- Configure dependency checkers to fail builds when high-severity vulnerabilities are detected.
- Maintain a clean SBOM to quickly locate and patch compromised libraries when global vulnerabilities are announced.

# Keywords
- DevSecOps
- Shift Left
- SAST
- DAST
- SCA
- Secrets Management
- Trivy
- SBOM
- Vulnerability Scanning
- Hardcoded Credentials

# Glossary

| Term | Meaning |
|---|---|
| Shift Left | Moving security reviews and tests to the early stages of software development. |
| SAST | Static Application Security Testing; scanning code files for vulnerability patterns without running them. |
| DAST | Dynamic Application Security Testing; scanning a live, running application to find security issues. |
| SCA | Software Composition Analysis; checking third-party dependencies against vulnerability registries. |
| SBOM | Software Bill of Materials; a complete list of every package and ingredient used in a software build. |
| Secrets Vault | A specialized database designed to securely store and distribute sensitive passwords and API keys. |

## Next Recommended Chapters
- 14-SRE-Fundamentals.md
