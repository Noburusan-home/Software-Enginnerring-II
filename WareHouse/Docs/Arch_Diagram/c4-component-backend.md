# C4 Level 3 — Backend components

Express app along the HTTP path.

```mermaid
C4Component
title Component Diagram — Warehouse API

Container_Boundary(api, "API Application") {
  Component(routes, "Routes", "app/routes/*", "Resource routers under /warehouse.")
  Component(authz, "Authorization", "app/authorization/*", "Session auth and role checks.")
  Component(controllers, "Controllers", "app/controllers/*", "Validation, feature rules, and responses.")
  Component(models, "Models", "app/models/*", "Sequelize entities and associations.")
}

Container_Ext(spa, "Web SPA", "Vue + axios")
ContainerDb_Ext(db, "MySQL", "System of record")

Rel(spa, routes, "HTTP JSON", "Bearer token")
Rel(routes, authz, "Protects")
Rel(routes, controllers, "Delegates")
Rel(controllers, authz, "Role / actor")
Rel(controllers, models, "CRUD")
Rel(authz, models, "Loads session / user")
Rel(models, db, "SQL", "Sequelize")

UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

## Notes

- Missing or invalid session → `401`. Insufficient role → `403`. Unknown id → `404`.
- Controllers ignore client-supplied `createdBy`.

**Related:** [ADR-0002](../adr/0002-security-and-role-architecture.md) · [ADR-0006](../adr/0006-node-express-api.md) · [ADR-0008](../adr/0008-sequelize-orm.md)
