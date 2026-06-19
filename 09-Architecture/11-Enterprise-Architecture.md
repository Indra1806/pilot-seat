> **Mode:** Book
> **Pilot-Seat Standard**

---

# Chapter 11: Enterprise Architecture

---

## 1. Introduction

Imagine you have been hired to run the technology department of a company with 30,000 employees, 400 software applications, 12 data centers, 6 acquired subsidiaries all running different tools, and a board demanding a "digital transformation" by next year. Where do you even begin?

This is the world that **Enterprise Architecture (EA)** was built for.

Enterprise Architecture is the practice of creating and maintaining a comprehensive, structured description of an entire organization — its business goals, its processes, its data, its applications, and its technology — and then using that description to make better decisions about where to invest, what to build, and how to change over time.

In the simplest terms: **EA is the map of your entire organization's technology landscape, aligned to the business strategy.**

It sits above software architecture, above solution architecture, and even above IT strategy. It is the view from 30,000 feet — the aerial photograph that lets you see not just one building, but the entire city and how every road, river, and district connects.

This chapter walks you through what EA is, why it exists, how its major frameworks work, and how real organizations use it to navigate enormous complexity.

---

## 2. Why It Exists

### The Chaos That Comes Before EA

Every organization that grows fast, acquires other companies, or lets individual teams make independent technology choices eventually ends up in the same place: **accidental complexity**.

Think about what happens inside a large bank over 30 years:

- The mortgage team builds a custom loan-origination system in COBOL.
- The retail banking team buys a CRM tool from Salesforce.
- The mobile team builds a React Native app that talks to 8 different backends.
- The analytics team buys a data warehouse.
- Three companies are acquired, each with their own email systems, HR platforms, and customer databases.

Nobody planned this. Every decision made local sense. But now there are **400+ applications**, **duplicated data**, **no single view of the customer**, and changing one system breaks three others. The IT budget swells, but the business gets slower.

Enterprise Architecture exists because **without deliberate design at the organizational level, complexity grows faster than value**.

### The City Planner Analogy

Think of EA like **city urban planning**.

A city planner does not design individual buildings. That is the architect's job. Instead, the city planner designs:
- **Zones** (residential, commercial, industrial — like business domains)
- **Roads and highways** (the connections between systems)
- **Utilities** (shared infrastructure — like cloud platforms and identity services)
- **Building codes** (architecture standards and principles)
- **Long-range growth plans** (the technology roadmap)

Individual engineers (software architects, developers) are like individual building architects — they design their building to code, in the right zone, connected to the right utilities. The city planner makes sure all the buildings together form a functional city that can grow for decades.

Without a city planner, you get sprawl: roads that dead-end, utilities that cannot handle growth, industrial zones next to schools, and no room to expand.

---

## 3. Problem It Solves

Enterprise Architecture directly addresses several painful organizational problems:

### Problem 1 — Duplication and Redundancy
Multiple teams independently build or buy systems that do the same thing. EA creates an **Application Portfolio** that makes duplication visible, so the organization can consolidate.

### Problem 2 — Integration Chaos
Systems cannot talk to each other because they were built independently. EA defines **integration patterns and standards**, so new systems know how to connect from day one.

### Problem 3 — Misaligned IT Investment
IT spending does not reflect business priorities. EA creates a link between **business capabilities** and the technology that supports them, making investment decisions rational.

### Problem 4 — Slow Response to Change
When a regulation changes or a new market opens, the organization cannot respond because no one knows which systems are affected. EA maps **dependencies** so impact analysis becomes possible.

### Problem 5 — Technology Debt and Obsolescence
Old systems linger because nobody wants to touch them. EA creates **technology roadmaps** that deliberately plan for modernization and retirement.

### Problem 6 — Acquisition Chaos
Merging with another company is catastrophic when there is no architectural view of either organization. EA provides the blueprint for integration planning.

---

## 4. Core Concepts

### 4.1 The Architecture Hierarchy

EA is not the only kind of architecture — it exists in a hierarchy:

```
+----------------------------------------------------------+
|          ENTERPRISE ARCHITECTURE (EA)                    |
|   "The whole organization's technology landscape"        |
|   Scope: All business units, all systems, all data       |
+----------------------------------------------------------+
         |                        |
         v                        v
+-------------------+    +--------------------+
| SOLUTION          |    |  SOLUTION          |
| ARCHITECTURE      |    |  ARCHITECTURE      |
| "One project or   |    |  "Another project" |
| program"          |    |                    |
+-------------------+    +--------------------+
         |
         v
+-------------------+
| SOFTWARE          |
| ARCHITECTURE      |
| "One application" |
+-------------------+
         |
         v
+-------------------+
| IMPLEMENTATION    |
| "Code"            |
+-------------------+
```

- **Enterprise Architecture** — the whole organization (hundreds of apps, thousands of services)
- **Solution Architecture** — a specific program or initiative (the "digital banking transformation project")
- **Software Architecture** — a specific application (the mobile banking app)
- **Implementation** — actual code

EA sets the rules, patterns, and boundaries. Solution architects work within those rules. Software architects work within solution boundaries.

### 4.2 The Four Architecture Domains

Almost every EA framework organizes the enterprise into **four architecture domains**:

```
+------------------------------------------------------------------+
|  BUSINESS ARCHITECTURE                                           |
|  Business capabilities, processes, organizational structure,     |
|  strategy goals, value streams                                   |
+------------------------------------------------------------------+
         |
         v
+------------------------------------------------------------------+
|  APPLICATION ARCHITECTURE                                        |
|  Application portfolio, application interactions,                |
|  services exposed by each system                                 |
+------------------------------------------------------------------+
         |
         v
+------------------------------------------------------------------+
|  DATA ARCHITECTURE                                               |
|  Data entities, data ownership, data flows, master data,        |
|  data governance                                                 |
+------------------------------------------------------------------+
         |
         v
+------------------------------------------------------------------+
|  TECHNOLOGY ARCHITECTURE                                         |
|  Infrastructure, cloud platforms, networks, servers,            |
|  middleware, operating systems                                   |
+------------------------------------------------------------------+
```

These four layers are sometimes called the **BDAT stack**. Business goals drive applications, applications use data, and all of it runs on technology.

### 4.3 Architecture Principles

Architecture principles are the rules that guide every decision in the organization. They are like the **constitution** of your technology estate. Examples:

