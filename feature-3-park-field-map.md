# Feature: Park Field Map

**Feature ID:** 3
**Branch pattern:** `feature/3-park-field-map`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Give players, families, and staff an interactive map of the park showing every soccer field and key amenities so they can find where to go.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md)

---

## User Stories

### US-3.1: View the park map
**As a** visitor
**I want to** see a map of the park with fields and amenities
**So that** I can find my way around on game day

**Priority:** P1
**Independent test:** Open the map and see every active field and map point as a marker.
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: Manage fields
**As a** league administrator
**I want to** add, edit, and close fields
**So that** only usable fields are available for scheduling

**Priority:** P1
**Independent test:** Create `Field 1` and confirm it appears in the field list as `open`.
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: Place amenities on the map
**As a** league administrator
**I want to** place markers for restrooms, first aid, entrances, food stands, and parking lots
**So that** visitors can locate them

**Priority:** P1
**Independent test:** Add a `RESTROOM` marker and see it on the map.
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

### US-3.4: View field details
**As a** visitor
**I want to** select a field and see its details
**So that** I know the surface and size of the field my team plays on

**Priority:** P2
**Independent test:** Click `Field 1` and see its surface type, size, and status.
**Acceptance scenarios:** see ### US-3.4 under Acceptance Criteria

### US-3.5: Upload the park map image
**As a** league administrator
**I want to** upload the park base map image
**So that** markers sit on an accurate picture of our park

**Priority:** P2
**Independent test:** Upload a PNG and see it displayed as the map background.
**Acceptance scenarios:** see ### US-3.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The map and field list MUST be viewable without signing in.
- **FR-002**: Only an `admin` MUST be able to create, edit, or close fields and map points.
- **FR-003**: A field name MUST be unique within a park.
- **FR-004**: A field MUST have a status of `open` or `closed`.
- **FR-005**: Closed fields MUST be shown on the map as greyed out and MUST NOT be selectable when scheduling games.
- **FR-006**: Each field and map point MUST store map coordinates (`mapX`, `mapY`) as percentages from 0 to 100.
- **FR-007**: Coordinates outside 0 to 100 MUST be rejected.
- **FR-008**: A map point MUST have one type from: `FOOD_STAND`, `PARKING_LOT`, `RESTROOM`, `FIRST_AID`, `ENTRANCE`.
- **FR-009**: The map image MUST be PNG or JPG and MUST NOT exceed 5 MB.
- **FR-010**: The map MUST include a legend identifying each marker type.
- **FR-011**: A field with scheduled future games MUST NOT be deleted; it MUST only be closed.

---

## Key Entities

- **Park**: the physical park; owns the base map image, fields, and map points.
- **Field**: a playable soccer field with surface, size, status, and a position on the map.
- **Map Point**: a non-field location on the map such as restroom, entrance, food stand, or parking lot.

---

## Initial Data Model

### `parks` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `name` | STRING(150) | Required |
| `address` | STRING(255) | Required |
| `mapImageUrl` | STRING(255) | Optional; PNG/JPG |

### `fields` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `parkId` | INTEGER FK | Required, references `parks.id` |
| `name` | STRING(100) | Required; unique with `parkId` |
| `surfaceType` | ENUM(`grass`,`turf`) | Required |
| `fieldSize` | ENUM(`small`,`medium`,`full`) | Required |
| `status` | ENUM(`open`,`closed`) | Required, default `open` |
| `mapX` | DECIMAL(5,2) | Required, 0–100 |
| `mapY` | DECIMAL(5,2) | Required, 0–100 |

### `map_points` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `parkId` | INTEGER FK | Required, references `parks.id` |
| `pointType` | ENUM(`FOOD_STAND`,`PARKING_LOT`,`RESTROOM`,`FIRST_AID`,`ENTRANCE`) | Required |
| `label` | STRING(100) | Required |
| `mapX` | DECIMAL(5,2) | Required, 0–100 |
| `mapY` | DECIMAL(5,2) | Required, 0–100 |

---

## Acceptance Criteria

### US-3.1 — View the park map

#### Scenario: Visitor sees fields and amenities on the map
*   **Given** a park has 2 open fields and 1 restroom marker
*   **When** I open the map page without signing in
*   **Then** I see 3 markers on the park map
*   **And** a legend lists each marker type

#### Scenario: Closed field is greyed out
*   **Given** `Field 2` has status `closed`
*   **When** I open the map page
*   **Then** the `Field 2` marker is displayed greyed out

### US-3.2 — Manage fields

#### Scenario: Admin creates a field
*   **Given** I am signed in as an `admin`
*   **When** I enter name `Field 1`, surface `grass`, size `full`, X `25`, Y `40` and click **Save**
*   **Then** the API returns `201` with `status` equal to `open`

#### Scenario: Duplicate field name is rejected
*   **Given** `Field 1` exists in the park
*   **When** I create another field named `Field 1`
*   **Then** the API returns `409` with `{ "message": "A field with this name already exists." }`

#### Scenario: Field with future games cannot be deleted
*   **Given** `Field 1` has a scheduled game next week
*   **When** I try to delete `Field 1`
*   **Then** the API returns `409` with `{ "message": "Field has scheduled games. Close it instead." }`

### US-3.3 — Place amenities on the map

#### Scenario: Admin adds a restroom marker
*   **Given** I am signed in as an `admin`
*   **When** I add a point of type `RESTROOM`, label `North Restrooms`, X `60`, Y `15`
*   **Then** the API returns `201`
*   **And** the marker appears on the map

#### Scenario: Out-of-range coordinates are rejected
*   **Given** I am signed in as an `admin`
*   **When** I add a point with X `130`
*   **Then** the API returns `400` with `{ "message": "Coordinates must be between 0 and 100." }`

### US-3.4 — View field details

#### Scenario: Visitor opens field details
*   **Given** `Field 1` is `grass`, `full`, and `open`
*   **When** I click the `Field 1` marker
*   **Then** a panel shows surface `grass`, size `full`, and status `open`

### US-3.5 — Upload the park map image

#### Scenario: Admin uploads a valid map image
*   **Given** I am signed in as an `admin`
*   **When** I upload a 2 MB PNG
*   **Then** the API returns `200` with `mapImageUrl`
*   **And** the image displays as the map background

#### Scenario: Oversized map image is rejected
*   **Given** I am signed in as an `admin`
*   **When** I upload a 9 MB PNG
*   **Then** the API returns `400` with `{ "message": "Map image must be 5 MB or smaller." }`

---

## Screen Requirements

### Screen: Park Map
**Route:** `/map` | **Access:** Public

| Element | Details |
|---------|---------|
| Map canvas | Park image with markers positioned by `mapX` / `mapY` |
| Markers | Distinct icon per type; fields shown as numbered pins |
| Legend | Always visible; lists all marker types |
| Detail panel | Opens on marker click with name, type, and details |
| Filter chips | Toggle visibility by marker type |
| **Edit map** button | Visible to `admin` only |
| Responsive | Pinch-zoom and pan on mobile |

### Screen: Field Management
**Route:** `/admin/fields` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Fields table | Columns: Name, Surface, Size, Status |
| **Add field** | Form with name, surface, size, status, X, Y |
| Click-to-place | Clicking the map fills X and Y automatically |
| **Close / Reopen** | Toggle with confirmation dialog |

### Screen: Map Editor
**Route:** `/admin/map` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Upload control | Accepts PNG/JPG up to 5 MB |
| Marker palette | Choose type, then click map to place |
| Drag to move | Updates coordinates on drop |
| **Save** | Toast "Map saved." |
