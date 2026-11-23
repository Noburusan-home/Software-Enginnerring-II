# C4 Level 2 — Containers

Browser SPA talks to a REST API. The API enforces auth and roles. MySQL holds warehouse rows.

```mermaid
C4Container
title Container Diagram — Warehouse Operations

Person(employee, "Warehouse employee / manager", "Uses the app in a browser.")

System_Boundary(wh, "Warehouse Operations") {
  Container(spa, "Web SPA", "Vue 3, Vuetify, axios", "Operational screens; UX-only route and role hiding.")
  Container(api, "API", "Node.js, Express, Sequelize", "REST /warehouse — auth and role enforcement.")
  ContainerDb(db, "Database", "MySQL", "Orders, shipments, inventory, locations, equipment, lots.")
}

Rel(employee, spa, "Uses", "HTTPS")
Rel(spa, api, "JSON", "Bearer token")
Rel(api, db, "SQL", "Sequelize")

UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Notes

- API routes are mounted under `/warehouse/`.
- The API sets actor fields from `req.user.id` and checks `req.user.role`. The browser never supplies a trusted `createdBy`.
- Shared warehouse tenancy: employees can read operational records; manager-only writes return `403`.

**Related:** [ADR-0001](../adr/0001-client-server-warehouse-architecture.md) · [ADR-0002](../adr/0002-security-and-role-architecture.md) · [ADR-0007](../adr/0007-rest-json-api.md)
