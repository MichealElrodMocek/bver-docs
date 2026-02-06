BVER System Layout (Recommended)

Version: 0.1.0
Status: Draft (MVP framing)
Audience: Plugin authors and platform engineers

This document proposes a standard "shape" for a BVER system when the deployment target is on-prem via BVER Sandbox.

Goals
- Keep the domain vocabulary and identity in one place (core app repo)
- Encourage plugin development by making plugins small, replaceable, and contract-driven
- Make "what runs where" obvious for local clusters (operators) and plugin authors (developers)

Non-goals (for MVP)
- Cloud control plane, marketplaces, or hosted execution
- Multi-tenant governance and billing

1. Repository Model

1.1 Core Application Repo (one primary repo)
The core repo owns:
- The domain vocabulary (base objects + kinds)
- Identity rules (IDs, versioning, migrations)
- Automations (triggers, policies, schedules)
- Orchestrators (multi-step workflows that call plugin capabilities)

Clarification: base objects vs value structures
- Base objects are referenceable by default (identity + lifecycle; can be pointed to, versioned, audited).
- Value structures are not referenceable by default (pure data like `EmailAddress`, `Money`, `GeoPoint`), and typically live inside base objects or appear only in request/response payloads.

Suggested name
- bver-app-<product> (e.g., bver-app-substation)

Suggested top-level tree
- app/
- app/domain/ (base objects + kinds)
- app/contracts/ (DTOs, schemas, compatibility notes)
- app/orch/ (orchestrators)
- app/automations/ (triggers, schedules, policies)
- app/config/ (runtime config, environment defaults)
- sandbox/ (local cluster topology, dev helpers)
- docs/ (system-specific docs; link back to platform docs)

Guideline: the core repo should not "embed" vendor dependencies that belong in adapters. Use plugins for that.

1.2 Plugin Repos (many, capability-driven)
Plugins are isolated by the author (you) based on how you want teams to depend on them.

Two grouping rules that work well together:
- Group by functional directive when the capability is cohesive (a simulator, a solver suite, a workflow pack)
- Group adapters by provider (not by type) when integration concerns dominate (auth, SDKs, rate limits, file formats)

Suggested naming conventions
- bver-plugin-sim-<name> (single simulator)
- bver-plugin-solve-<suite> (suite of generators/solvers)
- bver-plugin-orch-<pack> (orchestrator pack)
- bver-plugin-adapter-<provider> (provider-focused integration plugin)
- bver-plugin-tx-<provider|channel> (transmitters grouped by endpoint/provider)
- bver-plugin-rx-<provider|channel> (retrievers grouped by endpoint/provider)

Example plugin groupings
- bver-plugin-adapter-msal
  - MSAL auth + token vending
  - Graph API normalization
  - Any "Microsoft integration glue" that would otherwise be scattered

- bver-plugin-solve-sitegen
  - Generators that produce a Site or Site-related derived objects
  - Shared heuristics/config objects used across generators

1.3 Monorepo Option (when speed matters)
If you want to move fast early, you can keep core + plugins in one repo and still preserve boundaries:
- apps/core/ (the core application)
- plugins/<plugin-name>/ (independent plugin packages)

Rule: plugins must still be buildable/testable in isolation (no deep imports across plugin directories).

2. Runtime Assembly (On-Prem)

2.1 Local cluster composition
The on-prem Sandbox should be able to run:
- Core app services (domain ownership + orchestrators + automations)
- Plugin services (capabilities)
- Shared infra dependencies (DB, queues, object stores) as needed

MVP expectations
- Local plugin install from filesystem paths
- Explicit version pinning (lockfile) so environments reproduce
- Uniform execution envelope: logs, artifacts, error types, correlation IDs

2.2 Contract boundaries
To keep plugins replaceable:
- Plugins accept/produce contract objects (DTOs) rather than internal ORM models
- Plugins do not own domain identity; they operate on inputs and return outputs
- Orchestrators live in core (or an orchestrator-pack plugin) and compose capabilities explicitly

3. Plugin Shape (Minimal)

Every plugin should ship:
- A manifest (capabilities, inputs/outputs, permissions, runtime compatibility)
- Contracts (schemas/types) for requests and responses
- Tests (golden files for adapters; determinism checks for simulators/solvers)
- Example invocation (CLI snippet or minimal orchestrator)

4. Where Things Go (Rule of Thumb)

Put it in core when:
- It defines what something is (domain semantics, identity, migrations)
- It is an automation policy you want to enforce for your platform
- It coordinates multiple capabilities with gating and audit expectations

Put it in a plugin when:
- It is optional or replaceable compute (simulation, solver, perception stage)
- It is provider-specific integration (auth, API/SDK glue, file format normalization)
- It pushes or pulls data across the platform boundary (transmitters/retrievers)
