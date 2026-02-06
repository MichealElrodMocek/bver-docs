# When to Use BVER (and When Not To) + Simple Example
## BVER in One Sentence

BVER is a domain-first framework for building engineering platforms where **domain evolution and capability composition** matter more than hand-written endpoints.

---

## Use BVER When…

### ✅ Your project has a “real-world domain”
- You model assets, sites, documents, drawings, people, materials, rules
- Objects have identity, lifecycle, and relationships
- The system will live for years and evolve

### ✅ Your data model changes often (or will)
- New fields, new object kinds, new workflows
- You already feel pain from client breakage, endpoint churn, or migration anxiety

### ✅ You need capabilities, not just CRUD
- Generators, solvers, validators, orchestrators
- Long-running jobs with tracking and audit
- Repeatable workflows with “proof” and traceability

### ✅ Multiple teams/tools need to interoperate
- Shared contracts and DTOs
- Stable plugins that can be reused across projects
- A governed way to subscribe to change events (EAGER)

---

## Think Twice When…

### ⚠️ The project is small, stable, or short-lived
- A script, a small internal app, a proof-of-concept
- The model is simple and unlikely to evolve
- A standard web stack will do

### ⚠️ You don’t want any platform commitments
- You want maximum speed today and are fine with rework later
- You prefer handwritten endpoints and conventional deployment

---

## Don’t Use BVER When…

### ❌ You are building a generic consumer application
Examples that are usually a poor fit:
- Social media platforms
- Chat apps
- File-sharing / Dropbox-like services
- Generic CMS / blog systems
- CRUD-only SaaS with shallow domain semantics

These are not “bad” apps—just better served by traditional stacks and existing products.

### ❌ Your primary complexity is UI novelty, not domain semantics
If the hard part is a unique front-end experience rather than object lifecycles and capability orchestration, BVER won’t be the leverage point.

---

## The Honest Infrastructure Note

BVER does **not** remove infrastructure from the universe.

- Databases, message brokers, identity providers, and cloud services still exist.
- Many services will be built **for** BVER (hosting, CI, deployment, observability), not *on* BVER.
- The win is that you stop re-implementing infrastructure *per domain change*.

BVER reduces **accidental complexity**, not **essential complexity**.

---

## Why BVER Exists (Hypothetical Origin Story)

Imagine a domain engineer building a serious system (e.g., power planning, industrial layouts, or asset tracking).

They start simple with a desktop app. Deployment becomes the first trap:
- Distribute executables and become an IT department, or
- Move to a server architecture and inherit infrastructure work

Once the server architecture begins, the stack grows fast:
- Postgres for persistence
- RabbitMQ for jobs and events
- Search engines for fast queries
- Proxies/routers for secure access
- Job orchestration and monitoring

Even if the team can handle those tools, the real pain is this: every domain change multiplies the infrastructure work.
- New endpoints
- New SDKs
- UI rewrites
- Migration headaches

At scale, segmentation into microservices adds even more coordination and operational overhead.

BVER exists to close that gap. It makes domain-first modeling the center of the system and treats the infrastructure as a shared platform problem.

---

## What the Infrastructure Stuff Actually Does (Plain English)

Database (Postgres)
- Stores your objects permanently, safely, and consistently
- Becomes painful when schemas evolve (migrations, backups, performance)

Message Broker (RabbitMQ)
- Moves tasks and events between components
- Becomes painful with reliability, retries, and ordering

Search Engine (OpenSearch/Elasticsearch)
- Provides fast, flexible search
- Becomes painful with indexing pipelines and drift

Identity and Auth (OIDC, SSO, API keys)
- Controls who can access the system and what they can do
- Becomes painful with roles, policies, and audit requirements

Secrets Management (Vault, KMS)
- Stores credentials and keys safely
- Becomes painful with rotation, access policies, and incident response

Routers and Proxies (Nginx/Envoy)
- Safely route requests to the right place
- Becomes painful with TLS, routing rules, and scaling

Job System (Celery, etc.)
- Runs heavy tasks in the background
- Becomes painful with scheduling, retries, idempotency

SDKs and Client Glue
- Keep clients working as the backend evolves
- Becomes painful with versioning and docs

---

## How BVER Maps to Those Concepts

- References (base objects)
  - Define objects once; BVER persists and versions them
  - Migrations are planned and applied for you

- Jobs
  - Define long-running work; BVER runs and tracks it
  - Retries and logs are built in

- Orchestrators
  - Define multi-step workflows; BVER coordinates and audits

- Reference Pointers
  - Refer to objects without passing raw data
  - BVER resolves them on demand

- EAGER (Data Mesh)
  - Subscribe to change events without custom glue
  - Enables platform segmentation without manual wiring

---

## The Tough Sell (Decision Moment)

If you skip BVER, you are choosing to:
- Build and maintain infrastructure yourself
- Write custom CRUD and endpoints for every domain change
- Rebuild SDKs and UIs repeatedly as the model evolves
- Own scaling, reliability, and operational correctness forever

If you choose BVER, you are choosing to:
- Define the domain once and evolve it safely
- Get capabilities, jobs, and orchestration without bespoke plumbing
- Scale by declaring intent, not by hand-wiring infrastructure

---

## Simple Example Use Case (Minimal and Real)

### Goal
Create a tiny “Home Power Planning” platform that can:
1) Store a `Site/Home`
2) Store a `Generator` asset
3) Run a plugin capability: “Generate a starter home power layout”

### Base objects + kinds
- `Site/Home`
- `Asset/Generator`
- `Document/SingleLine` (optional)

### Capability plugin: `home.plugin.layout_generator`
**Type:** Generator  
**Inputs:**
- Pointer to `Site/Home`
- Config values (house size, load tier)

**Outputs:**
- New assets created (list of pointers)
- A “layout report” object (or job artifact)

### What happens
1) User creates a `Site/Home` object.
2) User invokes the generator plugin.
3) Runtime validates input contract, executes the job.
4) Plugin creates:
   - `Asset/Generator`
   - `Asset/TransferSwitch` (optional later)
5) Runtime stores objects, emits EAGER change events.
6) A subscriber (optional) listens and indexes changes.

### Why this is a good first example
- It proves the domain-first loop
- It proves plugins and capabilities
- It proves object persistence + evolution
- It stays simple enough to ship early

---

## Quick Checklist

Use BVER if you answer “yes” to **two or more**:

- Will the domain model evolve significantly?
- Do multiple tools/teams need shared contracts?
- Do I need jobs, workflows, validators, or generators?
- Will I regret hand-built endpoints in 6 months?
- Do I want reusable capability plugins?

If “no” across the board, use a normal stack and move fast.
