# C4 Level 1 — System context

Warehouse employees and managers share operational records (orders, inventory, locations) through one web application. There are no external SaaS dependencies in this draft.

```mermaid
C4Context
title System Context — Warehouse Operations

UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")

Person(employee, "Warehouse employee", "Records floor work: orders, receiving, picks, counts, damage.")
Person(manager, "Warehouse manager", "Reviews exceptions, dispositions, equipment, and stock policy.")
System(wh, "Warehouse Operations", "Web application for shared warehouse records.")
SystemDb_Ext(mysql, "MySQL", "Application system of record.")

Rel(employee, wh, "Uses", "HTTPS")
Rel(manager, wh, "Uses", "HTTPS")
Rel(wh, mysql, "Reads and writes", "Sequelize")
```

## Notes

- Operational data is **shared** for this warehouse (not private per user).
- The Vue SPA and Express API sit inside the warehouse system; the [container diagram](./c4-container.md) expands that boundary.

**Related:** [ADR-0001](../adr/0001-client-server-warehouse-architecture.md) · [ADR-0003](../adr/0003-mysql-relational-database.md)
