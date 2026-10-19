# Feature: Warehouse Equipment and Device Management

**Feature ID:** 3  
**Branch pattern:** `Software-Enginnering-2-projects`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Warehouse requirement for devices and equipment used to complete warehouse operations

---

## 1. Overview

The Equipment and Device Management feature will allow the warehouse to keep track of equipment used during operations.

Equipment can include scanners, computers, tablets, printers, scales, and other devices.

---

## 2. User Stories

### Story 1: View Equipment

**As a** warehouse employee,

**I want** view equipment

**so that** I know which equipment I can use.

### Story 2: Report Broken Equipment

**As a** warehouse employee,

**I want** report broken equipment

**so that**  it is taken out of use.

### Story 3: Track Equipment

**As a** warehouse manager,

**I want** to know the status of equipment,

**so that** I know what equipment is available.

---

## 3. Functional Requirements

### FR-1: Equipment Record

The system shall maintain equipment records.

### FR-2: Equipment Types

The system shall support equipment such as:

- Barcode scanners
- Tablets
- Computers
- Label printers
- Scales
- Inspection devices

### FR-3: Equipment Status

Equipment shall have a status such as:

- Available
- In Use
- Maintenance
- Broken
- Out of Service

### FR-4: Equipment Assignment

The system shall record the employee using equipment if assignment tracking is required.


### FR-5: Equipment Issue

Employees shall be able to report equipment issues.

### FR-6: Maintenance

Authorized users shall be able to mark equipment as being repaired or maintained.

---

## 4. Edge Cases

- A scanner breaks down while an order is being picked
- Equipment is already assigned.
- Equipment is missing.
- Equipment requires maintenance.
- A backup device is needed.

---

## 5. Success Criteria

- Equipment can be identified.
- Equipment status is visible.
- Broken equipment can be reported.
- Equipment can be assigned.
- Employees know when equipment is unavailable.

---

## 6. Acceptance Criteria

### Scenario 1: Report Broken equipment

```gherkin
Given an employee is using a scanner
When the scanner breaks down
Then the employee can report the scanner as broken
And the system shall change its status to Broken