# C4 Level 4 — Deployment

The **user PC** runs the SPA in a browser. The **web server** hosts static assets, the Node API, and MySQL.

```mermaid
C4Deployment
title Deployment Diagram — Warehouse Operations

Deployment_Node(userPc, "User PC", "Employee / manager / developer computer") {
  Container(spa, "Web SPA", "Browser + Vue", "Loaded from the web server; runs on the user PC.")
}

Deployment_Node(webServer, "Web Server", "Classroom or local deploy host") {
  Container(staticAssets, "Static Assets", "Apache or Vite preview", "Built Vue dist.")
  Container(api, "API", "Node.js + Express", "REST /warehouse.")
  ContainerDb(db, "Database", "MySQL", "Warehouse operational tables.")
}

Rel(staticAssets, spa, "Serves", "HTTPS")
Rel(spa, api, "JSON", "Bearer token")
Rel(api, db, "SQL", "TCP")

UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="2")
```

## Typical ports

| Location | Service | Port |
|----------|---------|------|
| User PC | Browser | — |
| Web Server | SPA (Vite dev or Apache) | `8082` / `80` |
| Web Server | Backend API | `3200` (or reverse-proxied) |
| Web Server | MySQL | `3306` |

**Related:** [ADR-0001](../adr/0001-client-server-warehouse-architecture.md)
