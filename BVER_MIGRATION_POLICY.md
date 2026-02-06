BVER Migration Policy

Version: 0.1.0
Status: Draft (MVP framing)
Audience: Platform engineers, plugin authors

Goal
Engineers should evolve domain objects without hand-authoring database migrations. Storage should feel like an implementation detail, while still protecting data when changes are destructive.

Scope
This policy covers schema evolution for persisted reference storage (base objects and their kinds), whether those reference types are defined by the core application repo or by plugins.

Key terms
- Reference (base object): a referenceable-by-default entity with identity and lifecycle.
- Value structure: non-referenceable-by-default data embedded in references or passed via contracts.
- Schema diff: a computed change set between "domain definition" and "persisted storage".
- Migration plan: an ordered set of actions to reconcile storage with the current domain definition.
- Sweep: an optional, prompted scan of persisted values to quantify and sample what would be affected by a destructive change.
- Artifact: a durable output produced during a migration (export, report, logs).

Principles
1. Domain-first: change the domain model first; migration planning follows automatically.
2. Safe-by-default: additive changes auto-apply; destructive changes require explicit confirmation.
3. Auditable: every applied migration produces a report and is recorded.
4. Reversible-when-possible: when a destructive change is requested, provide export/archive options.
5. Local-first: migrations and sweeps run in on-prem Sandbox environments without cloud dependencies.

Change classification
Safe / automatic (no prompt)
- Add optional attribute (nullable/optional field)
- Add new kind or kind-specific fields (when they do not change existing persisted values)
- Add index (when supported by the storage adapter)

Potentially destructive (requires prompt)
- Drop attribute / column
- Rename attribute (treated as drop + add unless a mapping is provided)
- Type change (e.g., string -> int, list -> scalar)
- Tighten constraints (nullable -> non-null, max length, enum narrowing)
- Move/reshape nested structures (object split/merge)

Default behaviors
- Additive changes apply automatically during `bver sandbox trial` (or equivalent) unless disabled.
- Destructive changes are blocked until explicitly acknowledged.
- When blocked, the user is shown:
  - what changed (schema diff)
  - what will happen (migration plan)
  - what could be lost (sweep summary, if requested)

Sweep behavior (prompted)
Sweep exists to make data loss visible before it happens.

When a destructive change is proposed, Sandbox can:
- Count impacted records
- Provide representative samples (bounded)
- Surface "unknowns" (e.g., values that cannot be coerced safely)
- Emit a migration report artifact

Destructive-change safety options
At prompt time, the platform should offer one or more of:
- Confirm drop: remove the field/column and accept loss
- Export: write affected values to an artifact (CSV/JSON) before dropping
- Archive: move values into an archive blob/namespace for later recovery
- Map/merge: provide a mapping strategy (rename, split, merge) so values are transformed rather than lost

Operational requirements
- Storage adapter abstraction: the runtime persists references through an adapter (Postgres, SQLite, etc.).
- Deterministic plan: given the same domain definition and storage state, the migration plan should be stable.
- Correlated logging: migrations emit correlation IDs and structured logs.
- Rollback strategy: when supported by adapter/transaction semantics, apply within a transaction; otherwise require explicit backups.

Plugin considerations
- Plugins may define their own base objects (reference types) and kinds to add capability to a platform.
- A plugin-defined base object is still a base object: referenceable by default, versioned, auditable, and persisted via the same adapter abstraction.
- Ownership is explicit: each base object type has exactly one owning service at runtime (core app or a plugin service).
- Collisions are not allowed: base object type identifiers must be globally unique (namespaced).

Recommended UX (MVP)
- `bver sandbox diff`: show schema diff and proposed plan
- `bver sandbox migrate`: apply safe changes; prompt for destructive steps
- `bver sandbox sweep`: run sweep and attach report artifact

The guiding promise
If you wake up tomorrow and decide an attribute on `Site` is misleading, you change the domain object first. BVER then plans the storage changes, shows you what would be lost, and only drops data after you explicitly confirm (or choose an export/archive/merge strategy).

