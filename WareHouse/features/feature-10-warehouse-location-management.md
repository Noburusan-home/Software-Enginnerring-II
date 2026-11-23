# Feature: Warehouse Location Management

**Feature ID:** 10  
**Branch pattern:** `feature/10-warehouse-location-management`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement to track the physical location of products  
**Depends on:** None  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-10.1: Assign Location
**As a** warehouse employee  
**I want** products to be assigned to a warehouse location  
**So that** employees know where to find them

**Priority:** P1  
**Independent test:** Assign SKU `SKU-9` to `A-12-B` and read it back  
**Acceptance scenarios:** see ### US-10.1 under Acceptance Criteria

### US-10.2: Find Product
**As a** warehouse employee  
**I want to** search for a product's location  
**So that** I can find it

**Priority:** P1  
**Independent test:** Search SKU `SKU-9` and see location `A-12-B`  
**Acceptance scenarios:** see ### US-10.2 under Acceptance Criteria

### US-10.3: Move Product
**As a** warehouse employee  
**I want to** record a product's movement  
**So that** the system knows its latest location

**Priority:** P1  
**Independent test:** Move `SKU-9` from `A-12-B` to `B-01-A` and verify the current location  
**Acceptance scenarios:** see ### US-10.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST maintain warehouse locations (code and optional capacity).
- **FR-002**: The system MUST keep products associated to a location.
- **FR-003**: The system MUST enable an employee to search a product's location by SKU.
- **FR-004**: The system MUST allow an employee to record a product's movement.
- **FR-005**: The system MUST enable employees to verify that the product is physically located where the system says it is.
- **FR-006**: A product with no location MUST be searchable and return a not-assigned message.
- **FR-007**: Recording a move MUST update the current location (system of record).

---

## Assumptions

- Location codes are unique (for example `A-12-B`).
- Two different SKUs MAY share a location unless capacity is exceeded.
- This feature tracks location; on-hand quantity remains Feature 5.

---

## Edge Cases

- Product is found in a wrong location → verification fails; employee can record a move to correct it.
- Product has no location → search result `"Location not assigned."`
- Location is full → move MUST return `400` if capacity would be exceeded.
- Product is moved physically but the system is not updated → verification against the old location fails until a move is recorded.
- Two products have the same location → allowed unless capacity is 1 and already occupied.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Products have identifiable locations.
- **SC-003**: Employees can search and find a product's location.
- **SC-004**: Movements are recorded as the latest location.
- **SC-005**: Wrong location information can be corrected by a move or verification.

---

## Key Entities

- **Location:** A bin/slot in the warehouse with a unique code.
- **Product location:** Current assignment of a SKU (or SKU+lot) to a location.
- **Move:** History of a location change.

---

## Initial Data Model

### `locations` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `code` | VARCHAR | Required, unique |
| `description` | VARCHAR | Optional |
| `capacity` | INTEGER | Optional; null = unlimited |

### `product_locations` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `locationId` | INTEGER FK | Required |
| `quantity` | INTEGER | Optional; default 1 |

### `location_moves` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sku` | VARCHAR | Required |
| `fromLocationId` | INTEGER FK | Optional if first assignment |
| `toLocationId` | INTEGER FK | Required |
| `movedBy` | INTEGER FK | From `req.user.id` |
| `movedAt` | DATE | Required |

---

## Gherkin AC

### US-10.1 — Assign Location

#### Scenario: Employee assigns a product to a location
*   **Given** location `A-12-B` exists
*   **When** I assign `SKU-9` to `A-12-B`
*   **Then** the API returns `200` or `201`
*   **And** the current location for `SKU-9` is `A-12-B`

#### Scenario: Location is full
*   **Given** location `A-12-B` has capacity `1` and already holds a product
*   **When** I assign another product to `A-12-B`
*   **Then** the API returns `400` with `{ "message": "Location is full." }`

---

### US-10.2 — Find Product

#### Scenario: Employee searches for a product location
*   **Given** a product has warehouse location `A-12-B`
*   **When** I search for the product
*   **Then** the system displays its location `A-12-B`

#### Scenario: Product has no location
*   **Given** `SKU-9` has no location assignment
*   **When** I search for `SKU-9`
*   **Then** I see `"Location not assigned."`

---

### US-10.3 — Move Product

#### Scenario: Employee records a move
*   **Given** `SKU-9` is at `A-12-B`
*   **When** I move it to `B-01-A`
*   **Then** the current location is `B-01-A`
*   **And** a move record stores from `A-12-B` to `B-01-A` and my user id

#### Scenario: Verification at expected location succeeds
*   **Given** `SKU-9` is assigned to `A-12-B`
*   **When** I verify `SKU-9` at `A-12-B`
*   **Then** verification succeeds

#### Scenario: Product found in the wrong location
*   **Given** `SKU-9` is assigned to `A-12-B`
*   **When** I verify `SKU-9` at `B-01-A`
*   **Then** verification fails
*   **And** I can record a move to correct the location

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in staff MAY search locations |
| **Write scope** | Employees MAY assign, move, and verify |
| **Create scope** | `movedBy` from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/locations` | Yes | List locations |
| `POST` | `/warehouse/locations` | Yes | Create a location |
| `GET` | `/warehouse/locations/search` | Yes | Query by `sku` — current location |
| `PUT` | `/warehouse/products/:sku/location` | Yes | Assign or move |
| `POST` | `/warehouse/products/:sku/location/verify` | Yes | Verify physical vs system location |

**Search:** `GET /warehouse/locations/search?sku=SKU-9`  
**Move body:** `{ "toLocationCode": "B-01-A" }`

---

## Screen Requirements

### [View: Locations] — route name `locations`
*   Heading: **Warehouse locations**
*   Primary action: **+ Add location**
*   Search field: SKU; result shows location code or `"Location not assigned."`
*   Actions: **Assign**, **Move**, **Verify location**
*   **Empty state:** `"No locations defined yet."`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Pick-path optimization ([Feature 4](./feature-4-fast-picking-and-packing.md))
*   Inventory quantity adjustments ([Feature 5](./feature-5-inventory-accuracy.md))
*   3D warehouse maps
