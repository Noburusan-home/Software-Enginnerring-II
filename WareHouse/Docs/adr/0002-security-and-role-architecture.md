# ADR-0002: Layered security and employee/manager roles

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

[ADR-0001](./0001-client-server-warehouse-architecture.md) puts the API in charge of warehouse data. Security still needs a model for **who** is calling, **what** they may do, and **how** errors are returned.

Threats and forces:

- Unauthenticated access to operational APIs.
- Employees performing manager-only actions (inventory adjust, final damage disposition, minimum stock).
- Client sending a fake `createdBy` or role in the body.
- Frontend-only checks treated as security.

This is **not** Todo’s “user A must never see user B’s lists.” Warehouse records are shared. Isolation is **authentication + role**, not per-user row ownership of every table.

## Decision

Adopt **layered security**: the **API enforces**; the **UI is UX only**.

### Trust boundaries

```text
Browser (untrusted)     →  Express API (enforcement)  →  MySQL
router / hide buttons      authenticate, validate,       FKs, actor columns
                           role check, ignore body ids
```

### Authentication

Protected `/warehouse/*` routes require `Authorization: Bearer <token>`. Missing or bad token → **`401`**. Login/register UI is out of scope of Features 1–10 but identity is assumed so actor fields can be set.

### Authorization (roles)

| Role | May |
|------|-----|
| **Employee** (`worker`) | Create/update floor records allowed by feature FRs |
| **Manager** (`manager`) | Employee actions plus inventory adjust, damage/return disposition, equipment maintenance status, stock minima |

Insufficient role → **`403`**. Missing resource → **`404`** (do not use 403 to mean “not found”).

### Create attribution

`createdBy` / `reportedBy` / `verifiedBy` come from `req.user.id`. Ignore those fields if the client sends them.

## Consequences

### Positive

- Shared warehouse data still has an audit trail of who acted.
- Manager-only rules are testable in Gherkin (`403`).
- Matches course HTTP conventions except role denial uses `403` (correct for “you are logged in but not allowed”).

### Negative / tradeoffs

- Full auth feature is assumed, not specified as Feature 0 yet.
- `403` on role differs from Todo’s “always 404 for other users’ rows” — documented here on purpose.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **404 for every denied action** | Confuses “no such shipment” with “you are not a manager” |
| **UI-only hide manager buttons** | Employees could still call the API |
| **No roles** | Inventory adjust and dispose would be available to every floor user |

## Related artifacts

- [design.md](../../features/design.md) Data Ownership
- Features 2, 5, 6, 7 (manager-only FRs)
- NFR **Security** row
