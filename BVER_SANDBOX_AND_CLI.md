
# BVER Sandbox and CLI
## Domain validation, execution, and integration environment

BVER Sandbox is the development and validation environment for BVER systems.

Sandbox validates domain correctness, pointer resolution, service contracts, and runtime compatibility.

Sandbox enables building complex systems without infrastructure expertise.

---

# Core Purpose

Sandbox provides:

- Service build validation
- Manifest generation
- Pointer contract validation
- Cross-service contract validation
- Runtime trial execution
- Contract correctness enforcement

Sandbox ensures domain correctness before deployment.

---

# Pointer Validation

Sandbox validates ReferencePointers.

Validation includes:

- pointer syntax correctness
- referenced service existence
- referenced repo existence
- attribute path validity
- version qualifier validity

Sandbox prevents invalid cross-service references.

---

# Service Boundary Validation

Sandbox enforces cross-service boundary rules.

Validation ensures:

- services do not embed foreign reference objects
- services use pointers for foreign references
- operations only use allowed cross-service contract types
- shared contract types are valid

Sandbox prevents illegal cross-service coupling.

---

# Trial Execution

Sandbox supports:

    bver sandbox trial <service>

Trial execution performs:

- build service
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

- service registry
- adapter endpoints
- cluster configuration

Sandbox uses settings to validate pointer resolution.

---

# Development Role

Sandbox enables:

- safe development
- contract validation
- pointer resolution testing
- service integration testing

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
