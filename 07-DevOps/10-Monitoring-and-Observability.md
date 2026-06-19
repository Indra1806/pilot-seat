> **Mode:** Book
> **Pilot-Seat Standard**

# Introduction
**Monitoring and Observability** in DevOps covers the operational implementation of telemetry agents, metric scrapers, visualization dashboards, and alert notification managers. This chapter focuses on the setup and tuning of the industry-standard **Prometheus and Grafana** stack to track cluster health, diagnose failures, and manage alerts without inducing notification fatigue.

# Why It Exists
Deploying applications inside container clusters like Kubernetes makes checking server health difficult. In a traditional setup, administrators checked if a server was running by trying to connect to it. In a clustered container environment where containers spin up, move, and shut down automatically, checking health manually is impossible. Operations teams need automated systems that continuously query (scrape) metrics from running containers, store them in time-series databases, and send instant notifications when things break.

# Problem It Solves
Monitoring and observability solve undetected system failures, delayed crash diagnostics, and overwhelming alarm flood fatigue (alert fatigue).

### Before Monitoring & Alerting (Passive Ops):
- Systems crashed silently, and companies only discovered the outage when customers called support.
- Locating why a query was running slowly required logging into multiple servers manually to check memory usage.
- Administrators received thousands of redundant email notifications for minor, temporary spikes, causing them to ignore all alerts.

### After Monitoring & Alerting (Proactive Ops):
- Systems continuously export metrics, displaying real-time cluster health on large wall-mounted screens (Grafana).
- Automated alert managers group related notifications and page the on-call engineer within seconds of a critical failure.
- Systems calculate health trends, warning administrators *before* a hard drive fills up completely.

# Core Concepts
To design monitoring systems, you must understand metrics collection styles and alert management:

1. **Pull-Based Metrics Collection (Scraping):** The collection method where the monitoring server (like Prometheus) periodically queries (pulls) metrics from a list of registered application endpoints (targets). This prevents the application servers from overloading the monitoring server with push traffic.
2. **Time-Series Database (TSDB):** A database optimized for storing sequences of values mapped to timestamps over time (e.g. tracking CPU usage percentages recorded every 5 seconds).
3. **Alert Deduplication and Grouping:** The practice of consolidating multiple related warnings into a single, unified alert notification. (e.g., if a database goes down, instead of sending 50 separate emails for 50 failing web servers, the alert manager sends a single message: *"Database offline: 50 web nodes affected"*).

# Architecture / Components
The flow of metrics data from target application pods to visual dashboards and paging services:

```text
  [ Application Pod 1 ] ──> Exposes /metrics text page
  [ Application Pod 2 ] ──> Exposes /metrics text page
             ▲
             │ (Prometheus pulls / scrapes every 15 seconds)
  [ Prometheus Server ] ──> Saves data to local Time-Series DB (TSDB)
             │
             ├──────────────────────┬──────────────────────┐
             ▼ (Queries data)       ▼ (Forwards alerts)    ▼ (Defines alert rules)
     [ Grafana Dashboard ]    [ AlertManager ]      [ Alert Rules Config ]
     - Visual charts          - Groups alerts       - "If CPU > 90% for 5 mins"
                              - Sends PagerDuty/Slack
```

- **Prometheus Exporters:** Helper agents that collect metrics from specific technologies (like database engines) and format them for Prometheus.
- **AlertManager:** The HashiCorp/CNCF tool that handles alert routing, silencing, and group messaging.

# Workflow
The automated alert escalation loop during a memory leak event:

```text
Step 1: Application Node 1 develops a memory leak. Its memory usage rises to 92%.
                             ↓
Step 2: Prometheus scrapes Node 1's `/metrics` endpoint and records the 92% value in the TSDB.
                             ↓
Step 3: The Prometheus evaluation engine evaluates the rule: `memory_usage > 90% for 5 minutes`.
                             ↓
Step 4: The rule evaluates to TRUE. Prometheus forwards the warning to AlertManager.
                             ↓
Step 5: AlertManager waits 30 seconds to see if other nodes report the same issue, then groups them.
                             ↓
Step 6: AlertManager sends a single, high-priority notification to the on-call engineer's phone via PagerDuty.
```

# Real World Examples
Think of Prometheus, Grafana, and AlertManager as a **school health coordinator, clinic dashboard, and emergency pager system**.
- **Without Monitoring:** You run a school, but you have no nurse or doctor. You only discover a kid is sick when they faint in the hallway.
- **Prometheus (Pull Scraper):** You hire a school nurse (**Prometheus**). Instead of sitting in the office waiting for kids to walk in (Push model), the nurse walks to every classroom desk (**Target**) every hour (**Scrape Interval**) with a thermometer, takes the kid's temperature (**Metrics**), and writes it down in a logbook (**TSDB**).
- **Grafana (Clinic Dashboard):** The school principal wants to see the health of the school. The nurse draws a line chart on a large whiteboard in the office (**Grafana**). The chart shows the average temperature of each classroom. The principal glances at the board and instantly sees if a flu outbreak is starting.
- **AlertManager (Emergency Coordinator):** The nurse writes a rule: *"If a kid's temperature is over 101°F for 3 checkups in a row, notify the parents."*
  - If 5 kids in the same classroom all get high temperatures, the nurse doesn't call the principal 5 separate times. The nurse groups the calls into a single alert: *"Flu outbreak suspected in Room 4B; 5 kids affected."* This is **Alert Grouping** and prevents **Alert Fatigue**.

