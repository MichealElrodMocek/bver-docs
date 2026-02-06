# BVER Base Type System (Capabilities-First)

Version: 0.1.0  
Status: Draft  
Audience: Runtime/Core developers, plugin authors, platform builders

---

## Core mindset

**Base types never define attributes. They define capabilities.**

BVER does not try to model *what an object has* (fields, columns, properties).  
BVER models *what an object can do* (capabilities) and *how it interacts with the world*.

This is the foundational separation:

- **Base Types**: a stable vocabulary of “things that exist” (identity + lifecycle)
- **Capabilities**: typed operations that can be attached to base types
- **Domain Models**: application-owned attribute schemas (“my Person has name + birthday”)

BVER’s runtime stays out of attribute debates and instead provides a clean capability composition system.

---

## Motivating example (Person)

In a typical application, a `Person` might have:

- name
- middle name
- last name
- gender
- birthday
- address
- phone numbers
- …plus 1,000 other fields depending on the domain

That approach breaks down at ecosystem scale: every team defines “Person” differently.

**BVER’s rule:** we do *not* define `Person` by attributes.  
We define `Person` by capabilities—what can be done with a person object in a given platform.

Examples of *capabilities* that may apply to `Person`:

- `format_display_name(person) -> str`
- `age_on(person, on_date: date) -> int`
- `validate_identity(person) -> ValidationResult`
- `notify(person, message: Message) -> NotificationReceipt`

The platform builder decides what their person “has” (name fields, birthdate field, etc.).  
Capabilities are implemented in plugins and can rely on those domain-defined attributes **through typed access contracts**, not by hard-coding a universal schema.

---

## Design goals

1. **No universal schema**
   - Base types do not ship with “common” fields beyond identity/lifecycle.
2. **Capability composition**
   - Capabilities are registered independently and can be installed/uninstalled.
3. **Local-first dev ergonomics**
   - Building on BVER should feel like writing normal Python with strong typing.
4. **Module isolation**
   - Domain packages define base types without importing the world.
5. **Typed discoverability**
   - Capabilities are discoverable and callable in a type-safe way.
6. **Plugin-friendly**
   - Plugin authors add capabilities by shipping a Python module that registers them.

---

## Key concepts

### 1) Base Type

A **base type** is a minimal, referenceable entity category (identity + lifecycle), e.g.:

- `Person`
- `Site`
- `Asset`
- `Document`
- `Organization`

A base type **does not** define business attributes.

Hard rule (compiler enforced):
- Base types cannot define intrinsic data fields.
- If a base type defines attributes, the compiler should fail.
- If no base type exists for a domain object, you must define one first and register it on the platform.

> The base type is the “socket.” Capabilities are the “plugs.”

---

## Reference Definition Workflow (Root Objects + Persistence)

BVER treats references as the core unit of identity. A reference is defined in Python by inheriting from a base type (or from the root reference type when no base type fits).

Two storage modes exist:

### Registry
A **registry** is a reference collection that is created entirely in code.
- Intended for fixed or seeded objects (standards, constants, system vocabularies).
- Instances are defined in code, not user-edited.
- No database persistence is required.

### Repository
A **repository** is a reference collection that is persisted in storage (Postgres) and is user-editable.
- Instances are created, edited, and versioned at runtime.
- Backed by the storage adapter and migration system.

Core rule: references are never embedded directly on other objects. Use BVER reference pointers instead.

Ergonomics note:
The Python API can still accept reference objects directly for usability. The runtime treats these as handles and
stores pointers behind the scenes, resolving and hydrating on access. This preserves strict reference semantics
without forcing pointer plumbing in user code.

---

## Platform Domain API (Conceptual)

The platform definition should be explicit and simple:

```python
platform = PlatformDomain()

platform.add_registry(StandardVoltage)
platform.add_registry(MaterialCatalog)

platform.add_repository(Site)
platform.add_repository(Asset)
platform.add_repository(Document)
```

This communicates the intent clearly:
- registries are code-defined and fixed
- repositories are persisted and user-editable

---

## Inheritance and Kinds (Design Principle)

Avoid deep inheritance chains. References should inherit once from a base type (or the root reference).
Relationships and specialization are modeled using:
- kinding taxonomies
- network structure graphs
- capability attachments

Inheritance is for identity and lifecycle only. Domain complexity should live in kinds and relationships.

### 2) Domain Object (Application-owned attributes)

A **domain object** is an application-level schema that extends a base type with attributes.  
This is where “name”, “birthday”, “rating”, “manufacturer_id”, etc. live.

The runtime never forces a global attribute set.

### 3) Capability

A **capability** is a typed callable bound to one or more base types.

- It has a **function signature**
- It may have metadata: name, version, permissions, tags
- It is discoverable through a **registry**
- It is invoked through the runtime in a consistent way

### 4) Capability Registry

The registry maps:

- `(base_type, capability_name)` → callable + metadata

It supports:

- discovery (what can I do with a Person?)
- conflict resolution (two plugins provide the same capability)
- policy checks (permissions, sandbox rules)
- runtime dispatch (local or remote execution)

---

## Packaging model (domain packages)

