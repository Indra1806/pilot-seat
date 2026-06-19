> **Mode:** Book
> **Pilot-Seat Standard**

---

# Chapter 14: Architecture Decision Records (ADRs)

---

## 1. Introduction

Every software system is a graveyard of decisions.

Not the code you can read — the *why* behind it. Why is this service split from that one? Why did the team choose PostgreSQL over MongoDB? Why is there a message queue between these two components, and why does it use Kafka instead of RabbitMQ?

If you have ever joined a project and stared at the codebase wondering *"who made this choice, and what were they thinking?"* — you have felt the pain that **Architecture Decision Records (ADRs)** were designed to cure.

An **Architecture Decision Record** is a short, structured document that captures a single important architectural decision. It records:

- The **context** — what was happening when the decision was made
- The **decision** — what was chosen and why
- The **consequences** — what changes, trade-offs, or future constraints follow from the decision

Think of an ADR as a sticky note pinned to an architectural choice — except this sticky note never falls off the wall, never gets thrown away, and anyone on the team can read it a year later and immediately understand the reasoning.

ADRs live alongside your code, in your repository, written in plain Markdown. They are version-controlled, reviewable, and permanent.

This chapter will teach you what ADRs are, how to write them, where to store them, and why every engineering team — from two-person startups to thousand-engineer organizations — benefits from maintaining them.

---

## 2. Why It Exists

### The "Why Did We Build It This Way?" Problem

Imagine you join a construction crew tasked with extending a bridge. The original crew is gone. There are no notes. You can see *what* was built — steel beams, concrete pillars, cable anchors — but you have no idea *why* the design looks the way it does. Was this the only option? Were cheaper materials considered? Was the unusual cable angle a deliberate engineering choice or an on-site compromise?

Without records, you are stuck guessing. And guessing leads to:

- Accidentally undoing careful decisions
- Repeating the same debates the original team already resolved
- Making new decisions that conflict with old ones

Software teams face exactly this problem. The original architects leave. Context disappears. New engineers arrive and question decisions without the benefit of the original reasoning. Weeks of meetings are spent re-debating choices that were settled two years ago.

### The Oral Tradition Does Not Scale

Early in a project, architectural knowledge lives in the heads of the people who made the decisions. It gets passed on through conversations, Slack threads, and "ask Sarah, she was here when we set this up."

This oral tradition breaks down the moment:
- Team members leave or change roles
- The team grows beyond a handful of people
- The project lives longer than a year
- Remote or distributed teams join

ADRs replace oral tradition with written record. They are the institutional memory of your architecture.

### Standards and Compliance

In regulated industries — finance, healthcare, government — architectural decisions often need to be justified to auditors, compliance officers, or security reviewers. ADRs provide a ready-made paper trail of *why* the system is built the way it is, which is enormously valuable during audits.

---

## 3. Problem It Solves

ADRs solve five concrete, painful problems:

```
PROBLEM                          WITHOUT ADRs              WITH ADRs
----------------------------------------------------------------------
"Why is it built this way?"      Mystery / guessing        Read the ADR
Same decision debated again       Hours of meetings         "We settled this in ADR-007"
New engineer onboarding           Weeks of oral hand-off    Read the ADR library
Conflicting future decisions      Silent breakage           ADR shows the constraint
Compliance / audit trail          Scramble to reconstruct   ADRs are the audit trail
```

### Problem 1 — Architectural Amnesia

Decisions made 18 months ago are forgotten. The people who made them may have moved on. ADRs cure amnesia by writing down the reasoning *at the time it is freshest*.

### Problem 2 — Decision Re-litigation

Without a record, the same architectural debate resurfaces every few months. "Should we use GraphQL or REST?" gets argued repeatedly. With an ADR, you can point to ADR-012 and say "we discussed this and here is what we decided and why. If the context has changed, let us write a new ADR."

### Problem 3 — Onboarding Friction

New engineers spend weeks learning why the system is structured a certain way. A well-maintained ADR library acts as an architectural orientation guide — a curated history of the important decisions that shaped the system.

### Problem 4 — Silent Constraint Violations

A new decision might unknowingly conflict with an old architectural constraint. If ADR-003 says "all inter-service communication must be asynchronous for resilience," and a new engineer designs a synchronous API between two services, ADR-003 surfaces the conflict during review — before the code is written.

### Problem 5 — No Audit Trail

In regulated environments, "why did you choose this technology?" needs a defensible answer. ADRs are that answer.

---

## 4. Core Concepts

### The Captain's Ship Log Analogy

Picture the captain of a large cargo ship keeping a ship's log. Every significant decision gets an entry:

```
SHIP'S LOG -- Entry 47
Date: March 14th
Decision: Changed course 15 degrees north
Context: Weather forecast shows severe storm system
         directly on planned route. Cargo includes
         fragile electronics -- risk unacceptable.
Alternatives Considered:
  - Hold position and wait (adds 3 days delay)
  - Continue original course (high risk to cargo)
  - Route through northern channel (chosen)
Consequences: Arrival delayed by 8 hours. Fuel
              consumption increases 12%. Cargo safe.
```

Six months later, a new first mate joins the ship. They can read every log entry and understand *exactly* why the ship is where it is, what conditions were faced, and what trade-offs were made. They do not have to guess. They do not have to ask the previous crew.

