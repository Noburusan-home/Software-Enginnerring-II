# Feature: <Human-readable title>

**Feature ID:** N  
**Branch pattern:** `feature/N-short-name`  
**Status:** Draft  
**Created:** YYYY-MM-DD  
**Input:** One sentence — what this feature is for (intent, not technology)  
**Depends on:** [Feature X — …](feature-X-….md)  
**Related:** optional links to ADRs or reference docs  

---

## User Stories

### US-N.1: Short title
**As a** <role>  
**I want to** <capability>  
**So that** <benefit>

**Priority:** P1  
**Independent test:** <how to verify this story alone, in one sentence>  
**Acceptance scenarios:** see ### US-N.1 under Acceptance Criteria

### US-N.2: Short title
**As a** <role>  
**I want to** <capability>  
**So that** <benefit>

**Priority:** P1  
**Independent test:** <how to verify this story alone, in one sentence>  
**Acceptance scenarios:** see ### US-N.2 under Acceptance Criteria

---

## Requirements

### Functional Requirements

- **FR-001**: System MUST …
- **FR-002**: Users MUST be able to …
- **FR-003**: … MUST NOT …

---

## Assumptions

- What already exists (e.g. Feature 1 auth is on `dev`)
- What you are deliberately not building yet

---

## Edge Cases

- Empty required field → …
- Cross-user access → …
- Duplicate / invalid input → …

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge
- **SC-002**: <measurable outcome for this feature>

---

## Key Entities

- **Entity**: short description; relationships in plain language
- **Entity**: …

---

## Data Model Requirements

### `table_name` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `…` | … | … |

### Associations (if known)
- …

---

## Acceptance Criteria

### US-N.1 — Short title (same as the story)

#### Scenario: Descriptive name (happy path)
*   **Given** <starting state>
*   **When** <action>
*   **Then** <observable result>
*   **And** <extra result if needed>

#### Scenario: Descriptive name (failure / edge)
*   **Given** …
*   **When** …
*   **Then** …

### US-N.2 — Short title (same as the story)

#### Scenario: Descriptive name (happy path)
*   **Given** …
*   **When** …
*   **Then** …

#### Scenario: Descriptive name (failure / edge)
*   **Given** …
*   **When** …
*   **Then** …
