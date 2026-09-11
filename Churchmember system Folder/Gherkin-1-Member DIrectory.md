# Church Member Directory

## Feature

The system shall allow users to view and search for church members in a directory.

## Acceptance Criteria

### Scenario: View the church member directory

**Given** the user is on the Church Member Directory page  
**When** the user opens the directory  
**Then** the system should display a list of church members  
**And** each member should display their name and contact information

### Scenario: Search for a church member

**Given** the user is viewing the Church Member Directory  
**When** the user searches for a member by name  
**Then** the system should display the matching church member

### Scenario: No matching member is found

**Given** the user is viewing the Church Member Directory  
**When** the user searches for a member who does not exist  
**Then** the system should display a message stating that no members were found