**ADRs are the ship's log for your software architecture.**

### What Counts as an "Architectural Decision"?

Not every decision needs an ADR. Reserve them for decisions that:

- Are **hard to reverse** (changing later would be expensive)
- Have **broad impact** (affect multiple teams or systems)
- Involve **significant trade-offs** (no clear "right" answer)
- Will **confuse future readers** without explanation

```
WORTH AN ADR                        NOT WORTH AN ADR
-------------------------------------------------------
Choosing a database engine          Choosing a variable name
Adopting microservices              Adding a utility function
Selecting an auth strategy          Fixing a bug
Choosing a message broker           Updating a dependency version
Deciding on API style (REST/gRPC)   Choosing a code formatter
Adopting a monorepo structure       Adding a new API endpoint
```

### ADR Statuses

Every ADR has a status that reflects its current state in the decision lifecycle:

```
  +-----------+    review     +----------+
  | PROPOSED  | ------------> | ACCEPTED |
  +-----------+               +----------+
                                    |
               +--------------------+--------------------+
               v                    v                    v
        +------------+       +-----------+      +--------------+
        | DEPRECATED |       | REJECTED  |      |  SUPERSEDED  |
        +------------+       +-----------+      +--------------+
                                                       |
                                                       v
                                                +------------+
                                                |  ADR-NNN   |
                                                | (new ADR)  |
                                                +------------+
```

| Status | Meaning |
|---|---|
| **Proposed** | Decision is being discussed, not yet finalized |
| **Accepted** | Decision was made and is currently in effect |
| **Rejected** | Decision was considered but not taken |
| **Deprecated** | Decision was accepted but is no longer recommended |
| **Superseded** | Decision was replaced by a newer ADR (always link to it) |

> **Golden Rule:** Never delete an ADR. Even rejected or superseded decisions have historical value. They explain paths that were considered and why they were not taken.

---

## 5. Architecture / Components

### The Michael Nygard Format (Original)

Michael Nygard introduced ADRs in a 2011 blog post. His format is intentionally minimal — a decision record should be short enough to read in under two minutes.

```
# ADR-NNN: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-NNN]

## Context
[What is the situation, the forces at play, the constraints?
 What problem are we trying to solve?]

## Decision
[What did we decide to do? State it clearly and actively.]

## Consequences
[What becomes easier? What becomes harder?
 What new constraints does this create?]
```

That is it. Four sections. Deliberately spartan.

### The Extended MADR Format

**MADR** (Markdown Architectural Decision Records) is a popular extended format that adds alternatives, pros/cons, and metadata. It is more thorough and suitable for larger teams.

```
# [short title of solved problem and solution]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-NNN]

## Date
YYYY-MM-DD

## Author(s)
[Name(s) of the decision makers]

## Context and Problem Statement
[Describe the context and problem statement]

## Decision Drivers
* [driver 1, e.g., a force, concern, or constraint]
* [driver 2, e.g., a force, concern, or constraint]

## Considered Options
* [Option 1]
* [Option 2]
* [Option 3]

## Decision Outcome
Chosen option: "[option 1]", because [justification].

### Positive Consequences
* [e.g., improvement of quality attribute]

### Negative Consequences
* [e.g., compromising quality attribute]

## Pros and Cons of the Options

### [Option 1]
**Pro:** [argument a]
**Con:** [argument b]

### [Option 2]
**Pro:** [argument a]
**Con:** [argument b]
```

### ADR Numbering and Naming

ADRs are numbered sequentially. The number never changes, even if the status changes.

```
docs/
+-- decisions/
    +-- ADR-001-use-postgresql-as-primary-database.md
    +-- ADR-002-adopt-microservices-architecture.md
    +-- ADR-003-use-kafka-for-event-streaming.md
    +-- ADR-004-adopt-jwt-for-api-authentication.md
    +-- ADR-005-use-react-for-frontend.md
```

The file name convention: `ADR-NNN-kebab-case-title.md`

---

## 6. Workflow

### Real Step-by-Step Scenario: Choosing a Caching Strategy

Let us walk through a real workflow. The team at a growing e-commerce platform is facing a performance problem: product catalog pages are slow because every request hits the database.

```
WORKFLOW: Writing and Ratifying an ADR
-----------------------------------------------------------

  Step 1: TRIGGER
  +----------------------------------------------------------+
  |  Engineer notices a problem / proposes a change          |
  |  "We need caching. Which solution do we use?"            |
  |  --> Hard to reverse, broad impact --> needs ADR          |
  +---------------------------+------------------------------+
                              |
  Step 2: DRAFT               v
  +----------------------------------------------------------+
  |  Author writes ADR-006 as a draft (Status: Proposed)     |
  |  Captures: Context, Options Considered, Initial Thinking |
  |  Creates a Pull Request with just the .md file           |
  +---------------------------+------------------------------+
                              |
  Step 3: REVIEW              v
  +----------------------------------------------------------+
  |  Team reviews the PR                                     |
  |  Comments added: "What about Memcached?"                 |
  |  Author updates ADR with the additional option           |
  |  Async discussion happens in PR comments                 |
  +---------------------------+------------------------------+
                              |
  Step 4: DECISION            v
  +----------------------------------------------------------+
  |  Team reaches consensus: Redis, because it supports      |
  |  data structures needed for cart and session data too    |
  |  Author updates Status --> Accepted, fills in Decision    |
  +---------------------------+------------------------------+
                              |
  Step 5: MERGE               v
  +----------------------------------------------------------+
  |  PR is merged into main branch                           |
  |  ADR lives in docs/decisions/ alongside the code        |
  |  It is now part of the repository's permanent history    |
  +---------------------------+------------------------------+
                              |
  Step 6: REFERENCE           v
  +----------------------------------------------------------+
  |  6 months later: new engineer asks "why Redis?"          |
  |  Senior engineer: "See ADR-006"                          |
  |  Problem solved in 30 seconds instead of 30 minutes      |
  +----------------------------------------------------------+
```

