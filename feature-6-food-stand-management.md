# Feature: Food Stand Management

**Feature ID:** 6
**Branch pattern:** `feature/6-food-stand-management`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Let administrators manage the park's food stands, vendors, game-day hours, and menus so visitors can see what is available and where.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md), [Feature 3 — Park Field Map](feature-3-park-field-map.md)

---

## User Stories

### US-6.1: Register a food stand
**As a** league administrator
**I want to** add a food stand with its vendor contact and map location
**So that** the league has a complete record of who sells food in the park

**Priority:** P1
**Independent test:** Create a stand and see it in the stand list with its map location.
**Acceptance scenarios:** see ### US-6.1 under Acceptance Criteria

### US-6.2: Schedule stand operating hours
**As a** league administrator
**I want to** set which stands are open on which game days and during what hours
**So that** visitors know when food is available

**Priority:** P1
**Independent test:** Add hours for a date and see the stand listed as open that day.
**Acceptance scenarios:** see ### US-6.2 under Acceptance Criteria

### US-6.3: Manage a stand's menu
**As a** league administrator
**I want to** add menu items with prices and mark them unavailable
**So that** visitors see accurate offerings

**Priority:** P2
**Independent test:** Add a menu item and mark it sold out; confirm it displays as unavailable.
**Acceptance scenarios:** see ### US-6.3 under Acceptance Criteria

### US-6.4: Browse food stands
**As a** visitor
**I want to** see which food stands are open today, their menus, and where they are
**So that** I can decide where to eat during a game

**Priority:** P1
**Independent test:** Open the food page without signing in and see today's open stands with menus.
**Acceptance scenarios:** see ### US-6.4 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Only an `admin` MUST be able to create, edit, or deactivate food stands, hours, and menu items.
- **FR-002**: A food stand MUST have a name, vendor name, vendor email, and a map location.
- **FR-003**: A food stand name MUST be unique.
- **FR-004**: A food stand's map location MUST reference a map point of type `FOOD_STAND`.
- **FR-005**: A map point MUST NOT be used by more than one food stand.
- **FR-006**: Vendor email MUST be a valid email and MUST NOT be shown to anonymous visitors.
- **FR-007**: Operating hours MUST include a date, open time, and close time, with close time after open time.
- **FR-008**: A stand MUST NOT have two operating-hour entries on the same date.
- **FR-009**: A menu item MUST have a name and a price of 0.00 or more with at most two decimal places.
- **FR-010**: A menu item MUST be flagged `isAvailable`, defaulting to true.
- **FR-011**: The public page MUST show only `active` stands that have operating hours on the selected date.
- **FR-012**: Deactivating a stand MUST hide it from the public page but MUST retain its history.

---

## Key Entities

- **Food Stand**: a vendor-operated stand at a fixed map location.
- **Stand Hours**: the date and time window a stand is open.
- **Menu Item**: a food or drink item with price and availability.
- **Map Point**: the location marker (Feature 3).

---

## Initial Data Model

### `food_stands` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `name` | STRING(100) | Required, unique |
| `vendorName` | STRING(150) | Required |
| `vendorEmail` | STRING(255) | Required, valid email |
| `vendorPhone` | STRING(20) | Optional |
| `mapPointId` | INTEGER FK | Required, unique, references `map_points.id` (type `FOOD_STAND`) |
| `status` | ENUM(`active`,`inactive`) | Required, default `active` |

### `food_stand_hours` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `foodStandId` | INTEGER FK | Required, references `food_stands.id` |
| `serviceDate` | DATE | Required; unique with `foodStandId` |
| `openTime` | TIME | Required |
| `closeTime` | TIME | Required, > `openTime` |

### `menu_items` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `foodStandId` | INTEGER FK | Required, references `food_stands.id` |
| `name` | STRING(100) | Required |
| `price` | DECIMAL(6,2) | Required, >= 0 |
| `isAvailable` | BOOLEAN | Required, default `true` |

---

## Acceptance Criteria

### US-6.1 — Register a food stand

