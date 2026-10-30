# Feature: User Authentication & Roles

**Feature ID:** 1
**Branch pattern:** `feature/1-user-authentication-roles`
**Status:** Draft
**Created:** 2026-10-29
**Input:** Let league staff, team managers, and volunteers create accounts, sign in, and only reach the parts of the Soccer League System their role allows.

---

## User Stories

### US-1.1: Register an account
**As a** new league member
**I want to** register with my name, email, and password
**So that** I can sign up for volunteer jobs and see member-only league information

**Priority:** P1
**Independent test:** Submit the registration form with valid data and confirm a new `member` account exists.
**Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

### US-1.2: Sign in
**As a** registered user
**I want to** sign in with my email and password
**So that** I can use the features that belong to my role

**Priority:** P1
**Independent test:** Sign in with known credentials and receive a session token plus a redirect to the home page.
**Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

### US-1.3: Sign out
**As a** signed-in user
**I want to** sign out
**So that** nobody else can use my account on a shared device

**Priority:** P1
**Independent test:** Sign out and confirm the old token is rejected on the next request.
**Acceptance scenarios:** see ### US-1.3 under Acceptance Criteria

### US-1.4: Restrict pages by role
**As the** application
**I want to** block users from screens and actions outside their role
**So that** only authorized people can change league data

**Priority:** P1
**Independent test:** Call an admin-only action as a `member` and receive `403`.
**Acceptance scenarios:** see ### US-1.4 under Acceptance Criteria

### US-1.5: Assign user roles
**As a** league administrator
**I want to** change a user's role
**So that** new team managers and staff get the right access

**Priority:** P2
**Independent test:** Promote a `member` to `manager` and confirm the change is saved.
**Acceptance scenarios:** see ### US-1.5 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Users MUST register with first name, last name, email, and password.
- **FR-002**: Email MUST be unique and stored in lowercase.
- **FR-003**: Passwords MUST be at least 8 characters and MUST be stored only as a bcrypt hash.
- **FR-004**: New accounts MUST be created with the role `member`.
- **FR-005**: Users MUST sign in with email and password.
- **FR-006**: A successful sign in MUST return a session token, `userId`, `email`, and `role`.
- **FR-007**: Failed sign in MUST return the same message whether the email or password was wrong.
- **FR-008**: Sign out MUST invalidate the session token.
- **FR-009**: The system MUST support exactly three roles: `admin`, `manager`, `member`.
- **FR-010**: Requests without a valid token to protected actions MUST return `401`.
- **FR-011**: Requests with a valid token but insufficient role MUST return `403`.
- **FR-012**: Only an `admin` MUST be able to change another user's role.
- **FR-013**: An admin MUST NOT be able to remove their own `admin` role.

---

## Key Entities

- **User**: a registered account with one role; may later own volunteer enrollments or manage a team.
- **Session**: server-side record tying a login token to one user until it expires or the user signs out.

---

## Initial Data Model

### `users` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `firstName` | STRING(100) | Required |
| `lastName` | STRING(100) | Required |
| `email` | STRING(255) | Required, unique, stored lowercase |
| `phone` | STRING(20) | Optional |
| `passwordHash` | STRING(255) | Required; bcrypt hash only |
| `role` | ENUM(`admin`,`manager`,`member`) | Required, default `member` |
| `createdAt` | DATETIME | Auto-set |

### `sessions` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `token` | STRING(255) | Required, unique |
| `userId` | INTEGER FK | Required, references `users.id` |
| `expiresAt` | DATETIME | Required `[NEEDS CLARIFICATION: session lifetime, e.g. 24 hours?]` |
| `createdAt` | DATETIME | Auto-set |

---

## Acceptance Criteria

### US-1.1 — Register an account

#### Scenario: New member registers with valid data
*   **Given** I am on the registration page
*   **When** I enter first name `Maria`, last name `Lopez`, email `maria@example.com`, and password `Goal2026!`
*   **And** I click **Create account**
*   **Then** the API returns `201` with `userId` and `role` equal to `member`
*   **And** the stored password is a bcrypt hash, not `Goal2026!`

