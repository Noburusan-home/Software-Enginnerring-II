# ADR-0007: REST and flat JSON as the API style

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Features need a stable HTTP contract for implementers and Gherkin. Envelope formats (`{ success, data }`) make tests noisier and diverge from the Todo kit.

## Decision

Use **REST** resources under `/warehouse/` with **flat JSON** (no success envelope).

| Result | Status | Body |
|--------|--------|------|
| Create | `201` | Created resource |
| Update | `200` | Updated resource |
| Delete | `200` or `204` | Empty or message |
| Validation | `400` | `{ "message": "…" }` |
| Unauthenticated | `401` | `{ "message": "…" }` |
| Forbidden role | `403` | `{ "message": "…" }` |
| Missing | `404` | `{ "message": "Not found." }` |

Feature files list **only** endpoints that feature adds or changes.

## Consequences

### Positive

- Status codes in Gherkin match implementation.
- Same error shape on every failure.

### Negative / tradeoffs

- No JSON:API / GraphQL batching.
- Pagination is not designed yet (classroom scale).

## Alternatives considered

| Option | Why not |
|--------|---------|
| **`{ success, data }` envelope** | Extra nesting; not the Speckit convention |
| **RPC-style `/doThing`** | Weaker resource mapping for orders/shipments |

## Related artifacts

- Feature **API Requirements**
- [design.md](../../features/design.md)
