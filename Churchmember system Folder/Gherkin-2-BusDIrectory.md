# Bus Directory

## Feature

The system shall allow users to view and search for buses in a directory.

## Acceptance Criteria

### Scenario: View the bus directory

**Given** the user is on the Bus Directory page  
**When** the user opens the directory  
**Then** the system should display a list of buses  
**And** each bus should display its bus number and available information

### Scenario: Search for a bus

**Given** the user is viewing the Bus Directory  
**When** the user searches for a bus by its bus number  
**Then** the system should display the matching bus

### Scenario: No matching bus is found

**Given** the user is viewing the Bus Directory  
**When** the user searches for a bus number that does not exist  
**Then** the system should display a message stating that no buses were found