- *"Cloud first"* — New systems must be cloud-hosted unless there is a compelling reason not to.
- *"Single source of truth"* — Every data entity has one authoritative system. No duplicated master data.
- *"API-first"* — All systems expose their data and capabilities via APIs. No direct database access.
- *"Buy before build"* — Prefer commercial off-the-shelf before custom development.

### 4.4 Architecture Runway

An **architecture runway** is the set of architectural work done ahead of delivery teams so they have a solid foundation to build on. Just like a real runway that planes need to take off, delivery teams need stable architecture underneath them. If the runway runs out, delivery slows to a crawl.

### 4.5 Reference Architecture

A **reference architecture** is a pre-approved, reusable architectural pattern for a specific problem domain. For example:
- *"Our reference architecture for microservices on Azure"*
- *"Our reference architecture for a data analytics platform"*

Teams use reference architectures as blueprints instead of designing from scratch every time.

### 4.6 Architecture Governance

**Governance** is the process by which the enterprise ensures that architecture decisions are made correctly, consistently, and by the right people. It includes:
- Architecture Review Boards (ARBs)
- Architecture approval workflows
- Standards compliance checking
- Exception management

---

## 5. Architecture / Components

### 5.1 TOGAF — The Open Group Architecture Framework

**TOGAF** is the most widely adopted EA framework in the world. Think of it as the **playbook** that tells enterprise architects how to do their job.

The centerpiece of TOGAF is the **Architecture Development Method (ADM)** — a cycle of phases that guides architects from strategy to implementation:

```
                    [Preliminary]
                    Set up EA capability,
                    define principles
                         |
                         v
              +-----> [A: Architecture Vision] <------+
              |       Define scope, stakeholders,      |
              |       high-level target state          |
              |                |                       |
              |                v                       |
              |    [B: Business Architecture]          |
              |    Current vs target business          |
              |    capabilities and processes          |
              |                |                       |
              |                v                       |
              |    [C: Information Systems Arch.]      |
              |    Application + Data architectures    |
              |                |                       |
              |                v                       |
              |    [D: Technology Architecture]        |
              |    Infrastructure to support C         |
              |                |                       |
              |                v                       |
              |    [E: Opportunities & Solutions]      |
              |    Identify projects, work packages    |
              |                |                       |
              |                v                       |
              |    [F: Migration Planning]             |
              |    Roadmap, transition architectures   |
              |                |                       |
              |                v                       |
              |    [G: Implementation Governance]      |
              |    Oversee delivery, ensure compliance |
              |                |                       |
              |                v                       |
              +---- [H: Architecture Change Mgmt] ----+
                    Monitor, trigger new cycles
```

The ADM is **iterative** — you run through the cycle repeatedly as the business evolves.

TOGAF also defines the **Architecture Repository**, where all architecture artifacts are stored:
- Architecture Principles
- Architecture Models
- Reference Architectures
- Standards Library
- Architecture Decisions

### 5.2 The Zachman Framework

The **Zachman Framework** is an older but still respected EA framework. Instead of a process (like TOGAF ADM), it is a **classification scheme** — a 6x6 grid that categorizes every possible architectural artifact.

```
         WHAT        HOW         WHERE       WHO         WHEN        WHY
       (Data)     (Function)  (Network)   (People)     (Time)    (Motivation)
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Scope | Entities | Processes | Locations | Org Units | Events    | Goals     |
|      | (things) |           |           |           |           |           |
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Owner | Business | Business  | Business  | Work      | Master    | Business  |
|      | Entities | Processes | Geography | Flow      | Schedule  | Plans     |
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Arch. | Logical  | App       | Distributed| Human    | Process   | Business  |
|      | Data Mdl | Functions | Sys Arch. | Interface | Structure | Rules     |
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Build | Physical | System    | Technology| Pres.     | Control   | Rule      |
|      | Data Mdl | Design    | Arch.     | Arch.     | Cycle     | Design    |
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Sub-  | Data     | Programs  | Network   | Security  | Timing    | Rule      |
|Cont. | Defn.    |           | Arch.     | Arch.     | Defn.     | Spec.     |
+------+----------+-----------+-----------+-----------+-----------+-----------+
|Entpr.| Data     | Working   | Network   | User      | Schedule  | Strategy  |
|      | (stored) | System    |           |           |           |           |
+------+----------+-----------+-----------+-----------+-----------+-----------+
```

Each **row** represents a perspective (Scope/Planner → Owner → Architect → Builder → Sub-contractor → Enterprise), and each **column** asks a different question (What / How / Where / Who / When / Why). Each **cell** is a specific type of architectural artifact.

The Zachman Framework is not a process — it is a **taxonomy**. It helps you understand what artifacts should exist, not how to create them.

### 5.3 ArchiMate — The Modeling Language

If TOGAF is the playbook and Zachman is the classification system, **ArchiMate** is the **language** used to draw EA diagrams. It is to enterprise architecture what UML is to software design.

ArchiMate has several layers that map to EA domains:

```
+--------------------------------------------------------------+
|  MOTIVATION LAYER  (Why — goals, drivers, principles)        |
+--------------------------------------------------------------+
|  STRATEGY LAYER    (What — capabilities and resources)       |
+--------------------------------------------------------------+
|  BUSINESS LAYER    (Business processes, actors, services)    |
+--------------------------------------------------------------+
|  APPLICATION LAYER (Application components, services, data)  |
+--------------------------------------------------------------+
|  TECHNOLOGY LAYER  (Infrastructure, networks, platforms)     |
+--------------------------------------------------------------+
|  PHYSICAL LAYER    (Physical equipment, facilities)          |
+--------------------------------------------------------------+
```

### 5.4 Enterprise Integration Patterns

A major part of EA is defining HOW systems communicate. The key patterns:

**Point-to-Point:**
```
[System A] <-----> [System B]
[System A] <-----> [System C]
[System B] <-----> [System C]
```
Simple but does not scale. N systems = N x (N-1)/2 connections. With 10 systems: 45 connections. With 100: 4,950.

**Hub and Spoke (ESB — Enterprise Service Bus):**
```
          [System A]
              |
[System C] --[HUB]-- [System B]
              |
          [System D]
```
All systems connect to a central hub. Easy to add new systems. But the hub is a single point of failure and a bottleneck.

