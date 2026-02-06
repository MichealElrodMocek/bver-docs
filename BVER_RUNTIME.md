
# BVER Runtime
## Define the domain. BVER builds the system.

BVER Runtime is a domain-native execution engine that compiles domain definitions into fully operational backend systems.

Developers define domain primitives such as references, jobs, orchestrators, and contracts. BVER constructs the execution system, APIs, orchestration environment, and pointer-based addressing automatically.

BVER Runtime provides:

- Automatic REST and MCP operation generation
- Reference storage and versioning integration
- Job and orchestrator execution engines
- Event emission and trigger integration
- Reference resolution (local-first)
- Pointer-based universal addressing
- Auth scope enforcement
- Contract and manifest generation

The runtime is the execution substrate of the platform.

---

# Core Philosophy

Traditional systems are built infrastructure-first:

    infrastructure → plumbing → endpoints → logic → domain

BVER is built domain-first:

    domain → references → operations → runtime → system

The domain defines the system. Infrastructure is a runtime detail.

---

# Core Primitives

## References

References are persistent domain objects managed by a platform application (and its installed plugins).

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

References are always owned by exactly one owner (core app or a plugin).

References are versioned. Every mutation produces:

- version snapshot
- diff record
- trigger event

References are never "passed around" as embedded objects between capabilities. Capabilities exchange pointers and contracted DTO payloads, and the runtime resolves pointers as needed.

---

# Storage Abstraction and Automatic Migration

Engineering domain modeling should not be blocked by database mechanics.

BVER is storage-agnostic by default and supports automatic migration planning and prompted data-loss sweeps.

Canonical policy: `BVER_MIGRATION_POLICY.md`

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

## Reference Locking (Leases) for Robust Jobs

Jobs should not directly receive embedded reference objects. They should receive pointers, and the runtime resolves references when needed.

To keep the system stable under concurrency, the runtime should support reference locking for writes:
- When a job starts, it can request a write lease on one or more reference pointers.
- While the lease is held, other mutations to those references are blocked (or forced into a conflict path) until the job completes, fails, or is terminated.
- Leases have timeouts/heartbeats so a crashed worker does not lock the system forever.

Recommended pattern (Celery/RabbitMQ or similar runners)
- Submit job with reference pointers only.
- Worker begins job -> acquire write lease(s) for the pointers it will mutate.
- Perform work with resolved references.
- Commit mutations -> release lease(s).
- On failure/termination -> release lease(s) (or let them expire).

Configurable healing behavior (optional)
- On write conflict: either fail fast, retry with backoff, or "restart on write" (re-run the job when the references become writable again).
- On lease loss/timeout: treat as cancellation and emit an audit event.

This makes "who is allowed to write right now" a runtime concern, not something plugin authors have to reinvent.

---

## Orchestrators

Orchestrators are dynamic workflows that coordinate operations across capabilities (core app code + plugins).

Orchestrators:

- invoke operations
- resolve reference pointers
- coordinate multi-step workflows
- emit events
- respond to triggers

Orchestrators are the primary mechanism for system composition inside a single BVER platform.

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

    bver://{namespace}/{repo}/{id}@{attribute_path}

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

ReferencePointers allow dynamic resolution without the designer thinking about networking or infrastructure.

ReferencePointers are first-class platform primitives.

---

# Pointer Resolution

Pointer resolution is performed by the runtime resolver.

Resolution process:

1. Parse pointer
2. Identify owning repo/owner
3. Resolve locally or via adapter (storage, files, external systems)
4. Fetch reference
5. Apply attribute path
6. Return value

Runtime interface:

    bver.get(pointer)
    bver.resolve(pointer)
    bver.exists(pointer)
    bver.list(repo)
    bver.listrefs(namespace)

Pointer resolution is a platform primitive; the runtime decides how/where the data is loaded.

---

# Composition and Boundaries (No Segmentation Required)

BVER does not require splitting the domain into separate networked components to stay sane.

Rules:

- Each reference type has exactly one owner in the platform (core app or a plugin).
- Capabilities operate via operations with explicit input/output contracts.
- Use pointers to refer to references, not embedded reference snapshots.
- Orchestrators compose capabilities and enforce gating/policy/audit.

If an organization truly needs separate platforms, they run separate BVER platforms and connect them via EAGER subscriptions (not in-process imports or bespoke networking glue).

---

# Shared Contracts

Cross-plugin (and cross-platform) data contracts should use:

- BVER core types (RefPtr, AttributePointer, JobHandle, etc)
- Shared contract types from a shared package

Shared packages define stable payload types for composition.

Shared packages are versioned and imported normally.

Plugins should not import internal models from the core app (or other plugins). Use contracts and EAGER channels for sharing.

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

Figments allow reuse across plugins and apps.

---

# Manifest

Each application bundle (core app and plugins) produces:

    bver.manifest.json

The manifest defines:

- references
- operations
- jobs
- orchestrators
- triggers
- pointer resolution contracts

Manifests define the capability contract.

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

Reference types have explicit owners. Orchestrators compose systems.

The runtime builds the system automatically.
