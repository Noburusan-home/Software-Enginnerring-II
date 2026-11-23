# ADR-0003: MySQL relational database

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Orders, shipments, locations, lots, and inventory have relationships (P.O. number, SKU, location code, FKs). We need durable storage that matches OC CS Speckit labs (often XAMPP / MySQL).

## Decision

Use **MySQL** as the system of record, accessed through **Sequelize** ([ADR-0008](./0008-sequelize-orm.md)).

- One database for this warehouse application.
- Integer auto-increment primary keys in feature data models.
- Foreign keys for document lines (`orderId`, `shipmentId`, `pickListId`, …).
- **SKU** is the business key for products ([ADR-0009](./0009-sku-product-identity.md)).

Schema for each slice is defined in that feature’s **Initial Data Model**, not in this ADR.

## Consequences

### Positive

- Joins and constraints match warehouse documents (order ↔ lines ↔ SKU).
- Same engine as the course Todo kit.

### Negative / tradeoffs

- Requires a local MySQL instance.
- No multi-region replication in this draft.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **SQLite-only** | Weaker match to course hosting and concurrent lab use |
| **Document DB (Mongo)** | Line items and FKs are relational; Gherkin expects stable tables |
| **Spreadsheets** | Not an API source of truth |

## Related artifacts

- Feature **Initial Data Model** sections
- NFR **Data integrity**
