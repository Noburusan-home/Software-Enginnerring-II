# Feature: Damaged Product Handling

**Feature ID:** 2  
**Branch pattern:** `feature/2-damaged-product-handling`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for handling broken, damaged, defective, wet, or unusable products  
**Depends on:** [Feature 1 — Supplier Orders and Shipments](./feature-1-supplier-orders-and-shipments.md)  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-2.1: Report Damaged Product
**As a** warehouse employee  
**I want to** report a damaged product  
**So that** the inventory record remains accurate

**Priority:** P1  
**Independent test:** Report a known SKU as damaged with a quantity and verify a damage record is created  
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: Record Damage Reason
**As a** warehouse employee  
**I want to** record why a product is damaged  
**So that** the warehouse has a record of what happened

**Priority:** P1  
**Independent test:** Save a damage report with a selected reason and optional description  
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

### US-2.3: Separate Damaged Inventory
**As a** warehouse employee  
**I want** damaged products separated from usable inventory  
**So that** damaged products are not accidentally shipped

**Priority:** P1  
**Independent test:** After a confirmed unusable quantity, available inventory no longer includes those units  
**Acceptance scenarios:** see ### US-2.3 under Acceptance Criteria

### US-2.4: Determine Product Disposition
**As a** warehouse manager  
**I want to** record what happens to damaged products  
**So that** each damaged item has a documented outcome

**Priority:** P1  
**Independent test:** As a manager, set disposition on a damage report and verify it is saved  
**Acceptance scenarios:** see ### US-2.4 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST allow employees to create a damaged product report.
- **FR-002**: The system MUST identify the damaged product using its SKU or product ID.
- **FR-003**: The system MUST record the number of damaged units.
- **FR-004**: The system MUST allow the employee to select a damage reason from: Broken, Wet, Defective, Missing parts, Damaged packaging, Expired, Contaminated, Unusable, Unknown.
- **FR-005**: The system MUST allow additional details in a description field.
- **FR-006**: The system MUST allow disposition of: Under Review, Repair, Return to Supplier, Reshelve, Dispose, Destroyed.
- **FR-007**: The system MUST remove confirmed unusable quantities from available inventory.
- **FR-008**: The system MUST record which employee reported the damage.
- **FR-009**: Default disposition on create MUST be `Under Review`.
- **FR-010**: Only a manager MUST set a final disposition other than `Under Review`.

---

## Assumptions

- Products already exist in warehouse records (SKU is known).
- Feature 1 documents exist when damage is tied to a shipment or order; a damage report MAY omit those IDs.
- Partial-quantity damage is allowed (not the whole shipment).
- Signed-in identity is available.

---

## Edge Cases

- Only part of a shipment is damaged → record affected quantity only.
- Product is damaged during picking or receiving → still create a damage report.
- A damaged product can be repaired → disposition `Repair` does not destroy the units.
- Reason is unknown → reason `Unknown` is allowed; description SHOULD be used.
- The same product is reported damaged twice → each report is a separate record; quantities accumulate against available inventory only when marked unusable.
- Employee (non-manager) attempts final disposition → `403`.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Damaged products can be reported with SKU, quantity, reason, and reporter.
- **SC-003**: Confirmed unusable quantities are removed from available inventory.
- **SC-004**: Managers can set repair, return, reshelve, dispose, or destroy outcomes.
- **SC-005**: Damaged units MUST NOT appear as available to ship.

---

## Key Entities

- **Damage report:** A record that a quantity of a product is damaged, with reason, description, reporter, and disposition.
- **Product:** Identified by SKU.
- **Employee:** Who reported the damage.
- **Disposition:** What happens to the damaged quantity.

---

## Initial Data Model

### `damage_reports` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `quantity` | INTEGER | Required; MUST be greater than zero |
| `reason` | VARCHAR | Required; enumerated reasons |
| `description` | TEXT | Optional |
| `disposition` | VARCHAR | Required; default `Under Review` |
| `source` | VARCHAR | Optional: `receiving`, `picking`, `on_hand`, `other` |
| `shipmentId` | INTEGER FK | Optional |
| `reportedBy` | INTEGER FK | Required; from `req.user.id` |
| `createdAt` | DATE | Required |

