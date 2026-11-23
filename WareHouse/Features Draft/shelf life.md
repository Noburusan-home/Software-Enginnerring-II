# Feature: Expiration and Shelf-Life Tracking

**Feature ID:** 9  
**Branch pattern:** `Software-Enginnering-2-projects`  
**Status:** Draft  
**Created:** 2026-09-24   
**Input:** Warehouse requirement of identifying products which are close to their expiration date or have a shelf life

---

## 1. Overview

This feature allows warehouse employees to keep track of products with expiration dates or one which have shelf life.

---

## 2. User Stories

### Story 1: Record Expiration

**As a** warehouse employee,

**I want** to record expiration dates of products

**so that** I can know products which are expired

### Story 2: View Upcoming Expiration

**As a** warehouse manager

**I want**  to view products with approaching expiration dates

**so that**  I can take action on products about to expire

### Story 3: Prevent Expired Shipment

**As a** warehouse employee,

**I want** to be alerted of expired products

**so that** I can avoid shipping expired products

---

## 3. Functional Requirements

### FR-1: Expiration Date

The system shall record an expiration date when applicable.

### FR-2: Expiration Status

System shall record an expiration status.

### FR-3: Warning

The system shall warn warehouse employees of products that are close to expiry.

### FR-4: Expired Product

The system shall identify expired products.

### FR-5: Shipment Restriction

When applicable, the system shall restrict the shipment of expired products.

---

## 4. Edge Cases

- An product might not have an expiration date.
- A product might expire soon.
- A product might have already expired
- A products might have multiple expiration dates
- A product might not have a shelf life

---

## 5. Success Criteria

- Expiration dates shall be recorded.
- Employees shall be able to identify products which are about to expire.
- Products with an expiration date shall be identified as expired.
- Expired products shall be restricted from being shipped.

---

## 6. Acceptance Criteria

```gherkin
Given a product has an expiration date
When the expiration date is approaching
Then the system shall display an expiration warning