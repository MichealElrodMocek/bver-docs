
# BVER Runtime
## Define the domain. BVER builds the system.

BVER Runtime is a domain-native execution engine that compiles domain definitions into fully operational backend systems.

Developers define domain primitives such as references, jobs, orchestrators, and contracts. BVER constructs the execution system, APIs, orchestration environment, and cross-service addressing automatically.

BVER Runtime provides:

- Automatic REST and MCP operation generation
- Reference storage and versioning integration
- Job and orchestrator execution engines
- Event emission and trigger integration
- Cross-service reference resolution
- Pointer-based universal addressing
- Auth scope enforcement
- Contract and manifest generation

The runtime is the execution substrate of the platform.

---

# Core Philosophy

Traditional systems are built infrastructure-first:

    infrastructure → services → endpoints → logic → domain

BVER is built domain-first:

    domain → references → operations → runtime → system

The domain defines the system. Infrastructure is a runtime detail.

---

# Core Primitives

## References

References are persistent domain objects managed by a service.

Examples:

- Site
- Drawing
- Patient
- Device
- Case

References define:

- schema
- kind taxonomy
- version history
- diff tracking
- auth policies
- trigger emission

References are always owned by exactly one service.

References are versioned. Every mutation produces:

- version snapshot
- diff record
- trigger event

References never cross service boundaries as objects. Only pointers cross boundaries.

---

# Storage Abstraction and Automatic Migration

Engineering domain modeling should not be blocked by database mechanics.

BVER is storage-agnostic by default:
- Reference schemas are defined in the domain (not in hand-written SQL migrations)
- The runtime persists references through a storage adapter (Postgres, SQLite, etc.)
- Schema evolution is handled via generated migration plans

Migration behavior (target design)
- Additive changes (new optional fields) apply automatically.
- Potentially destructive changes (renames, type changes, deletes) require an explicit prompt/ack.
- Before applying destructive changes, Sandbox can run a "sweep" to surface what data would be lost.

Destructive change safety options (evolves over time)
- Drop after confirmation: remove the column/field only when the user accepts the loss.
- Export: write an artifact snapshot (CSV/JSON) of the affected values before dropping.
- Archive: move old values into a generic extension blob for later recovery.
- Merge strategy: propose a mapping from old -> new fields when a rename/split/merge is intended.

The guiding principle: if you wake up and decide a field is misleading, you change the domain object first; BVER helps you evolve storage safely and deliberately.

---

## Jobs

Jobs are deterministic compute tasks.

Jobs represent:

    known input → known output

The runtime provides:

- execution tracking
- status persistence
- retry handling
- event emission
- job handles

Jobs are exposed automatically as operations.

---

## Orchestrators

Orchestrators are dynamic workflows that coordinate operations across services.

Orchestrators:

- invoke operations
- resolve reference pointers
- coordinate multi-step workflows
- emit events
- respond to triggers

Orchestrators are the primary mechanism for cross-service system composition.

Services own their data. Orchestrators compose across services.

---

## Operations

Operations are the universal execution interface.

Examples:

    refs.site.create
    refs.site.patch
    refs.site.get
    jobs.reindex.start
    orch.design_site.start

Operations define:

- input schema
- output schema
- auth scopes
- execution behavior

Operations are exposed automatically via REST, MCP, and SDKs.

---

# ReferencePointers and AttributePaths

ReferencePointers are the universal addressing mechanism in BVER.

ReferencePointer canonical form:

    bver://{service}/{repo}/{id}@{attribute_path}

Examples:

    bver://sites/site/123
    bver://sites/site/123@name
    bver://drawings/drawing/456@meta.sheet_number

ReferencePointers uniquely identify:

- references
- attributes
- nested attribute values

AttributePaths use human-readable dotted syntax:

    name
    meta.owner.name
    terminals[0].label

ReferencePointers allow dynamic resolution across service boundaries.

ReferencePointers are first-class platform primitives.

---

# Pointer Resolution

Pointer resolution is performed by the runtime resolver.

Resolution process:

1. Parse pointer
2. Identify owning service
3. Resolve locally or via adapter
4. Fetch reference
5. Apply attribute path
6. Return value

Runtime interface:

    bver.get(pointer)
    bver.resolve(pointer)
    bver.exists(pointer)
    bver.list(repo)
    bver.listrefs(service)

Pointer resolution works across service boundaries transparently.

---

# Cross-Service Boundary Model

BVER enforces strict service ownership boundaries.

Rules:

- References are owned by exactly one service
- Foreign references are represented as pointers
- Services do not embed foreign reference objects
- Services do not mutate foreign reference state directly
- Cross-service changes occur via operations
- Orchestrators coordinate cross-service workflows

Pointers allow safe cross-service interaction without breaking service ownership.

---

# Shared Contracts

Cross-service data contracts must use:

- BVER core types (RefPtr, AttributePointer, JobHandle, etc)
- Shared contract types from a shared package

Shared packages define stable cross-service payload types.

Shared packages are versioned and imported normally.

Services must not import internal models from other services.

---

# Dynamic Kinding

References may support dynamic kinds.

Kinds extend reference schema dynamically while preserving identity.

Example:

    Site(kind="power.substation")
    Site(kind="telecom.node")

Kinds allow polymorphic domain modeling.

---

# Figments

Figments are reusable capability fragments.

Figments define:

- schema fragments
- operations
- triggers
- validation logic

Figments allow reuse across services.

---

# Manifest

Each service produces:

    bver.manifest.json

The manifest defines:

- references
- operations
- jobs
- orchestrators
- triggers
- pointer resolution contracts

Manifests define the service contract.

---

# Runtime Responsibilities

The runtime manages:

- capability registration
- operation generation
- pointer resolution
- job execution
- orchestrator execution
- trigger emission
- contract generation

---

# Summary

BVER Runtime transforms domain definitions into operational systems.

ReferencePointers provide universal addressing.

Services own references. Orchestrators compose systems.

The runtime builds the system automatically.
