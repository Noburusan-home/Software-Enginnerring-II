# Feature: Recieving inspection 

**Feature ID:** 8  
**Branch pattern:** `Software-Enginnering-2-projects`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Requirement of warehouse to make inspection of products arriving to them

---

## 1. Overview

The purpose of the feature Product Receiving & Inspection is to give employees at the warehouse possibility to check products coming in the warehouse before adding those products to available inventory

---

## 2. User Stories

### Story 1: Receive Shipment

**As a** warehouse employee

**I want** to record an incoming shipment

**so that** the warehouse will know about arrival of products

### Story 2: Check Quantity

**As a** warehouse employee

**I want**  to check received quantity against expected quantity

**so that** so that receiving exception will be recorded

### Story 3: Inspect Product

**As a** warehouse employee

**I want** to inspect received products

**so that** damaged products will not go as normal available inventory

---

## 3. Functional Requirements

### FR-1: Shipment Identification

The system shall identify the shipment that is coming

### FR-2: Product Verification

The employee shall verify the product that is received

### FR-3: Quantity Verification

The employee shall verify the received quantity

### FR-4: Damage Inspection

The employee shall report the damage of the product

### FR-5: Receiving Status

The shipment will have a status indicating current state of receiving. The status shall be one of:

- Expected
- Arrived
- Inspecting
- Received
- Exception

### FR-6: Inventory

The products shall not be added to normal available inventory before required inspection of the products is completed

---

## 4. Edge Cases

- The shipment has lesser quantity of products than expected
- The shipment has higher quantity of products than expected
- Product is received damaged
- Wrong product is received
- Shipment was made without any documentation
- The product was impossible to identify

---

## 5. Success Criteria

- Incoming shipments can be entered to the system
- Products will be checked before entering to the inventory
- Quantity of products will be checked
- Products will be examined for damage
- Records of receiving products will be kept

---

## 6. Acceptance Criteria

```gherkin
Given a shipment is expected
When the shipment arrives
Then the employee shall be able to mark it as Arrived
And start inspecting the shipment