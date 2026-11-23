# Feature: Product Receiving and Inspection

**Feature ID:** 8  
**Branch pattern:** `feature/8-receiving-inspection`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Requirement of warehouse to make inspection of products arriving to them  
**Depends on:** [Feature 1 — Supplier Orders and Shipments](./feature-1-supplier-orders-and-shipments.md)  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-8.1: Receive Shipment
**As a** warehouse employee  
**I want to** record an incoming shipment  
**So that** the warehouse will know about arrival of products

**Priority:** P1  
**Independent test:** Mark an expected inbound shipment `Arrived` and start inspection  
**Acceptance scenarios:** see ### US-8.1 under Acceptance Criteria

### US-8.2: Check Quantity
**As a** warehouse employee  
**I want to** check received quantity against expected quantity  
**So that** a receiving exception will be recorded

**Priority:** P1  
**Independent test:** Receive fewer units than expected and see an exception  
**Acceptance scenarios:** see ### US-8.2 under Acceptance Criteria

### US-8.3: Inspect Product
**As a** warehouse employee  
**I want to** inspect received products  
**So that** damaged products will not go as normal available inventory

**Priority:** P1  
**Independent test:** Complete inspection; damaged quantity is not added to available inventory  
**Acceptance scenarios:** see ### US-8.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST identify the incoming shipment (P.O. number, shipment id, or receiving reference).
- **FR-002**: The employee MUST verify the product that is received (SKU).
- **FR-003**: The employee MUST verify the received quantity.
- **FR-004**: The employee MUST be able to report damage on received product.
- **FR-005**: Receiving status MUST be one of: Expected, Arrived, Inspecting, Received, Exception.
- **FR-006**: Products MUST NOT be added to available inventory before required inspection is completed.
- **FR-007**: Wrong SKU or unidentifiable product MUST set status `Exception`.

---

## Assumptions

- Expected inbound lines come from a Feature 1 supplier order / shipment when a P.O. is known.
- A shipment without documentation can still be received with status starting at `Arrived` and `Exception` if identity is unknown.
- Feature 2 damage reports MAY be created from receiving damage; this feature records the receiving outcome either way.

---

## Edge Cases

- Lesser quantity than expected → `Exception`; record expected vs received.
- Higher quantity than expected → `Exception`.
- Product received damaged → damaged qty excluded from available inventory.
- Wrong product received → `Exception`; line not received as expected SKU.
- Shipment without documentation → create a receiving record with no P.O.; status `Exception` until identified.
- Product impossible to identify → line marked unidentified; not added to inventory.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Incoming shipments can be entered and marked arrived.
- **SC-003**: Products and quantities are checked before inventory increase.
- **SC-004**: Damage found at receiving is recorded and kept out of available stock.
- **SC-005**: Receiving records are kept with status history.

---

## Key Entities

- **Receiving record:** One inbound truck/shipment being received.
- **Receiving line:** Expected vs received SKU and quantity, inspection notes.
- **Receiving status:** Expected → Arrived → Inspecting → Received or Exception.

---

## Initial Data Model

### `receivings` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `po_number` | VARCHAR | Optional |
| `shipmentId` | INTEGER FK | Optional |
| `status` | VARCHAR | Required; default `Expected` |
| `arrivedAt` | DATE | Optional |
| `receivedBy` | INTEGER FK | From `req.user.id` when arrived |

### `receiving_lines` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `receivingId` | INTEGER FK | Required |
| `sku` | VARCHAR | Optional if unidentified |
| `expectedQty` | INTEGER | Optional if no document |
| `receivedQty` | INTEGER | Optional until counted |
| `damagedQty` | INTEGER | Default 0 |
| `unidentified` | BOOLEAN | Default false |
| `notes` | TEXT | Optional |

---

## Gherkin AC

### US-8.1 — Receive Shipment

#### Scenario: Expected shipment arrives
*   **Given** a shipment is expected
*   **When** the shipment arrives
*   **Then** I can mark it as `Arrived`
*   **And** I can start inspecting the shipment (`Inspecting`)

#### Scenario: Shipment without documentation
*   **Given** no P.O. number is available
*   **When** I create a receiving record without `po_number`
*   **Then** the record is saved
*   **And** status is `Exception` until the shipment is identified

---

### US-8.2 — Check Quantity

#### Scenario: Short quantity
*   **Given** expected quantity is `10`
*   **When** I record received quantity `8`
*   **Then** receiving status is `Exception`
*   **And** expected and received quantities are stored

#### Scenario: Over quantity
*   **Given** expected quantity is `10`
*   **When** I record received quantity `12`
*   **Then** status is `Exception`

---

### US-8.3 — Inspect Product

#### Scenario: Clean receipt is completed
*   **Given** received SKU and quantity match expected
*   **And** damaged quantity is `0`
*   **When** I complete inspection
*   **Then** status is `Received`
*   **And** available inventory increases by the received quantity

#### Scenario: Damaged units are excluded
*   **Given** I receive `10` units and mark `2` damaged
*   **When** inspection is completed
*   **Then** available inventory increases by `8`
*   **And** `2` are not available

#### Scenario: Wrong product
*   **Given** expected SKU is `SKU-9`
*   **When** I verify SKU `SKU-2`
*   **Then** status is `Exception`
*   **And** available inventory is not increased for `SKU-9`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY list receivings |
| **Write scope** | Employees MAY arrive, count, and inspect |
| **Create scope** | `receivedBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/receivings` | Yes | List receiving records |
| `POST` | `/warehouse/receivings` | Yes | Create expected/arrived receiving |
| `GET` | `/warehouse/receivings/:id` | Yes | Get receiving with lines |
| `PUT` | `/warehouse/receivings/:id` | Yes | Update status (Arrived / Inspecting / Received / Exception) |
| `PUT` | `/warehouse/receivings/:id/lines/:lineId` | Yes | Record counts, damage, SKU |

---

## Screen Requirements

### [View: Receiving] — route name `receiving`
*   Heading: **Receiving and inspection**
*   Primary action: **+ Record arrival**
*   Status chip: Expected, Arrived, Inspecting, Received, Exception
*   Line fields: expected SKU/qty, received SKU/qty, damaged qty
*   **Empty state:** `"No inbound shipments to receive."`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Full damage disposition workflow ([Feature 2](./feature-2-damaged-product-handling.md))
*   Putaway bin assignment UI ([Feature 10](./feature-10-warehouse-location-management.md))
*   Carrier appointment scheduling