**Message Bus / Event-Driven:**
```
[System A] --publish--> [===== MESSAGE BUS =====] --subscribe--> [System C]
[System B] --publish--> [=======================] --subscribe--> [System D]
```
Systems publish events; other systems subscribe to what they need. Decoupled, scalable, and resilient.

**Data Warehouse vs Data Lake:**
```
DATA WAREHOUSE                      DATA LAKE
- Structured data only              - Structured + unstructured
- Schema on write                   - Schema on read
- Cleaned, transformed (ETL)        - Raw data stored as-is
- Fast queries, known questions     - Flexible, exploratory analytics
- Examples: Redshift, Snowflake     - Examples: S3 + Athena, Delta Lake
```

---

## 6. Workflow

### Real Scenario: A Bank Embarks on Digital Transformation

Let's walk through how an enterprise architect actually works — from the boardroom to the delivery team — using a fictional bank ("NorthBank") with 200 applications, 8,000 employees, and an ambition to launch a fully digital banking product.

---

**Step 1 — Understand the Business Strategy (Weeks 1-2)**

The EA team interviews the C-suite and reads the 3-year strategy document. They extract key **business drivers**:
- Acquire 1 million new digital-only customers
- Reduce operational costs by 20%
- Launch in 3 new countries

From this they derive **Architecture Principles** for the transformation:
- API-first for all customer-facing services
- Cloud-native on Azure (cloud-first principle)
- Single customer record — one Customer MDM system
- Decommission all on-premise middleware within 24 months

---

**Step 2 — Assess the Current State (Weeks 3-6)**

The architects survey all 200 applications, cataloguing:
- What each system does
- Which business capabilities it supports
- Its health (aging, well-maintained, critical, redundant)
- Its integration touchpoints

They produce an **Application Portfolio Map**:

```
+---------------------+--------------------+-------------------+
| BUSINESS CAPABILITY | CURRENT SYSTEM     | HEALTH            |
+---------------------+--------------------+-------------------+
| Customer Identity   | LDAP + 3 custom DB | CRITICAL RISK     |
| Loan Origination    | COBOL system (1987)| HIGH TECH DEBT    |
| Payments            | FiServ + custom    | MODERATE          |
| Mobile Banking      | React Native app   | GOOD              |
| Fraud Detection     | Rules engine       | AGING             |
| Analytics           | Oracle DW          | ADEQUATE          |
+---------------------+--------------------+-------------------+
```

---

**Step 3 — Define the Target Architecture (Weeks 7-12)**

The EA team designs the **Target State Architecture** — what the organization should look like in 3 years:

```
                         CUSTOMERS
                             |
              +-------------------------------+
              |    API GATEWAY (Azure APIM)   |
              +-------------------------------+
                    |              |
           +--------+              +--------+
           |                                |
  [Mobile Banking App]           [Web Banking Portal]
           |                                |
           +--------+    +------------------+
                    |    |
              +-----+----+--------+
              |  MICROSERVICES    |
              |  - Identity Svc   |
              |  - Account Svc    |
              |  - Payment Svc    |
              |  - Loan Svc       |
              +-------------------+
                    |     |
             +------+     +-------+
             |                   |
    [Event Bus: Azure SB]  [Customer MDM]
             |
    [Analytics: Azure Synapse]
```

---

**Step 4 — Plan the Migration (Weeks 13-18)**

The architects build a **Transition Architecture** — the intermediate states between current and target. For NorthBank, this is a 3-phase plan:

- **Phase 1 (Months 1-6):** Stand up API Gateway. Wrap legacy systems with APIs. Do NOT touch internals yet.
- **Phase 2 (Months 7-18):** Build new microservices. Migrate customer data to new Customer MDM. Retire LDAP.
- **Phase 3 (Months 19-36):** Decommission COBOL loan system. Complete cloud migration. Retire on-premise DW.

This becomes the **Technology Roadmap**.

---

**Step 5 — Establish Governance (Ongoing)**

An **Architecture Review Board (ARB)** meets bi-weekly. Every new project must submit an Architecture Decision Record (ADR) before being approved to start. The ARB checks:
- Does the proposed design follow architecture principles?
- Does it use approved reference architectures?
- Does it comply with security and data governance standards?

Projects that deviate must request an exception with documented justification.

---

**Step 6 — Monitor and Evolve (Quarterly)**

The EA team runs quarterly reviews:
- Are delivery teams following the architecture?
- Has the business strategy changed?
- Are new technology options available (e.g., a better cloud service)?

The architecture evolves — it is never "done."

---

## 7. Real World Examples

### 7.1 Large Bank — JPMorgan Chase

JPMorgan Chase operates over **4,000 applications** across consumer banking, investment banking, asset management, and treasury. They use **TOGAF-aligned practices** with a central Enterprise Architecture team that:

- Maintains a complete Application Portfolio catalogue
- Enforces architecture standards via an Architecture Review Board
- Runs a formal decommissioning program (retiring approximately 100 legacy apps per year)
- Manages enterprise-wide data governance through a Chief Data Office aligned to the EA function
- Has published enterprise-wide API standards that all teams must follow

Their EA program helped them identify over $1.2 billion in technology savings by eliminating redundant systems across acquired entities like Bear Stearns and Washington Mutual.

---

### 7.2 Government — UK Government Digital Service (GDS)

The UK Government uses EA at the national level to ensure **interoperability** between hundreds of government agencies. Their approach:

- Published the **Government Service Standard** — a reference architecture that all UK government digital services must conform to
- Defined **GOV.UK Verify** as the shared identity platform (later replaced by One Login)
- Uses the **Technology Code of Practice** as a set of architecture principles (cloud-first, open standards, user-centered design)
- Mandates that all new systems use **REST APIs with JSON** for interoperability
- Maintains a government-wide **spend controls** process — no tech project over £5m proceeds without EA review

This EA-level governance means a citizen's data can flow from HMRC (tax) to DWP (benefits) to NHS (health) using shared identity and data standards.

---

### 7.3 Large Retailer — Walmart

Walmart operates in 24 countries, with both physical stores and a growing e-commerce platform. Their EA challenge is **omnichannel integration** — ensuring a customer can:
- Browse online, pick up in store
- Return online orders in-store
- Get consistent pricing everywhere
- Have one loyalty profile across all channels