# Implementation
Here is how developers expose metrics in Node.js and configure a Prometheus scraper to pull them:

### 1. Exposing Application Metrics (Node.js using prom-client)
```javascript
const express = require('express');
const client = require('prom-client'); // Official Prometheus client library

const app = express();

// 1. Enable collection of default system metrics (CPU, Memory, Event Loop Lag)
const collectDefaultMetrics = client.collectDefaultMetrics;
collectDefaultMetrics({ register: client.register });

// 2. Create a custom metric (Request counter)
const httpRequestCounter = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests processed',
  labelNames: ['method', 'route', 'status']
});

app.get('/api/users', (req, res) => {
  // Process request...
  res.send([{ id: 1, name: 'Alice' }]);
  
  // Increment counter with labels
  httpRequestCounter.inc({ method: 'GET', route: '/api/users', status: 200 });
});

// 3. Expose the /metrics endpoint for Prometheus to scrape
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});

app.listen(3000, () => console.log('App running on port 3000'));
```

### 2. Prometheus Configuration File (`prometheus.yml`)
Configure Prometheus to scrape your application every 15 seconds:

```yaml
global:
  scrape_interval: 15s # How often to query targets

# Define the list of servers to monitor
scrape_configs:
  - job_name: 'node_web_app'
    static_configs:
      # Target is the IP/Port where our Node app is running
      - targets: ['localhost:3000']
```

# Best Practices
- **Define Symptom-Based Alerts, Not Cause-Based:** Do not pager-alert engineers for minor CPU spikes. An engineer should only be woken up at night if users are experiencing issues (e.g. HTTP 500 error rates exceed 5% or page latency exceeds 2 seconds). High CPU is a *cause*, but if the page is still fast, it is not an emergency.
- **Implement Alert Deduplication:** Configure your AlertManager to group alerts by service name or error class. This prevents your team's pager from ringing continuously during a database outage.
- **Expose a lightweight `/metrics` endpoint:** Ensure the code that compiles metrics is highly optimized. If your `/metrics` endpoint is slow or locks the database to count rows, the scraper itself will slow down the application.

# Industry Standards
Modern Kubernetes observability stacks use the **Prometheus Operator**. This operator automates the installation, configuration, and scaling of Prometheus instances inside Kubernetes clusters, allowing developers to register new target pods simply by adding a custom resource file called a **ServiceMonitor**.

# Common Mistakes
- **Creating Too Many Alerts:** Sending a Slack or pager alert for every minor system warning. This leads to alert fatigue, causing developers to mute the alarm channels and eventually miss a real production crash.
- **High Cardinality Metrics:** Creating custom metrics with high-cardinality labels, such as placing a unique `user_id` or `session_id` inside a metric label (e.g. `http_requests_total{user_id="105289"}`). This creates millions of unique time-series entries, bloating Prometheus memory and eventually crashing the monitoring server.
- **Monitoring from inside the same network:** Hosting your monitoring servers inside the exact same cloud cluster as your application. If the cluster crashes or loses network connectivity, your monitoring system goes down with it, leaving you blind to the outage.

# Security & Performance Considerations
- **Exposing `/metrics` to the Public:** The `/metrics` endpoint reveals detailed internal names, IPs, database query rates, and operating system versions of your server. Leaving this page open to the public internet allows attackers to easily map your internal infrastructure. Secure the endpoint by restricting access to the Prometheus server IP only.
- **Prometheus Memory Allocation:** Prometheus stores active metrics in RAM before flushing them to disk. Under high traffic, if you scrape thousands of metrics, Prometheus will consume significant memory. Administrators must configure memory caps (`storage.tsdb.retention.size`) to prevent out-of-memory crashes.

# Related Technologies
- **PromQL:** Prometheus Query Language, used to query time-series metrics.
- **Grafana Loki:** A log aggregation system designed to integrate seamlessly with Grafana and Prometheus.
- **PagerDuty:** A managed incident response platform that coordinates on-call schedules and pages engineers during alerts.

# Summary

## What We Learned
- Monitoring and alerting systems proactively measure cluster health to detect and notify administrators of failures.
- Prometheus scrapes metrics from target endpoints, storing them in a time-series database.
- AlertManager groups related warnings to prevent alert fatigue, while Grafana visualizes metrics on customizable dashboards.

## Key Takeaways
- Restrict access to `/metrics` endpoints to protect internal server metadata from public scrapers.
- Focus pager alarms on user-facing symptoms (errors, latency) rather than internal hardware indicators (CPU).
- Avoid high-cardinality labels (like User IDs) in custom Prometheus metrics to prevent database memory exhaustion.

# Keywords
- Prometheus
- Grafana
- Scrape Interval
- Time-Series Database (TSDB)
- AlertManager
- Alert Fatigue
- Cardinallity
- Exporter
- PromQL
- ServiceMonitor

# Glossary

| Term | Meaning |
|---|---|
| Scraping | The pull-based process where a monitoring server queries target endpoints to fetch metrics data. |
| Time-Series Database | A database optimized for storing values mapped to sequential timestamps over time. |
| Cardinallity | In monitoring, this refers to the number of unique combinations of label values in a metric. |
| Exporter | A helper utility that translates third-party system stats into Prometheus-compatible formats. |
| PromQL | The query language used to filter, aggregate, and calculate statistics on Prometheus metrics. |
| Alert Fatigue | A state where operations teams become desensitized to system alarms due to receiving too many false alerts. |

## Next Recommended Chapters
- 11-Logging-Systems.md
