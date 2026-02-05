
# BVER Adapters
## Infrastructure integration layer

Adapters connect BVER runtime to external systems.

Adapters provide infrastructure capabilities while keeping domain logic independent.

Adapters translate runtime operations into infrastructure actions.

---

# Adapter Philosophy

Domain logic operates on:

- references
- pointers
- operations

Adapters handle:

- storage
- messaging
- indexing
- identity
- external communication

Domain logic remains infrastructure-independent.

---

# Package Model

Each adapter is a separate installable package.

Examples:

    bver-adapter-postgres
    bver-adapter-rabbitmq
    bver-adapter-opensearch
    bver-adapter-keycloak

Users install only needed adapters.

Adapters are independently versioned.

---

# Adapter Types

Storage adapters:

- postgres
- sqlite

Messaging adapters:

- rabbitmq
- kafka

Search adapters:

- opensearch

Identity adapters:

- keycloak

Communication adapters:

- smtp
- sms

---

# Pointer Resolution via Adapters

Adapters enable cross-service pointer resolution.

Adapters allow runtime to fetch references from external services.

Adapters allow distributed system operation.

---

# Open Ecosystem

Adapters are open source.

Anyone can build adapters.

Adapters follow runtime contract interfaces.

---

# Summary

Adapters connect runtime to infrastructure.

Adapters preserve domain purity.

Adapters enable distributed systems.
