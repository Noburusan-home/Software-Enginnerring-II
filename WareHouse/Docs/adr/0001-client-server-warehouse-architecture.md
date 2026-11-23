# ADR-0001: Client–server architecture for shared warehouse data

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Warehouse employees and managers must record orders, shipments, inventory, locations, equipment, returns, receiving, damage, expiration, and picking. Several people work the **same** warehouse records at once. The browser cannot be the source of truth.

We needed to decide:

1. Whether the SPA holds authoritative state or only talks to a server.
2. How callers are identified on each request.
3. Whether data is private per user (Todo-style) or shared for the warehouse.

## Decision

Adopt a **classic client–server split** with a **REST API** and **server-enforced identity**. Operational rows are **shared warehouse data**, not private per-user documents.

| Layer | Choice |
|-------|--------|
| **Client** | Vue 3 SPA (Vite), Vuetify, axios |
| **Server** | Node.js + Express + Sequelize |
| **Database** | MySQL — one warehouse database |
| **Transport** | JSON over HTTP; API base path `/warehouse/` |
| **Auth** | Signed-in employee; `Authorization: Bearer <token>` |
| **Identity** | `req.user.id` and `req.user.role` (`worker` or `manager`) |

```text
Browser (Vue SPA)                 Express API                    MySQL
─────────────────                 ───────────                    ─────
route guards / role UI  ─Bearer─► authenticate + role checks  ──► operational tables
axios services                    createdBy from req.user.id
```

**Invariants**

1. The **server** is the source of truth for warehouse records.
2. Every authenticated request resolves to exactly one employee (`req.user.id`) and a role.
3. Employees of this warehouse **share** operational data (orders, inventory, locations).
4. The client never sends a trusted `createdBy` — the server assigns the acting employee.
5. Unknown ids → **`404`**. Missing auth → **`401`**. Wrong role → **`403`**.

## Consequences

### Positive

- Matches OC CS Speckit classroom stack so specs, APIs, and tests line up.
- Shared records match real warehouse work (many people touch the same shipment).
- Actor fields (`createdBy`, `reportedBy`) still support audit without private tenancy.

### Negative / tradeoffs

- Needs a running MySQL and API for a full demo (not a static page).
- Multi-warehouse / multi-company tenancy is not modeled (one warehouse database).
- Concurrent edits are last-write-wins unless a later feature adds locking.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **localStorage-only warehouse** | No shared truth; two employees would not see the same inventory |
| **Per-user private records (Todo isolation)** | Wrong domain: a pick list and a bin location belong to the warehouse, not one user |
| **GraphQL / tRPC** | Heavier than the course REST + flat JSON pattern |

## Related artifacts

- [ADR-0002](./0002-security-and-role-architecture.md), [ADR-0003](./0003-mysql-relational-database.md), [ADR-0007](./0007-rest-json-api.md)
- [architecture.md](../../features/architecture.md), [design.md](../../features/design.md)
- NFRs: [quality-attributes.md](../nfr/quality-attributes.md)
