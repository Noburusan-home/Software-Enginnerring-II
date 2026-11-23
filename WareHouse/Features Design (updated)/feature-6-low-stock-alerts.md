# Feature: Low-Stock and Reorder Alerts

**Feature ID:** 6  
**Branch pattern:** `feature/6-low-stock-alerts`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for identifying products with low inventory  
**Depends on:** [Feature 5 — Inventory Accuracy and Verification](./feature-5-inventory-accuracy.md)  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-6.1: Set Minimum Stock
**As a** warehouse manager  
**I want to** set the lowest amount of inventory  
**So that** the warehouse recognizes when inventory is low

**Priority:** P1  
**Independent test:** As a manager, set minimum `10` on a SKU and persist it  
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Receive Alert
**As a** warehouse employee  
**I want to** receive a low stock alert  
**So that** I can address the issue

**Priority:** P1  
**Independent test:** When on-hand falls below minimum, an alert with status `new` exists  
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: See Low Stock
**As a** warehouse manager  
**I want to** view products with low inventory  
**So that** I can verify which items need attention

**Priority:** P1  
**Independent test:** Open the low-stock list and see SKUs below minimum  
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Managers MUST be able to set a minimum on-hand quantity per SKU.
- **FR-002**: The system MUST monitor on-hand inventory against the minimum.
- **FR-003**: The system MUST create a low-stock alert when on-hand is under the minimum.
- **FR-004**: Alerts MUST have a status of `new`, `reviewed`, or `resolved`.
- **FR-005**: The system MUST recognize low inventory for multiple SKUs at the same time.
- **FR-006**: On-hand of zero MUST still produce an alert when a minimum is set.
- **FR-007**: Employees MUST NOT change minimum quantities.

---

## Assumptions

- On-hand quantity is maintained by Feature 5 (and later receiving/picking updates).
- Negative on-hand is treated as an error state but still below minimum.
- Creating a supplier order (Feature 1) is a separate action; resolving an alert does not auto-create a P.O.

---

## Edge Cases

- Inventory is fully used up (on-hand `0`) → alert is created.
- Inventory is negative due to an error → alert is created; status remains until reviewed.
- Minimum is updated → open alerts are recalculated (below new min → stay/create; at or above → `resolved`).
- Product is removed from the warehouse → open alerts for that SKU are `resolved`.
- Inventory was damaged (Feature 2 reduced available) → available/on-hand used for the comparison is the post-damage quantity.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Managers can set a minimum quantity.
- **SC-003**: The system recognizes low supply and creates alerts.
- **SC-004**: Staff can view items with low inventory and review alerts.

---

## Key Entities

- **Stock policy:** Minimum quantity for a SKU.
- **Low-stock alert:** A notice that on-hand is below minimum, with review status.

---

## Initial Data Model

### `stock_policies` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required, unique |
| `minimumQty` | INTEGER | Required; MUST be greater than or equal to zero |
| `updatedBy` | INTEGER FK | Manager who last set it |

### `low_stock_alerts` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `onHand` | INTEGER | Snapshot when created/updated |
| `minimumQty` | INTEGER | Snapshot of policy |
| `status` | VARCHAR | `new`, `reviewed`, `resolved` |
| `createdAt` | DATE | Required |

---

## Gherkin AC

### US-6.1 — Set Minimum Stock

#### Scenario: Manager sets a minimum
*   **Given** I am signed in as a manager
*   **When** I set minimum quantity `10` for `SKU-9`
*   **Then** the API returns `200` or `201`
*   **And** the policy for `SKU-9` is `10`

#### Scenario: Employee cannot set minimum
*   **Given** I am an employee
*   **When** I try to set a minimum quantity
*   **Then** the API returns `403`

---

### US-6.2 — Receive Alert

#### Scenario: On-hand falls below minimum
*   **Given** `SKU-9` has a minimum of `10`
*   **When** on-hand becomes `9`
*   **Then** the system creates a low-stock alert
*   **And** the alert status is `new`

#### Scenario: On-hand is zero
*   **Given** `SKU-9` has a minimum of `10`
*   **When** on-hand is `0`
*   **Then** a low-stock alert exists for `SKU-9`

#### Scenario: Alert can be reviewed
*   **Given** a `new` alert exists
*   **When** I mark it `reviewed`
*   **Then** the status is `reviewed`

---

### US-6.3 — See Low Stock

#### Scenario: Manager views low-stock products
*   **Given** `SKU-9` is below minimum and `SKU-2` is not
*   **When** I open the low-stock list
*   **Then** I see `SKU-9`
*   **And** I do not see `SKU-2` as low stock

#### Scenario: Multiple products are low
*   **Given** two SKUs are below their minimums
*   **When** I list alerts with status `new`
*   **Then** both SKUs appear

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY read alerts and policies |
| **Write scope** | Managers set minima; employees and managers MAY change alert status to `reviewed` / `resolved` |
| **Create scope** | Alerts are created by the system when on-hand is below minimum |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/stock-policies` | Yes | List minima |
| `PUT` | `/warehouse/stock-policies/:sku` | Yes (manager) | Set minimum |
| `GET` | `/warehouse/low-stock-alerts` | Yes | List alerts (filter by status) |
| `PUT` | `/warehouse/low-stock-alerts/:id` | Yes | Update alert status |

---

## Screen Requirements

### [View: Low Stock] — route name `low-stock`
*   Heading: **Low-stock alerts**
*   Manager fields: SKU, minimum quantity; action **Save minimum**
*   Alert table: SKU, on-hand, minimum, status
*   Actions: **Mark reviewed**, **Mark resolved**
*   **Empty state:** `"No low-stock alerts."`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Automatic purchase-order creation ([Feature 1](./feature-1-supplier-orders-and-shipments.md))
*   Email/SMS notifications (in-app list is enough for this draft)
*   Supplier lead-time calculations
