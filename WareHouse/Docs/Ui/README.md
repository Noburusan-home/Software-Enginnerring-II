# UI

Optional screen mockups for warehouse features. **Screen Requirements** in each feature spec remain the source of truth for labels, routes, and empty/error copy.

## Dashboard mockups (linked to features)

| Mockup | Shows | Features |
|--------|--------|----------|
| [dashboard/dashboard-operations.jpg](./dashboard/dashboard-operations.jpg) | Orders, shipments, receiving, inventory trend, low-stock table, quick actions | [1](./feature-1-orders/), [5](./feature-5-inventory/), [6](./feature-6-low-stock/), [8](./feature-8-receiving/) |
| [dashboard/dashboard-orders-and-items.jpg](./dashboard/dashboard-orders-and-items.jpg) | Recent orders, top items, recent alerts (low stock + P.O.) | [1](./feature-1-orders/), [5](./feature-5-inventory/), [6](./feature-6-low-stock/) |
| [dashboard/dashboard-inventory-and-shipments.jpg](./dashboard/dashboard-inventory-and-shipments.jpg) | Inventory mix, low-stock SKUs, pending shipments | [1](./feature-1-orders/), [5](./feature-5-inventory/), [6](./feature-6-low-stock/) |
| [dashboard/dashboard-oc-warehouse.jpg](./dashboard/dashboard-oc-warehouse.jpg) | OC Warehouse home: low stock, activity, receive/create/record actions | [1](./feature-1-orders/), [5](./feature-5-inventory/), [6](./feature-6-low-stock/), [8](./feature-8-receiving/) |

The same files are copied into the feature folders below so each feature has its mockups next to it.

```text
Docs/Ui/
  dashboard/                ← all four dashboard images
  feature-1-orders/         ← Feature 1 — Supplier Orders and Shipments
  feature-5-inventory/      ← Feature 5 — Inventory Accuracy
  feature-6-low-stock/      ← Feature 6 — Low-Stock Alerts
  feature-8-receiving/      ← Feature 8 — Receiving and Inspection
  feature-2-damage/ …       ← other features (add mockups later)
```

Feature specs link these images from **Screen Requirements**.

**Related:** [ADR-0004](../adr/0004-vue-frontend-framework.md) · [ADR-0005](../adr/0005-vuetify-ui-library.md)
