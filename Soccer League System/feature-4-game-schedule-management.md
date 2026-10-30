# Feature: Game Schedule Management

**Feature ID:** 4
**Branch pattern:** `feature/4-game-schedule-management`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Let administrators schedule games with teams, date, time, and field, avoid conflicts, and let everyone view the schedule and results.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md), [Feature 2 — Team Management](feature-2-team-management.md), [Feature 3 — Park Field Map](feature-3-park-field-map.md)

---

## User Stories

### US-4.1: Schedule a game
**As a** league administrator
**I want to** schedule a game between two teams on a field at a date and time
**So that** families and teams know when and where to show up

**Priority:** P1
**Independent test:** Create a game and see it in the schedule with correct teams, field, date, and time.
**Acceptance scenarios:** see ### US-4.1 under Acceptance Criteria

### US-4.2: Prevent scheduling conflicts
**As the** application
**I want to** block double-booked fields and teams
**So that** no two games overlap

**Priority:** P1
**Independent test:** Create two overlapping games on the same field and receive a conflict error.
**Acceptance scenarios:** see ### US-4.2 under Acceptance Criteria

### US-4.3: View and filter the schedule
**As a** visitor
**I want to** view upcoming games filtered by team, date, or field
**So that** I can quickly find the games I care about

**Priority:** P1
**Independent test:** Filter by team and see only that team's games.
**Acceptance scenarios:** see ### US-4.3 under Acceptance Criteria

### US-4.4: Reschedule or cancel a game
**As a** league administrator
**I want to** change a game's time or field, or cancel it
**So that** weather and field issues are handled without confusion

**Priority:** P2
**Independent test:** Move a game to a new time and see the updated time in the schedule.
**Acceptance scenarios:** see ### US-4.4 under Acceptance Criteria

### US-4.5: Record final scores
**As a** league administrator
**I want to** enter the final score for a completed game
**So that** results are available to the league

**Priority:** P2
**Independent test:** Enter 3–1 for a game and see the result and status `completed`.
**Acceptance scenarios:** see ### US-4.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Only an `admin` MUST be able to create, reschedule, cancel, or score games.
- **FR-002**: A game MUST have a home team, away team, field, game date, start time, and end time.
- **FR-003**: Home team and away team MUST be different.
- **FR-004**: Both teams MUST be `active` and in the same division.
- **FR-005**: The field MUST have status `open`.
- **FR-006**: End time MUST be after start time.
- **FR-007**: Two games on the same field MUST NOT overlap in time on the same date.
- **FR-008**: A team MUST NOT be in two overlapping games on the same date.
- **FR-009**: A new game MUST have status `scheduled`.
- **FR-010**: Game status MUST be one of `scheduled`, `completed`, `cancelled`.
- **FR-011**: Cancelling a game MUST require a cancellation reason.
- **FR-012**: Scores MUST be whole numbers of 0 or more and MUST only be recorded for `scheduled` games whose date is today or earlier.
- **FR-013**: The schedule MUST be viewable without signing in and MUST be sorted by date, then start time.
- **FR-014**: The schedule MUST support filters for team, field, and date range.

---

## Key Entities

- **Game**: a match between two teams at a field, date, and time, with status and optional score.
- **Team**: home and away participants (from Feature 2).
- **Field**: the location of the game (from Feature 3).

---

## Initial Data Model

### `games` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `homeTeamId` | INTEGER FK | Required, references `teams.id` |
| `awayTeamId` | INTEGER FK | Required, references `teams.id`, != `homeTeamId` |
| `fieldId` | INTEGER FK | Required, references `fields.id` |
| `gameDate` | DATE | Required |
| `startTime` | TIME | Required |
| `endTime` | TIME | Required, > `startTime` |
| `status` | ENUM(`scheduled`,`completed`,`cancelled`) | Required, default `scheduled` |
| `homeScore` | INTEGER | Optional, >= 0 |
| `awayScore` | INTEGER | Optional, >= 0 |
| `cancelReason` | STRING(255) | Required when status is `cancelled` |
| `createdBy` | INTEGER FK | Required, references `users.id` |

---

## Acceptance Criteria

### US-4.1 — Schedule a game

#### Scenario: Admin schedules a valid game
*   **Given** I am signed in as an `admin`
*   **And** teams `Blue Lightning` and `Red Rockets` are active in `U10`
*   **And** `Field 1` is open
*   **When** I schedule them on `2026-11-14` from `09:00` to `10:00` on `Field 1`
*   **Then** the API returns `201` with `status` equal to `scheduled`

