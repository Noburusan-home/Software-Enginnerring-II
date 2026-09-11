
### US-1.2: Bus Route Management
**As a** church staff member  
**I want to** track bus routes, drivers, substitute drivers, and timetables  
**So that** the church can organize and manage its bus transportation.

**Priority:** P1  
**Independent test:** Verify that a church staff member can be able to add a bus route with a driver, substitute driver, and timetable and view the information.  
**Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

---

### US-1.2 — Bus Route Management

#### Scenario: Add a bus route
*   **Given** bus route system is available
*   **When** user enters a route, driver, substitute driver, and timetable
*   **Then** system adds the bus route information
*   **And** route information can be viewed

#### Scenario: View bus route information
*   **Given** bus route has been added
*   **When** user searches for or selects the bus route
*   **Then** system displays the route, assigned driver, substitute driver, and timetable

#### Scenario: Update a bus driver
*   **Given** bus route has an assigned driver
*   **When** user changes the assigned driver or substitute driver
*   **Then** system updates the bus route information
*   **And** new driver information is displayed

#### Scenario: Missing required bus information
*   **Given** user is adding a bus route
*   **When** required route or timetable information is missing
*   **Then** system must not save the incomplete bus route
*   **And** system tells the user which required information is missing