#### Scenario: Admin registers a food stand
*   **Given** I am signed in as an `admin`
*   **And** an unused `FOOD_STAND` map point `Snack Shack Spot` exists
*   **When** I enter name `Snack Shack`, vendor `Joe's Concessions`, vendor email `joe@example.com`, and select that map point
*   **And** I click **Save**
*   **Then** the API returns `201` with `status` equal to `active`

#### Scenario: Duplicate stand name is rejected
*   **Given** a stand named `Snack Shack` exists
*   **When** I create another stand named `Snack Shack`
*   **Then** the API returns `409` with `{ "message": "A food stand with this name already exists." }`

#### Scenario: Map point already in use is rejected
*   **Given** map point `Snack Shack Spot` is assigned to `Snack Shack`
*   **When** I assign the same map point to a new stand
*   **Then** the API returns `409` with `{ "message": "This map location is already in use." }`

#### Scenario: Non-food map point is rejected
*   **Given** map point `North Restrooms` has type `RESTROOM`
*   **When** I assign it to a new stand
*   **Then** the API returns `400` with `{ "message": "Location must be a food stand map point." }`

### US-6.2 — Schedule stand operating hours

#### Scenario: Admin sets hours for a game day
*   **Given** I am signed in as an `admin` and `Snack Shack` exists
*   **When** I add hours for `2026-11-14` from `08:30` to `14:00`
*   **Then** the API returns `201`

#### Scenario: Duplicate date is rejected
*   **Given** `Snack Shack` already has hours for `2026-11-14`
*   **When** I add hours for `2026-11-14` again
*   **Then** the API returns `409` with `{ "message": "Hours already exist for this date." }`

#### Scenario: Close time before open time is rejected
*   **Given** I am signed in as an `admin`
*   **When** I enter open `14:00` and close `08:30`
*   **Then** the API returns `400` with `{ "message": "Close time must be after open time." }`

### US-6.3 — Manage a stand's menu

#### Scenario: Admin adds a menu item
*   **Given** I am signed in as an `admin` and `Snack Shack` exists
*   **When** I add item `Hot Dog` priced `3.50`
*   **Then** the API returns `201` with `isAvailable` equal to `true`

#### Scenario: Admin marks an item sold out
*   **Given** `Hot Dog` is available
*   **When** I toggle **Available** off
*   **Then** the public menu shows `Hot Dog` as `Sold out`

#### Scenario: Negative price is rejected
*   **Given** I am signed in as an `admin`
*   **When** I add an item with price `-1.00`
*   **Then** the API returns `400` with `{ "message": "Price must be 0 or more." }`

### US-6.4 — Browse food stands

#### Scenario: Visitor sees stands open on a date
*   **Given** `Snack Shack` is active with hours on `2026-11-14`
*   **When** I open the food page for `2026-11-14` without signing in
*   **Then** I see `Snack Shack` with its hours and menu items

#### Scenario: Stand with no hours is hidden
*   **Given** `Taco Tent` has no hours on `2026-11-14`
*   **When** I open the food page for that date
*   **Then** `Taco Tent` is not shown

#### Scenario: Vendor email is not exposed publicly
*   **Given** I am not signed in
*   **When** I request the food stand list
*   **Then** the response does not include `vendorEmail`

---

## Screen Requirements

### Screen: Food Stands (Public)
**Route:** `/food` | **Access:** Public

| Element | Details |
|---------|---------|
| Date picker | Defaults to today |
| Stand cards | Name, hours, "Show on map" link, menu list |
| Menu list | Item, price, `Sold out` badge when unavailable |
| Empty state | "No food stands are open on this date." |

### Screen: Food Stand Management
**Route:** `/admin/food-stands` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Stands table | Columns: Name, Vendor, Map point, Status |
| **Add stand** | Form: name, vendor name, vendor email, vendor phone, map point dropdown |
| Row actions | Edit, Manage hours, Manage menu, Deactivate |

### Screen: Stand Hours & Menu
**Route:** `/admin/food-stands/:id` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Hours tab | Table of date, open, close; **Add hours** form |
| Menu tab | Table of item, price, availability toggle; **Add item** form |