### When to Write an ADR

Write an ADR **at the moment of decision** — not before, not after. If you write it before the decision, it becomes a proposal document. If you write it after, you lose the original context and the reasoning becomes reconstructed rather than genuine.

The right trigger questions:
- "Are we about to choose between two or more significant options?"
- "Would a new team member be confused by this choice in 6 months?"
- "Is this difficult or expensive to reverse?"

If yes to any of these — write an ADR.

### Who Writes ADRs?

Anyone can write an ADR, but typically:
- The **engineer proposing the change** drafts it
- The **tech lead or architect** reviews and approves it
- The **team** discusses it, often via Pull Request comments

The author field in the ADR records who wrote it, but the decision belongs to the team.

---

## 7. Real World Examples

### Example 1 — GitHub

GitHub uses ADRs extensively and some of their decision records are publicly visible in open-source projects they maintain. The GitHub team uses ADRs for decisions about their own infrastructure and has advocated for ADRs as a practice in engineering blog posts. Their pattern: ADRs live in a `/docs/adr/` directory, numbered sequentially, and referenced from pull requests when new code implements a decision.

### Example 2 — Spotify

Spotify's engineering culture is famous for its "Squad, Tribe, Chapter, Guild" model. As part of their architecture governance, Spotify teams maintain "Tech Radar" entries and decision records that explain why certain technologies are adopted or retired across the organization. Their extended format includes an "impact assessment" section that notes which squads are affected by an architectural decision.

### Example 3 — CNCF Projects (Kubernetes, Prometheus, etc.)

The Cloud Native Computing Foundation (CNCF) projects use ADRs (sometimes called "design proposals" or "enhancement proposals") to document major architectural decisions. Kubernetes Enhancement Proposals (KEPs) are a formalized version of ADRs used to govern significant changes to Kubernetes architecture. Each KEP follows a structured format and must be approved before implementation begins.

### Example 4 — Zalando

Zalando, the European e-commerce giant, open-sourced their Architecture Decision Record templates and actively uses ADRs across their engineering teams. Their format adds a "Participants" field (who was in the room when the decision was made) and a "Links" section (pointing to related technical documents, RFCs, or blog posts). Zalando engineers cite ADRs in code comments using references like `// See ADR-023` when implementing a decision.

---

## 8. Implementation

### Complete Example ADRs

#### ADR-001: Use PostgreSQL as Primary Database

```markdown
# ADR-001: Use PostgreSQL as Primary Database

## Status
Accepted

## Date
2024-03-10

## Author(s)
Priya Sharma (Lead Backend Engineer)

## Context and Problem Statement
We are building a new e-commerce platform from scratch. We need
to choose a primary relational database. The system will handle
product catalogs, user accounts, orders, and inventory. We
expect complex queries with joins across multiple entity types.
Transactional consistency (ACID compliance) is a hard requirement
for order processing.

## Decision Drivers
* Must support ACID transactions (order integrity is critical)
* Must handle complex relational queries (product + inventory + orders)
* Team has strong existing expertise
* Must be open-source (budget constraints)
* Must have good cloud-managed options (we plan to use AWS RDS)

## Considered Options
* PostgreSQL
* MySQL
* MongoDB
* DynamoDB

## Decision Outcome
Chosen option: **PostgreSQL**, because it satisfies all decision
drivers, the team has deep expertise, it has excellent JSON
support for semi-structured data (product attributes), and AWS
RDS for PostgreSQL is a mature managed offering.

### Positive Consequences
* Full ACID compliance for order transactions
* Rich query language supports complex reporting queries
* JSON/JSONB columns allow flexible product attribute storage
* Excellent tooling ecosystem (pgAdmin, Prisma, Sequelize, etc.)
* Team productivity is high due to existing expertise

### Negative Consequences
* Horizontal write-scaling requires additional tooling (e.g., Citus)
  if we exceed ~10,000 writes/second -- revisit at that scale
* Schema migrations require careful planning (altering large tables
  can be slow) -- we will adopt a migration tool (Flyway) from day one

## Pros and Cons of the Options

### PostgreSQL
**Pro:** ACID compliant, rich SQL, JSONB support, mature ecosystem
**Con:** Vertical scaling limits; complex HA setup vs. managed solutions

### MySQL
**Pro:** Widely known, fast reads
**Con:** Historically weaker ACID guarantees, fewer advanced features

### MongoDB
**Pro:** Flexible schema, horizontal scaling
**Con:** No true multi-document ACID transactions until v4.0+;
         less suited to our strongly relational data model

### DynamoDB
**Pro:** Infinite horizontal scale, managed by AWS
**Con:** No SQL joins; query patterns must be known upfront;
         higher cost at our scale; team has no expertise
```

