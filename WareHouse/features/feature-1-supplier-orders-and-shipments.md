# Feature: Supplier Orders and Shipments

**Feature ID:** 1  
**Branch pattern:** `feature/1-supplier-orders-and-shipments`  
**Status:** Draft  
**Created:** 2026-09-24  
**Input:** Manage supplier orders and shipment information so that warehouse employees can record, track, and process orders and shipments.  
**Depends on:** None  
**Related:** Bill of Lading; Supplier Order Form; [architecture.md](./architecture.md)

---

## User Stories

### US-1.1: Create a Supplier Order
**As a** warehouse employee  
**I want to** create a supplier order that includes the supplier, customer number, order date, P.O. number, and product information  
**So that** the warehouse has a record of the products it ordered from the supplier

**Priority:** P1  
**Independent test:** Create an order with valid supplier and product information and verify the saved order matches what was entered  
**Acceptance scenarios:** see ### US-1.1 under Acceptance Criteria

### US-1.2: Record Shipment Information
**As a** warehouse employee  
**I want to** record the source, destination, customer number, P.O. number, driver, and ship date of a shipment  
**So that** the warehouse can track outgoing shipments

**Priority:** P1  
**Independent test:** Create a shipment with valid Bill of Lading information and verify it is saved  
**Acceptance scenarios:** see ### US-1.2 under Acceptance Criteria

### US-1.3: Record Products
**As a** warehouse employee  
**I want to** include the product's SKU, description, quantity or cases, price, and total with an order or shipment  
**So that** the warehouse knows what products are included in an order or shipment

**Priority:** P1  
**Independent test:** Add a product to an order or shipment and verify the line is saved with that document  
**Acceptance scenarios:** see ### US-1.3 under Acceptance Criteria

### US-1.4: Record Delivery and Receiving Information
**As a** warehouse employee  
**I want to** record the delivery date and who received the shipment  
**So that** the warehouse has the date and person who received the shipment

**Priority:** P1  
**Independent test:** Provide a delivery date and receiver on an existing shipment and verify those fields persist  
**Acceptance scenarios:** see ### US-1.4 under Acceptance Criteria

---

## Functional Requirements

- **FR-001**: Warehouse employees MUST be able to create a supplier order.
- **FR-002**: A supplier order MUST include the supplier or manufacturer.
- **FR-003**: A supplier order MUST include the customer number.
- **FR-004**: A supplier order MUST include the order date.
- **FR-005**: A supplier order MUST include a P.O. number.
- **FR-006**: Products MUST be addable to a supplier order.
- **FR-007**: Each product line MUST include a SKU, description, number of cases, price, and total.
- **FR-008**: Warehouse employees MUST be able to create shipment records.
- **FR-009**: A shipment MUST include where the products are being shipped to.
- **FR-010**: A shipment MUST include where the products are being shipped from.
- **FR-011**: A shipment MUST include the customer number.
- **FR-012**: A shipment MUST include the date it was shipped.
- **FR-013**: A shipment MUST include the P.O. number.
- **FR-014**: A shipment MUST include the driver.
- **FR-015**: Products on a shipment MUST include a SKU, description, quantity, price, and total.
- **FR-016**: A shipment MUST allow a delivery date once it has been delivered.
- **FR-017**: A shipment MUST allow recording who received it.
- **FR-018**: A supplier order MUST allow recording who authorized the order.
- **FR-019**: The order date MUST be persisted with the supplier order.
- **FR-020**: Required information MUST be entered before an order or shipment can be saved.

---

## Assumptions

- Warehouse employees use this feature to keep track of supplier orders and shipments.
- Each supplier order is connected to a supplier or manufacturer.
- A product can be identified by its SKU.
- The P.O. number can associate an order with a shipment.
- Fields on the Supplier Order Form and Bill of Lading are represented in this feature's data models.
- Signed-in employee identity is available (`req.user.id`); login/register UI is out of scope.
- Inventory on-hand updates, warehouse bin tracking, payment processing, and carrier booking are out of scope.

---

## Edge Cases

- Missing required information → do not save; tell the employee what is missing.
- Duplicate P.O. number → warn; do not silently overwrite another document.
- Invalid quantity → quantity/cases MUST be greater than zero.
- Invalid price → MUST be a valid monetary value.
- Missing SKU → product line MUST NOT save.
- Missing destination → shipment MUST NOT save.
- Missing driver when required → identify the missing driver.
- Shipment has not arrived → delivery and receiving fields MAY remain blank.
- Missing P.O. number when connecting a shipment to an order → identify the missing P.O. number.

---

## Success Criteria

