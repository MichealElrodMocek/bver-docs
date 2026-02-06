# BVER Core Compatibility & Stability Policy

**Status:** Foundational  
**Audience:** Platform developers, plugin authors, operators  
**Scope:** BVER Core framework (runtime, compiler contracts, core registries)

---

## Purpose

BVER is a platform, not a fast-moving application framework.

For BVER to be usable in real organizations—especially with **on‑prem deployments, long‑lived plugins, and independent teams**—its core contracts must be stable over long periods of time.

This document defines what *“stable”* means in practice and how BVER evolves **without breaking the ecosystem**.

---

## Core Principle

> **BVER Core is immutable by default.**

BVER Core evolves through:
1. **Bug fixes** (behavior corrections without contract changes)
2. **Additive extensions** (new optional fields, new capabilities, new content)
3. **New Runtime API Levels** (opt‑in, side‑by‑side compatibility)

Silent breaking changes are not permitted.

---

## What Counts as “BVER Core”

The following are considered **core contracts** and are covered by this stability guarantee:

### 1. Identity & Reference Model
- Global object identifiers
- Pointer/reference semantics
- Reference resolution guarantees

### 2. Base Object Rules
- Base objects vs value structures
- Referenceability guarantees
- Lifecycle and versioning expectations

### 3. Capability Invocation Contract
- Capability discovery
- Invocation envelope
- Job execution lifecycle
- Standard error and result models

### 4. Plugin Manifest Schema
- Required manifest fields
- Semantic meaning of fields
- Compatibility declarations

### 5. Runtime API Surface
- Runtime service interfaces
- Capability registration and execution APIs
- Adapter hosting rules

### 6. Schema & DTO Governance
- Schema identifiers
- Versioning rules
- Compatibility checks and enforcement

### 7. Event Envelope (EAGER)
- Event metadata fields
- Schema reference rules
- Change notification semantics

### 8. Migration Policy Interface
- Schema diff representation
- Migration plan structure
- Destructive‑change signaling

Changes to the *meaning* of any of the above are considered breaking.

---

## Runtime API Levels

BVER introduces breaking changes **only** by defining a new **Runtime API Level**.

- Runtime API Levels are versioned (e.g. `API_LEVEL_1`, `API_LEVEL_2`)
- A plugin declares:
  - `runtime_api_min`
  - `runtime_api_max`
- Multiple API levels may be supported by the same runtime concurrently

**Key rule:**  
Older API levels remain supported for extended periods to protect on‑prem and long‑lived deployments.

---

## Allowed Changes (Safe)

The following changes are explicitly allowed and encouraged:

- Adding new optional fields to existing schemas
- Adding new capability types
- Adding new base objects or kinds (without changing existing ones)
- Adding new adapters, plugins, or standard libraries
- Performance improvements and bug fixes
- Adding new Runtime API Levels (opt‑in)

These changes must not alter the interpretation of existing data or contracts.

---

## Disallowed Changes (Breaking)

The following are **not permitted** within an existing Runtime API Level:

- Renaming or removing fields
- Changing field meaning or default behavior
- Making optional fields required
- Changing capability invocation semantics
- Altering reference or identity rules
- Breaking plugin manifests without version isolation

If a change cannot be expressed additively, it requires a **new Runtime API Level**.

---

## Deprecation Policy

Deprecation is a process, not an event.

When something must eventually be replaced:
1. Mark it as deprecated in documentation and metadata
2. Provide a supported alternative
3. Offer tooling to detect deprecated usage
4. Maintain support through a published sunset window

For on‑prem platforms, deprecation timelines may span **years**, not months.

---

## Compatibility Testing (Platform ABI)

BVER maintains a **compatibility test suite** that acts as a platform ABI:

- Golden contract tests
- Event envelope validation
- Plugin compatibility checks
- Migration plan invariants

Any change to BVER Core must pass this suite to be released.

---

## Core vs Standard Library

To preserve long‑term stability:

- **BVER Core** is intentionally small, stable, and slow‑moving
- **BVER Standard Library** evolves rapidly
  - Adapters
  - Capability types
  - UI widgets
  - Reference plugins

Innovation belongs in libraries and plugins—not in core contracts.

---

## On‑Prem & Offline Guarantees

BVER explicitly supports:
- Offline operation
- Long‑term on‑prem installations
- Independent upgrade schedules

No upgrade should force a rewrite of plugins or domain code.

---

## Policy Statement

> BVER Core guarantees long‑term stability of its core contracts.  
> Changes are additive by default.  
> Breaking changes are introduced only through new Runtime API Levels and never invalidate existing platforms without an explicit, supported migration path.

---

*This policy exists to protect users, plugin authors, and operators—and to ensure BVER remains a trustworthy platform over time.*