Walmart's EA team:
- Defined a **Customer Data Platform** as the single source of truth for all customer interactions
- Built an **Event-Driven Architecture** where store checkout events, website clicks, and app interactions all publish to a central event bus
- Created a **Reference Architecture for Microservices** that all development teams (thousands of engineers) must follow
- Uses an internal **Technology Radar** (similar to ThoughtWorks) to classify technologies as Adopt, Trial, Assess, or Hold
- Runs a formal **Application Portfolio Management** process — any team wanting to build a new application must justify it against the existing portfolio

---

### 7.4 Healthcare — Epic Systems + Hospital Networks

Major hospital networks using Epic Systems as their Electronic Health Records (EHR) platform use EA to:
- Define the **boundary** between Epic (vendor-managed) and custom systems (hospital-managed)
- Create **integration reference architectures** using HL7/FHIR standards for health data exchange
- Govern data flows to ensure **HIPAA compliance** — EA maps which systems touch Protected Health Information (PHI) and ensures encryption, audit logging, and access controls are in place
- Run **Application Portfolio rationalization** across merged hospital systems (a large hospital system may have acquired 5 smaller hospitals, each with their own lab systems, billing systems, and departmental tools)

The EA team is the key to answering: *"If we change the pharmacy system, which other systems are impacted?"*

---

## 8. Implementation

### 8.1 EA Artifacts You Will Actually Produce

Enterprise architects do not write code — they produce **documents and models**. Here are the key artifacts:

**Application Portfolio Map (APM):**
A catalogue of every application in the organization, with metadata about health, owner, business capability served, cost, and integration points.

**Architecture Principles Document:**
A formal document listing principles with rationale, implications, and examples.

**Business Capability Map:**
A structured decomposition of everything the business does, independent of how it is done today.

```
NorthBank Business Capability Map (excerpt):

[Customer Management]
  +-- Customer Onboarding
  +-- Customer Identity and KYC
  +-- Customer Communication
  +-- Customer Analytics

[Product Management]
  +-- Product Design
  +-- Product Pricing
  +-- Product Lifecycle

[Payments]
  +-- Domestic Payments
  +-- International Payments
  +-- Fraud Detection
```

**Architecture Decision Records (ADRs):**
Lightweight documents recording a decision, the context, the options considered, and the rationale. ADRs are version-controlled alongside code.

Example ADR format:
```
Title:   Use Azure Service Bus as the enterprise message broker
Status:  Accepted
Context: We need a message broker for async communication between
         microservices. We need pub/sub and dead-letter queuing.
Decision: Azure Service Bus (Premium tier)

Options Considered:
  - RabbitMQ       (rejected: operational overhead, not managed)
  - AWS SQS + SNS  (rejected: we are Azure-first)
  - Azure Svc Bus  (selected)

Consequences:
  - Teams must use .NET SDK or AMQP 1.0 compatible clients
  - All topics must be registered in the enterprise broker catalogue
```

**Technology Roadmap:**
A timeline showing which technologies will be **adopted**, **sunset**, or **replaced** and when.

```
TECHNOLOGY ROADMAP 2024-2027

2024 Q1  +-[API Gateway rollout]
         |
2024 Q3  +---[Customer MDM deployment]
         |
2025 Q1  +------[Microservices Phase 1: Identity + Account services]
         |
2025 Q3  +----------[COBOL system retirement begins]
         |
2026 Q2  +-------------[Legacy Oracle DW replaced by Synapse Analytics]
         |
2027 Q1  +----------------[Full cloud migration complete]
```

### 8.2 Working with EA Tools

**ArchiMate + Archi (free tool):**
ArchiMate is the standard notation. Archi is an open-source, free ArchiMate modeler. Enterprise architects use it to draw layer diagrams showing how business capabilities are supported by applications, which use data entities, running on infrastructure.

**Sparx Enterprise Architect:**
A widely used commercial tool that supports UML, ArchiMate, BPMN, and SysML. Used for detailed architectural modeling, generating reports, and maintaining the architecture repository.

**Avolution ABACUS:**
A cloud-based EA portfolio management tool. Specializes in Application Portfolio Management, technology rationalization, and EA analytics dashboards.

**LeanIX:**
Modern cloud-based EA tool popular with large enterprises. Integrates with Jira, ServiceNow, and GitHub to pull live data about applications and create automated portfolio reports.

### 8.3 A Node.js Architecture Registry Example

While EA tools are specialized, it helps to understand the data structures involved. Here is a simplified Node.js example showing how an application registry might work:

