# Architecture diagrams (C4)

C4 views for the **warehouse operations** application, as Mermaid. *Why* this shape exists: [ADR-0001](../adr/0001-client-server-warehouse-architecture.md).

| File | C4 level | Shows |
|------|----------|--------|
| [c4-context.md](./c4-context.md) | Context | People and the warehouse system |
| [c4-container.md](./c4-container.md) | Container | SPA, API, MySQL |
| [c4-component-backend.md](./c4-component-backend.md) | Component | API layers |
| [c4-component-frontend.md](./c4-component-frontend.md) | Component | SPA layers |
| [c4-deployment.md](./c4-deployment.md) | Deployment | Browser + web server |

## Preview in Cursor

1. Use a ` ```mermaid ` fence with `C4Context` / `C4Container` / `C4Component` / `C4Deployment` on the first line.
2. Cursor’s default preview often does not render C4 — use a Mermaid/C4-capable preview if the diagram stays blank.

**Related:** [../adr/README.md](../adr/README.md) · [Features Design (updated)](../../Features%20Design%20(updated)/architecture.md)