---

#### ADR-002: Adopt Microservices Architecture

```markdown
# ADR-002: Adopt Microservices Architecture

## Status
Accepted

## Date
2024-04-02

## Author(s)
James Okafor (Principal Architect)

## Context and Problem Statement
Our monolithic application has reached a point where:
- A single team of 40 engineers is working on one codebase
- Deployments are 2-hour affairs that require full system freeze
- A bug in the payment module causes the entire app to go down
- Different parts of the system have wildly different scaling needs
  (catalog is read-heavy; checkout is write-heavy; search needs
  specialized infrastructure)
We need an architecture that allows independent deployment,
independent scaling, and team autonomy.

## Decision Drivers
* Enable independent deployment per team
* Allow different scaling strategies per service
* Isolate failure domains (payment failure != catalog failure)
* Support team autonomy (Conway's Law alignment)
* Each service should be replaceable/rewriteable independently

## Considered Options
* Monolith (status quo)
* Modular Monolith (well-structured single deployable)
* Microservices

## Decision Outcome
Chosen option: **Microservices**, because the team size, deployment
frequency goals, and independent scaling requirements justify the
operational overhead. We will begin with domain-driven decomposition:
User Service, Product Service, Order Service, Payment Service,
Notification Service.

### Positive Consequences
* Teams deploy independently -- no more 2-hour deployment windows
* Services can be scaled independently (Product Service on read
  replicas; Payment Service on high-availability clusters)
* Failure isolation: a Notification Service outage does not affect
  Order processing
* Teams own their services end-to-end (you build it, you run it)

### Negative Consequences
* Distributed systems complexity: network failures, latency,
  eventual consistency must all be handled explicitly
* Operational overhead: each service needs its own CI/CD pipeline,
  monitoring, alerting, and deployment configuration
* Inter-service communication requires design (we will use
  async messaging via Kafka -- see ADR-003)
* Debugging across services requires distributed tracing
  (we will adopt OpenTelemetry -- see ADR-008)

## Related ADRs
* ADR-003: Use Kafka for Event Streaming
* ADR-008: Adopt OpenTelemetry for Distributed Tracing
```

---

#### ADR-003: Use Kafka for Event Streaming

```markdown
# ADR-003: Use Kafka for Event Streaming

## Status
Accepted

## Date
2024-04-15

## Author(s)
Mei Chen (Senior Infrastructure Engineer)

## Context and Problem Statement
With microservices adopted (ADR-002), we need a strategy for
inter-service communication. Requirements:
- Services should be decoupled (publisher does not know consumers)
- Events must be durable (if Notification Service is down, it
  should process events when it comes back up)
- We expect high throughput: up to 100,000 events/second at peak
- We need event replay capability for disaster recovery
- Multiple consumers per event type (Order Placed triggers both
  Notification Service and Inventory Service)

## Decision Drivers
* Durability: events must survive consumer downtime
* High throughput (100k events/sec target)
* Event replay for recovery and new consumer bootstrapping
* Decoupling of producers and consumers
* Mature ecosystem and operational tooling

## Considered Options
* Apache Kafka
* RabbitMQ
* AWS SQS/SNS
* Redis Pub/Sub

## Decision Outcome
Chosen option: **Apache Kafka**, because it uniquely satisfies our
durability, replay, and throughput requirements. We will use
Confluent Cloud as a managed Kafka offering to reduce operational
burden.

### Positive Consequences
* Events are durable -- consumer downtime does not cause data loss
* New services can replay event history from day one
* 100k+ events/second well within Kafka's capabilities
* Strong ecosystem: Kafka Connect for data integration,
  Kafka Streams for stream processing

### Negative Consequences
* Higher operational complexity than SQS/SNS (offset management,
  consumer groups, partition strategy)
* Higher latency than Redis Pub/Sub (~10ms vs <1ms) -- acceptable
  for our use case but synchronous operations will still use REST
* Confluent Cloud cost is higher than SQS at low volume --
  acceptable given our growth projections

## Pros and Cons of the Options

### Apache Kafka
**Pro:** Durable, high-throughput, replayable, strong ecosystem
**Con:** Operational complexity, higher latency than in-memory brokers

### RabbitMQ
**Pro:** Simpler to operate, lower latency
**Con:** No native replay; messages deleted after consumption;
         lower throughput ceiling

### AWS SQS/SNS
**Pro:** Fully managed, no operational overhead
**Con:** No replay (messages deleted after max 14 days);
         SNS fan-out is limited; tied to AWS vendor

### Redis Pub/Sub
**Pro:** Extremely low latency (<1ms)
**Con:** No durability (messages lost if consumer offline);
         not designed for high-throughput streaming workloads
```

---

### Tooling: adr-tools (CLI)

`adr-tools` is a command-line tool for managing ADRs. It scaffolds new ADRs without copying templates manually.

```bash
# Install adr-tools (macOS via Homebrew)
brew install adr-tools

# Initialize ADR directory in your project
adr init docs/decisions

# Create a new ADR (auto-increments the number)
adr new "Use Redis for caching"
# Creates: docs/decisions/0004-use-redis-for-caching.md

# List all ADRs
adr list

# Mark an ADR as superseded by a new one
adr supersede 3 7
# Updates ADR-003 status to "Superseded by ADR-007"
# Updates ADR-007 to note it supersedes ADR-003

# Generate a table of contents
adr generate toc
```