```javascript
// enterprise-registry/registry.js
// A simplified in-memory Application Portfolio Registry

const applications = [
  {
    id: "app-001",
    name: "Core Banking System",
    vendor: "Temenos",
    language: "Java",
    businessCapabilities: ["Account Management", "Loans", "Deposits"],
    status: "ACTIVE",
    health: "CRITICAL", // GOOD | MODERATE | CRITICAL | SUNSET
    hostingModel: "ON_PREMISE",
    owner: "Retail Banking Division",
    integrations: ["app-002", "app-005", "app-009"],
    annualCost: 2400000,
    decommissionDate: null,
    complianceTags: ["PCI-DSS", "SOX", "GDPR"],
  },
  {
    id: "app-002",
    name: "Customer Identity Platform",
    vendor: "Custom",
    language: "Node.js",
    businessCapabilities: ["Customer Identity", "Authentication"],
    status: "ACTIVE",
    health: "GOOD",
    hostingModel: "CLOUD_AZURE",
    owner: "Digital Platform Team",
    integrations: ["app-001", "app-003"],
    annualCost: 180000,
    decommissionDate: null,
    complianceTags: ["GDPR", "ISO27001"],
  },
  {
    id: "app-003",
    name: "Legacy COBOL Loan System",
    vendor: "Custom",
    language: "COBOL",
    businessCapabilities: ["Loan Origination"],
    status: "SUNSET", // marked for retirement
    health: "CRITICAL",
    hostingModel: "ON_PREMISE",
    owner: "Loans Division",
    integrations: ["app-001"],
    annualCost: 850000,
    decommissionDate: "2026-06-30",
    complianceTags: ["SOX"],
  },
];

// --- Portfolio Analysis Functions ---

/**
 * Find all applications with CRITICAL health
 * Used for: Risk reporting, investment prioritization
 */
function getCriticalApplications() {
  return applications.filter((app) => app.health === "CRITICAL");
}

/**
 * Find applications supporting a given business capability
 * Used for: Impact analysis when a capability changes
 */
function getApplicationsByCapability(capability) {
  return applications.filter((app) =>
    app.businessCapabilities.includes(capability)
  );
}

/**
 * Find all systems that integrate with a given application (impact analysis)
 * Used for: "What breaks if we change app X?"
 */
function getImpactedApplications(appId) {
  const impacted = applications.filter((app) =>
    app.integrations.includes(appId)
  );
  return impacted.map((app) => app.name);
}

/**
 * Calculate total annual IT spend by hosting model
 * Used for: Cloud migration ROI analysis
 */
function spendByHostingModel() {
  return applications.reduce((summary, app) => {
    const key = app.hostingModel;
    summary[key] = (summary[key] || 0) + app.annualCost;
    return summary;
  }, {});
}

/**
 * Find applications due for decommission within the next N months
 */
function getUpcomingDecommissions(months) {
  const cutoff = new Date();
  cutoff.setMonth(cutoff.getMonth() + months);

  return applications.filter((app) => {
    if (!app.decommissionDate) return false;
    return new Date(app.decommissionDate) <= cutoff;
  });
}

// --- Demo Output ---
console.log("=== ENTERPRISE PORTFOLIO REPORT ===\n");

console.log("Critical Applications:");
getCriticalApplications().forEach((app) =>
  console.log(`  [!] ${app.name} — Owner: ${app.owner}`)
);

console.log("\nApplications supporting 'Loan Origination':");
getApplicationsByCapability("Loan Origination").forEach((app) =>
  console.log(`  - ${app.name} (${app.status})`)
);

console.log("\nImpact if 'app-001' (Core Banking) changes:");
const impacted = getImpactedApplications("app-001");
console.log(`  Impacted systems: ${impacted.join(", ")}`);

console.log("\nIT Spend by Hosting Model:");
const spend = spendByHostingModel();
Object.entries(spend).forEach(([model, total]) =>
  console.log(`  ${model}: $${total.toLocaleString()} / year`)
);

console.log("\nSystems due for decommission in next 18 months:");
getUpcomingDecommissions(18).forEach((app) =>
  console.log(`  - ${app.name} (Planned: ${app.decommissionDate})`)
);
```

**Expected output:**
```
=== ENTERPRISE PORTFOLIO REPORT ===

Critical Applications:
  [!] Core Banking System — Owner: Retail Banking Division
  [!] Legacy COBOL Loan System — Owner: Loans Division

Applications supporting 'Loan Origination':
  - Legacy COBOL Loan System (SUNSET)

Impact if 'app-001' (Core Banking) changes:
  Impacted systems: Customer Identity Platform

IT Spend by Hosting Model:
  ON_PREMISE: $3,250,000 / year
  CLOUD_AZURE: $180,000 / year

Systems due for decommission in next 18 months:
  - Legacy COBOL Loan System (Planned: 2026-06-30)
```

This data is the foundation of every EA conversation — who owns what, what is at risk, what depends on what, and where the money goes.

---

## 9. Best Practices

### 9.1 Start With Business, Not Technology

Always begin with business capabilities and strategy. Never start with "we should use Kubernetes." Start with "the business needs to onboard customers in under 2 minutes — what architecture enables that?"

### 9.2 Make Architecture Visible and Accessible

Architecture that lives in a locked SharePoint folder nobody reads is worthless. EA artifacts should be:
- Published on an internal wiki (Confluence, Notion)
- Linked from project documents
- Referenced in sprint planning
- Reviewed during onboarding for new engineers

### 9.3 Version Control Your Architecture Decisions

Store Architecture Decision Records (ADRs) in Git alongside code. This means when a developer asks *"why are we using Kafka instead of RabbitMQ?"* the answer is in the repo with context and date.

### 9.4 Use the Strangler Fig Pattern for Legacy Migration

When retiring a legacy system, do not try to replace it all at once (a "big bang" rewrite almost always fails). Instead:

```
PHASE 1: Route new features to new system
  [Old System] <-- existing traffic still flows here
  [New System] <-- new features go here

PHASE 2: Gradually migrate old functionality
  [Old System] <-- less and less traffic
  [New System] <-- more and more traffic

PHASE 3: Decommission
  [Old System] <-- retired, switched off
  [New System] <-- all traffic
```

This pattern is named for the Strangler Fig tree, which grows around old trees and eventually replaces them entirely.

### 9.5 Distinguish Between Descriptive and Prescriptive Architecture

- **Descriptive** (as-is) architecture documents what exists today. Accurate, honest, sometimes ugly.
- **Prescriptive** (to-be) architecture defines the target state. Aspirational but grounded.
- **Normative** architecture defines the rules and standards. Binding.

Confusing these three types leads to architectures that look great on paper but have no connection to reality.

### 9.6 Measure Architecture Health Regularly

Track metrics like:
- **Technical debt ratio** — percentage of applications in CRITICAL health
- **Cloud adoption rate** — percentage of workloads on cloud
- **API coverage** — percentage of systems exposing APIs
- **Duplication rate** — percentage of business capabilities served by multiple systems

Without measurement, architecture improvement is just opinion.

---

## 10. Industry Standards

| Standard / Framework  | Description                                       | Governing Body            |
|-----------------------|---------------------------------------------------|---------------------------|
| **TOGAF 10**          | EA Framework and ADM process                      | The Open Group            |
| **Zachman Framework** | EA taxonomy (6x6 grid)                            | Zachman International     |
| **ArchiMate 3.2**     | EA modeling language                              | The Open Group            |
| **COBIT 2019**        | IT governance and management framework            | ISACA                     |
| **ITIL 4**            | IT service management (aligned to EA)             | AXELOS / PeopleCert       |
| **ISO/IEC 42010**     | Standard for architecture description             | ISO / IEC                 |
| **FEAF (Federal)**    | US Federal Enterprise Architecture Framework     | US Federal Government     |
| **NAF (NATO)**        | NATO Architecture Framework                       | NATO                      |
| **BIAN**              | Banking Industry Architecture Network standards   | BIAN consortium           |

**Compliance Frameworks that EA governs:**

