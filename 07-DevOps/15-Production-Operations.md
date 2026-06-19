> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Production Operations** is the collection of engineering strategies, deployment methodologies, and incident response practices used to safely manage live software environments. It ensures that application updates are deployed without interrupting users, systems recover automatically from infrastructure faults, and engineering teams can quickly resolve issues when outages occur.

# Why It Exists
Historically, deploying software was a high-risk event. Developers would upload new code directly to live servers all at once, replacing the old application. If the new code had a bug, the website crashed for 100% of users, and recovery took hours of panic-inducing troubleshooting. Production Operations was created to turn code releases into gradual, reversible steps. By using smart traffic routing, automated rollback safety nets, and controlled failure testing, teams can deploy code daily without risking service uptime.

# Problem It Solves
Production Operations solves the problems of high-risk releases, long recovery times during outages, untested system recovery behaviors, and chaotic emergency responses.

### Before Production Operations (All-or-Nothing Releases):
- Upgrading software required taking the site down and displaying a "maintenance window" banner to users.
- A single bug in a new release affected every user instantly, requiring hours of manual code edits to fix.
- On-call engineers had to guess how to troubleshoot complex server alerts under high pressure at 3 AM.

### After Production Operations (Zero-Downtime Operations):
- Upgrades are deployed side-by-side with old versions, allowing immediate, zero-downtime transitions.
- Traffic is slowly funneled to new releases, restricting bug exposure to a tiny fraction of users before full rollout.
- Automated safety checkers detect rising error rates and revert (rollback) deployments instantly without human intervention.

# Core Concepts
Managing live production environments safely relies on progressive delivery and system resilience:

1. **Blue-Green Deployment:** A release strategy using two identical production environments: "Blue" (which runs the current live version) and "Green" (which runs the new version). Code is deployed to Green, verified privately, and then traffic is switched instantly at the router level from Blue to Green.
2. **Canary Deployment:** A release strategy where a new software version is deployed to a small subset of servers. Only a tiny percentage of user traffic (e.g. 5%) is routed to these servers. If the version is stable, traffic is gradually increased to 100%.
3. **Rollback:** The automated or manual process of returning a system to its previous stable version immediately after a deployment failure is detected.
4. **Chaos Engineering:** The practice of proactively injecting controlled failures (like cutting network connections or shutting down servers) directly into production to test if the system's automated backup systems work correctly.
5. **Runbooks (or Playbooks):** Detailed, step-by-step written guides that tell on-call engineers exactly how to diagnose, troubleshoot, and fix specific system alerts.

# Architecture / Components
Traffic routing models during Blue-Green and Canary deployments:

### Blue-Green Deployment (Track Switching)
```text
  [ Active Users ]                      [ Active Users ]
         │                                     │
         ▼                                     ▼
  [ Traffic Router ]                    [ Traffic Router ]
    ├──► [ Blue Environment (v1) ]        │    [ Blue Environment (v1) ]
    └─── [ Green Environment (v2) ]       └──► [ Green Environment (v2) ]
         (Deploying & Testing)                 (Traffic Switched - Active)
```

### Canary Deployment (Gradual Splitting)
```text
                       [ All User Traffic ]
                                │
                                ▼
                       [ Load Balancer ]
                        ├─── 95% ───► [ Stable Service Pool (v1.0.0) ]
                        └───  5%  ───► [ Canary Service Pool (v1.1.0) ]
```

# Workflow
The steps involved in running a safe Canary Deployment and automated rollback:

```text
Step 1: The CI/CD system builds the new application version (v2.0.0).
                             ↓
Step 2: Version v2.0.0 is deployed to a small group of canary servers (5% of the fleet).
                             ↓
Step 3: The load balancer redirects 5% of public traffic to the new canary servers.
                             ↓
Step 4: Monitoring agents track the error rate and performance of the canary servers.
                             ↓
Step 5: A memory leak causes the canary servers to return HTTP 500 error codes.
                             ↓
Step 6: The monitoring tool detects the SLO violation and triggers an automated alert.
                             ↓
Step 7: The deployment controller triggers a rollback, pointing 100% of traffic back to the stable servers.
                             ↓
Step 8: Version v2.0.0 is shut down, and developers review the system logs to diagnose the memory leak.
```

# Real World Examples
Think of Production Operations strategies as **railway switches, restaurant menu testing, and fire drills**.
- **Blue-Green Deployment (Railway Track Switch):** Imagine a train station with two identical parallel tracks: Track A (Blue) and Track B (Green). Passengers are boarding the train on Track A. Meanwhile, a brand new train is prepared and tested on Track B. Once ready, the dispatcher flips a lever (updates the router). All incoming passengers are instantly directed to Track B. If the new train has an issue, the dispatcher simply flips the lever back, routing passengers back to the train on Track A.
- **Canary Deployment (Restaurant Menu Test):** A restaurant chef wants to introduce a new spicy pasta recipe. Instead of changing the menu for all diners immediately, they serve the new recipe to only 5% of tables (the canary test group). If those diners complain that the pasta is too spicy (errors), the chef removes the recipe immediately (rollback). If the diners love it, the chef increases it to 25%, 50%, and eventually updates the entire menu.
- **Chaos Engineering (Fire Drills):** A building manager intentionally sets off the fire alarms when the building is full. This isn't done to cause panic, but to verify that the alarm sensors work, the emergency doors open, and people know the evacuation paths. Doing this regularly ensures the building remains safe during a real emergency.

