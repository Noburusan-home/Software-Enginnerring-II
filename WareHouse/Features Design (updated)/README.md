# Warehouse Feature Specifications

Spec-driven source of truth for the **warehouse operations** application.  
No application code may be written unless it maps to a requirement in one of these files.

**Methodology:** OC CS Speckit (same section style as the course POS / Todo examples).  
**Architecture:** [architecture.md](./architecture.md)  
**Cross-cutting design:** [design.md](./design.md)  
**ADRs:** [../Docs/adr/README.md](../Docs/adr/README.md)  
**NFRs:** [../Docs/nfr/quality-attributes.md](../Docs/nfr/quality-attributes.md)

**Sprints** live in the agile tool — they are not part of these specs.

## Feature catalog

| ID | File | Branch | Depends on |
|----|------|--------|------------|
| 1 | [feature-1-supplier-orders-and-shipments.md](./feature-1-supplier-orders-and-shipments.md) | `feature/1-supplier-orders-and-shipments` | — |
| 2 | [feature-2-damaged-product-handling.md](./feature-2-damaged-product-handling.md) | `feature/2-damaged-product-handling` | [1](./feature-1-supplier-orders-and-shipments.md) |
| 3 | [feature-3-equipment-device-management.md](./feature-3-equipment-device-management.md) | `feature/3-equipment-device-management` | — |
| 4 | [feature-4-fast-picking-and-packing.md](./feature-4-fast-picking-and-packing.md) | `feature/4-fast-picking-and-packing` | [10](./feature-10-warehouse-location-management.md) |
| 5 | [feature-5-inventory-accuracy.md](./feature-5-inventory-accuracy.md) | `feature/5-inventory-accuracy` | [10](./feature-10-warehouse-location-management.md) |
| 6 | [feature-6-low-stock-alerts.md](./feature-6-low-stock-alerts.md) | `feature/6-low-stock-alerts` | [5](./feature-5-inventory-accuracy.md) |
| 7 | [feature-7-product-returns.md](./feature-7-product-returns.md) | `feature/7-product-returns` | — |
| 8 | [feature-8-receiving-inspection.md](./feature-8-receiving-inspection.md) | `feature/8-receiving-inspection` | [1](./feature-1-supplier-orders-and-shipments.md) |
| 9 | [feature-9-expiration-shelf-life.md](./feature-9-expiration-shelf-life.md) | `feature/9-expiration-shelf-life` | — |
| 10 | [feature-10-warehouse-location-management.md](./feature-10-warehouse-location-management.md) | `feature/10-warehouse-location-management` | — |

**Status:** all features are **Draft** until stories, FRs, initial data model, Gherkin, and design sections are reviewed.

**Branch roles:** `Warehouse-Design` holds these specs · `feature/N-*` is the implementation branch for each feature (branch from `dev`).
