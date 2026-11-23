# Feature: Inventory Accuracy and Verification

**Feature ID:** 5  
**Branch pattern:** `feature/5-inventory-accuracy`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse need of product identification and quantity verification  
**Depends on:** [Feature 10 — Warehouse Location Management](./feature-10-warehouse-location-management.md)  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-5.1: Verify Product
**As a** warehouse employee  
**I want to** verify a product before processing it  
**So that** I don't process the wrong product

**Priority:** P1  
**Independent test:** Submit a SKU that matches expected stock and a SKU that does not  
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Verify Quantity
**As a** warehouse employee  
**I want to** compare the physical quantity with the recorded quantity  
**So that** I ensure the inventory is accurate

**Priority:** P1  
**Independent test:** Count 8 against expected 10 and see a recorded discrepancy  
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: Report Discrepancy
**As a** warehouse employee  
**I want to** report inventory differences  
**So that** inventory bookkeeping errors can be reviewed

**Priority:** P1  
**Independent test:** A count with a difference creates a verification record with expected vs actual  
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST provide SKU verification for products.
- **FR-002**: The system MUST accept a physical count quantity.
- **FR-003**: The system MUST compare physical quantity to expected quantity.
- **FR-004**: The system MUST identify a difference between expected and actual quantity.
- **FR-005**: Managers MUST be able to adjust on-hand inventory after a discrepancy is reviewed.
- **FR-006**: The system MUST record product, expected quantity, verified quantity, employee, date, and result.
- **FR-007**: Employees MUST NOT apply inventory adjustments (manager only).

---

## Assumptions

- Expected quantity comes from on-hand inventory for the SKU (and location when provided).
- A product that does not exist cannot be verified as a match.
- Damaged product handling is Feature 2; a count may still record actual quantity including damaged units as a note.

---

## Edge Cases

- SKU does not match expected product → verification result `sku_mismatch`.
- Physical quantity higher than expected → discrepancy recorded (positive variance).
- Physical quantity lower than expected → discrepancy recorded (negative variance).
- Product does not exist → `404` or `400` with `"Product not found."`
- Wrong product location → employee can still count; location on the record may differ from Feature 10 assignment.
- Damaged product found during count → count still records quantity; damage report is a separate Feature 2 action.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Products and quantities can be verified.
- **SC-003**: Discrepancies are detected and stored with expected vs actual.
- **SC-004**: Inventory corrections by managers are tracked on the verification record.

---

## Key Entities

- **Inventory balance:** On-hand quantity for a SKU (optionally at a location).
- **Verification:** A count event comparing expected vs actual.
- **Adjustment:** Manager-approved change to on-hand quantity.

---

## Initial Data Model

### `inventory_balances` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `locationCode` | VARCHAR | Optional |
| `onHand` | INTEGER | Required; default 0 |

### `inventory_verifications` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `locationCode` | VARCHAR | Optional |
| `expectedQty` | INTEGER | Required |
| `actualQty` | INTEGER | Required; MAY be zero |
| `result` | VARCHAR | `match`, `over`, `short`, `sku_mismatch` |
| `adjusted` | BOOLEAN | Default false |
| `verifiedBy` | INTEGER FK | From `req.user.id` |
| `verifiedAt` | DATE | Required |

---

## Gherkin AC

### US-5.1 — Verify Product

#### Scenario: Employee verifies the expected SKU
*   **Given** inventory exists for `SKU-9`
*   **When** I verify SKU `SKU-9`
*   **Then** the system accepts the product for counting

#### Scenario: SKU does not match
*   **Given** I am counting location stock expected to be `SKU-9`
*   **When** I enter SKU `SKU-2`
*   **Then** the verification result is `sku_mismatch`
*   **And** on-hand inventory is not changed

#### Scenario: Product does not exist
*   **Given** SKU `NO-SUCH` is not in the catalog
*   **When** I try to verify it
*   **Then** the API returns `400` with `{ "message": "Product not found." }`

---

### US-5.2 — Verify Quantity

#### Scenario: Physical count is short
*   **Given** the system expects `10` units of `SKU-9`
*   **When** I count `8` units
*   **Then** the system identifies a discrepancy
*   **And** it records expected `10` and actual `8`
*   **And** result is `short`

#### Scenario: Physical count is over
*   **Given** the system expects `10` units
*   **When** I count `12` units
*   **Then** result is `over`
*   **And** expected and actual are stored

#### Scenario: Count matches
*   **Given** expected quantity is `10`
*   **When** I count `10`
*   **Then** result is `match`

---

### US-5.3 — Report Discrepancy

#### Scenario: Discrepancy is stored for review
*   **Given** I recorded a short count for `SKU-9`
*   **When** I view verification history
*   **Then** I see product, expected, actual, my user id, date, and result

#### Scenario: Manager adjusts inventory
*   **Given** I am a manager
*   **And** a short verification exists for `SKU-9` expected `10` actual `8`
*   **When** I apply the adjustment
*   **Then** on-hand becomes `8`
*   **And** `adjusted` is true

#### Scenario: Employee cannot adjust inventory
*   **Given** I am an employee
*   **When** I try to apply an inventory adjustment
*   **Then** the API returns `403`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY read balances and verification history |
| **Write scope** | Employees MAY create verifications; managers MAY adjust `onHand` |
| **Create scope** | `verifiedBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/inventory` | Yes | List balances |
| `GET` | `/warehouse/inventory/:sku` | Yes | Get expected on-hand |
| `POST` | `/warehouse/inventory/verifications` | Yes | Record a count |
| `GET` | `/warehouse/inventory/verifications` | Yes | List counts |
| `POST` | `/warehouse/inventory/verifications/:id/adjust` | Yes (manager) | Apply on-hand adjustment |

**Count request body:**
```json
{
  "sku": "SKU-9",
  "locationCode": "A-12-B",
  "actualQty": 8
}
```

---

## Screen Requirements

### [View: Inventory Count] — route name `inventory-count`
*   Heading: **Inventory accuracy**
*   Fields: SKU, location, expected quantity (read-only), actual quantity
*   Primary action: **Record count**
*   Manager action: **Apply adjustment**
*   **Empty state:** `"No verification records yet."`
*   Discrepancy rows highlighted when result is `over` or `short`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Low-stock alerts ([Feature 6](./feature-6-low-stock-alerts.md))
*   Damage dispositions ([Feature 2](./feature-2-damaged-product-handling.md))
*   Cycle-count scheduling / ABC analysis