### Automating ADR Validation with Node.js

Here is a Node.js script that validates your ADR directory — checking that every ADR has required sections, that superseded ADRs link to their successors, and that numbering is sequential:

```javascript
// scripts/validate-adrs.js
// Run: node scripts/validate-adrs.js

const fs = require('fs');
const path = require('path');

const ADR_DIR = path.join(__dirname, '..', 'docs', 'decisions');
const REQUIRED_SECTIONS = [
  '## Status',
  '## Context',
  '## Decision',
  '## Consequences'
];
const VALID_STATUSES = [
  'Proposed',
  'Accepted',
  'Rejected',
  'Deprecated',
  'Superseded'
];

function validateADRs() {
  const files = fs.readdirSync(ADR_DIR)
    .filter(f => f.endsWith('.md'))
    .sort();

  const errors = [];
  const warnings = [];

  files.forEach((file, index) => {
    const filePath = path.join(ADR_DIR, file);
    const content = fs.readFileSync(filePath, 'utf8');
    const fileNum = parseInt(file.match(/\d+/)?.[0], 10);

    // Check numbering is sequential
    if (fileNum !== index + 1) {
      errors.push(
        `${file}: Expected ADR number ${index + 1}, got ${fileNum}`
      );
    }

    // Check required sections exist
    REQUIRED_SECTIONS.forEach(section => {
      if (!content.includes(section)) {
        errors.push(
          `${file}: Missing required section "${section}"`
        );
      }
    });

    // Check status is valid
    const statusMatch = content.match(/## Status\s+\n([^\n]+)/);
    if (statusMatch) {
      const status = statusMatch[1].trim();
      const statusWord = VALID_STATUSES.find(s => status.startsWith(s));
      if (!statusWord) {
        errors.push(
          `${file}: Invalid status "${status}". ` +
          `Must be one of: ${VALID_STATUSES.join(', ')}`
        );
      }
      // If superseded, check it references another ADR
      if (status.startsWith('Superseded') && !status.includes('ADR-')) {
        warnings.push(
          `${file}: Status is "Superseded" but does not reference successor ADR`
        );
      }
    }

    // Warn if ADR is still in Proposed status (might be forgotten)
    if (content.match(/## Status\r?\n\r?\nProposed/)) {
      warnings.push(
        `${file}: Still in Proposed status — has this been reviewed?`
      );
    }
  });

  // Report results
  if (errors.length > 0) {
    console.error('\n[ERROR] ADR VALIDATION ERRORS:');
    errors.forEach(e => console.error(`  * ${e}`));
  }

  if (warnings.length > 0) {
    console.warn('\n[WARN] ADR VALIDATION WARNINGS:');
    warnings.forEach(w => console.warn(`  * ${w}`));
  }

  if (errors.length === 0 && warnings.length === 0) {
    console.log(`\n[OK] All ${files.length} ADRs are valid.\n`);
  }

  process.exit(errors.length > 0 ? 1 : 0);
}

validateADRs();
```

Run this as part of your CI pipeline to maintain ADR hygiene automatically:

```yaml
# .github/workflows/validate-adrs.yml
name: Validate ADRs

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Validate ADR structure
        run: node scripts/validate-adrs.js
```

### Generating an ADR Index Automatically

Keep a `README.md` in your `docs/decisions/` directory that lists all ADRs as a table of contents, auto-generated by this script:

```javascript
// scripts/generate-adr-index.js
// Run: node scripts/generate-adr-index.js

const fs = require('fs');
const path = require('path');

const ADR_DIR = path.join(__dirname, '..', 'docs', 'decisions');
const INDEX_FILE = path.join(ADR_DIR, 'README.md');

function extractTitle(content) {
  const match = content.match(/^# (.+)$/m);
  return match ? match[1] : 'Untitled';
}

function extractStatus(content) {
  const match = content.match(/## Status\s*\n+([^\n]+)/);
  return match ? match[1].trim() : 'Unknown';
}

function extractDate(content) {
  const match = content.match(/## Date\s*\n+([^\n]+)/);
  return match ? match[1].trim() : '--';
}

function generateIndex() {
  const files = fs.readdirSync(ADR_DIR)
    .filter(f => f.endsWith('.md') && f !== 'README.md')
    .sort();

  const rows = files.map(file => {
    const content = fs.readFileSync(path.join(ADR_DIR, file), 'utf8');
    const title = extractTitle(content);
    const status = extractStatus(content);
    const date = extractDate(content);
    return `| [${title}](./${file}) | ${status} | ${date} |`;
  });

  const index = [
    '# Architecture Decision Records',
    '',
    'This directory contains all ADRs for this project.',
    '',
    '## Index',
    '',
    '| Title | Status | Date |',
    '|-------|--------|------|',
    ...rows,
    '',
    '---',
    '*Generated by `scripts/generate-adr-index.js`*',
  ].join('\n');

  fs.writeFileSync(INDEX_FILE, index);
  console.log(`[OK] ADR index written to ${INDEX_FILE}`);
}

generateIndex();
```

---

## 9. Best Practices

### The One-Page Rule