- **SC-001**: Every Gherkin scenario has at least one automated test before merge.
- **SC-002**: Employees can create and save a supplier order with supplier, customer number, order date, P.O. number, and product information.
- **SC-003**: Employees can create and save a shipment with source, destination, customer number, ship date, P.O. number, and driver.
- **SC-004**: Employees can add products to an order or shipment using SKU, description, quantity or cases, price, and total.
- **SC-005**: Employees can record the delivery date and who received a shipment.

---

## Key Entities

- **Supplier Order:** An order placed with a supplier or manufacturer. Contains supplier, customer number, order date, P.O. number, authorization, and products.
- **Shipment:** Products moving from one location to another. Contains source, destination, customer number, ship date, P.O. number, driver, delivery date, and receiving information.
- **Product line:** An item on an order or shipment, identified by SKU, with description, quantity or cases, price, and total.
- **Customer:** Connected through customer number.
- **Supplier/Manufacturer:** The company supplying the products.
- **Driver:** Person transporting the shipment.
- **Address:** Source or destination of a shipment.

---

## Initial Data Model

### `supplier_orders` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `supplier_name` | VARCHAR | Required |
| `supplier_address` | VARCHAR | Required |
| `customer_number` | VARCHAR | Required |
| `order_date` | DATE | Required |
| `po_number` | VARCHAR | Required |
| `authorized_by` | VARCHAR | Optional until the order is authorized |
| `createdBy` | INTEGER FK | Set from `req.user.id` on create |

### `order_products` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `supplierOrderId` | INTEGER FK | Required; references `supplier_orders.id` |
| `sku` | VARCHAR | Required |
| `description` | VARCHAR | Required |
| `cases` | INTEGER | Required; MUST be greater than zero |
| `price` | DECIMAL | Required; valid money |
| `total` | DECIMAL | Calculated or entered total |

### `shipments` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `customer_number` | VARCHAR | Required |
| `shipped_date` | DATE | Required |
| `po_number` | VARCHAR | Required |
| `driver_name` | VARCHAR | Required |
| `shipped_from` | VARCHAR | Required |
| `shipped_to` | VARCHAR | Required |
| `delivery_date` | DATE | Optional until delivery |
| `received_by` | VARCHAR | Optional until received |
| `createdBy` | INTEGER FK | Set from `req.user.id` on create |

### `shipment_products` table

| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `shipmentId` | INTEGER FK | Required; references `shipments.id` |
| `sku` | VARCHAR | Required |
| `description` | VARCHAR | Required |
| `quantity` | INTEGER | Required; MUST be greater than zero |
| `price` | DECIMAL | Required; valid money |
| `total` | DECIMAL | Calculated or entered total |

### Associations

- A supplier order has many order products.
- A shipment has many shipment products.
- A shipment MAY be connected to a supplier order using `po_number`.

---

## Gherkin AC

### US-1.1 — Create a Supplier Order

#### Scenario: Create a valid supplier order
*   **Given** I am a signed-in warehouse employee creating a new supplier order
*   **When** I enter the supplier, customer number, order date, P.O. number, and product information
*   **And** I save the order
*   **Then** the API returns `201` with the saved supplier order
*   **And** the saved order contains the information that was entered

#### Scenario: Required information is missing
*   **Given** I am creating a new supplier order
*   **When** I try to save the order without required information
*   **Then** the API returns `400` with a message naming the missing information
*   **And** the system does not save the order

#### Scenario: Duplicate P.O. number is used
*   **Given** a supplier order already exists with P.O. number `PO-1001`
*   **When** I try to save another supplier order with P.O. number `PO-1001`
*   **Then** the API returns `400` with `{ "message": "P.O. number is already in use." }`
*   **And** the new order is not saved

---

### US-1.2 — Record Shipment Information

#### Scenario: Create a valid shipment
*   **Given** I am a signed-in warehouse employee creating a shipment
*   **When** I enter the source, destination, customer number, ship date, P.O. number, and driver
*   **And** I save the shipment
*   **Then** the API returns `201` with the saved shipment
*   **And** the shipment contains the information that was entered

#### Scenario: Required shipment information is missing
*   **Given** I am creating a shipment
*   **When** I try to save the shipment without required information
*   **Then** the API returns `400` with a message naming the missing information
*   **And** the system does not save the shipment

---

### US-1.3 — Record Products

#### Scenario: Add a product to an order
*   **Given** supplier order `PO-1001` already exists
*   **When** I enter SKU, description, cases, price, and total
*   **And** I save the product line
*   **Then** the API returns `201` with the product line
*   **And** the product can be viewed with the order