| Framework    | Domain      | What It Covers                                              |
|--------------|-------------|-------------------------------------------------------------|
| **SOC 2**    | Security    | Controls for SaaS/cloud vendors (availability, security)   |
| **ISO 27001**| Security    | International standard for information security management |
| **GDPR**     | Privacy     | Data protection for EU citizens                             |
| **HIPAA**    | Healthcare  | Protection of health information (US)                       |
| **PCI-DSS**  | Payments    | Security for cardholder data                               |
| **SOX**      | Finance     | Financial IT controls (US public companies)                 |

Enterprise architects do not implement compliance — but they map which systems are in scope for which frameworks and ensure the architecture supports compliance by design.

---

## 11. Common Mistakes

### 11.1 The Ivory Tower Anti-Pattern

The EA team sits in a glass office, creates beautiful architecture diagrams, presents them to executives, and then never talks to the delivery teams building the actual software.

**Result:** Architecture documents that are theoretically perfect but completely disconnected from reality. Developers ignore them because they offer no practical guidance.

**Fix:** Architects must be embedded with or regularly visiting delivery teams. Architecture must be co-created, not handed down from above.

### 11.2 The Architecture Police Anti-Pattern

Every change requires an architecture review. Every review takes 3 weeks. The ARB rejects anything that deviates from the standard. Teams spend more time in governance meetings than building software.

**Result:** Teams bypass the architecture function, build shadow IT, and the architecture team loses all credibility.

**Fix:** Governance should be **lightweight and enabling**, not obstructive. Use a tiered approach:
- Small/standard changes: **self-service** (team checks against a published checklist)
- Medium changes: **async review** (team submits ADR, gets feedback within 48 hours)
- Large/strategic changes: **ARB review** (scheduled, but with a guaranteed SLA)

### 11.3 Architecture by Committee

Every stakeholder must approve every decision. Every department head has veto power. Architecture discussions become political negotiations.

**Result:** Decisions take forever and are often compromises that satisfy nobody and serve no clear architectural purpose.

**Fix:** Identify a clear **Architecture Decision Authority**. Use RACI matrices. Distinguish between *consulted* and *approver*.

### 11.4 Point-in-Time Architecture

The architecture is documented once and then never updated. Eighteen months later, the diagram shows systems that no longer exist and misses a dozen systems that were recently added.

**Result:** The architecture repository is not trusted and therefore not used.

**Fix:** Treat architecture as a **living product**. Assign owners. Schedule quarterly reviews. Use tools (like LeanIX) that auto-discover applications from cloud APIs.

### 11.5 Treating Technology as the Starting Point

Starting with "we should use microservices" or "let's go serverless" before understanding the actual business problem.

**Fix:** Always derive technology choices from business requirements. The business need drives the architecture — never the other way around.

### 11.6 Ignoring the Human Side of Architecture

Architecture is as much about **change management** as it is about technology. An architect who designs a perfect system but fails to bring teams along will see the design ignored or abandoned.

**Fix:** Spend as much time communicating, educating, and building consensus as you do modeling.

---

## 12. Security & Performance Considerations

### 12.1 Enterprise Security Architecture

Security in EA is not just firewalls and passwords. It is a **discipline** that mirrors the four BDAT domains:

```
BUSINESS SECURITY ARCHITECTURE
  - Security policies, acceptable use, risk appetite
  - Business continuity and disaster recovery planning
  - Security training and awareness programs

APPLICATION SECURITY ARCHITECTURE
  - Secure SDLC standards (OWASP, threat modeling)
  - API security standards (OAuth 2.0, OpenID Connect)
  - Secret management (no hardcoded credentials)

DATA SECURITY ARCHITECTURE
  - Data classification (Public / Internal / Confidential / Restricted)
  - Encryption standards (at rest: AES-256, in transit: TLS 1.3)
  - Data residency requirements (GDPR: EU data stays in EU)
  - Data masking and anonymization for non-production environments

TECHNOLOGY SECURITY ARCHITECTURE
  - Network segmentation (DMZ, internal zones, zero-trust)
  - Identity and Access Management (IAM) standards
  - Endpoint protection standards
  - Vulnerability management and patch cadence
```

### 12.2 Zero Trust Architecture

Traditional enterprise security assumed that anything inside the network perimeter could be trusted. Zero Trust rejects this assumption entirely:

```
TRADITIONAL (Perimeter-based):
[Internet] --[Firewall]--> [TRUSTED ZONE: everything inside is allowed]

ZERO TRUST:
[Internet]         --[Identity Verify]--> [Minimal Access Granted]
[Internal Network] --[Identity Verify]--> [Minimal Access Granted]

  Rule: "Never trust, always verify — regardless of network location"
```

Zero Trust is now an **EA-level mandate** in most large enterprises. It requires:
- Every user, device, and service to be continuously authenticated
- Least-privilege access enforced for every resource
- All traffic encrypted, even internal traffic
- Continuous monitoring and anomaly detection

### 12.3 Compliance Governance in EA

EA maintains a **Compliance Landscape Map** — which systems are in scope for which frameworks:

```
SYSTEM                 SOC2   ISO27001   GDPR   PCI-DSS   HIPAA
-----------------------------------------------------------------
Core Banking             Y        Y        Y        Y        N
Customer MDM             Y        Y        Y        N        N
Payment Gateway          Y        Y        Y        Y        N
EHR System               Y        Y        Y        N        Y
Analytics Platform       Y        Y        Y        N        N
```

This map determines:
- Which systems need annual external audits
- Which data must be encrypted
- Which regions data can be stored in
- What access controls must be in place

### 12.4 Security Reference Architecture

EA publishes a **Security Reference Architecture** that all solution architectures must comply with. Key elements typically include:

- All external APIs must sit behind an API Gateway with rate limiting and authentication
- All secrets stored in a centralized vault (e.g., Azure Key Vault) — no credentials in environment variables or code
- All production databases encrypted at rest with customer-managed keys
- All application logs forwarded to a central SIEM (Security Information and Event Management)
- All applications must undergo DAST (Dynamic Application Security Testing) before production release

### 12.5 Performance at Enterprise Scale

Performance in EA is about **capacity planning** and **resiliency**, not just individual system speed:

- **SLA governance:** EA defines enterprise-wide SLA tiers.
  ```
  Gold:   99.99% uptime — mission-critical (payment processing, core banking)
  Silver: 99.9%  uptime — important (CRM, reporting systems)
  Bronze: 99.0%  uptime — internal tools (HR systems, intranet)
  ```
