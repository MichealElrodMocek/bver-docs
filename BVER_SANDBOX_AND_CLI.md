
# BVER Sandbox and CLI
## Domain validation, execution, and integration environment

BVER Sandbox is the development and validation environment for BVER systems.

Sandbox validates domain correctness, pointer resolution, capability contracts, and runtime compatibility.

Sandbox enables building complex systems without infrastructure expertise.

---

# Core Purpose

Sandbox provides:

- Bundle build validation (core app + plugins)
- Manifest generation
- Pointer contract validation
- Cross-plugin contract validation
- Runtime trial execution
- Contract correctness enforcement

Sandbox ensures domain correctness before deployment.

---

# Pointer Validation

Sandbox validates ReferencePointers.

Validation includes:

- pointer syntax correctness
- referenced namespace/repo existence
- attribute path validity
- version qualifier validity

Sandbox prevents invalid references and contract drift.

---

# Capability Boundary Validation

Sandbox enforces boundary rules without requiring segmented networking or infrastructure concerns.

Validation ensures:

- plugins do not embed reference snapshots as nested objects in contracts
- plugins use pointers and shared DTO contracts for referencing persisted objects
- plugins do not import internal models from other plugins (or the core app)
- shared contract types are valid and versioned

Sandbox prevents accidental tight coupling between plugins and the core app.

---

# Schema Diff, Migrations, and Sweeps

Sandbox helps engineers evolve domain objects without hand-authoring database migrations.

Sandbox supports:
- Computing schema diffs and generating migration plans
- Flagging potentially destructive changes and requiring explicit confirmation
- Prompted "sweeps" to quantify/summarize potential data loss

Canonical policy: `BVER_MIGRATION_POLICY.md`

---

# Trial Execution

Sandbox supports:

    bver sandbox trial <bundle>

Trial execution performs:

- build bundle
- generate manifest
- validate contracts
- validate pointers
- start runtime
- test pointer resolution
- test operations

Produces validation report.

---

# Settings

Sandbox uses:

    bver.settings.json

Defines:

- bundle registry (core app + plugins)
- adapter endpoints
- cluster configuration

Sandbox uses settings to validate pointer resolution.

---

# Development Role

Sandbox enables:

- safe development
- contract validation
- pointer resolution testing
- plugin integration testing

Sandbox ensures system correctness.

---

# CLI

CLI provides commands:

    bver sandbox trial
    bver sandbox validate
    bver sandbox resolve

CLI integrates runtime and sandbox.

---

# Summary

Sandbox validates domain correctness.

Sandbox enforces pointer safety.

Sandbox ensures system integrity.
