# Feature: Team Management

**Feature ID:** 2
**Branch pattern:** `feature/2-team-management`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Let league administrators organize teams into age divisions and assign each team a manager so games and contacts can be tied to real teams.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md)

---

## User Stories

### US-2.1: Create divisions
**As a** league administrator
**I want to** define age divisions such as U8 and U10
**So that** teams are grouped fairly for scheduling

**Priority:** P1
**Independent test:** Create division `U10` and see it in the division list.
**Acceptance scenarios:** see ### US-2.1 under Acceptance Criteria

### US-2.2: Create a team
**As a** league administrator
**I want to** create a team in a division for a season
**So that** the team can be scheduled for games

**Priority:** P1
**Independent test:** Create team `Blue Lightning` in `U10` and confirm it is saved as `active`.
**Acceptance scenarios:** see ### US-2.2 under Acceptance Criteria

### US-2.3: Assign a manager to a team
**As a** league administrator
**I want to** assign a manager to a team
**So that** families and staff know who runs each team

**Priority:** P1
**Independent test:** Assign a user with role `manager` to a team and see the manager name on the team.
**Acceptance scenarios:** see ### US-2.3 under Acceptance Criteria

### US-2.4: Browse teams
**As a** league visitor
**I want to** view teams by division
**So that** I can find a team and its manager

**Priority:** P2
**Independent test:** Open the team list, filter by `U10`, and see only `U10` teams.
**Acceptance scenarios:** see ### US-2.4 under Acceptance Criteria

### US-2.5: Edit or deactivate a team
**As a** league administrator
**I want to** rename or deactivate a team
**So that** the roster of teams stays accurate during the season

**Priority:** P2
**Independent test:** Deactivate a team and confirm it no longer appears in the active team list.
**Acceptance scenarios:** see ### US-2.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Only an `admin` MUST be able to create, edit, or deactivate divisions and teams.
- **FR-002**: A division MUST have a unique name.
- **FR-003**: A team MUST belong to exactly one division and one season year.
- **FR-004**: A team name MUST be unique within its division and season year.
- **FR-005**: A team MUST have at most one manager.
- **FR-006**: Only a user with role `manager` MUST be assignable as a team manager.
- **FR-007**: A user MAY manage more than one team.
- **FR-008**: A new team MUST default to status `active`.
- **FR-009**: Deactivated teams MUST NOT appear in the default team list but MUST remain linked to past games.
- **FR-010**: The team list MUST be viewable without signing in and MUST support filtering by division.
- **FR-011**: The team list MUST NOT expose manager email to anonymous visitors.

---

## Key Entities

- **Division**: an age group that teams compete in.
- **Team**: a named squad in one division and season; has zero or one manager.
- **User (manager)**: an existing account with role `manager` who runs a team.

---

## Initial Data Model

### `divisions` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `name` | STRING(50) | Required, unique |
| `minAge` | INTEGER | Required |
| `maxAge` | INTEGER | Required, >= `minAge` |

### `teams` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `name` | STRING(100) | Required; unique with `divisionId` + `seasonYear` |
| `divisionId` | INTEGER FK | Required, references `divisions.id` |
| `seasonYear` | INTEGER | Required |
| `managerId` | INTEGER FK | Optional, references `users.id` (role must be `manager`) |
| `primaryColor` | STRING(20) | Optional |
| `status` | ENUM(`active`,`inactive`) | Required, default `active` |

---

## Acceptance Criteria

### US-2.1 — Create divisions

#### Scenario: Admin creates a division
*   **Given** I am signed in as an `admin`
*   **When** I enter name `U10`, minimum age `9`, maximum age `10` and click **Save**
*   **Then** the API returns `201` with the new division
*   **And** `U10` appears in the division list

#### Scenario: Duplicate division name is rejected
*   **Given** a division named `U10` exists
*   **When** I create another division named `U10`
*   **Then** the API returns `409` with `{ "message": "A division with this name already exists." }`

### US-2.2 — Create a team

#### Scenario: Admin creates a team
*   **Given** I am signed in as an `admin`
*   **And** division `U10` exists
*   **When** I create team `Blue Lightning` in `U10` for season `2026`
*   **Then** the API returns `201` with `status` equal to `active`

#### Scenario: Duplicate team name in same division and season
*   **Given** team `Blue Lightning` exists in `U10` for `2026`
*   **When** I create another team `Blue Lightning` in `U10` for `2026`
*   **Then** the API returns `409` with `{ "message": "A team with this name already exists in this division and season." }`

#### Scenario: Manager cannot create a team
*   **Given** I am signed in as a `manager`
*   **When** I try to create a team
*   **Then** the API returns `403`

### US-2.3 — Assign a manager to a team

#### Scenario: Admin assigns a manager
*   **Given** team `Blue Lightning` has no manager
*   **And** user `maria@example.com` has role `manager`
*   **When** I assign her to `Blue Lightning` and click **Save**
*   **Then** the API returns `200` with `managerId` equal to her `userId`

#### Scenario: Assigning a non-manager user is rejected
*   **Given** user `sam@example.com` has role `member`
*   **When** I try to assign him as manager of `Blue Lightning`
*   **Then** the API returns `400` with `{ "message": "Selected user is not a manager." }`

### US-2.4 — Browse teams

#### Scenario: Visitor filters teams by division
*   **Given** teams exist in divisions `U8` and `U10`
*   **When** I open the team list and choose division `U10`
*   **Then** only `U10` teams are shown

#### Scenario: Anonymous visitor does not see manager email
*   **Given** I am not signed in
*   **When** I open the team list
*   **Then** each team shows the manager's name
*   **And** no manager email is present in the response

### US-2.5 — Edit or deactivate a team

#### Scenario: Admin deactivates a team
*   **Given** I am signed in as an `admin`
*   **When** I click **Deactivate** on `Blue Lightning` and confirm
*   **Then** the team `status` is `inactive`
*   **And** it no longer appears in the default team list

#### Scenario: Past games keep the deactivated team
*   **Given** `Blue Lightning` has a completed game and is now `inactive`
*   **When** I view that game
*   **Then** the team name `Blue Lightning` is still shown

---

## Screen Requirements

### Screen: Team List
**Route:** `/teams` | **Access:** Public

| Element | Details |
|---------|---------|
| Division filter | Dropdown with **All divisions** plus each division |
| Season filter | Dropdown, defaults to current season |
| Teams table | Columns: Team, Division, Manager name, Status |
| **Add team** button | Visible to `admin` only |
| Empty state | "No teams found for this filter." |

### Screen: Team Form (Create / Edit)
**Route:** `/admin/teams/new`, `/admin/teams/:id/edit` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Name | Text, required |
| Division | Dropdown, required |
| Season year | Number, required |
| Manager | Searchable dropdown listing users with role `manager` |
| Primary color | Color picker, optional |
| **Save** / **Cancel** | Save shows toast "Team saved." |
| **Deactivate** | Edit mode only; requires confirmation dialog |

### Screen: Division Management
**Route:** `/admin/divisions` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Divisions table | Columns: Name, Min age, Max age |
| **Add division** | Opens inline form with name, min age, max age |
