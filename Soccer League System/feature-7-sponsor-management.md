# Feature: Sponsor Management

**Feature ID:** 7
**Branch pattern:** `feature/7-sponsor-management`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Track league sponsors, their contact info and sponsorship level, and recognize them publicly on the league site and on the teams or fields they support.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md), [Feature 2 — Team Management](feature-2-team-management.md), [Feature 3 — Park Field Map](feature-3-park-field-map.md)

---

## User Stories

### US-7.1: Add a sponsor
**As a** league administrator
**I want to** record a sponsor's business name, contact person, email, phone, and logo
**So that** the league has one reliable sponsor contact list

**Priority:** P1
**Independent test:** Create a sponsor and see it in the admin sponsor list.
**Acceptance scenarios:** see ### US-7.1 under Acceptance Criteria

### US-7.2: Set sponsorship tier and term
**As a** league administrator
**I want to** set each sponsor's tier, pledged amount, and start and end dates
**So that** I can track commitments and renewals

**Priority:** P1
**Independent test:** Set tier `gold` with dates and see them saved.
**Acceptance scenarios:** see ### US-7.2 under Acceptance Criteria

### US-7.3: Recognize sponsors publicly
**As a** visitor
**I want to** see current sponsors and their logos
**So that** I know who supports the league

**Priority:** P1
**Independent test:** Open the sponsors page and see only sponsors whose term includes today.
**Acceptance scenarios:** see ### US-7.3 under Acceptance Criteria

### US-7.4: Link a sponsor to a team or field
**As a** league administrator
**I want to** attach a sponsor to a team or a field
**So that** "Sponsored by" credit shows where they are supporting

**Priority:** P2
**Independent test:** Link a sponsor to a team and see the credit on the team list.
**Acceptance scenarios:** see ### US-7.4 under Acceptance Criteria

### US-7.5: Identify expiring sponsorships
**As a** league administrator
**I want to** see sponsors whose terms end within 60 days
**So that** I can follow up on renewals

**Priority:** P3
**Independent test:** Create a sponsor ending in 30 days and see it in the expiring list.
**Acceptance scenarios:** see ### US-7.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Only an `admin` MUST be able to create, edit, link, or deactivate sponsors.
- **FR-002**: A sponsor MUST have a business name, contact name, and contact email.
- **FR-003**: Sponsor business name MUST be unique.
- **FR-004**: Contact email MUST be a valid email format.
- **FR-005**: Sponsor tier MUST be one of `bronze`, `silver`, `gold`, `platinum`.
- **FR-006**: Pledged amount MUST be 0.00 or more.
- **FR-007**: End date MUST be on or after start date.
- **FR-008**: Logo MUST be PNG or JPG and MUST NOT exceed 2 MB.
- **FR-009**: The public sponsors page MUST show only `active` sponsors whose term includes today, ordered platinum to bronze.
- **FR-010**: Contact name, email, phone, and pledged amount MUST NOT be returned to non-admin users.
- **FR-011**: A sponsor assignment MUST target exactly one team or one field.
- **FR-012**: The same sponsor MUST NOT be linked twice to the same target.
- **FR-013**: The expiring list MUST include active sponsors whose end date is within the next 60 days.

---

## Key Entities

- **Sponsor**: a business supporting the league, with contact, tier, term, and logo.
- **Sponsor Assignment**: a link between a sponsor and a team or field for recognition.
- **Team / Field**: the supported entities (Features 2 and 3).

---

## Initial Data Model

### `sponsors` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `businessName` | STRING(150) | Required, unique |
| `contactName` | STRING(150) | Required |
| `contactEmail` | STRING(255) | Required, valid email |
| `contactPhone` | STRING(20) | Optional |
| `websiteUrl` | STRING(255) | Optional |
| `logoUrl` | STRING(255) | Optional; PNG/JPG, max 2 MB |
| `tier` | ENUM(`bronze`,`silver`,`gold`,`platinum`) | Required |
| `pledgedAmount` | DECIMAL(10,2) | Required, >= 0 |
| `startDate` | DATE | Required |
| `endDate` | DATE | Required, >= `startDate` |
| `status` | ENUM(`active`,`inactive`) | Required, default `active` |

### `sponsor_assignments` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `sponsorId` | INTEGER FK | Required, references `sponsors.id` |
| `teamId` | INTEGER FK | Optional, references `teams.id` |
| `fieldId` | INTEGER FK | Optional, references `fields.id` |
| | | Exactly one of `teamId` / `fieldId` required; unique per sponsor + target |

---

## Acceptance Criteria

### US-7.1 — Add a sponsor

