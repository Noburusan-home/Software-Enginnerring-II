# Warehouse Cross-Cutting Design (Draft)

**Status:** Draft  
**Date:** 2026-11-23  
**Related:** [architecture.md](./architecture.md)

Use this file for rules that apply to **every** warehouse feature. Feature files add only their delta.

---

## Data ownership (warehouse-wide)

Warehouse records are **shared operational data**, not private user documents.

| Rule | Requirement |
|------|-------------|
| **Read scope** | Signed-in employees and managers MAY read operational records for this warehouse |
| **Write scope** | Employees MAY create/update records allowed by the feature FRs; manager-only actions MUST be rejected with `403` |
| **Create scope** | Server sets `createdBy` (or equivalent) from `req.user.id` — ignore body `createdBy` |
| **Unknown id** | `404` `{ "message": "Not found." }` |
| **Unauthenticated** | `401` |
| **UI scope** | Screens show only data returned by the API |

Per-feature ownership tables repeat this pattern and add any extra restriction (for example, “available inventory MUST NOT include damaged units”).

---

## Error and validation

| Situation | HTTP | Notes |
|-----------|------|-------|
| Missing/invalid required field | `400` | Message names the field |
| Unauthenticated | `401` | |
| Authenticated but role not allowed | `403` | Manager-only actions |
| Missing record | `404` | |
| Duplicate business key (e.g. P.O. number) | `400` | Warn; do not silently overwrite |

Quantities that represent physical counts MUST be integers **greater than zero** unless a feature explicitly allows zero (e.g. counted quantity during a cycle count).

---

## Product identity

- **SKU** is the business key for a product.
- Line items store SKU, description, quantity (or cases), price, and total when the source form requires them.
- Location codes are assigned in Feature 10 and reused by picking and inventory accuracy.

---

## Screen shell

When the SPA exists:

*   App bar on operational routes (not on login).
*   Primary labeled actions use a clear CTA (course `oc-cta` class when the Vue kit is used).
*   Empty, loading, and error states are required on list views.
*   Icon-only actions need `aria-label`s.

Feature Screen Requirements name the route and quoted copy for that slice only.