#### Scenario: Registration with a duplicate email
*   **Given** a user with email `maria@example.com` exists
*   **When** I register using `Maria@Example.com`
*   **Then** the API returns `409` with `{ "message": "An account with this email already exists." }`

#### Scenario: Registration with a short password
*   **Given** I am on the registration page
*   **When** I enter a valid email and the password `abc123`
*   **Then** the API returns `400` with `{ "message": "Password must be at least 8 characters." }`

### US-1.2 — Sign in

#### Scenario: User signs in with valid credentials
*   **Given** a registered user exists with email `maria@example.com` and a known password
*   **When** I enter the correct email and password
*   **And** I click **Sign in**
*   **Then** the API returns `200` with `userId`, `email`, `token`, and `role`
*   **And** I am redirected to the home page

#### Scenario: User signs in with an invalid password
*   **Given** a registered user exists with email `maria@example.com`
*   **When** I enter that email and an incorrect password
*   **Then** the API returns `401` with `{ "message": "Invalid email or password." }`
*   **And** I remain on the sign-in page

#### Scenario: User signs in with an unknown email
*   **Given** no user exists with email `nobody@example.com`
*   **When** I enter `nobody@example.com` and any password
*   **Then** the API returns `401` with `{ "message": "Invalid email or password." }`

### US-1.3 — Sign out

#### Scenario: Signed-in user signs out
*   **Given** I am signed in as `maria@example.com`
*   **When** I click **Sign out**
*   **Then** my session token is invalidated
*   **And** I am redirected to the sign-in page

#### Scenario: Old token is rejected after sign out
*   **Given** I signed out and kept my previous token
*   **When** I call a protected action with that token
*   **Then** the API returns `401`

### US-1.4 — Restrict pages by role

#### Scenario: Unauthenticated request to a protected action
*   **Given** I have no session token
*   **When** I call an action that requires sign in
*   **Then** the API returns `401` with `{ "message": "Please sign in." }`

#### Scenario: Member attempts an admin-only action
*   **Given** I am signed in with role `member`
*   **When** I call an admin-only action
*   **Then** the API returns `403` with `{ "message": "You do not have permission to do that." }`

#### Scenario: Admin-only menu items are hidden from members
*   **Given** I am signed in with role `member`
*   **When** I view the main navigation
*   **Then** I do not see the **Admin** menu item

### US-1.5 — Assign user roles

#### Scenario: Admin promotes a member to manager
*   **Given** I am signed in as an `admin`
*   **And** a user `maria@example.com` has role `member`
*   **When** I change her role to `manager` and click **Save**
*   **Then** the API returns `200` with `role` equal to `manager`

#### Scenario: Admin cannot remove their own admin role
*   **Given** I am signed in as an `admin`
*   **When** I try to change my own role to `member`
*   **Then** the API returns `400` with `{ "message": "You cannot remove your own admin role." }`

#### Scenario: Non-admin cannot change roles
*   **Given** I am signed in as a `manager`
*   **When** I try to change another user's role
*   **Then** the API returns `403`

---

## Screen Requirements

### Screen: Registration
**Route:** `/register` | **Access:** Public

| Element | Details |
|---------|---------|
| First name, Last name | Text inputs, required |
| Email | Email input, required |
| Password | Password input with show/hide toggle, required, min 8 characters |
| **Create account** button | Disabled until required fields are filled |
| Link | "Already have an account? Sign in" |
| Error area | Inline alert showing the exact API message |

### Screen: Sign In
**Route:** `/login` | **Access:** Public

| Element | Details |
|---------|---------|
| Email, Password | Required inputs |
| **Sign in** button | Shows loading state while request is pending |
| Link | "Need an account? Register" |
| Error area | Alert showing `"Invalid email or password."` on failure |

### Screen: User Management (Admin)
**Route:** `/admin/users` | **Access:** `admin` only

| Element | Details |
|---------|---------|
| Users table | Columns: Name, Email, Role, Created |
| Role dropdown | Per row: `admin`, `manager`, `member` |
| **Save** action | Per row; success toast "Role updated." |
| Search box | Filters by name or email |

### Global: Navigation Bar
- Shows **Sign in** / **Register** when signed out.
- Shows user's name and **Sign out** when signed in.
- Shows **Admin** menu only for `admin`.
