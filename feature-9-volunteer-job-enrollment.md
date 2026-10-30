# Feature: Volunteer Job Enrollment

**Feature ID:** 9
**Branch pattern:** `feature/9-volunteer-job-enrollment`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Let administrators post volunteer jobs for game days and let members enroll in open slots so the league is staffed without double-booking people.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md), [Feature 4 — Game Schedule Management](feature-4-game-schedule-management.md)

---

## User Stories

### US-9.1: Post a volunteer job
**As a** league administrator
**I want to** create a volunteer job with a date, time, location, and number of slots
**So that** members can see where help is needed

**Priority:** P1
**Independent test:** Create a job with 3 slots and see it in the open jobs list.
**Acceptance scenarios:** see ### US-9.1 under Acceptance Criteria

### US-9.2: Browse open jobs
**As a** signed-in member
**I want to** see open volunteer jobs with remaining slots
**So that** I can choose a job that fits my schedule

**Priority:** P1
**Independent test:** Open the job board and see only jobs with available slots, soonest first.
**Acceptance scenarios:** see ### US-9.2 under Acceptance Criteria

### US-9.3: Enroll in a job
**As a** signed-in member
**I want to** sign up for a volunteer job
**So that** the league knows I will be there

**Priority:** P1
**Independent test:** Enroll in a job and see my enrollment and a reduced slot count.
**Acceptance scenarios:** see ### US-9.3 under Acceptance Criteria

### US-9.4: Withdraw from a job
**As a** signed-in member
**I want to** cancel my enrollment
**So that** the slot is freed if I can no longer attend

**Priority:** P1
**Independent test:** Withdraw and confirm the slot becomes available again.
**Acceptance scenarios:** see ### US-9.4 under Acceptance Criteria

### US-9.5: View job rosters
**As a** league administrator
**I want to** see who is enrolled in each job
**So that** I can confirm coverage and contact volunteers

**Priority:** P2
**Independent test:** Open a job's roster and see enrolled names and emails.
**Acceptance scenarios:** see ### US-9.5 under Acceptance Criteria

### US-9.6: View my volunteer schedule
**As a** signed-in member
**I want to** see all jobs I am enrolled in
**So that** I remember where to be and when

**Priority:** P2
**Independent test:** Enroll in two jobs and see both on My Jobs in date order.
**Acceptance scenarios:** see ### US-9.6 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Only an `admin` MUST be able to create, edit, or cancel volunteer jobs.
- **FR-002**: A job MUST have a title, job type, date, start time, end time, location label, and total slots.
- **FR-003**: Job type MUST be one of `scorekeeper`, `concessions`, `parking`, `field_setup`, `first_aid`, `other`.
- **FR-004**: Total slots MUST be a positive integer.
- **FR-005**: End time MUST be after start time.
- **FR-006**: A job MAY reference a game; when it does, the job date MUST match the game's date.
- **FR-007**: Only signed-in users MUST be able to view jobs and enroll.
- **FR-008**: A user MUST NOT enroll in the same job twice.
- **FR-009**: Enrollment MUST be rejected when no slots remain.
- **FR-010**: A user MUST NOT enroll in two jobs whose times overlap on the same date.
- **FR-011**: A user MUST NOT enroll in a job that has already started or is cancelled.
- **FR-012**: A user MUST be able to withdraw only from their own enrollment, and only before the job starts.
- **FR-013**: Cancelling a job MUST mark all its enrollments `cancelled` and keep their history.
- **FR-014**: Total slots MUST NOT be reduced below the number of active enrollments.
- **FR-015**: Only an `admin` MUST see volunteer emails and phone numbers on a roster.

---

## Key Entities

- **Volunteer Job**: a task with date, time, location, type, and a limited number of slots.
- **Job Enrollment**: one user's commitment to one job.
- **Game**: optional link to the game the job supports (Feature 4).
- **User**: the volunteer (Feature 1).

---

## Initial Data Model

### `volunteer_jobs` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `title` | STRING(150) | Required |
| `jobType` | ENUM(`scorekeeper`,`concessions`,`parking`,`field_setup`,`first_aid`,`other`) | Required |
| `description` | TEXT | Optional |
| `gameId` | INTEGER FK | Optional, references `games.id` |
| `jobDate` | DATE | Required; equals game date when `gameId` is set |
| `startTime` | TIME | Required |
| `endTime` | TIME | Required, > `startTime` |
| `locationLabel` | STRING(150) | Required, e.g. "Field 1" or "East Lot" |
| `totalSlots` | INTEGER | Required, > 0 |
| `status` | ENUM(`open`,`cancelled`) | Required, default `open` |
| `createdBy` | INTEGER FK | Required, references `users.id` |

### `job_enrollments` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `jobId` | INTEGER FK | Required, references `volunteer_jobs.id` |
| `userId` | INTEGER FK | Required, references `users.id`; unique with `jobId` |
| `status` | ENUM(`enrolled`,`withdrawn`,`cancelled`) | Required, default `enrolled` |
| `enrolledAt` | DATETIME | Auto-set |

---

## Acceptance Criteria

### US-9.1 — Post a volunteer job

#### Scenario: Admin posts a job
*   **Given** I am signed in as an `admin`
*   **When** I enter title `Scorekeeper — Field 1`, type `scorekeeper`, date `2026-11-14`, `09:00` to `10:00`, location `Field 1`, slots `2`
*   **And** I click **Save**
*   **Then** the API returns `201` with `status` equal to `open`

#### Scenario: Zero slots is rejected
*   **Given** I am signed in as an `admin`
*   **When** I enter total slots `0`
*   **Then** the API returns `400` with `{ "message": "Total slots must be at least 1." }`

