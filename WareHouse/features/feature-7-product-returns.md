# Feature: Product Returns

**Feature ID:** 7  
**Branch pattern:** `feature/7-product-returns`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for processing returned products  
**Depends on:** None  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-7.1: Record Return
**As a** warehouse employee  
**I want to** record a return  
**So that** the warehouse has a reason for the return

**Priority:** P1  
**Independent test:** Create a return with SKU and reason; record is saved as awaiting inspection  
**Acceptance scenarios:** see ### US-7.1 under Acceptance Criteria

### US-7.2: Inspect Return
**As a** warehouse employee  
**I want to** inspect returned products  
**So that** I can decide if they can be resold

**Priority:** P1  
**Independent test:** Set condition on a return (e.g. Damaged) and persist it  
**Acceptance scenarios:** see ### US-7.2 under Acceptance Criteria

### US-7.3: Process Return
**As a** warehouse employee  
**I want to** give disposition to the product  
**So that** the warehouse knows how to handle it

**Priority:** P2  
**Independent test:** After inspection, a manager sets disposition; inventory is not increased until approved  
**Acceptance scenarios:** see ### US-7.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST allow products to be returned.
- **FR-002**: The system MUST record a reason for the return.
- **FR-003**: The system MUST allow the product to be inspected.
- **FR-004**: Product condition MUST be one of: New, Used, Damaged, Defective, Unusable.
- **FR-005**: Disposition MUST be one of: Reshelved, Repaired, Returned to supplier, Disposed, Destroyed, Awaiting inspection.
- **FR-006**: The system MUST NOT add returned products into available inventory until a manager approves a reshelve disposition.
- **FR-007**: A return MAY be recorded without an original order number; the missing order number MUST be allowed with a warning in the UI.

---

## Assumptions

- Returns may come from customers or other locations.
- Unidentifiable products can still be logged with description and condition `Unusable` until identified.
- Damage reports (Feature 2) are separate if damage is found on owned stock rather than a customer return.

---

## Edge Cases

- Product is returned without an order number → return still saves; `orderReference` optional.
- Product is damaged or missing parts → condition `Damaged` or `Defective`.
- Product is still usable → condition `New` or `Used`; disposition may be `Reshelved` after approval.
- Returned product cannot be identified → SKU optional only if `unidentified` flag is true; description required.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Returns can be recorded and inspected with a condition.
- **SC-003**: Returned products are not available inventory until approved.

---

## Key Entities

- **Return:** A inbound returned product (or quantity) with reason, condition, and disposition.
- **Inspection:** Condition recorded before inventory impact.

---

## Initial Data Model

### `product_returns` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required unless `unidentified` is true |
| `unidentified` | BOOLEAN | Default false |
| `description` | VARCHAR | Required if unidentified |
| `quantity` | INTEGER | Required; > 0 |
| `reason` | VARCHAR | Required |
| `orderReference` | VARCHAR | Optional |
| `condition` | VARCHAR | Optional until inspected |
| `disposition` | VARCHAR | Default `Awaiting inspection` |
| `receivedBy` | INTEGER FK | From `req.user.id` |
| `createdAt` | DATE | Required |

---

## Gherkin AC

### US-7.1 — Record Return

#### Scenario: Employee records a return
*   **Given** a product has been returned
*   **When** I receive the product with SKU `SKU-9`, quantity `1`, and reason `Customer changed mind`
*   **Then** the API returns `201`
*   **And** disposition is `Awaiting inspection`

#### Scenario: Return without order number
*   **Given** I am recording a return
*   **When** I omit `orderReference`
*   **Then** the return is still saved
*   **And** the UI shows `"No order number provided."`

---

### US-7.2 — Inspect Return

#### Scenario: Employee records condition
*   **Given** a return exists with disposition `Awaiting inspection`
*   **When** I set condition `Damaged`
*   **Then** the API returns `200`
*   **And** condition is `Damaged`

#### Scenario: Unidentified product
*   **Given** the returned item has no readable SKU
*   **When** I save with `unidentified` true and a description
*   **Then** the return is saved without a SKU

---

### US-7.3 — Process Return

#### Scenario: Manager approves reshelve
*   **Given** I am a manager
*   **And** a return is inspected with condition `New`
*   **When** I set disposition to `Reshelved`
*   **Then** available inventory MAY increase by the return quantity
*   **And** the return is no longer `Awaiting inspection`

#### Scenario: Return is not in inventory before approval
*   **Given** a return is `Awaiting inspection`
*   **When** I request available inventory for that SKU
*   **Then** the returned quantity is not included

#### Scenario: Employee cannot approve reshelve
*   **Given** I am an employee
*   **When** I set disposition to `Reshelved`
*   **Then** the API returns `403`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY list returns |
| **Write scope** | Employees create and inspect; managers set inventory-affecting dispositions |
| **Create scope** | `receivedBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/returns` | Yes | List returns |
| `POST` | `/warehouse/returns` | Yes | Create a return |
| `GET` | `/warehouse/returns/:id` | Yes | Get one return |
| `PUT` | `/warehouse/returns/:id` | Yes | Inspect / set disposition (role-checked) |

---

## Screen Requirements

### [View: Returns] — route name `returns`
*   Heading: **Product returns**
*   Primary action: **+ Record return**
*   Fields: SKU, quantity, reason, order reference, unidentified checkbox, description
*   Inspection: condition select; manager disposition select
*   **Empty state:** `"No returns recorded."`
*   Quote: `"No order number provided."` when order reference is blank

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Customer refunds and payment capture
*   Carrier return labels
*   Supplier RMA documents beyond disposition `Returned to supplier`