An ADR should fit on a single printed page — roughly 400–600 words. If your ADR is longer, you are probably trying to document too many decisions at once, or you are writing a design document rather than a decision record. Split it.

```
GOOD ADR LENGTH:
-------------------------------------------------
Title:         1 line
Status:        1 line
Date/Author:   2 lines
Context:       3-5 sentences
Decision:      2-3 sentences (clear, active voice)
Consequences:  5-10 bullet points
Alternatives:  3-6 bullet points per option
-------------------------------------------------
Total: ~400-600 words. Fits on one page.
```

### Write ADRs at Decision Time

The most important rule. Write the ADR **while the context is fresh** — ideally as part of the same pull request or sprint that implements the decision. Writing ADRs retrospectively produces lower-quality records because:

- The original options are hard to remember
- The reasoning becomes post-hoc rationalization
- The author's certainty about the decision's consequences is exaggerated

### Use Active, Decisive Language

**Bad:** *"The team felt that PostgreSQL might be a good choice due to various factors..."*

**Good:** *"We will use PostgreSQL as the primary database because it provides ACID transactions required for order integrity and our team has deep expertise with it."*

An ADR records a decision. Write it like one.

### Keep the History Intact — Only Supersede, Never Delete

When a decision changes, create a **new ADR** and mark the old one as "Superseded by ADR-NNN." The old ADR remains in the repository forever. This is crucial because:

- It shows the evolution of thinking
- The old decision's context explains why the new one was needed
- It prevents "why did we ever do it that way?" confusion

### Link ADRs to Code

When implementing a decision, link back to the ADR in code comments and pull request descriptions:

```javascript
// Using Redis for session storage
// See: docs/decisions/ADR-006-use-redis-for-caching.md
const session = require('express-session');
const RedisStore = require('connect-redis')(session);

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false
}));
```

### Store ADRs in the Repository

ADRs should live in the **same repository as the code** they govern. This ensures they are versioned together, visible to all contributors, and part of the pull request review process.

```
project-root/
+-- src/
+-- tests/
+-- docs/
|   +-- decisions/          <-- ADRs live here
|   |   +-- README.md       <-- Auto-generated index
|   |   +-- ADR-001-use-postgresql.md
|   |   +-- ADR-002-adopt-microservices.md
|   |   +-- ADR-003-use-kafka.md
|   +-- architecture/       <-- Higher-level architecture docs
+-- package.json
+-- README.md
```

---

## 10. Industry Standards

### RFC 2119 Language in ADRs

Many organizations adopt RFC 2119 keyword conventions in ADRs. Words like **MUST**, **SHOULD**, **MAY** carry precise meanings:

| Keyword | Meaning |
|---|---|
| **MUST** / **REQUIRED** | Absolute requirement |
| **MUST NOT** / **SHALL NOT** | Absolute prohibition |
| **SHOULD** / **RECOMMENDED** | Expected unless there is a good reason not to |
| **MAY** / **OPTIONAL** | Truly optional |

### Y-Statements

Some teams use **Y-statements** (a compact decision format) as a complement or alternative to full ADRs:

> *"In the context of [situation], facing [concern], we decided [option], to achieve [quality], accepting [downside]."*

Example:
> *"In the context of high-traffic product catalog pages, facing slow database response times, we decided to use Redis as a caching layer, to achieve sub-50ms page load times, accepting the added operational complexity of managing a Redis cluster."*

Y-statements can serve as the opening line of an ADR's Decision section to give instant clarity.

### Architecture Review Boards (ARB)

In large organizations, ADRs are submitted to an **Architecture Review Board** — a committee of senior engineers and architects who review and approve significant decisions before they are implemented. The ADR is the submission artifact.

```
ENGINEER            ARB PROCESS             OUTCOME
--------            -----------             -------
Writes ADR   -->   Submits for review  -->  Approved (Accepted)
                                        -->  Needs revision (Proposed)
                                        -->  Not taken (Rejected)
```

### ISO/IEC 42010 (Systems and Software Engineering)

ISO 42010 is the international standard for architecture description. It formalizes the idea that architectural decisions should be documented, including their rationale. ADRs are a lightweight, practical implementation of the rationale documentation that ISO 42010 calls for.

---

## 11. Common Mistakes

### Mistake 1 — Writing ADRs After the Fact

Writing ADRs months after a decision was made produces reconstructed history, not genuine reasoning. The options you did not choose are hard to remember accurately, and the context has changed. Write ADRs at the time of decision.

### Mistake 2 — Making ADRs Too Long

ADRs that become 10-page design documents lose their purpose. Nobody reads a 10-page ADR. Keep them to one page. If you need more detail, link to a separate design document.

### Mistake 3 — Vague Context Sections

"We needed a database" is not context. Context should explain *the specific constraints and forces* at the time of the decision: team size, traffic levels, budget, existing expertise, regulatory requirements.

### Mistake 4 — Ignoring ADRs After Writing Them

ADRs only have value if they are actually read. Common failure modes:
- Nobody knows ADRs exist (fix: mention them in your README and onboarding docs)
- They are not searchable (fix: use a proper index or search tool)
- They are not linked from code (fix: add ADR references in code comments and PR descriptions)

### Mistake 5 — Deleting or Editing Old ADRs

Editing an accepted ADR to reflect the new reality destroys the historical record. If the decision has changed, **supersede it with a new ADR**. Never alter the original.

