# BVER Development Roadmap (Practical, Build-Order Focused)

**Purpose:** A staged plan for building BVER so you can keep momentum, avoid architecture thrash, and prove value early—without committing to the full ecosystem all at once.

**Audience:** You (primary), early contributors, future plugin authors  
**Assumption:** You want *domain-first* development (objects + capabilities) while still designing the ecosystem pieces (Sandbox, local deploy, plugin store, EAGER).

---

## Guiding Constraints (Non‑Negotiables)

1. **Ship a running system early.** Every phase ends with something you can install and run.
2. **Define contracts before endpoints.** Capabilities expose typed contracts; transport is secondary.
3. **Don’t build the cloud first.** Local-first must be credible; cloud is an additive convenience.
4. **Ecosystem comes after kernel.** Marketplace / EAGER only work if Runtime + Compiler are stable enough.
5. **Version everything.** Runtime API level, plugin manifests, DTO schemas, migration plans.

---

## Roadmap Overview

- **Phase 0 — Foundations (1–2 weeks)**
- **Phase 1 — Minimum Runnable Platform (v0 “it works”)**
- **Phase 2 — Plugin System (local-first)**
- **Phase 3 — EAGER (schemas + events, minimal)**
- **Phase 4 — Sandbox as a product (dev UX)**
- **Phase 5 — Local Deploy (on‑prem delivery)**
- **Phase 6 — Marketplace + Control Plane (optional, later)**
- **Phase 7 — Hosted execution (Forge) (much later)**

> You can pause after Phase 2 and still have a real, valuable internal platform foundation.

---

## Phase 0 — Foundations (No Magic, Just Discipline)

### Outcomes
- A repo structure and naming that won’t collapse later
- A minimal “platform domain” package format
- The first set of invariants: identifiers, manifests, version rules

### Build
- **Canonical IDs**
  - Object type IDs (namespaced)
  - Capability IDs (namespaced)
  - Contract IDs (schema/DTO IDs)
- **Manifest formats** (JSON/YAML/Pydantic models)
  - `platform.toml` / `bver_platform.yaml` (pick one)
  - `plugin.toml` / `bver_plugin.yaml`
  - Version fields everywhere

### Decide early
- **Your “unit of distribution”:** wheel / OCI image / both  
  Recommend: *wheels for dev + OCI images for deploy*.
- **Your “unit of execution”:** process / container  
  Recommend: *process first; container later*.

---

## Phase 1 — Minimum Runnable Platform (Runtime + Compiler + One Adapter)

### Outcomes
- A platform that boots locally
- A small set of base objects you can create/read/update
- A job runner that can execute one “capability”
- A migration plan generator that prints diffs (even if it doesn’t apply yet)

### Build (in this order)

#### 1) Runtime kernel (minimal)
- Service lifecycle (start/stop)
- Capability registry (in-memory to start)
- Adapter registry (in-memory to start)
- Job execution interface (sync first; async later)
- Logging + correlation IDs (required)

#### 2) Domain object model
- Base objects + value structures rules
- Pointer/reference model (how objects point to objects)
- Storage abstraction (but only implement Postgres initially)

#### 3) One adapter: Postgres
- Object persistence for base objects
- Simple query mechanism (by ID + basic filters)
- Minimal audit trail (created_at, updated_at, version integer)

#### 4) Compiler (v0)
- Reads `PlatformDomain` definition
- Produces a “compiled graph” artifact (your `.bdam` seed)
- Validates manifests and prints a “plan”:
  - objects discovered
  - capabilities discovered
  - adapters required

### Exit criteria
- You can run: `bver up` and create a `Site/Substation` object and store it.
- You can register one toy capability and execute it.

---

## Phase 2 — Plugin System (Local-First, No Marketplace Yet)

### Outcomes
- Plugins install locally, register capabilities, and run
- Capability contracts are enforced (typed DTOs)
- Compatibility is checked (runtime API level, schema versions)

### Build

#### 1) Plugin packaging format
- Plugin manifest fields:
  - `plugin_id`, `version`, `runtime_api_min/max`
  - capabilities (IDs, input/output contract IDs)
  - permissions (declared scopes)
  - optional object packs (new kinds/base objects)
- Local install:
  - from wheel / from folder / from git ref (pick one first)