#### Scenario: Job date must match linked game date
*   **Given** a game exists on `2026-11-14`
*   **When** I create a job linked to that game with date `2026-11-15`
*   **Then** the API returns `400` with `{ "message": "Job date must match the game date." }`

#### Scenario: Member cannot post a job
*   **Given** I am signed in as a `member`
*   **When** I try to create a job
*   **Then** the API returns `403`

### US-9.2 — Browse open jobs

#### Scenario: Member sees open jobs with remaining slots
*   **Given** I am signed in as a `member`
*   **And** a job has `3` slots with `1` enrolled
*   **When** I open the job board
*   **Then** the job shows `2 slots left`

#### Scenario: Full and cancelled jobs are not shown as open
*   **Given** one job is full and another is cancelled
*   **When** I open the job board
*   **Then** neither appears in the **Open** list

#### Scenario: Anonymous visitor is blocked
*   **Given** I am not signed in
*   **When** I request the job board
*   **Then** the API returns `401`

### US-9.3 — Enroll in a job

#### Scenario: Member enrolls in an open job
*   **Given** I am signed in as `maria@example.com`
*   **And** job `Scorekeeper — Field 1` has `2` slots and `0` enrolled
*   **When** I click **Sign up**
*   **Then** the API returns `201` with `status` equal to `enrolled`
*   **And** the job shows `1 slot left`

#### Scenario: Duplicate enrollment is rejected
*   **Given** I am already enrolled in `Scorekeeper — Field 1`
*   **When** I click **Sign up** again
*   **Then** the API returns `409` with `{ "message": "You are already signed up for this job." }`

#### Scenario: Full job is rejected
*   **Given** a job has `1` slot and it is already taken
*   **When** I try to sign up
*   **Then** the API returns `409` with `{ "message": "This job is full." }`

#### Scenario: Overlapping job is rejected
*   **Given** I am enrolled in a job on `2026-11-14` from `09:00` to `10:00`
*   **When** I sign up for another job that day from `09:30` to `10:30`
*   **Then** the API returns `409` with `{ "message": "You have another job at that time." }`

#### Scenario: Job already started is rejected
*   **Given** a job started one hour ago
*   **When** I try to sign up
*   **Then** the API returns `400` with `{ "message": "This job has already started." }`

### US-9.4 — Withdraw from a job

#### Scenario: Member withdraws before the job starts
*   **Given** I am enrolled in a job that starts tomorrow
*   **When** I click **Withdraw** and confirm
*   **Then** my enrollment `status` is `withdrawn`
*   **And** the job's remaining slots increase by one

#### Scenario: Member cannot withdraw another person
*   **Given** another member is enrolled in a job
*   **When** I try to withdraw their enrollment
*   **Then** the API returns `403`

#### Scenario: Withdrawal after start is rejected
*   **Given** I am enrolled in a job that started one hour ago
*   **When** I try to withdraw
*   **Then** the API returns `400` with `{ "message": "You cannot withdraw after the job has started." }`

### US-9.5 — View job rosters

#### Scenario: Admin views a roster
*   **Given** I am signed in as an `admin`
*   **And** `maria@example.com` is enrolled in a job
*   **When** I open the job's roster
*   **Then** I see her name, email, and phone

#### Scenario: Member cannot view rosters
*   **Given** I am signed in as a `member`
*   **When** I request a job roster
*   **Then** the API returns `403`

#### Scenario: Admin cancels a job with enrollments
*   **Given** a job has 2 enrolled volunteers
*   **When** I click **Cancel job** and confirm
*   **Then** the job `status` is `cancelled`
*   **And** both enrollments have `status` equal to `cancelled`

#### Scenario: Reducing slots below enrollments is rejected
*   **Given** a job has 3 active enrollments
*   **When** I change total slots to `2`
*   **Then** the API returns `400` with `{ "message": "Total slots cannot be less than current enrollments." }`

### US-9.6 — View my volunteer schedule

#### Scenario: Member sees own jobs in date order
*   **Given** I am enrolled in jobs on `2026-11-14` and `2026-11-07`
*   **When** I open **My Jobs**
*   **Then** the `2026-11-07` job is listed before the `2026-11-14` job

#### Scenario: Member with no jobs
*   **Given** I have no active enrollments
*   **When** I open **My Jobs**
*   **Then** I see `You have not signed up for any jobs.`

---

## Screen Requirements

### Screen: Volunteer Job Board
**Route:** `/volunteer` | **Access:** Signed-in users

| Element | Details |
|---------|---------|
| Filters | Date range, Job type |
| Job cards | Title, type badge, date, time, location, `N slots left` |
| **Sign up** button | Disabled with label `Full` when no slots remain |
| Linked game | Shows teams and a link to the game if `gameId` is set |
| Empty state | "No open jobs right now." |

### Screen: My Jobs
**Route:** `/my/jobs` | **Access:** Signed-in users

| Element | Details |
|---------|---------|
| Enrollment list | Date, time, title, location, status, sorted by date |
| **Withdraw** | Visible only before the job starts; confirmation dialog |
| Empty state | "You have not signed up for any jobs." |

### Screen: Job Management
**Route:** `/admin/jobs` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Jobs table | Columns: Title, Type, Date, Time, Slots filled / total, Status |
| **Add job** | Form: title, type, description, optional game, date, start, end, location, slots |
| Row actions | Edit, View roster, Cancel job |

### Screen: Job Roster
**Route:** `/admin/jobs/:id/roster` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Roster table | Columns: Name, Email, Phone, Enrolled at, Status |
| Summary | `X of Y slots filled` |