#### Scenario: Product information is missing
*   **Given** I am adding a product
*   **When** I try to save the product without a SKU
*   **Then** the API returns `400` with `{ "message": "SKU is required." }`
*   **And** the product is not saved

#### Scenario: Invalid quantity is entered
*   **Given** I am adding a product to a shipment
*   **When** I enter quantity `0` or a negative number
*   **Then** the API returns `400` with `{ "message": "Quantity must be greater than zero." }`

---

### US-1.4 — Record Delivery and Receiving Information

#### Scenario: Record a completed delivery
*   **Given** a shipment has already been created
*   **When** I enter the delivery date and who received the shipment
*   **And** I save
*   **Then** the API returns `200` with the updated shipment
*   **And** the shipment shows the delivery and receiving information

#### Scenario: Shipment has not been received
*   **Given** a shipment has been created but has not arrived yet
*   **When** I view the shipment
*   **Then** the delivery date and received-by fields can remain empty
*   **And** the shipment can be updated later when it arrives

---

## Data Ownership

Operational order and shipment records are **shared** among signed-in warehouse employees.

| Rule | Requirement |
|------|-------------|
| **Read scope** | Signed-in employees MAY list and view all supplier orders and shipments in this warehouse |
| **Write scope** | Signed-in employees MAY create and update orders, shipments, and product lines |
| **Create scope** | `createdBy` is set from `req.user.id`; ignore `createdBy` in the body |
| **Cross-user access** | Another employee's records are visible (shared warehouse). Unknown id → `404` |
| **UI scope** | Screens show only records returned by the API |
| **Unauthenticated** | `401` |

---

## API Requirements

| Method | Endpoint | Auth | Purpose |
|--------|----------|------|---------|
| `GET` | `/warehouse/orders` | Yes | List supplier orders |
| `POST` | `/warehouse/orders` | Yes | Create a supplier order |
| `GET` | `/warehouse/orders/:orderId` | Yes | Get one supplier order with product lines |
| `PUT` | `/warehouse/orders/:orderId` | Yes | Update header fields (including `authorized_by`) |
| `POST` | `/warehouse/orders/:orderId/products` | Yes | Add a product line to an order |
| `GET` | `/warehouse/shipments` | Yes | List shipments |
| `POST` | `/warehouse/shipments` | Yes | Create a shipment |
| `GET` | `/warehouse/shipments/:shipmentId` | Yes | Get one shipment with product lines |
| `PUT` | `/warehouse/shipments/:shipmentId` | Yes | Update shipment, including delivery/receiving |
| `POST` | `/warehouse/shipments/:shipmentId/products` | Yes | Add a product line to a shipment |

**Create order request body:**
```json
{
  "supplier_name": "Acme Supply",
  "supplier_address": "100 Industrial Rd",
  "customer_number": "C-22",
  "order_date": "2026-11-01",
  "po_number": "PO-1001"
}
```

**Error response:** `{ "message": "Human-readable explanation." }`  
**Not found:** `404`. **Validation:** `400`.

---

## Screen Requirements

### [View: Supplier Orders] — route name `orders`
*   Heading: **Supplier Orders**
*   Primary action: **+ New Order**
*   Fields: supplier, supplier address, customer number, order date, P.O. number, authorized by
*   Product lines table: SKU, description, cases, price, total; action **Add product**
*   **Empty state:** `"No supplier orders yet. Create your first order."`
*   **Loading / error:** progress while loading; `<v-alert type="error">` on failure

### [View: Shipments] — route name `shipments`
*   Heading: **Shipments**
*   Primary action: **+ New Shipment**
*   Fields: shipped from, shipped to, customer number, ship date, P.O. number, driver, delivery date, received by
*   Product lines: SKU, description, quantity, price, total
*   **Empty state:** `"No shipments yet. Create your first shipment."`

---

## Definition of Done

*   [ ] Backend and frontend implemented per this spec (**FR-00N** satisfied)
*   [ ] **Success Criteria (SC-00N)** met
*   [ ] All Gherkin scenarios have automated tests and `npm test` passes
*   [ ] `features/reference/data-model.md` updated (if schema changed)
*   [ ] `features/reference/api.md` updated (if API changed)
*   [ ] `features/reference/behavior.md` updated (if product rules changed)

---

## Out of Scope

*   User registration and login UI
*   On-hand inventory adjustments ([Feature 5](./feature-5-inventory-accuracy.md))
*   Warehouse bin assignment ([Feature 10](./feature-10-warehouse-location-management.md))
*   Incoming receiving inspection workflow ([Feature 8](./feature-8-receiving-inspection.md))
*   Payment handling and carrier booking
