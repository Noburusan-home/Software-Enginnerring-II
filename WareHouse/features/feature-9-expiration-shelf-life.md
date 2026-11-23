# Feature: Expiration and Shelf-Life Tracking

**Feature ID:** 9  
**Branch pattern:** `feature/9-expiration-shelf-life`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement of identifying products which are close to their expiration date or have a shelf life  
**Depends on:** None  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-9.1: Record Expiration
**As a** warehouse employee  
**I want to** record expiration dates of products  
**So that** I can know which products are expired

**Priority:** P1  
**Independent test:** Save an expiration date on a lot and retrieve it  
**Acceptance scenarios:** see ### US-9.1 under Acceptance Criteria

### US-9.2: View Upcoming Expiration
**As a** warehouse manager  
**I want to** view products with approaching expiration dates  
**So that** I can take action on products about to expire

**Priority:** P1  
**Independent test:** List lots within the warning window and see them on the expiration view  
**Acceptance scenarios:** see ### US-9.2 under Acceptance Criteria

### US-9.3: Prevent Expired Shipment
**As a** warehouse employee  
**I want to** be alerted of expired products  
**So that** I can avoid shipping expired products

**Priority:** P1  
**Independent test:** Attempt to ship an expired lot and receive a restriction  
**Acceptance scenarios:** see ### US-9.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST record an expiration date when applicable.
- **FR-002**: The system MUST record an expiration status: `ok`, `warning`, `expired`, or `none`.
- **FR-003**: The system MUST warn warehouse employees of products close to expiry (within 30 days unless a lot-specific window is set).
- **FR-004**: The system MUST identify expired products (expiration date before today).
- **FR-005**: The system MUST restrict shipment of expired products.
- **FR-006**: Products without an expiration date MUST have status `none` and MUST NOT be blocked from shipment by this feature.

---

## Assumptions

- Expiration is stored per **lot** (a SKU may have multiple lots and dates).
- “Today” is the server’s current date.
- Picking/shipping integration uses this restriction when Feature 4 exists; this feature still exposes an API to check shippability.

---

## Edge Cases

- A product might not have an expiration date → status `none`.
- A product might expire soon → status `warning`.
- A product might have already expired → status `expired`; shipment blocked.
- A product might have multiple expiration dates → one row per lot.
- A product might not have a shelf life → no lot required.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Expiration dates can be recorded.
- **SC-003**: Employees can identify products about to expire and already expired.
- **SC-004**: Expired products are restricted from being shipped.

---

## Key Entities

- **Lot:** A quantity of a SKU sharing one expiration date.
- **Expiration status:** Derived from expiration date vs today and warning window.

---

## Initial Data Model

### `product_lots` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `lotCode` | VARCHAR | Required |
| `expirationDate` | DATE | Optional (null = no shelf life) |
| `warningDays` | INTEGER | Default 30 |
| `quantity` | INTEGER | Required; >= 0 |
| `createdBy` | INTEGER FK | From `req.user.id` |

Expiration status is **derived**, not stored as the only source of truth (MAY be cached if documented).

---

## Gherkin AC

### US-9.1 — Record Expiration

#### Scenario: Employee records an expiration date
*   **Given** I am a signed-in employee
*   **When** I save lot `L-1` for `SKU-9` with expiration date `2026-12-31`
*   **Then** the API returns `201`
*   **And** the lot stores that expiration date

#### Scenario: Product has no expiration date
*   **Given** I am recording lot `L-2` for `SKU-9`
*   **When** I omit `expirationDate`
*   **Then** the lot is saved
*   **And** expiration status is `none`

---

### US-9.2 — View Upcoming Expiration

#### Scenario: Manager sees products approaching expiration
*   **Given** a product has an expiration date
*   **And** that date is within 30 days
*   **When** I open the expiration list
*   **Then** the system displays an expiration warning
*   **And** status is `warning`

#### Scenario: Multiple lots different dates
*   **Given** `SKU-9` has lot `L-1` expiring tomorrow and lot `L-2` expiring in one year
*   **When** I view upcoming expirations
*   **Then** `L-1` appears in the warning list
*   **And** `L-2` does not

---

### US-9.3 — Prevent Expired Shipment

#### Scenario: Expired product is flagged
*   **Given** lot `L-1` expiration date is yesterday
*   **When** I view the lot
*   **Then** status is `expired`

#### Scenario: Shipping expired product is blocked
*   **Given** lot `L-1` is `expired`
*   **When** I request to ship `L-1`
*   **Then** the API returns `400` with `{ "message": "Expired products cannot be shipped." }`

#### Scenario: Product with no shelf life can ship
*   **Given** a lot has no expiration date
*   **When** I request to ship it
*   **Then** this feature does not block the shipment

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY list lots and statuses |
| **Write scope** | Employees MAY create/update lots |
| **Create scope** | `createdBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/lots` | Yes | List lots (filter `status=warning\|expired`) |
| `POST` | `/warehouse/lots` | Yes | Record a lot / expiration |
| `GET` | `/warehouse/lots/:id` | Yes | Get one lot with derived status |
| `PUT` | `/warehouse/lots/:id` | Yes | Update expiration or quantity |
| `POST` | `/warehouse/lots/:id/ship-check` | Yes | Returns whether the lot may be shipped |

---

## Screen Requirements

### [View: Expiration] — route name `expiration`
*   Heading: **Expiration and shelf life**
*   Primary action: **+ Record lot**
*   Fields: SKU, lot code, expiration date, quantity
*   Table: SKU, lot, expiration date, status
*   **Empty state:** `"No lots with expiration dates."`
*   Warning/expired rows visually distinct

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   FEFO slotting algorithms
*   Automatic destruction work orders
*   Full pick-list UI ([Feature 4](./feature-4-fast-picking-and-packing.md)) — only the ship-check rule