### Mistake 6 — Treating Every Decision as an ADR

Not every decision needs an ADR. If you write ADRs for choosing variable names or updating library versions, the ADR library becomes noise and engineers stop reading it. Reserve ADRs for genuinely significant architectural decisions.

```
ADR QUALITY CHECK -- ASK THESE BEFORE WRITING:
--------------------------------------------------
[?] Is this decision hard to reverse?
[?] Does it affect multiple teams or services?
[?] Are there meaningful trade-offs?
[?] Would a future engineer be confused without this?

YES to 2+ questions  -->  Write the ADR
NO to all questions  -->  Skip it
--------------------------------------------------
```

---

## 12. Security & Performance Considerations

### Security: What to Include and Exclude

ADRs often contain sensitive reasoning — for example, "we chose not to use vendor X because of a security audit finding." Be thoughtful about what goes into ADRs that live in public repositories:

- **Do include:** Technology choices, architectural patterns, trade-offs
- **Do NOT include:** Specific vulnerability details, API keys, credentials, internal security audit findings verbatim

For ADRs containing sensitive context, maintain a **private ADR store** (a private repository or internal wiki) separate from your public-facing ADR library.

### Security Decision ADRs

Security-relevant decisions are among the most important to document:

```
Examples of security-critical ADRs:
----------------------------------------------------
ADR-012: Authentication Strategy (JWT vs. Session)
ADR-018: Secret Management (Vault vs. Secrets Manager)
ADR-024: Data Encryption at Rest Strategy
ADR-031: API Rate Limiting Strategy
ADR-039: Dependency Security Scanning Tooling
----------------------------------------------------
```

### Performance: ADR Review as a Gate

ADRs can serve as a **performance governance gate** — decisions that might significantly impact performance should include performance benchmarks or load estimates in their consequences section. This creates a record of the expected performance profile at decision time, which can later be compared to actual production metrics.

### ADR Storage: Repository vs. Wiki

```
STORAGE       PROS                      CONS
-----------   ------------------------  -------------------------
Git repo      Versioned, searchable,    Needs Markdown tooling
              linked from code,
              part of PR review

Wiki          Easy editing,             Drift from code, hard to
(Confluence,  non-engineers can read    link from source, search
Notion)                                 less precise

RECOMMENDATION: Git repository is primary source of truth.
                Wiki can mirror for discoverability only.
```

---

## 13. Related Technologies

### Architecture Decision Records vs. Related Documents

```
DOCUMENT TYPE          PURPOSE                       TYPICAL LENGTH
----------------------------------------------------------------------
ADR                    Single decision + rationale   1 page
RFC (Request for       Broad technical proposal      5-20 pages
Comments)              inviting community input
Design Document        Detailed technical design      10-50 pages
                       of a system or feature
Runbook                Operational procedures         5-15 pages
Post-Mortem            Analysis of an incident        2-5 pages
Architecture           High-level system overview     Varies
Diagram (C4, etc.)
----------------------------------------------------------------------
```

### Tech Radar

A **Tech Radar** (pioneered by ThoughtWorks) is a complementary tool — it shows which technologies are recommended (Adopt), being evaluated (Trial), used cautiously (Assess), or avoided (Hold) across an organization. Where a Tech Radar gives the *current state*, ADRs explain the *journey that got you there*.

### Architecture as Code (Structurizr, C4 Model)