- **Disaster Recovery tiers:** Gold: RTO < 1 hour, RPO < 15 minutes. Bronze: RTO < 24 hours, RPO < 4 hours.
- **Capacity planning:** EA works with cloud teams to forecast compute and storage growth based on business plans — e.g., "the mobile app will onboard 1 million new users — what additional capacity do we need and at what cost?"

---

## 13. Related Technologies

| Technology / Concept          | Relationship to EA                                                          |
|-------------------------------|-----------------------------------------------------------------------------|
| **Microservices**             | An application architecture pattern governed by EA principles               |
| **Event-Driven Architecture** | An enterprise integration pattern defined at the EA level                   |
| **API Management**            | EA defines the API gateway standard; solution architects implement it        |
| **Cloud Computing**           | EA defines the cloud strategy (cloud-first, multi-cloud, hybrid)            |
| **DevOps / Platform Eng.**    | EA defines standards that DevOps platforms must meet                         |
| **Data Governance**           | Part of EA's Data Architecture domain; Chief Data Office often reports here  |
| **Service Mesh**              | A technology pattern for service-to-service communication governed by EA     |
| **Identity & Access Mgmt.**   | Defined at EA level (enterprise SSO, LDAP/AD standards, Zero Trust)          |
| **Business Process Mgmt.**    | EA's Business Architecture domain often uses BPMN process models             |
| **Domain-Driven Design**      | DDD's bounded contexts inform how EA draws application boundaries            |
| **Solution Architecture**     | Works within EA guardrails to design specific programs                       |
| **Software Architecture**     | Works within Solution Architecture guidelines                                |
| **ITIL**                      | IT service management framework that works alongside EA governance            |
| **SAFe (Agile at Scale)**     | EA provides the "Architectural Runway" concept that SAFe builds on           |

---

## 14. Summary

### What We Learned

We began this chapter by looking at the chaos that emerges when large organizations make independent technology decisions without any overarching plan — hundreds of applications, duplicated data, impossible integrations, and IT budgets that grow while agility shrinks.

We introduced **Enterprise Architecture** as the discipline that solves this: a structured, organization-wide view of business strategy, applications, data, and technology — aligned so that IT serves business goals deliberately, not accidentally.

We explored the **two major frameworks**:
- **TOGAF** — the process framework, with its Architecture Development Method (ADM) that guides architects from strategy through to governed implementation in an iterative cycle
- **Zachman** — the classification framework, a 6x6 grid that categorizes every possible architectural artifact by perspective and interrogative question

We examined the **four domains** (Business, Application, Data, Technology) and the architecture hierarchy (EA → Solution Architecture → Software Architecture → Code).

We walked through a **real workflow** — NorthBank's digital transformation — seeing how EA work flows from executive strategy documents through application portfolio assessment, target architecture design, migration roadmaps, and ongoing governance.

We saw how **real organizations** — banks, governments, retailers, and hospitals — use EA to manage complexity at scale, and we looked at common **anti-patterns** (ivory tower, architecture police, architecture by committee) that cause EA programs to fail.

Finally, we covered the critical role of EA in **security** — governance of compliance frameworks (SOC 2, ISO 27001, GDPR, HIPAA, PCI-DSS), Zero Trust architecture mandates, and enterprise-wide security reference architectures.

### Key Takeaways

- Enterprise Architecture is to an organization what urban planning is to a city — it designs not one building, but how all buildings work together.
- EA sits above Solution Architecture and Software Architecture; it sets the rules and guardrails that all levels work within.
- The four EA domains are Business, Application, Data, and Technology (BDAT).
- TOGAF's ADM provides a repeatable process for EA; Zachman provides a classification taxonomy; ArchiMate provides the modeling language.
- Enterprise architects produce artifacts — not code. Their outputs are principles, roadmaps, portfolio maps, reference architectures, and ADRs.
- EA governance must be enabling, not obstructing. Ivory tower and architecture police patterns destroy EA credibility.
- Security architecture is an EA responsibility — compliance frameworks, zero trust mandates, and data classification policies are defined at the EA level.
- Good EA is never "done" — it is a living practice that evolves as business strategy and the technology landscape change.

---

## 15. Keywords

`Enterprise Architecture`, `TOGAF`, `Zachman Framework`, `ArchiMate`, `Architecture Development Method`, `ADM`, `Application Portfolio Management`, `Business Architecture`, `Application Architecture`, `Data Architecture`, `Technology Architecture`, `Architecture Principles`, `Architecture Governance`, `Architecture Review Board`, `ARB`, `Architecture Decision Record`, `ADR`, `Reference Architecture`, `Architecture Runway`, `Hub and Spoke`, `Point-to-Point Integration`, `Enterprise Service Bus`, `ESB`, `Event-Driven Architecture`, `Message Bus`, `Technology Roadmap`, `Strangler Fig Pattern`, `Transition Architecture`, `Target Architecture`, `Current State Architecture`, `Business Capability Map`, `BDAT`, `Zero Trust Architecture`, `SOC 2`, `ISO 27001`, `GDPR`, `HIPAA`, `PCI-DSS`, `SOX`, `Security Reference Architecture`, `Data Lake`, `Data Warehouse`, `Ivory Tower Anti-pattern`, `Architecture Police Anti-pattern`, `Architecture by Committee`, `Solution Architecture`, `Software Architecture`, `EA Hierarchy`, `LeanIX`, `Sparx Enterprise Architect`, `Avolution ABACUS`, `Architecture Repository`, `COBIT`, `ITIL`, `Digital Transformation`, `Application Portfolio`, `Compliance Landscape Map`, `Master Data Management`

---

## 16. Glossary

