
## User Stories

### US-1.1: Church Member Directory
**As a** church staff member  
**I want to** have a directory containing church member information  
**So that** easier to enter, find, and track church member information.

**Priority:** P1  
**Independent test:** Verify that a church staff member can add a church member and view their name, family name, address, and phone number in the directory.  
**Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

---

### US-1.1 — Church Member Directory

#### Scenario: Add a church member
*   **Given** church member directory is available
*   **When** the user enters a member's family name, name, address, and phone number
*   **Then** the system adds the church member to the directory
*   **And** the member's information can be viewed in the directory

#### Scenario: View a church member's information
*   **Given** a church member has been added to the directory
*   **When** user searches for or selects the church member
*   **Then** system displays the member's family name, name, address, and phone number

#### Scenario: Required information is missing
*   **Given** user is adding a church member
*   **When** user does not provide the required family name, address, or phone number
*   **Then** system must not save the incomplete member information
*   **And** system tells the user which required information is missing