Tools like **Structurizr** (based on Simon Brown's C4 Model) create living architecture diagrams from code. ADRs can be linked from Structurizr diagrams — clicking on a component surfaces the ADRs that govern it.

### Lightweight RFC Process

Some teams use a **lightweight RFC** (Request for Comments) process as a precursor to ADRs:
1. Engineer writes an RFC proposing a change
2. Community provides feedback via comments
3. RFC is finalized into an ADR once the decision is made

GitHub, Rust, and React all use RFC processes. RFCs capture the deliberation; ADRs capture the final decision.

### Architecture Governance Ecosystem

```
                  +------------------+
                  |   Tech Radar     |  <-- What is recommended org-wide
                  +------------------+
                           |
                           v
                  +------------------+
                  |  Architecture    |  <-- Who approves decisions
                  |  Review Board    |
                  +------------------+
                           |
                           v
                  +------------------+
                  |      ADRs        |  <-- What was decided and why
                  +------------------+
                           |
                           v
                  +------------------+
                  |  Code + Tests    |  <-- How the decision was implemented
                  +------------------+
```

---

## 14. Summary

### What We Learned

In this chapter, we started with a fundamental problem in software engineering: **architectural decisions get made, but their reasoning gets lost**. We introduced Architecture Decision Records as the solution — a lightweight, structured document format that captures the context, decision, and consequences of each significant architectural choice.

We explored:
- The **captain's log analogy** — ADRs as the ship's log of your architecture
- The **Michael Nygard format** — minimal, four-section ADRs
- The **MADR format** — extended ADRs with alternatives and pros/cons
- **ADR lifecycle statuses** — Proposed, Accepted, Deprecated, Superseded
- Three complete, realistic ADR examples: PostgreSQL, Microservices, Kafka
- The **full workflow** — from identifying a decision, to drafting, reviewing, and ratifying
- **Real-world adoption** at GitHub, Spotify, CNCF projects, and Zalando
- **Tooling** — adr-tools CLI, Node.js validation scripts, auto-generated indexes
- The **golden rules**: write at decision time, keep them short, never delete — only supersede

### Key Takeaways

```
+------------------------------------------------------------------+
|                      ADR KEY TAKEAWAYS                           |
+------------------------------------------------------------------+
| 1. ADRs capture WHY, not just WHAT. Code shows what was built;  |
|    ADRs show why it was built that way.                          |
|                                                                  |
| 2. Write at the moment of decision -- context fades fast.        |
|                                                                  |
| 3. One page maximum. Short ADRs get read; long ADRs do not.     |
|                                                                  |
| 4. Never delete -- only supersede. History has value.            |
|                                                                  |
| 5. Store in the repository alongside the code.                   |
|                                                                  |
| 6. Not every decision needs an ADR -- only the significant ones. |
|                                                                  |
| 7. Link ADRs from code, PRs, and README files.                  |
|                                                                  |
| 8. ADRs are a team practice, not an individual one.              |
+------------------------------------------------------------------+
```

---

## 15. Keywords

`Architecture Decision Record`, `ADR`, `Michael Nygard`, `MADR`,
`decision log`, `architectural decision`, `decision rationale`,
`software documentation`, `technical documentation`, `adr-tools`,
`architecture governance`, `Architecture Review Board`, `ARB`,
`decision status`, `superseded`, `Y-statement`, `RFC`, `Tech Radar`,
`Conway's Law`, `ISO 42010`, `institutional memory`, `onboarding`,
`decision lifecycle`, `Markdown ADR`, `docs as code`, `decision record`,
`architectural knowledge management`, `compliance`, `audit trail`,
`Proposed`, `Accepted`, `Rejected`, `Deprecated`

---

## 16. Glossary

| Term | Definition |
|---|---|
| **ADR** | Architecture Decision Record — a short document capturing a single architectural decision, its context, and its consequences |
| **MADR** | Markdown Architectural Decision Records — an extended ADR format that adds alternatives, pros/cons, and metadata |
| **Proposed** | ADR status indicating a decision is under discussion and not yet finalized |
| **Accepted** | ADR status indicating a decision was made and is currently in effect |
| **Rejected** | ADR status indicating a decision was considered but not taken |
| **Deprecated** | ADR status indicating a decision was once accepted but is no longer recommended |
| **Superseded** | ADR status indicating a decision was replaced by a newer ADR (always links to the successor) |
| **adr-tools** | A command-line tool for scaffolding and managing ADRs in a project directory |
| **Architecture Review Board (ARB)** | A committee of senior engineers and architects who review and approve significant architectural decisions |
| **Y-statement** | A compact one-sentence decision format: "In the context of X, facing Y, we decided Z, to achieve Q, accepting R" |
| **Tech Radar** | A visualization (pioneered by ThoughtWorks) of an organization's technology adoption posture across Adopt, Trial, Assess, and Hold rings |
| **RFC (Request for Comments)** | A document inviting broad community input on a proposed change, often preceding a formal ADR |
| **ISO 42010** | The international standard for systems and software architecture description, which formalizes architectural rationale documentation |
| **Conway's Law** | "Organizations design systems that mirror their own communication structure" — a principle often referenced in ADRs about team and service organization |
| **Institutional Memory** | The accumulated knowledge and reasoning of an organization, preserved in documents like ADRs rather than in the heads of individuals |
| **Decision Driver** | A force, constraint, or quality attribute that influenced which option was chosen |
| **Positive Consequence** | A benefit or improvement that follows from the decision |
| **Negative Consequence** | A trade-off, constraint, or cost that follows from the decision |
| **Architectural Amnesia** | The phenomenon where architectural reasoning is forgotten over time as team members leave and context disappears |
| **Decision Re-litigation** | The recurring debate of a decision that was already made, caused by the absence of a written record |

---

## 17. Next Recommended Chapters

Having mastered Architecture Decision Records, the following chapters build naturally on this foundation:

1. **Chapter 15: Architecture Governance** — Learn how ADRs fit into a broader governance process, including Architecture Review Boards, compliance frameworks, and how organizations enforce architectural standards at scale.

2. **Chapter 16: The C4 Model for Architecture Diagrams** — ADRs explain *why* decisions were made; C4 diagrams show *what* the resulting architecture looks like. Learn how to create Context, Container, Component, and Code diagrams that live alongside your ADRs.

3. **Chapter 17: Domain-Driven Design (DDD)** — Many ADRs describe decisions about bounded contexts and domain decomposition. DDD gives you the language and tools to make those decisions rigorously.

4. **Chapter 18: Technical Debt Management** — ADRs that record pragmatic choices ("we will use X for now, revisit when we reach Y scale") are the foundation of a technical debt register. Learn how to track, prioritize, and pay down technical debt systematically.

5. **Chapter 10: Microservices Architecture** *(if not yet read)* — ADR-002 in this chapter adopted microservices. Chapter 10 goes deep on the patterns, anti-patterns, and trade-offs of microservices — the kind of content that arms you to *write* that ADR with confidence.

---

*End of Chapter 14: Architecture Decision Records (ADRs)*

---

> **Pilot-Seat Curriculum** | Module 9: Architecture Fundamentals
> Chapter 14 of 20 | Estimated reading time: 35 minutes