#### 2) Capability invocation API (stable surface)
- `invoke(capability_id, inputs, context) -> outputs`
- The runtime:
  - validates input schemas
  - provides object pointers/resolution
  - records job results + logs

#### 3) Local registry
- File-based registry (SQLite or JSON) is fine for v0
- Tracks installed plugins, versions, compatibility status

### Exit criteria
- Install a plugin, see it in `bver plugins list`, run it on a `Site/Substation`.

---

## Phase 3 — EAGER (Minimal: Schema Registry + Change Events)

### Outcomes
- You can define/version DTO schemas
- You can publish object-change events and subscribe locally

### Build (keep it small)

#### 1) Schema registry (local)
- Store schemas (JSON Schema / Pydantic exported)
- Versioning policy:
  - semantic versioning for schemas
  - compatibility checks (backward/forward rules)

#### 2) Change event bus (local)
- Start with one broker (RabbitMQ *or* NATS *or* Postgres LISTEN/NOTIFY)
- Event envelope:
  - `event_id`, `timestamp`, `channel`, `schema_id@version`
  - `object_pointer`, `change_type`, `payload`

#### 3) Subscription API
- Local subscriber can listen for channel changes
- Useful first use case: “indexer” or “audit watcher”

### Exit criteria
- Updating an object emits an event and a subscriber receives it.

---

## Phase 4 — Sandbox Becomes a Product (Dev UX)

### Outcomes
- Reproducible local environments
- CI-friendly
- Migration planning + safety checks become real

### Build

#### 1) Sandbox CLI commands
- `bver init`, `bver up`, `bver down`, `bver reset`
- `bver doctor` (diagnostics)
- `bver logs`, `bver status`

#### 2) Migration planning pipeline (real)
- Detect schema changes from domain definitions
- Produce:
  - diff report
  - migration plan
  - “destructive change” prompts and safeguards

#### 3) Dev artifact outputs
- Build runtime bundle (zip/dir)
- Build plugin packages
- Generate docs and SDK stubs (even minimal)

### Exit criteria
- A new machine can clone + run USDP example in < 10 minutes.

---

## Phase 5 — Local Deploy (On‑Prem Delivery)

### Outcomes
- Installation/upgrade path that is boring and reliable
- Rollback plan
- Backups + health checks

### Build

#### 1) Deployment artifact format
- “runtime bundle” + “plugin bundle”
- signed manifest (optional early; mandatory later)

#### 2) Upgrade workflow
- preflight checks
- backup
- apply migration plan
- restart services
- verify health
- rollback on failure

#### 3) Ops surfaces (minimal)
- health endpoints
- metrics export (even basic)
- structured logs

### Exit criteria
- You can upgrade a running local install without deleting data.

---

## Phase 6 — Marketplace + Control Plane (Later, Not First)

### Outcomes
- Multi-tenant governance
- Plugin entitlements, signing, provenance
- Shared registries that teams can rely on

### Build (only after you have users)
- Control plane:
  - org/project/tenant model
  - identity integration
  - plugin entitlements
- Marketplace:
  - publishing workflow
  - version compatibility verification
  - signing/provenance

### Exit criteria
- A second team (not you) can safely install and update plugins.

---

## Phase 7 — Hosted Execution (Forge) (Much Later)

### Outcomes
- Remote execution of “hosted plugins”
- Metering, quotas, billing events (if needed)

> Don’t touch this until local plugins + deploy are stable and people ask for hosted compute.

---

## Recommended “First Proof” Use Cases (Pick 1–2)

1) **Substation Site + Asset inventory**
- Base objects: `Site/Substation`, `Asset/Transformer`, `Document/OneLine`
- One generator plugin: “Create starter substation skeleton”
- One validator plugin: “Standards sanity check”

2) **Drawing ingest pipeline (toy)**
- Upload file -> create `Document` -> emit EAGER event -> subscriber indexes metadata

---

## What to Defer (Avoid the Trap)

- Full UI framework generation
- Full multi-tenant control plane
- Global marketplace
- Hosted GPU scheduling
- Perfect object-kind taxonomy exchange

Build the kernel, prove the loop, then expand.

---

## “Definition of Done” for v0

You have v0 when you can:
- define a domain in code
- boot locally with Sandbox
- install a plugin
- run a capability as a job
- store objects in Postgres
- produce a migration plan
- emit an EAGER change event

That’s already a real platform.
