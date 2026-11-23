# C4 Level 3 — Frontend components

Vue SPA along the user interaction and API path.

```mermaid
C4Component
title Component Diagram — Warehouse Web SPA

Container_Boundary(spa, "Web SPA") {
  Component(router, "Router", "vue-router", "Named routes from Screen Requirements; UX-only auth redirects.")
  Component(views, "Views", "views/*.vue", "Orders, shipments, receiving, picking, counts, and other floor screens.")
  Component(ui, "UI Components", "components/*.vue", "Tables, dialogs, status chips, and forms.")
  Component(services, "API Services", "*Services.js", "axios modules for /warehouse resources.")
  Component(config, "Client Config", "config + plugins", "Token storage, helpers, and Vuetify.")
}

Container_Ext(api, "API", "Express /warehouse")

Rel(router, views, "Renders")
Rel(views, ui, "Uses")
Rel(views, services, "Calls")
Rel(services, config, "Token")
Rel(services, api, "HTTP JSON", "Bearer token")

UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

## Notes

- Router guards and hidden manager buttons are UX only; the API enforces roles.
- Optional mockups live in [../Ui](../Ui/README.md).

**Related:** [ADR-0004](../adr/0004-vue-frontend-framework.md) · [ADR-0005](../adr/0005-vuetify-ui-library.md)