| Term | Definition |
|------|------------|
| **Enterprise Architecture (EA)** | The practice of aligning business strategy with IT systems across the entire organization, producing a structured description of the organization's capabilities, processes, applications, data, and technology. |
| **TOGAF** | The Open Group Architecture Framework — the world's most widely adopted EA framework, providing the Architecture Development Method (ADM) as a structured process for creating and maintaining EA. |
| **ADM (Architecture Development Method)** | The core process of TOGAF — a cyclical series of phases (Preliminary through H) guiding architects from strategy to governed implementation. |
| **Zachman Framework** | A 6x6 classification grid that categorizes all architectural artifacts by stakeholder perspective (rows) and interrogative question — What, How, Where, Who, When, Why (columns). |
| **ArchiMate** | An EA modeling language standardized by The Open Group, used to describe and visualize enterprise architecture across motivation, strategy, business, application, technology, and physical layers. |
| **BDAT** | Business, Data, Application, Technology — the four standard domains of enterprise architecture. |
| **Business Capability** | A specific thing the business can do, expressed independently of how it is done today. Example: "Loan Origination" is a capability; the COBOL system is one implementation of it. |
| **Application Portfolio Management (APM)** | The practice of cataloguing and governing all applications in the organization, assessing their health, cost, and alignment to business capabilities. |
| **Architecture Principles** | Formal statements that guide all architectural decisions across the organization. Examples: "Cloud First," "API First," "Single Source of Truth." |
| **Reference Architecture** | A pre-approved, reusable architectural pattern for a specific type of solution, that teams can adopt as a starting point instead of designing from scratch. |
| **Architecture Runway** | Architectural work done ahead of delivery teams to ensure they have a stable foundation to build on — a term from SAFe Lean-Agile thinking. |
| **Architecture Review Board (ARB)** | A governance body that reviews and approves major architectural decisions to ensure they comply with enterprise standards and principles. |
| **Architecture Decision Record (ADR)** | A lightweight document recording a specific architectural decision, its context, the options considered, and the rationale. Typically version-controlled alongside code. |
| **Technology Roadmap** | A multi-year timeline showing planned technology changes — adoptions, upgrades, migrations, and decommissions — aligned to business strategy. |
| **Strangler Fig Pattern** | A migration strategy where new functionality is built to incrementally replace old functionality, until the old system can be safely retired — named for the Strangler Fig tree that grows around and replaces a host tree. |
| **Hub and Spoke** | An integration pattern where all systems connect to a central hub (Enterprise Service Bus) rather than directly to each other, reducing the number of direct connections. |
| **Enterprise Service Bus (ESB)** | Middleware that acts as the hub in a hub-and-spoke integration architecture, routing and transforming messages between systems. |
| **Zero Trust Architecture** | A security model based on "never trust, always verify" — no user, device, or service is trusted by default regardless of network location. Every request must be authenticated and authorized. |
| **SOC 2** | Service Organization Control 2 — a compliance standard for technology service providers covering security, availability, processing integrity, confidentiality, and privacy. |
| **ISO 27001** | International standard for Information Security Management Systems (ISMS) — a comprehensive framework for managing information security risks. |
| **GDPR** | General Data Protection Regulation — EU law governing the collection, processing, and storage of personal data of EU citizens. |
| **HIPAA** | Health Insurance Portability and Accountability Act — US law protecting the privacy and security of Protected Health Information. |
| **PCI-DSS** | Payment Card Industry Data Security Standard — security requirements for organizations that handle payment card data. |
| **Business Capability Map** | A structured decomposition of everything a business can do, used to connect business strategy to IT systems and identify gaps and redundancies. |
| **Ivory Tower Anti-pattern** | An EA anti-pattern where architects design in isolation without engaging delivery teams, resulting in architecture that is theoretically correct but practically ignored. |
| **Architecture Police Anti-pattern** | An EA anti-pattern where governance is so strict and slow that it blocks delivery teams, causing them to work around the architecture function entirely. |
| **Solution Architecture** | Architecture for a specific project or program, operating within the guardrails and principles set by Enterprise Architecture. |
| **LeanIX** | A cloud-based EA and Application Portfolio Management tool that integrates with development tools to maintain a live, automatically updated architecture registry. |
| **Compliance Landscape Map** | An EA artifact mapping which applications and data flows are in scope for which regulatory and compliance frameworks, used to govern audit readiness. |
| **Data Lake** | A storage repository that holds vast amounts of raw data in native format, supporting flexible and exploratory analytics with schema applied at read time. |
| **Data Warehouse** | A structured database system optimized for reporting and known analytical queries, storing cleaned and transformed data with schema applied at write time. |
| **COBIT** | Control Objectives for Information and Related Technologies — an IT governance and management framework published by ISACA that complements EA. |
| **BIAN** | Banking Industry Architecture Network — a non-profit consortium that defines a standard service landscape and canonical data model for the banking industry. |
| **Master Data Management (MDM)** | The discipline of creating and maintaining a single, authoritative, consistent record for key business entities (e.g., Customer, Product, Location) across all systems. |

---

## 17. Next Recommended Chapters

Having built a comprehensive understanding of Enterprise Architecture — the top of the architecture hierarchy — you are ready to go deeper into the layers below, or broader into related disciplines:

| Chapter | Why It Follows |
|---------|----------------|
| **Chapter 12: Solution Architecture** | Zoom in one level — how architects design specific programs and initiatives within EA guardrails. Covers solution design patterns, stakeholder management, and architecture for delivery. |
| **Chapter 13: Domain-Driven Design (DDD)** | EA uses Business Capability Maps to understand what the business does. DDD provides the detailed technique for identifying Bounded Contexts and designing application boundaries. A natural deepening of EA concepts. |
| **Chapter 14: Event-Driven Architecture** | We introduced event-driven architecture as an enterprise integration pattern. This chapter goes deep: event sourcing, CQRS, event brokers (Kafka, Azure Service Bus), and saga patterns. |
| **Chapter 15: API Design and Management** | EA mandates API-first; this chapter covers how to design, version, govern, and manage APIs at scale — including API gateways, OpenAPI specifications, developer portals, and API lifecycle management. |
| **Chapter 16: Data Architecture and Governance** | The Data Architecture domain of EA gets its own deep dive: master data management, data lineage, data catalogs, data mesh, and the modern data stack. |
| **Chapter 17: Cloud Architecture Patterns** | EA defines the cloud strategy; this chapter covers the technical patterns — landing zones, multi-cloud, hybrid cloud, cloud-native design, and FinOps for cost governance. |
| **Chapter 18: Security Architecture** | We touched on enterprise security governance in this chapter. Chapter 18 goes deep into threat modeling, security design patterns, Zero Trust implementation, and DevSecOps integration. |
| **Chapter 19: Technology Governance and ITIL** | Explores how IT service management (ITIL 4) works alongside EA governance — change management, incident management, and service portfolio management. |

---

*Chapter 11 — Enterprise Architecture | Pilot-Seat Curriculum | Architecture Track*
