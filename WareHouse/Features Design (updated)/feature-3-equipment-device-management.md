# Feature: Warehouse Equipment and Device Management

**Feature ID:** 3  
**Branch pattern:** `feature/3-equipment-device-management`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for devices and equipment used to complete warehouse operations  
**Depends on:** None  
**Related:** [architecture.md](./architecture.md)

---

## User Stories

### US-3.1: View Equipment
**As a** warehouse employee  
**I want to** view equipment  
**So that** I know which equipment I can use

**Priority:** P1  
**Independent test:** Open the equipment list and see identifier, type, and status for each device  
**Acceptance scenarios:** see ### US-3.1 under Acceptance Criteria

### US-3.2: Report Broken Equipment
**As a** warehouse employee  
**I want to** report broken equipment  
**So that** it is taken out of use

**Priority:** P1  
**Independent test:** Report a scanner as broken and verify status becomes `Broken`  
**Acceptance scenarios:** see ### US-3.2 under Acceptance Criteria

### US-3.3: Track Equipment
**As a** warehouse manager  
**I want to** know the status of equipment  
**So that** I know what equipment is available

**Priority:** P1  
**Independent test:** As a manager, view statuses and mark a device `Maintenance`  
**Acceptance scenarios:** see ### US-3.3 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The system MUST maintain equipment records.
- **FR-002**: The system MUST support equipment types: Barcode scanner, Tablet, Computer, Label printer, Scale, Inspection device.
- **FR-003**: Equipment MUST have a status of: Available, In Use, Maintenance, Broken, Out of Service, Missing.
- **FR-004**: The system MUST record the employee using equipment when it is assigned.
- **FR-005**: Employees MUST be able to report equipment issues.
- **FR-006**: Managers MUST be able to mark equipment as being repaired or maintained.
- **FR-007**: Equipment with status `Broken`, `Maintenance`, `Out of Service`, or `Missing` MUST NOT be assignable as available.

---

## Assumptions

- Equipment is warehouse-owned, not personal property.
- A backup device is another equipment record, not a special type.
- Signed-in identity is available.

---

## Edge Cases

- A scanner breaks while an order is being picked → employee reports it `Broken`; assignment is cleared.
- Equipment is already assigned → new assignment MUST fail with `400`.
- Equipment is missing → status `Missing`.
- Equipment requires maintenance → manager sets `Maintenance`.
- A backup device is needed → employee uses another record with status `Available`.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Equipment can be identified and its status is visible.
- **SC-003**: Broken equipment can be reported and is no longer shown as available.
- **SC-004**: Equipment can be assigned to an employee when available.
- **SC-005**: Employees can see when equipment is unavailable.

---

## Key Entities

- **Equipment:** A tracked device (scanner, tablet, printer, and so on) with type and status.
- **Assignment:** Optional link from equipment to the employee currently using it.
- **Issue report:** Note that equipment failed or is missing.

---

## Initial Data Model

### `equipment` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `code` | VARCHAR | Required, unique |
| `name` | VARCHAR | Required |
| `type` | VARCHAR | Required; enumerated types |
| `status` | VARCHAR | Required; default `Available` |
| `assignedTo` | INTEGER FK | Optional; employee using it |
| `notes` | TEXT | Optional |

### `equipment_issues` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `equipmentId` | INTEGER FK | Required |
| `reportedBy` | INTEGER FK | From `req.user.id` |
| `statusAfter` | VARCHAR | e.g. `Broken` |
| `description` | TEXT | Optional |
| `createdAt` | DATE | Required |

---

## Gherkin AC

### US-3.1 — View Equipment

#### Scenario: Employee views equipment list
*   **Given** equipment `SCAN-01` (Barcode scanner) exists with status `Available`
*   **When** I open the equipment list
*   **Then** I see `SCAN-01`, type Barcode scanner, and status `Available`

#### Scenario: No equipment exists
*   **Given** there are no equipment records
*   **When** I open the equipment list
*   **Then** I see `"No equipment recorded yet."`

---

### US-3.2 — Report Broken Equipment

#### Scenario: Employee reports a scanner as broken
*   **Given** I am using scanner `SCAN-01`
*   **When** the scanner breaks down
*   **And** I report `SCAN-01` as broken
*   **Then** the API returns `200`
*   **And** status is `Broken`
*   **And** `assignedTo` is cleared

#### Scenario: Cannot assign broken equipment
*   **Given** `SCAN-01` has status `Broken`
*   **When** I try to assign it to myself
*   **Then** the API returns `400` with `{ "message": "Equipment is not available." }`

---

### US-3.3 — Track Equipment

#### Scenario: Manager views status
*   **Given** I am signed in as a manager
*   **And** devices exist with statuses `Available` and `Broken`
*   **When** I view equipment
*   **Then** I see each device's status

#### Scenario: Manager marks maintenance
*   **Given** I am a manager
*   **And** `SCAN-02` is `Available`
*   **When** I mark `SCAN-02` as `Maintenance`
*   **Then** the API returns `200`
*   **And** status is `Maintenance`

#### Scenario: Equipment already assigned
*   **Given** `SCAN-01` is assigned to employee B
*   **When** I try to assign `SCAN-01` to myself
*   **Then** the API returns `400` with `{ "message": "Equipment is already assigned." }`

---

## Data Ownership

| Rule | Requirement |
|------|-------------|
| **Read scope** | All signed-in warehouse staff MAY list equipment |
| **Write scope** | Employees MAY report issues and assign available devices to themselves; managers MAY change maintenance statuses |
| **Create scope** | New equipment records are warehouse-owned; `reportedBy` on issues from `req.user.id` |
| **Unknown id** | `404` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/equipment` | Yes | List equipment |
| `POST` | `/warehouse/equipment` | Yes (manager) | Create an equipment record |
| `GET` | `/warehouse/equipment/:id` | Yes | Get one device |
| `PUT` | `/warehouse/equipment/:id` | Yes | Update status/assignment (role-checked) |
| `POST` | `/warehouse/equipment/:id/issues` | Yes | Report an issue |

---

## Screen Requirements

### [View: Equipment] — route name `equipment`
*   Heading: **Equipment**
*   Primary action: **+ Add equipment** (manager)
*   Employee action: **Report issue**, **Check out** / **Check in**
*   Columns: code, name, type, status, assigned to
*   **Empty state:** `"No equipment recorded yet."`
*   Status `Broken` and `Maintenance` visually distinct from `Available`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] Living reference updated if schema, API, or rules changed

---

## Out of Scope

*   Purchasing new hardware
*   Device firmware or scanner SDK integration
*   Picking screens ([Feature 4](./feature-4-fast-picking-and-packing.md))