# Implementation
Here is how engineers configure a Kubernetes Ingress resource using Nginx annotations to split traffic for a Canary deployment, routing 10% of user requests to the canary service:

### Implementing a 10% Canary Traffic Split in Kubernetes (YAML)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-app-ingress-canary
  namespace: production
  annotations:
    # 1. Enable the Nginx controller's canary feature
    nginx.ingress.kubernetes.io/canary: "true"
    # 2. Direct exactly 10% of incoming requests to this ingress backend
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  ingressClassName: nginx
  rules:
    - host: my-app.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                # 3. Direct the 10% split to the canary deployment service
                name: web-app-canary-service
                port:
                  number: 80
```

# Best Practices
- **Decouple Deployments from Releases:** Use **Feature Flags** (software switches in code). This allows you to deploy code to production inertly, and then turn features on or off for users dynamically without redeploying code.
- **Maintain Database Backwards Compatibility:** When using Blue-Green deployments, ensure your database schema works with both the old and new code versions simultaneously. If the new code requires a database structure change, apply it in a way that does not break the active, older application.
- **Automate the Rollback Trigger:** Do not wait for a human to review charts during a failed deployment. Define clear, automated metrics (like latency spikes or error count thresholds) that trigger automatic rollbacks within seconds.

# Industry Standards
Modern production environments run **Game Days**. Teams simulate major outages (e.g. database failure, cloud zone outage) in a staging or production environment to practice incident response. These sessions test both the system's automated recovery systems and the on-call team's ability to coordinate and troubleshoot using runbooks.

# Common Mistakes
- **Deploying Database Schema Changes Too Late:** Changing database tables directly during a Blue-Green deployment swap. If you rename a table column and then need to roll back the code, the older version will crash because the old column is gone. Use multi-step migration strategies (Add Column -> Deploy Code -> Remove Old Column).
- **Ignoring Canary Performance Data:** Running canary deployments but failing to check their metrics, or letting a canary run for days without completing the promotion to 100% traffic.
- **Failing to Test Rollbacks:** Designing complex automated rollback pipelines but never testing them in test environments. A rollback process must be tested as thoroughly as the deployment process itself.

# Security & Performance Considerations
- **Session Persistence (Sticky Sessions):** When transitioning users between versions, ensure their session data (like shopping cart contents) is shared or stored in a centralized cache (like Redis). Otherwise, when users are routed to the new version, they will be logged out.
- **Blast Radius Reduction:** Keep the "blast radius" (the number of users impacted by a failure) as small as possible. Run canary tests during low-traffic periods, and start with the smallest possible traffic percentage (e.g. 1%).

# Related Technologies
- **Argo Rollouts:** A Kubernetes controller that provides automated Blue-Green and Canary deployment strategies.
- **LaunchDarkly:** A cloud platform used to manage feature flags dynamically.
- **Chaos Mesh / Gremlin:** Tools used to run chaos engineering failure injection tests.

# Summary

## What We Learned
- Production Operations focuses on safely upgrading and maintaining live applications.
- Blue-Green deployments switch traffic instantly between two identical environments, while Canary deployments slowly phase traffic in.
- Rollbacks must be automated using telemetry threshold safety checks, and database updates must remain backwards compatible.

## Key Takeaways
- Use canary releases to limit the blast radius of new bugs to a small percentage of users.
- Practice chaos engineering and run regular game days to ensure system reliability under pressure.
- Document step-by-step runbooks for every alert to guide on-call engineers during outages.

# Keywords
- Production Operations
- Blue-Green Deployment
- Canary Deployment
- Rollback
- Chaos Engineering
- Runbook
- Blast Radius
- Game Day
- Feature Flags
- Zero-Downtime Release

# Glossary

| Term | Meaning |
|---|---|
| Blue-Green Deployment | A deployment technique that switches user traffic instantly from an old environment to a new one. |
| Canary Deployment | A release strategy that rolls out updates to a small fraction of users before full deployment. |
| Rollback | Reverting an application to its previous stable version after a failed release. |
| Chaos Engineering | Intentionally injecting system faults to test stability and recovery systems. |
| Runbook | A written guide containing step-by-step procedures for handling system alerts and outages. |
| Blast Radius | The share of users or systems affected when a software release or component fails. |

## Next Recommended Chapters
- 08-Cloud/01-Cloud-Fundamentals.md