---

## Gherkin AC

### US-2.1 — Report Damaged Product

#### Scenario: Employee reports damage
*   **Given** a product with SKU `SKU-9` exists
*   **And** I am a signed-in warehouse employee
*   **When** I report `SKU-9` as damaged with quantity `3`
*   **Then** the API returns `201` with a damage record
*   **And** the record stores quantity `3` and `reportedBy` as my user id

#### Scenario: Quantity is missing or invalid
*   **Given** I am reporting damage
*   **When** I submit quantity `0`
*   **Then** the API returns `400` with `{ "message": "Quantity must be greater than zero." }`

---

### US-2.2 — Record Damage Reason

#### Scenario: Employee records a damage reason
*   **Given** I am creating a damage report for `SKU-9`
*   **When** I select reason `Wet` and enter description `Pallet soaked in aisle B`
*   **And** I save
*   **Then** the saved record includes reason `Wet` and the description

#### Scenario: Reason is unknown
*   **Given** I am creating a damage report
*   **When** I select reason `Unknown`
*   **Then** the report is saved with reason `Unknown`

---

### US-2.3 — Separate Damaged Inventory

#### Scenario: Unusable quantity is removed from available inventory
*   **Given** `SKU-9` has `10` available units
*   **And** a damage report for `3` units is confirmed unusable
*   **When** available inventory is requested for `SKU-9`
*   **Then** available quantity is `7`
*   **And** the damaged units MUST NOT be offered for shipment

#### Scenario: Partial shipment damage
*   **Given** a shipment contains `12` units of `SKU-9`
*   **When** I report `2` units damaged from that shipment
*   **Then** the damage record quantity is `2`
*   **And** the remaining `10` units are not marked damaged by this report

---

### US-2.4 — Determine Product Disposition

#### Scenario: Manager sets disposition
*   **Given** I am signed in as a manager
*   **And** a damage report exists with disposition `Under Review`
*   **When** I set disposition to `Return to Supplier`
*   **Then** the API returns `200`
*   **And** the report shows `Return to Supplier`

#### Scenario: Employee cannot set final disposition
*   **Given** I am signed in as an employee
*   **And** a damage report exists
*   **When** I try to set disposition to `Dispose`
*   **Then** the API returns `403`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | Signed-in employees and managers MAY list all damage reports |
| **Write scope** | Employees MAY create reports; managers MAY set disposition |
| **Create scope** | `reportedBy` from `req.user.id` |
| **Unknown id** | `404` |
| **UI scope** | Damage list shows API results only |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/damage-reports` | Yes | List damage reports |
| `POST` | `/warehouse/damage-reports` | Yes | Create a damage report |
| `GET` | `/warehouse/damage-reports/:id` | Yes | Get one report |
| `PUT` | `/warehouse/damage-reports/:id` | Yes | Update description/source; manager updates disposition |

**Create request body:**
```json
{
  "sku": "SKU-9",
  "quantity": 3,
  "reason": "Wet",
  "description": "Pallet soaked in aisle B",
  "source": "receiving"
}
```

---

## Screen Requirements

### [View: Damage Reports] — route name `damage`
*   Heading: **Damaged Products**
*   Primary action: **+ Report Damage**
*   Fields: SKU, quantity, reason (select), description, source
*   Table columns: SKU, quantity, reason, disposition, reported by, date
*   Manager action: **Set disposition**
*   **Empty state:** `"No damage reports yet."`
*   **Error:** `<v-alert type="error">`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Full supplier RMA shipping paperwork
*   Financial write-off / accounting
*   Picking workflow ([Feature 4](./feature-4-fast-picking-and-packing.md))
*   Receiving inspection UI ([Feature 8](./feature-8-receiving-inspection.md)) — this feature only stores damage found anywhere