#### Scenario: Same team on both sides is rejected
*   **Given** I am signed in as an `admin`
*   **When** I choose `Blue Lightning` as both home and away team
*   **Then** the API returns `400` with `{ "message": "Home and away teams must be different." }`

#### Scenario: Teams from different divisions are rejected
*   **Given** `Blue Lightning` is in `U10` and `Green Giants` is in `U8`
*   **When** I try to schedule them against each other
*   **Then** the API returns `400` with `{ "message": "Teams must be in the same division." }`

#### Scenario: Closed field is rejected
*   **Given** `Field 2` has status `closed`
*   **When** I try to schedule a game on `Field 2`
*   **Then** the API returns `400` with `{ "message": "Selected field is closed." }`

### US-4.2 — Prevent scheduling conflicts

#### Scenario: Overlapping game on the same field is rejected
*   **Given** a game exists on `Field 1` on `2026-11-14` from `09:00` to `10:00`
*   **When** I schedule another game on `Field 1` that day from `09:30` to `10:30`
*   **Then** the API returns `409` with `{ "message": "Field is already booked for that time." }`

#### Scenario: Team double-booked is rejected
*   **Given** `Blue Lightning` plays on `2026-11-14` from `09:00` to `10:00`
*   **When** I schedule `Blue Lightning` on `Field 3` that day from `09:30` to `10:30`
*   **Then** the API returns `409` with `{ "message": "A team is already scheduled at that time." }`

#### Scenario: Back-to-back games on the same field are allowed
*   **Given** a game on `Field 1` ends at `10:00`
*   **When** I schedule another game on `Field 1` starting at `10:00`
*   **Then** the API returns `201`

### US-4.3 — View and filter the schedule

#### Scenario: Visitor views upcoming games in order
*   **Given** several scheduled games exist
*   **When** I open the schedule page without signing in
*   **Then** games are listed by date and then start time

#### Scenario: Visitor filters by team
*   **Given** games exist for `Blue Lightning` and other teams
*   **When** I filter by `Blue Lightning`
*   **Then** only games where `Blue Lightning` is home or away are shown

### US-4.4 — Reschedule or cancel a game

#### Scenario: Admin reschedules a game
*   **Given** a scheduled game on `2026-11-14` at `09:00`
*   **When** I change the start time to `11:00` and end time to `12:00` and click **Save**
*   **Then** the API returns `200` with the new times

#### Scenario: Admin cancels a game with a reason
*   **Given** a scheduled game exists
*   **When** I click **Cancel game** and enter the reason `Lightning in the area`
*   **Then** the game `status` is `cancelled`
*   **And** the schedule shows the reason

#### Scenario: Cancelling without a reason is rejected
*   **Given** a scheduled game exists
*   **When** I click **Cancel game** and leave the reason empty
*   **Then** the API returns `400` with `{ "message": "A cancellation reason is required." }`

### US-4.5 — Record final scores

#### Scenario: Admin records a final score
*   **Given** a scheduled game dated today or earlier
*   **When** I enter home score `3` and away score `1` and click **Save result**
*   **Then** the game `status` is `completed`
*   **And** the schedule displays `3 - 1`

#### Scenario: Negative score is rejected
*   **Given** a scheduled game dated today or earlier
*   **When** I enter home score `-1`
*   **Then** the API returns `400` with `{ "message": "Scores must be whole numbers of 0 or more." }`

#### Scenario: Score for a future game is rejected
*   **Given** a scheduled game dated next week
*   **When** I try to record a score
*   **Then** the API returns `400` with `{ "message": "Cannot score a game that has not been played." }`

---

## Screen Requirements

### Screen: Game Schedule
**Route:** `/schedule` | **Access:** Public

| Element | Details |
|---------|---------|
| Filters | Team, Field, From date, To date |
| View toggle | **List** and **Calendar** |
| Game row/card | Date, time, home vs away, field name, status badge, score if completed |
| Field link | Opens the field on the park map |
| Status styling | `cancelled` shown struck through with reason |
| Empty state | "No games match your filters." |
| **Add game** | Visible to `admin` only |

### Screen: Game Form (Create / Edit)
**Route:** `/admin/games/new`, `/admin/games/:id/edit` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Home team, Away team | Dropdowns of active teams |
| Field | Dropdown of open fields |
| Date | Date picker |
| Start time, End time | Time pickers |
| **Save** | Inline conflict errors with exact API messages |
| **Cancel game** | Opens dialog requiring reason |

### Screen: Record Result
**Route:** `/admin/games/:id/result` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Home score, Away score | Number inputs, min 0 |
| **Save result** | Sets status to `completed` |
