# Feature: Fast Picking and Packing

**Feature ID:** 4  
**Branch pattern:** `feature/4-fast-picking-and-packing`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for faster product picking and packing while maintaining accuracy  
**Depends on:** [Feature 10 — Warehouse Location Management](./feature-10-warehouse-location-management.md)  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-4.1: Locate Product
**As a** warehouse employee  
**I want to** easily locate the product  
**So that** I can save time

**Priority:** P1  
**Independent test:** Open a pick list line and see the warehouse location for that SKU  
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: Verify Product
**As a** warehouse employee  
**I want to** verify the product while picking it  
**So that** the correct item is picked

**Priority:** P1  
**Independent test:** Confirm a matching SKU on a pick line; reject a mismatched SKU  
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: Complete Multiple Items
**As a** warehouse employee  
**I want to** complete multiple items  
**So that** I can save time

**Priority:** P1  
**Independent test:** Mark several pick lines complete on one list and verify each completion is recorded  
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST provide the warehouse location of the product on each pick line.
- **FR-002**: The system MUST provide a picking list of products for the employee.
- **FR-003**: The system MUST provide a way for the employee to verify the product (SKU).
- **FR-004**: The system MUST provide a way for the employee to verify the quantity.
- **FR-005**: The system MUST record completion of picking an item.
- **FR-006**: The system MUST verify products at packing so the packed set matches the pick list.
- **FR-007**: A pick line MUST NOT be marked picked if the verified SKU does not match.
- **FR-008**: A pick line MUST NOT be marked picked if verified quantity is less than required (short pick is recorded as an exception).

---

## Assumptions

- Feature 10 location assignment exists so a location can be shown.
- Pick lists are created from an order or shipment document (Feature 1) or a simple list of SKUs for this draft.
- Equipment used while picking is Feature 3; this feature does not assign scanners.

---

## Edge Cases

- Product is not in the expected location → show location anyway; employee can record `not found`.
- Quantity is not enough → short-pick exception; line not fully complete.
- Product is defective → employee does not complete the pick; may open a damage report (Feature 2) separately.
- Wrong product is picked → SKU verification fails; line stays open.
- Product has already been picked → completing again MUST return `400`.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Products can be located from the pick list.
- **SC-003**: Products are picked only after SKU and quantity verification.
- **SC-004**: Completion of each pick line is recorded.
- **SC-005**: Packing checks that packed SKUs match the pick list.

---

## Key Entities

- **Pick list:** A set of lines an employee must pick for an order.
- **Pick line:** One SKU, required quantity, location, verification, and completion status.
- **Pack check:** Confirmation that packed items match picked lines.

---

## Initial Data Model

### `pick_lists` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `reference` | VARCHAR | Required (order/shipment/P.O. number) |
| `status` | VARCHAR | `open`, `picking`, `picked`, `packed` |
| `assignedTo` | INTEGER FK | Optional |
| `createdBy` | INTEGER FK | From `req.user.id` |

### `pick_lines` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `pickListId` | INTEGER FK | Required |
| `sku` | VARCHAR | Required |
| `requiredQty` | INTEGER | Required; > 0 |
| `locationCode` | VARCHAR | Required when known |
| `verifiedSku` | VARCHAR | Optional until verified |
| `pickedQty` | INTEGER | Optional until picked |
| `status` | VARCHAR | `pending`, `picked`, `short`, `not_found` |

---

## Gherkin AC

### US-4.1 — Locate Product

#### Scenario: Pick list shows warehouse location
*   **Given** an order has product `SKU-9`
*   **And** `SKU-9` is assigned to location `A-12-B`
*   **When** I open the pick list
*   **Then** the system shows location `A-12-B` and required quantity

#### Scenario: Product has no location
*   **Given** a pick line for `SKU-9` has no location
*   **When** I view the line
*   **Then** I see `"Location not assigned."`

---

### US-4.2 — Verify Product

#### Scenario: Employee verifies the correct SKU and quantity
*   **Given** a pick line requires `SKU-9` quantity `4` at `A-12-B`
*   **When** I verify SKU `SKU-9` and quantity `4`
*   **And** I mark the line picked
*   **Then** the API returns `200`
*   **And** the line status is `picked`

#### Scenario: Wrong product is verified
*   **Given** a pick line requires `SKU-9`
*   **When** I verify SKU `SKU-2`
*   **Then** the API returns `400` with `{ "message": "SKU does not match this pick line." }`
*   **And** the line remains `pending`

#### Scenario: Short quantity
*   **Given** a pick line requires quantity `4`
*   **When** I verify quantity `2`
*   **Then** the line status is `short`
*   **And** the line is not fully complete

---

### US-4.3 — Complete Multiple Items

#### Scenario: Employee completes several lines on one list
*   **Given** a pick list has three pending lines
*   **When** I successfully pick two of them
*   **Then** those two lines are `picked`
*   **And** the remaining line stays `pending`

#### Scenario: Line already picked
*   **Given** a pick line is already `picked`
*   **When** I try to pick it again
*   **Then** the API returns `400` with `{ "message": "This line is already picked." }`

#### Scenario: Packing verification succeeds
*   **Given** all pick lines are `picked`
*   **When** I confirm packed SKUs match the list
*   **Then** the pick list status is `packed`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | Signed-in employees MAY view pick lists for this warehouse |
| **Write scope** | Assigned employee MAY update line verification and completion |
| **Create scope** | `createdBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/pick-lists` | Yes | List pick lists |
| `POST` | `/warehouse/pick-lists` | Yes | Create a pick list |
| `GET` | `/warehouse/pick-lists/:id` | Yes | Get list with lines and locations |
| `PUT` | `/warehouse/pick-lists/:id/lines/:lineId` | Yes | Verify / complete a line |
| `POST` | `/warehouse/pick-lists/:id/pack` | Yes | Confirm packing |

---

## Screen Requirements

### [View: Pick List] — route name `picking`
*   Heading: **Picking**
*   Shows location, SKU, required quantity per line
*   Actions: **Verify SKU**, **Confirm quantity**, **Mark picked**
*   Packing action: **Confirm packed**
*   **Empty state:** `"No pick lists yet."`
*   Failed SKU match shows `"SKU does not match this pick line."`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Wave/batch path optimization algorithms
*   Damage reporting UI ([Feature 2](./feature-2-damaged-product-handling.md))
*   Creating supplier orders ([Feature 1](./feature-1-supplier-orders-and-shipments.md))
