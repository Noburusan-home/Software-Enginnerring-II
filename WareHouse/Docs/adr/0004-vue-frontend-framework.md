# ADR-0004: Vue 3 as the frontend framework

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Warehouse screens (orders, receiving, picking, counts) need a SPA that talks to the REST API. The course Speckit uses Vue 3 + Vite.

## Decision

Use **Vue 3** (Composition API as needed) with **Vite** as the frontend. One view per major operational route named in **Screen Requirements**. The SPA is not the source of truth ([ADR-0001](./0001-client-server-warehouse-architecture.md)).

## Consequences

### Positive

- Same patterns as OC CS Speckit examples (views, router, axios services).
- Screen Requirements map cleanly to named routes.

### Negative / tradeoffs

- Two dev ports (frontend + API).
- Handheld-native apps are out of scope.

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Server-rendered only** | Weaker match to course SPA + API testing |
| **React / Angular** | Extra stack vs the kit the class already uses |

## Related artifacts

- [ADR-0005](./0005-vuetify-ui-library.md)
- Feature **Screen Requirements**
