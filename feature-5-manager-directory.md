# Feature: Manager Directory

**Feature ID:** 5
**Branch pattern:** `feature/5-manager-directory`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Provide a directory of team managers with email and phone contact info so parents, staff, and sponsors can reach the right person.
**Depends on:** [Feature 1 — User Authentication & Roles](feature-1-user-authentication-roles.md), [Feature 2 — Team Management](feature-2-team-management.md)

---

## User Stories

### US-5.1: View the manager directory
**As a** signed-in member
**I want to** see a list of managers with their teams and contact info
**So that** I can reach the right person with questions

**Priority:** P1
**Independent test:** Sign in as a member and see managers listed with name, team, and email.
**Acceptance scenarios:** see ### US-5.1 under Acceptance Criteria

### US-5.2: Update my own contact info
**As a** team manager
**I want to** edit my phone, contact email, and preferred contact method
**So that** people reach me the way I prefer

**Priority:** P1
**Independent test:** Change phone number and see the new value in the directory.
**Acceptance scenarios:** see ### US-5.2 under Acceptance Criteria

### US-5.3: Control my contact visibility
**As a** team manager
**I want to** hide my phone number from the directory
**So that** I can keep my number private while staying reachable by email

**Priority:** P2
**Independent test:** Hide phone and confirm a member no longer sees it.
**Acceptance scenarios:** see ### US-5.3 under Acceptance Criteria

### US-5.4: Search the directory
**As a** signed-in member
**I want to** search managers by name or team
**So that** I can find a specific manager quickly

**Priority:** P2
**Independent test:** Search `Lopez` and see only matching managers.
**Acceptance scenarios:** see ### US-5.4 under Acceptance Criteria

### US-5.5: Maintain any manager's record
**As a** league administrator
**I want to** edit any manager's contact info
**So that** the directory stays correct when managers change or are unreachable

**Priority:** P2
**Independent test:** Edit another manager's email as `admin` and see the change saved.
**Acceptance scenarios:** see ### US-5.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: The directory MUST only be viewable by signed-in users; anonymous requests MUST return `401`.
- **FR-002**: The directory MUST list every user with role `manager`, showing name, team names, contact email, and phone (when visible).
- **FR-003**: A manager MUST be able to edit only their own contact profile.
- **FR-004**: An `admin` MUST be able to edit any manager's contact profile.
- **FR-005**: Contact email MUST be a valid email format.
- **FR-006**: Phone MUST contain 10 digits when provided.
- **FR-007**: Preferred contact method MUST be one of `email`, `phone`, `text`.
- **FR-008**: When `showPhone` is false, the phone MUST NOT appear in responses to non-admin users.
- **FR-009**: Admins MUST always see all contact fields regardless of visibility settings.
- **FR-010**: Search MUST match manager first name, last name, or team name, case-insensitive.
- **FR-011**: A manager with no assigned team MUST still be listed with "No team assigned."
- **FR-012**: Each directory entry MUST include a link to email that manager.

---

## Key Entities

- **Manager Profile**: contact details and visibility preferences for one manager user.
- **User**: the manager's account (Feature 1).
- **Team**: the team(s) the manager runs (Feature 2).

---

## Initial Data Model

### `manager_profiles` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `userId` | INTEGER FK | Required, unique, references `users.id` (role must be `manager`) |
| `contactEmail` | STRING(255) | Required, valid email; defaults to `users.email` |
| `phone` | STRING(20) | Optional, 10 digits |
| `preferredContact` | ENUM(`email`,`phone`,`text`) | Required, default `email` |
| `showPhone` | BOOLEAN | Required, default `true` |
| `updatedAt` | DATETIME | Auto-set |

---

## Acceptance Criteria

### US-5.1 — View the manager directory

#### Scenario: Signed-in member views the directory
*   **Given** I am signed in as a `member`
*   **And** manager `Maria Lopez` manages `Blue Lightning`
*   **When** I open the manager directory
*   **Then** I see `Maria Lopez`, `Blue Lightning`, and her contact email

#### Scenario: Anonymous visitor is blocked
*   **Given** I am not signed in
*   **When** I request the manager directory
*   **Then** the API returns `401` with `{ "message": "Please sign in." }`

#### Scenario: Manager without a team is still listed
*   **Given** manager `Tom Green` has no team
*   **When** I open the directory
*   **Then** his row shows `No team assigned.`

### US-5.2 — Update my own contact info

#### Scenario: Manager updates own phone
*   **Given** I am signed in as manager `Maria Lopez`
*   **When** I change my phone to `4055550123` and click **Save**
*   **Then** the API returns `200` with the new phone
*   **And** the directory shows the updated phone

#### Scenario: Invalid contact email is rejected
*   **Given** I am signed in as manager `Maria Lopez`
*   **When** I enter contact email `maria-at-example`
*   **Then** the API returns `400` with `{ "message": "Enter a valid email address." }`

#### Scenario: Manager cannot edit another manager
*   **Given** I am signed in as manager `Maria Lopez`
*   **When** I try to edit the profile of `Tom Green`
*   **Then** the API returns `403`

### US-5.3 — Control my contact visibility

#### Scenario: Manager hides phone from members
*   **Given** I am signed in as manager `Maria Lopez` with a saved phone
*   **When** I turn off **Show phone in directory** and click **Save**
*   **Then** a signed-in `member` viewing the directory does not see her phone
*   **And** her email is still visible

#### Scenario: Admin still sees hidden phone
*   **Given** `Maria Lopez` has `showPhone` set to false
*   **When** I view the directory as an `admin`
*   **Then** her phone is visible

### US-5.4 — Search the directory

#### Scenario: Search by last name
*   **Given** managers `Maria Lopez` and `Tom Green` exist
*   **When** I search `lopez`
*   **Then** only `Maria Lopez` is shown

#### Scenario: Search by team name
*   **Given** `Maria Lopez` manages `Blue Lightning`
*   **When** I search `lightning`
*   **Then** `Maria Lopez` is shown

#### Scenario: Search with no matches
*   **Given** no manager matches `zzz`
*   **When** I search `zzz`
*   **Then** I see `No managers found.`

### US-5.5 — Maintain any manager's record

#### Scenario: Admin edits another manager's email
*   **Given** I am signed in as an `admin`
*   **When** I change `Tom Green`'s contact email to `tom.green@example.com` and click **Save**
*   **Then** the API returns `200` with the new email

---

## Screen Requirements

### Screen: Manager Directory
**Route:** `/managers` | **Access:** Signed-in users

| Element | Details |
|---------|---------|
| Search box | Placeholder "Search by name or team" |
| Directory table | Columns: Name, Team(s), Email, Phone, Preferred contact |
| **Email** link | `mailto:` link on each row |
| Phone cell | Shows "Hidden" for `admin` only when `showPhone` is false; blank for others |
| Empty state | "No managers found." |
| **Edit** button | Shown on own row for managers; on every row for `admin` |

### Screen: Edit Contact Info
**Route:** `/managers/:id/edit` | **Access:** Owning manager or `admin`

| Element | Details |
|---------|---------|
| Contact email | Email input, required |
| Phone | Text input with 10-digit mask |
| Preferred contact | Radio group: Email, Phone, Text |
| Show phone in directory | Toggle, default on |
| **Save** / **Cancel** | Save shows toast "Contact info updated." |