To avoid importing everything at once:

- A **domain package** defines base types (and optionally kinds) in lightweight modules.
- Plugins can import only the base types they need to declare capabilities for.
- The runtime loads capability modules via entry points or explicit imports.

Example package structure:

```text
bver_domain_people/
  __init__.py
  base_types.py        # defines Person (and maybe Organization)
  contracts.py         # typed access contracts for domain attributes (optional)
  capabilities/        # optional built-in capabilities shipped by the domain package
    display_name.py

bver_plugin_identity/
  __init__.py
  capabilities/
    validate_identity.py
```

---

## Core API sketch (Python)

### Base type definition (minimal)

```python
# bver_domain_people/base_types.py
from __future__ import annotations
from dataclasses import dataclass
from typing import NewType
from uuid import UUID

BaseId = NewType("BaseId", UUID)

@dataclass(frozen=True)
class BaseRef:
    """Reference to a persisted base object (identity only)."""
    type_id: str        # e.g. "bver.core/Person"
    id: BaseId

@dataclass(frozen=True)
class Person:
    """Base type: identity + lifecycle only. No attribute schema."""
    ref: BaseRef
```

**Important:** this `Person` class is *not* a “person model.”  
It’s a typed handle that tells the runtime: “this is a Person-thing.”

### Reference implementation sketch (base type + domain reference)

```python
# bver_runtime/reference.py
from __future__ import annotations
from dataclasses import dataclass, fields
from typing import Any, ClassVar, NewType, Type
from uuid import UUID

BaseId = NewType("BaseId", UUID)

@dataclass(frozen=True)
class BaseRef:
    type_id: str
    id: BaseId

class BaseTypeMeta(type):
    def __new__(mcls, name: str, bases: tuple[type, ...], ns: dict[str, Any]):
        annotations = ns.get("__annotations__", {})
        extra = [k for k in annotations.keys() if k not in {"ref"}]
        if extra:
            raise TypeError(f"Base types cannot define data fields: {', '.join(extra)}")
        return super().__new__(mcls, name, bases, ns)

@dataclass(frozen=True)
class BaseType(metaclass=BaseTypeMeta):
    """Identity-only base type."""
    ref: BaseRef

class Reference:
    """Domain reference with attributes; backed by a base type."""
    base_type: ClassVar[Type[BaseType]]

    def __init_subclass__(cls) -> None:
        if not hasattr(cls, "base_type"):
            raise TypeError("Reference subclasses must define base_type")

    def bver_ref(self) -> BaseRef:
        return getattr(self, "ref")
```

```python
# bver_domain_power/objects.py
from dataclasses import dataclass
from bver_runtime.reference import BaseType, Reference

@dataclass(frozen=True)
class SiteBase(BaseType):
    pass

@dataclass
class Site(Reference):
    base_type = SiteBase
    ref: BaseRef
    name: str
    voltage_level: str
```

### @tracked decorator (automatic diff + version support)

```python
# bver_runtime/tracking.py
from dataclasses import fields
from typing import Any, Callable

def _snapshot(obj: Any) -> dict[str, Any]:
    return {f.name: getattr(obj, f.name) for f in fields(obj) if not f.name.startswith("_")}

def _diff(before: dict[str, Any], after: Any) -> dict[str, dict[str, Any]]:
    changes: dict[str, dict[str, Any]] = {}
    for key, old in before.items():
        new = getattr(after, key)
        if new != old:
            changes[key] = {"from": old, "to": new}
    return changes

def tracked(cls: type) -> type:
    original_post_init = getattr(cls, "__post_init__", None)

    def __post_init__(self):
        self._bver_original = _snapshot(self)
        if original_post_init:
            original_post_init(self)

    def bver_diff(self) -> dict[str, dict[str, Any]]:
        return _diff(getattr(self, "_bver_original", {}), self)

    cls.__post_init__ = __post_init__
    cls.bver_diff = bver_diff
    cls.__bver_tracked__ = True
    return cls
```

```python
# usage
from dataclasses import dataclass
from bver_runtime.tracking import tracked
from bver_domain_power.objects import Site

@tracked
@dataclass
class SiteEditable(Site):
    owner: str | None = None
```

The runtime can use `__bver_tracked__` and `bver_diff()` to produce version deltas automatically.

### Capability decorator

```python
# bver_runtime/capabilities.py
from __future__ import annotations
from dataclasses import dataclass
from typing import Any, Callable, Generic, Optional, Type, TypeVar

TBase = TypeVar("TBase")

@dataclass(frozen=True)
class CapabilityMeta:
    name: str
    version: str = "0.0.0"
    tags: tuple[str, ...] = ()
    permissions: tuple[str, ...] = ()

class CapabilityRegistry:
    def __init__(self) -> None:
        self._by_base: dict[type, dict[str, tuple[Callable[..., Any], CapabilityMeta]]] = {}

    def register(self, base: type, fn: Callable[..., Any], meta: CapabilityMeta) -> None:
        self._by_base.setdefault(base, {})
        self._by_base[base][meta.name] = (fn, meta)

    def get(self, base: type, name: str) -> tuple[Callable[..., Any], CapabilityMeta]:
        return self._by_base[base][name]

REGISTRY = CapabilityRegistry()

def capability(base: type[TBase], *, name: Optional[str] = None, version: str = "0.0.0",
               tags: tuple[str, ...] = (), permissions: tuple[str, ...] = ()):
    """Decorator: attach a capability to a base type."""
    def _decorator(fn: Callable[..., Any]) -> Callable[..., Any]:
        meta = CapabilityMeta(
            name=name or fn.__name__,
            version=version,
            tags=tags,
            permissions=permissions,
        )
        REGISTRY.register(base, fn, meta)
        return fn
    return _decorator
```

