# ADR-0009: SKU as the product business key

**Status:** Accepted  
**Date:** 2026-11-23  
**Deciders:** Warehouse operations project (draft)

## Context

Almost every warehouse feature names a product: orders, damage, picking, counts, returns, receiving, expiration, locations. We needed one identifier that floor staff and APIs share.

## Decision

**SKU** (string) is the **business key** for a product across features.

- Line items and reports store `sku` (plus description when the form requires it).
- Optional integer `id` keys exist on *documents* (orders, reports), not as the only way to talk about a product on the floor.
- Lots (Feature 9) are `sku` + `lotCode`.
- Locations (Feature 10) assign a `sku` to a location code.

A full product catalog table may be introduced when implementation starts; until then, SKU strings on documents are enough for the draft specs.

## Consequences

### Positive

- Gherkin can use stable examples (`SKU-9`) across features.
- Picking verification and receiving checks compare SKU strings.

### Negative / tradeoffs

- Duplicate/typo SKUs are possible until a catalog with uniqueness is implemented.
- Barcode hardware integration is out of scope (equipment is tracked as devices, Feature 3).

## Alternatives considered

| Option | Why not |
|--------|---------|
| **Integer productId only** | Poor match to warehouse forms and barcodes |
| **Name-only identity** | Names collide; SKU is the form field on orders/BOL |

## Related artifacts

- Features 1–10 Key Entities / data models
- NFR **Data integrity**