#### Scenario: Admin adds a sponsor
*   **Given** I am signed in as an `admin`
*   **When** I enter business `Sooner Pizza`, contact `Ana Ruiz`, email `ana@soonerpizza.com`, and click **Save**
*   **Then** the API returns `201` with the new sponsor

#### Scenario: Duplicate business name is rejected
*   **Given** a sponsor `Sooner Pizza` exists
*   **When** I add another sponsor named `Sooner Pizza`
*   **Then** the API returns `409` with `{ "message": "A sponsor with this name already exists." }`

#### Scenario: Oversized logo is rejected
*   **Given** I am signed in as an `admin`
*   **When** I upload a 4 MB logo
*   **Then** the API returns `400` with `{ "message": "Logo must be 2 MB or smaller." }`

### US-7.2 — Set sponsorship tier and term

#### Scenario: Admin sets tier and dates
*   **Given** sponsor `Sooner Pizza` exists
*   **When** I set tier `gold`, pledged `1500.00`, start `2026-08-01`, end `2027-07-31`
*   **Then** the API returns `200` with those values

#### Scenario: End date before start date is rejected
*   **Given** I am signed in as an `admin`
*   **When** I set start `2026-08-01` and end `2026-07-01`
*   **Then** the API returns `400` with `{ "message": "End date must be on or after start date." }`

#### Scenario: Invalid tier is rejected
*   **Given** I am signed in as an `admin`
*   **When** I submit tier `diamond`
*   **Then** the API returns `400` with `{ "message": "Tier must be bronze, silver, gold, or platinum." }`

### US-7.3 — Recognize sponsors publicly

#### Scenario: Visitor sees current sponsors by tier
*   **Given** active sponsors exist at `platinum` and `bronze` with terms including today
*   **When** I open the sponsors page without signing in
*   **Then** the platinum sponsor is listed before the bronze sponsor

#### Scenario: Expired sponsor is hidden
*   **Given** sponsor `Old Hardware` has an end date last month
*   **When** I open the sponsors page
*   **Then** `Old Hardware` is not shown

#### Scenario: Private sponsor details are not exposed
*   **Given** I am not signed in
*   **When** I request the public sponsor list
*   **Then** the response does not include `contactEmail`, `contactPhone`, or `pledgedAmount`

### US-7.4 — Link a sponsor to a team or field

#### Scenario: Admin links a sponsor to a team
*   **Given** sponsor `Sooner Pizza` and team `Blue Lightning` exist
*   **When** I link them
*   **Then** the API returns `201`
*   **And** the team list shows `Sponsored by Sooner Pizza`

#### Scenario: Duplicate link is rejected
*   **Given** `Sooner Pizza` is already linked to `Blue Lightning`
*   **When** I link them again
*   **Then** the API returns `409` with `{ "message": "This sponsor is already linked to that target." }`

#### Scenario: Link must target exactly one item
*   **Given** I am signed in as an `admin`
*   **When** I submit a link with both a team and a field
*   **Then** the API returns `400` with `{ "message": "Choose either a team or a field." }`

### US-7.5 — Identify expiring sponsorships

#### Scenario: Sponsor ending soon appears in the expiring list
*   **Given** sponsor `Sooner Pizza` has an end date 30 days from today
*   **When** I open **Expiring sponsors** as an `admin`
*   **Then** `Sooner Pizza` is listed

#### Scenario: Sponsor ending later is not listed
*   **Given** sponsor `Big Bank` has an end date 200 days from today
*   **When** I open **Expiring sponsors**
*   **Then** `Big Bank` is not listed

---

## Screen Requirements

### Screen: Our Sponsors (Public)
**Route:** `/sponsors` | **Access:** Public

| Element | Details |
|---------|---------|
| Tier sections | Platinum, Gold, Silver, Bronze headings |
| Sponsor card | Logo, business name, website link |
| Placeholder | Business name text when no logo |
| Empty state | "Sponsors coming soon." |

### Screen: Sponsor Management
**Route:** `/admin/sponsors` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Sponsors table | Columns: Business, Contact, Tier, Term, Status |
| **Add sponsor** | Form: business, contact name/email/phone, website, logo upload, tier, amount, start, end |
| Row actions | Edit, Link to team/field, Deactivate |
| **Expiring soon** tab | Sponsors ending within 60 days, with days remaining |

### Screen: Link Sponsor
**Route:** `/admin/sponsors/:id/links` | **Access:** `admin`

| Element | Details |
|---------|---------|
| Target type | Radio: Team or Field |
| Target dropdown | Lists teams or fields by selection |
| Existing links | Table with **Remove** action |