### Defining a capability in a plugin

```python
# bver_plugin_identity/capabilities/validate_identity.py
from __future__ import annotations
from dataclasses import dataclass
from bver_domain_people.base_types import Person
from bver_runtime.capabilities import capability

@dataclass(frozen=True)
class ValidationResult:
    ok: bool
    reasons: tuple[str, ...] = ()

@capability(Person, name="validate_identity", version="1.0.0", permissions=("person:read",))
def validate_identity(person: Person) -> ValidationResult:
    # Implementation chooses how to access domain attributes:
    # - via runtime-provided attribute access
    # - via a typed domain contract interface
    # - via adapters (directory services, HR systems, etc.)
    return ValidationResult(ok=True)
```

### Invoking a capability (runtime dispatch)

```python
# user code
from bver_domain_people.base_types import Person
from bver_runtime.capabilities import REGISTRY

fn, meta = REGISTRY.get(Person, "validate_identity")
result = fn(person)
```

In the real runtime, invocation should go through a single API so it can:

- enforce permissions
- attach correlation IDs / tracing
- route execution (local vs hosted)
- record audit events and artifacts

---

## How domain attributes are accessed (without breaking the rule)

Capabilities often need data. But the runtime cannot hard-code schemas.

The pattern is:

1. The **platform** owns attribute schemas (domain objects, kinds, storage mapping).
2. The **runtime** provides a **typed access interface** (contracts) to query required data.
3. A capability declares what it needs through typed inputs or a contract interface, not through “Person has fields X/Y/Z”.

Two workable approaches (can coexist):

### A) Explicit inputs (pure function style)
The capability takes the required data as parameters:

```python
@capability(Person, name="format_display_name")
def format_display_name(first: str, last: str, *, style: str = "first_last") -> str:
    ...
```

Pros: very explicit, easy to test.  
Cons: caller must extract attributes first.

### B) Typed contract interface (runtime-provided)
The capability receives a `Person` handle and asks the runtime for what it needs via a contract:

```python
from typing import Protocol

class PersonIdentityContract(Protocol):
    def first_name(self, person: Person) -> str: ...
    def last_name(self, person: Person) -> str: ...

@capability(Person, name="format_display_name")
def format_display_name(person: Person, contract: PersonIdentityContract) -> str:
    return f"{contract.first_name(person)} {contract.last_name(person)}"
```

Pros: plugin stays decoupled from storage and schema layout.  
Cons: requires a contract binding step (the platform provides the implementation).

**Rule stays intact:** base type still defines no attributes.

---

## Collision and versioning policy (capabilities)

When two plugins register the same capability name for the same base type:

- The runtime must define a deterministic rule:
  - prefer higher semantic version
  - or prefer platform-pinned provider
  - or allow multiple providers with explicit selection

Recommended baseline:

- Registry key: `(base_type, capability_name, provider_id)`
- Platform “wires” defaults (a pinned provider for each capability)
- “Unwired” capabilities are still discoverable but not auto-selected

---

## Discovery UX (developer ergonomics)

Goal: it should feel like normal Python scripting.

Two layers:

1. **Static** (IDE typing):
   - base types are actual Python classes
   - capability signatures are normal functions
2. **Dynamic** (runtime discovery):
   - list available capabilities for a base type
   - inspect signature + metadata
   - call via a single dispatcher

Potential helper API:

```python
runtime.can(Person).list()
runtime.can(Person).call("validate_identity", person=person)
```

This is sugar over the registry + dispatcher.

---

## Load model (how capabilities are registered)

Capability registration occurs when the module is imported.

Recommended loading options:

- **Explicit imports** in the platform package (simple and predictable)
- **Python entry points** (plugin auto-discovery)
- **Compiler-generated import map** (runtime loads exact plugin modules needed)

In all cases, the runtime should be able to produce an inventory:

- capability name
- provider package + version
- supported base types
- permissions/tags
- function signature

---

## Non-goals (for this layer)

This document intentionally does **not** define:

- persistence schemas
- migrations
- object/kind taxonomy rules
- permission engine details
- remote execution (Forge) mechanics

Those are separate layers. The base type system only guarantees:
**base types are stable handles + capabilities are typed operations.**

---

## Next decisions (to lock v0)

1. **Canonical base type list** (small and stable)
2. **Registry keying and collision policy**
3. **Contract binding mechanism** (Protocol + runtime injection vs explicit inputs)
4. **Runtime invocation API** (audit/trace hooks)
5. **Plugin discovery mechanism** (explicit imports vs entry points)
