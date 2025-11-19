# Lizzy API Field Mapping Document

**To:** Lizzy Technical Team  
**From:** Jonathan (Lizzy Agentic Commerce Project)  
**Date:** November 17, 2025  
**Version:** 1.2.0
**Status:** Draft - In Review
**Purpose:** Map API requirements to Lizzy database schema

-----

## Changelog (v1.2.0)

### Changed
- Field name model to model name in Equipment Root Fields list
- Updated answers for Critical Questions under Table 2.1: Service Order Fields
- Answered questions under Table 2.2: Service Order Invoice Fields and moved under Critical Questions Section under Table 2.1: Service Order Fields
- In Parts Order Fields section, replaced equipment_name with Make, Model Name and Model Number

### Added
- Model Number to Equipment Root Fields list
- multiple Lizzy App field names and comments with sample data to Equipment Root Fields list
- Type field to Service Order Fields
- Status field to Service Order Fields
- processed_date field to Service Order Fields

### Removed
- Status field from Equipment Root Fields list (applies to orders, not equipment)
- vin field from Equipment Root Fields (same as SN)
- purchase_date field from Equipment Root Fields (applies to orders, not equipment)
- last_service_date field from Equipment Root Fields (applies to orders, not equipment)
- Entire section Table 2.2: Service Order Invoice Fields (there are not separate entities for service orders and service invoices)

-----

## Instructions for Lizzy Team

This document contains **detailed field mapping tables** for each API endpoint. Please fill in the columns to help us understand your data model:

- **Column B:** What you call this field in your app/UI
- **Column C:** Navigation path in Lizzy app (e.g., "Customer → Equipment → Details")
- **Column D:** Actual database table and column name (e.g., "customers.phone_number")
- **Column E:** Any comments, notes, or clarifications

**Please complete all tables and return to us within 1 week.**

-----

## Table of Contents

1. [API 1: Get Customer Profile](#api-1-get-customer-profile)
2. [API 2: Get Customer Transactions](#api-2-get-customer-transactions)
3. [API 3: Create Service Order](#api-3-create-service-order)
4. [API 4: Create Parts Order](#api-4-create-parts-order)
5. [API 5: Log SMS Message](#api-5-log-sms-message)
6. [API 6: Flag Conversation](#api-6-flag-conversation)
7. [API 7: Get Conversation Status](#api-7-get-conversation-status)
8. [Data Type Reference](#data-type-reference)

-----

## API 1: Get Customer Profile

**Endpoint:** `GET /api/customers/by-phone/{phone}`

**Purpose:** Retrieve complete customer profile with equipment, addresses, resources, and parts catalog

### Table 1.1: Customer Profile Root Fields

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| customer_id | | | | Unique customer identifier |
| name | | | | Customer full name |
| email | | | | Customer email address |
| phone | | | | Customer phone number |

**Questions:**
1. What format is `phone` stored in your database? (E.164, national, other?)
2. Is `customer_id` a numeric ID, UUID, or string?
3. Is `name` stored as single field or separate first/last name fields?

---

### Table 1.2: Customer Address Fields

**Path in response:** `addresses.primary`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| addresses (object) | | | | Container for all addresses |
| addresses.primary (object) | | | | Primary/default address |
| addresses.primary.street | | | | Street address line 1 |
| addresses.primary.city | | | | City name |
| addresses.primary.state | | | | State abbreviation (e.g., "WA") |
| addresses.primary.zip | | | | ZIP or postal code |
| addresses.additional (array) | | | | Additional addresses (if any) |

**Questions:**
1. Do you support multiple addresses per customer?
2. What distinguishes "primary" address from others?
3. Is there a separate "street2" field for apt/suite numbers?
4. Do you store shipping vs billing addresses separately?

---

### Table 1.3: Equipment Root Fields

**Path in response:** `equipment[]` (array of equipment objects)

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| equipment_id | | | | Unique equipment identifier |
| make | Make | | | Manufacturer (e.g., "Honda") |
| model name | Model Name | | | Model name (e.g., "HRX217") |
| model number | Model Number | | | Model number (e.g., "HRX217K6VKA") |
| sku | stock number | | | dealer assigned value (e.g., "143 NCLC") |
| unit_type | unit category | | | 40 dealer-specific values (e.g., "Walk Behind Mower", "Zero-Turn"), else Lizzy defaults values as last 5 digits of SN+date received |
| serial_number | vin/serial# | | | Serial number for this unit |

**Critical Questions:**

1. **unit_type field:**
   - What do you call this field in Lizzy?
   - What are ALL possible values? (Mower, Chainsaw, Tractor, ATV, etc?)
   - Is this a free text field or controlled list?
   - Can we get a complete list of values?

2. **SKU field:**
   - Do you have a SKU field distinct from model number?
   - Can customers search equipment by SKU in your system?
   - Is this field always populated?

3. **status field:**
   - What are ALL possible status values?
   - We assume: "Active", "In Service", "Inactive" - correct?
   - Are there others?
   - What triggers status changes?

4. **serial_number:**
   - Always populated?
   - Unique across all equipment?
   - Searchable in your system?

---

### Table 1.4: Equipment Resources Fields

**Path in response:** `equipment[].resources`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| resources (object) | | | | Container for resource URLs |
| resources.tutorial_video_url | | | | URL to tutorial/demo video |
| resources.owners_manual_url | | | | URL to owner's manual PDF |
| resources.parts_diagram_url | | | | URL to parts diagram/exploded view |

**Questions:**
1. Do you store these URLs per equipment or per model?
2. Are these public URLs or require authentication?
3. What other resource types do you have?
4. Are resources always available or sometimes null?
5. How are these maintained (manually uploaded, synced from manufacturer)?

---

### Table 1.5: Parts Catalog Fields

**Path in response:** `equipment[].parts_catalog[]` (array of parts)

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| parts_catalog (array) | | | | All parts for this equipment |
| part_number | | | | Unique part identifier |
| description | | | | Part name/description |
| price | | | | Current price (decimal) |
| availability | | | | Stock status |
| category | | | | Part category (e.g., "Filters") |
| popularity_rank | | | | For sorting "Top 10 common" |

**Critical Questions:**

1. **Parts per equipment:**
   - How many parts typically per equipment? (10? 50? 100?)
   - Should we paginate or return all?

2. **availability field:**
   - What are ALL possible values?
   - We assume: "in_stock", "order", "out_of_stock" - correct?
   - Others: "backordered", "discontinued"?

3. **popularity_rank / Top 10 common parts:**
   - ⭐ **CRITICAL:** How should we determine "Top 10 common parts"?
   - Do you track:
     - Sales frequency?
     - Order count?
     - Replacement interval?
   - Can you provide a sort field for this?
   - Or should we just show first 10 alphabetically for MVP?

4. **category field:**
   - What categories do you use?
   - Complete list available?

5. **Price:**
   - Does price include tax?
   - Different prices for different customers?

---

### Table 1.6: Include Parameter Support

**Query parameter:** `?include=equipment,addresses,resources,parts`

| Include Value | Returns | Lizzy Table Joins Required | Performance Impact | Supported? (Y/N) |
|---------------|---------|---------------------------|-------------------|------------------|
| equipment | Equipment list only | | | |
| addresses | All customer addresses | | | |
| resources | Tutorial videos, manuals per equipment | | | |
| parts | Parts catalog per equipment | | | |
| equipment,addresses | Equipment + addresses | | | |
| equipment,addresses,resources | Equipment + addresses + resources | | | |
| (all) equipment,addresses,resources,parts | Everything | | | |

**Questions:**
1. Is this "include" parameter approach feasible?
2. Or would you prefer separate API endpoints for each?
3. What's the performance impact of returning full parts catalogs?
4. Should we limit parts to X per equipment in combined response?

---

## API 2: Get Customer Transactions

**Endpoint:** `GET /api/customers/{customer_id}/transactions`

**Purpose:** Retrieve all customer service orders, parts orders, and invoices

### Table 2.1: Service Order Fields

**Path in response:** `service_orders[]`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| order_id | | | | Service order identifier |
| type | type | | | Values: "Service Ticket", "Major Unit Sale", "Major Unit Quote", "Parts Quote", "Parts Order", "Internet". May use type=Internet for new parts orders? |
| equipment_id | | | | Related equipment ID |
| equipment_name | | | | Equipment make/model for display |
| status | Service Ticket Status | | | values: "Open", "Awaiting Approval", "This is an Estimate", "Completed", "Processed" |
| estimated_completion | | | | Estimated completion date |
| created_date | | | | Order creation date |
| processed_date | | | | date service order was processed |
| description | | | | Service description/problem |

**Critical Questions:**

1. **status field - Service Orders:**
   - What are ALL possible status values? - Answer: See comments in table above for field status
   - We need to filter for "in_progress" or "in_service" - Answer: Business logic will be built into twillio (not API's), so no need to define here.
   - Status definitions:
     - "open" (just created)
     - "This is an Estimate" (order has been updated with estimate details including parts and labor estimates)
     - "awaiting_parts" (waiting for parts)
     - "Awaiting Approval" (Estimate provided to customer, awaiting customer approval before performing service work)
     - "Completed" (service complete and ready for pickup)
     - "Processed" (picked up and paid for)

2. **equipment_name:**
   - Is this stored or should we construct from make + model?
   - Format you use (e.g., "Honda HRX217" or "HRX217 Honda")?

3. **estimated_completion:**
   - Always populated?
   - Who sets this (automatically, technician, system)?
   - How reliable for customer communication?

4. Is invoice created immediately when service order created? - Answer: Service orders and their invoices are not separate entities (at least for the purpose of our project). Invoices are generated after Service Orders are put in the 'Processed' status, and then URL links are sent to customer to view their invoice and/or make payment. For the purposes of our agentic workflow, we will just use service orders (also known as Service Tickets)
5. Or created later in workflow? - Answer: See answer above.
6. Are invoice_url and payment_url public (no auth required)? - Answer: Yes
7. Do these URLs work on mobile devices? - Answer: Yes
8. Do payment URLs support Apple Pay / Google Pay? - Answer: Do not believe so

---

### Table 2.3: Parts Order Fields

**Path in response:** `parts_orders[]`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| order_id | | | | Parts order identifier |
| type | type | | | Values: "Service Ticket", "Major Unit Sale", "Major Unit Quote", "Parts Quote", "Parts Order", "Internet". May use type=Internet for new parts orders? |
| equipment_id | | | | Related equipment ID |
| make | Make | | | Manufacturer (e.g., "Honda") |
| model name | Model Name | | | Model name (e.g., "HRX217") |
| model number | Model Number | | | Model number (e.g., "HRX217K6VKA") |
| status | | | | Order status |
| tracking_number | | | | Shipping tracking number |
| estimated_delivery | | | | Estimated delivery date |
| created_date | | | | Order creation date |
| items (array) | | | | Ordered parts |

**Critical Questions:**

1. **status field - Parts Orders:**
   - What are ALL possible status values?
   - Suggested values:
     - "pending" (just ordered)
     - "processing" (being prepared)
     - "shipped" (in transit)
     - "delivered" (received)
     - "will_call_ready" (ready for pickup)
     - "cancelled"
   - Please provide complete list

2. **tracking_number:**
   - Always available?
   - Format (just number or full URL)?
   - Which carriers do you use?
   - Should we construct tracking URL?

3. **estimated_delivery:**
   - How calculated?
   - Reliable for customer communication?

---

### Table 2.4: Parts Order Items Fields

**Path in response:** `parts_orders[].items[]`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| part_number | | | | Part identifier |
| description | | | | Part name |
| quantity | | | | Quantity ordered |
| price | | | | Price per unit |

**Questions:**
1. Is price the per-unit price or total for quantity?
2. Does this include shipping cost?

---

### Table 2.5: Parts Order Invoice Fields

**Path in response:** `parts_orders[].invoice`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| invoice (object) | | | | Invoice for this parts order |
| invoice.invoice_id | | | | Invoice identifier |
| invoice.amount | | | | Total amount |
| invoice.status | | | | Payment status |
| invoice.invoice_url | | | | URL to view invoice |
| invoice.payment_url | | | | URL to make payment |

---

### Table 2.6: Standalone Invoices Fields

**Path in response:** `invoices[]`

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| invoice_id | | | | Invoice identifier |
| type | | | | Invoice type |
| equipment_id | | | | Related equipment (if applicable) |
| amount | | | | Total amount |
| status | | | | Payment status |
| due_date | | | | Payment due date |
| created_date | | | | Invoice creation date |
| description | | | | Invoice description |
| invoice_url | | | | URL to view invoice |
| payment_url | | | | URL to make payment |

**Critical Questions:**

1. **type field:**
   - What are ALL possible invoice types?
   - We assume: "unit_sale", "service", "parts"
   - Others: "rental", "warranty", "labor_only"?

2. **status field - Invoices:**
   - What are ALL possible payment status values?
   - Suggested:
     - "unpaid"
     - "partially_paid"
     - "paid"
     - "overdue"
     - "voided"
   - Please provide complete list

3. **Invoice relationship:**
   - Are service/parts invoices ALSO in the invoices array?
   - Or only standalone invoices (unit sales)?
   - Should we deduplicate?

---

### Table 2.7: Query Parameters

| Parameter | Purpose | Example | Supported? (Y/N) | Comments |
|-----------|---------|---------|------------------|----------|
| include | Filter response sections | ?include=service_orders,parts_orders | | |
| equipment_id | Filter by equipment | ?equipment_id=EQ123 | | |
| status | Filter by status | ?status=in_progress | | |
| date_from | Historical data start | ?date_from=2025-01-01 | | |
| date_to | Historical data end | ?date_to=2025-12-31 | | |

**Questions:**
1. Which query parameters can you support?
2. Default date range if not specified?
3. Maximum date range allowed?
4. Performance implications of large result sets?

---

## API 3: Create Service Order

**Endpoint:** `POST /api/service-orders`

**Purpose:** Create new service order for equipment repair/maintenance

### Table 3.1: Request Fields

| Field Name (Our Requirement) | Required? | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|-----------|---------------------|-------------------|----------|
| customer_id | Yes | string | | | Customer identifier |
| equipment_id | Yes | string | | | Equipment identifier |
| service_type | Yes | string | | | Type of service |
| description | Yes | string | | | Problem description |
| drop_off_type | Yes | string | | | Drop-off method |
| preferred_date | Yes | date | | | Preferred service date |
| pickup_requested | No | boolean | | | Customer wants pickup? |
| pickup_date | No | date | | | Scheduled pickup date |
| pickup_address | No | object | | | Pickup location |
| pickup_address.street | No | string | | | Street address |
| pickup_address.city | No | string | | | City |
| pickup_address.state | No | string | | | State |
| pickup_address.zip | No | string | | | ZIP code |

**Critical Questions:**

1. **service_type field:**
   - What are ALL valid values?
   - We suggest: "repair", "maintenance", "inspection"
   - Do you use different values?
   - Complete list?

2. **drop_off_type field:**
   - What are ALL valid values?
   - We suggest: "drop_off_today", "drop_off_tomorrow", "schedule_pickup"
   - Do you use different values?
   - How do these map to your internal workflow?

3. **description field:**
   - Any length limits? (we assume 10-500 characters)
   - Free text or structured?
   - Any validation rules?

4. **preferred_date:**
   - How far in advance can customers schedule?
   - Any blackout dates?
   - Validation rules?

5. **pickup_requested workflow:**
   - How is pickup handled in Lizzy?
   - Different service order type?
   - Additional fields needed?
   - Cost implications?

---

### Table 3.2: Response Fields

| Field Name (Our Requirement) | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|---------------------|-------------------|----------|
| order_id | string | | | Created service order ID |
| invoice_id | string | | | Associated invoice ID |
| status | string | | | Initial order status |
| invoice_url | string | | | URL to view invoice |
| payment_url | string | | | URL to make payment |
| created_at | datetime | | | Timestamp of creation |

**Questions:**
1. Is invoice created immediately or async?
2. What is the initial status after creation?
3. How quickly can endpoint respond? (<2 seconds?)
4. Should we handle async on our end?

---

## API 4: Create Parts Order

**Endpoint:** `POST /api/parts-orders`

**Purpose:** Create new parts order

### Table 4.1: Request Fields

| Field Name (Our Requirement) | Required? | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|-----------|---------------------|-------------------|----------|
| customer_id | Yes | string | | | Customer identifier |
| equipment_id | Yes | string | | | Equipment identifier |
| items | Yes | array | | | Parts to order |
| items[].part_number | Yes | string | | | Part identifier |
| items[].quantity | Yes | integer | | | Quantity (1-99) |
| ship_to | Yes | string | | | Shipping option |
| shipping_address | Conditional | object | | | Address (if ship_to="ship_to_custom") |
| shipping_address.street | Conditional | string | | | Street address |
| shipping_address.city | Conditional | string | | | City |
| shipping_address.state | Conditional | string | | | State |
| shipping_address.zip | Conditional | string | | | ZIP code |

**Critical Questions:**

1. **items array - Multiple parts:**
   - ⭐ **MVP:** We only support 1 part per order
   - Is this acceptable?
   - Or must we support multiple parts in single order?
   - Would separate orders for multiple parts cause issues?

2. **ship_to field:**
   - What are ALL valid values?
   - We suggest: "ship_to_primary", "ship_to_custom", "will_call"
   - Do you use different values?
   - How does "will_call" work in your system?

3. **Inventory validation:**
   - Does endpoint check inventory availability?
   - What happens if part is out of stock?
   - 409 Conflict error?
   - Should we check availability before submitting order?

4. **Shipping cost:**
   - How is shipping calculated?
   - Included in response?
   - Added to invoice amount?

---

### Table 4.2: Response Fields

| Field Name (Our Requirement) | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|---------------------|-------------------|----------|
| order_id | string | | | Created parts order ID |
| invoice_id | string | | | Associated invoice ID |
| total | decimal | | | Total amount |
| status | string | | | Initial order status |
| invoice_url | string | | | URL to view invoice |
| payment_url | string | | | URL to make payment |
| estimated_delivery | date | | | Estimated delivery date |
| created_at | datetime | | | Timestamp of creation |

**Questions:**
1. Is total inclusive of shipping and tax?
2. How is estimated_delivery calculated?
3. What is initial status after creation?

---

## API 5: Log SMS Message

**Endpoint:** `POST /api/sms/log`

**Purpose:** Log SMS messages to Lizzy for dealer visibility

### Table 5.1: Request Fields

| Field Name (Our Requirement) | Required? | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|-----------|---------------------|-------------------|----------|
| phone | Yes | string | | | Customer phone number |
| customer_id | Yes | string | | | Customer identifier |
| direction | Yes | string | | | "inbound" or "outbound" |
| message | Yes | string | | | SMS message text |
| timestamp | Yes | datetime | | | Message timestamp (ISO 8601) |
| source | Yes | string | | | "workflow_bot" or "dealer" |

**Questions:**

1. **Existing SMS interface:**
   - Do you currently have an SMS messaging interface in Lizzy?
   - How do dealers view SMS conversations now?
   - Can we integrate with existing system?

2. **Message threading:**
   - How are conversations grouped/threaded?
   - By phone number? By customer ID? By date range?
   - Do you maintain conversation history?

3. **Real-time updates:**
   - Do dealers see messages in real-time?
   - Or do they need to refresh?
   - Push notifications available?

4. **Additional metadata:**
   - Any additional fields needed?
   - Equipment ID for context?
   - Order ID if discussing an order?
   - Workflow state?

5. **Priority:**
   - Is this endpoint critical for MVP?
   - Or can we defer if SMS interface doesn't exist yet?

---

### Table 5.2: Response Fields

| Field Name (Our Requirement) | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|---------------------|-------------------|----------|
| message_id | string | | | Logged message ID |
| logged | boolean | | | Success flag |

---

## API 6: Flag Conversation for Dealer

**Endpoint:** `POST /api/sms/conversation/{phone}/flag`

**Purpose:** Notify dealer that customer needs human attention

### Table 6.1: Request Fields

| Field Name (Our Requirement) | Required? | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|-----------|---------------------|-------------------|----------|
| customer_id | Yes | string | | | Customer identifier |
| reason | Yes | string | | | Reason for escalation |
| priority | Yes | string | | | "urgent", "normal", "low" |
| context | No | object | | | Additional context |
| context.last_menu | No | string | | | Last workflow state |
| context.selected_equipment | No | string | | | Equipment ID in context |
| context.equipment_name | No | string | | | Equipment name |

**Questions:**

1. **Dealer notifications:**
   - How should dealers be notified?
   - Email? SMS? In-app notification?
   - Real-time alerts or periodic checks?

2. **Priority levels:**
   - Do you support priority flagging?
   - Values: "urgent", "normal", "low"?
   - Or different levels?

3. **Context information:**
   - What context is most helpful for dealers?
   - Equipment info? Order history? Last interaction?
   - Other fields needed?

4. **Priority:**
   - Critical for MVP or can defer?
   - How important is dealer handoff?

---

### Table 6.2: Response Fields

| Field Name (Our Requirement) | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|---------------------|-------------------|----------|
| conversation_id | string | | | Conversation identifier |
| flagged | boolean | | | Success flag |
| dealer_notified | boolean | | | Was dealer notified? |

---

## API 7: Get Conversation Status

**Endpoint:** `GET /api/sms/conversation/{phone}/status`

**Purpose:** Check if dealer has taken over conversation

### Table 7.1: Response Fields

| Field Name (Our Requirement) | Data Type | Lizzy App Field Name | Lizzy Table.Column | Comments |
|------------------------------|-----------|---------------------|-------------------|----------|
| conversation_id | string | | | Conversation identifier |
| dealer_active | boolean | | | Is dealer currently handling? |
| last_dealer_message | datetime | | | When dealer last responded |
| bot_paused | boolean | | | Should bot pause? |

**Questions:**

1. **Dealer takeover mechanism:**
   - How does dealer "take over" a conversation?
   - Manual action in interface?
   - Automatic based on dealer response?

2. **Bot resume:**
   - How does bot know when to resume?
   - Dealer explicitly releases conversation?
   - Timeout after dealer inactivity?
   - Customer types "MENU" to restart bot?

3. **State management:**
   - Who maintains conversation state?
   - Lizzy or our system?

4. **Priority:**
   - Critical for MVP?
   - Can we simplify for MVP?

---

## Data Type Reference

### Standard Data Types

| Type | Format | Example | Validation Rules |
|------|--------|---------|------------------|
| string | UTF-8 text | "Honda" | Max length? |
| integer | Whole number | 123 | Min/max values? |
| decimal | Number with decimals | 99.99 | 2 decimal places? |
| boolean | true/false | true | - |
| date | ISO 8601 date | "2025-11-17" | YYYY-MM-DD |
| datetime | ISO 8601 datetime | "2025-11-17T10:30:00Z" | ISO 8601 format |
| array | List of items | [1, 2, 3] | Max items? |
| object | Key-value pairs | {"key": "value"} | - |

### Phone Number Format

| Format | Example | Your Format? |
|--------|---------|--------------|
| E.164 | +12065550100 | |
| National | (206) 555-0100 | |
| Digits only | 2065550100 | |

**Question:** Which format does Lizzy use? Should we normalize on our end?

---

## Summary Questions for Lizzy Team

### High Priority (Critical for MVP)

1. **Parts catalog sorting:**
   - How to determine "Top 10 common parts"?
   - Do you have popularity/frequency data?
   - Or should we use alphabetical for MVP?

2. **Status values:**
   - Complete list of status values for:
     - Service orders
     - Parts orders
     - Invoices
   - Definitions for each status

3. **unit_type (equipment category):**
   - Field name in Lizzy
   - Complete list of possible values
   - Free text or controlled list?

4. **service_type and drop_off_type:**
   - Valid values for service_type
   - Valid values for drop_off_type
   - How they map to your workflow

5. **Invoice creation:**
   - When is invoice created?
   - Immediately or async?
   - Response time expectations?

### Medium Priority (Important but workarounds exist)

6. **Multi-part orders:**
   - Must support multiple parts per order?
   - Or OK with one part per order for MVP?

7. **Include parameter:**
   - Can you support consolidated endpoint?
   - Or prefer separate endpoints?

8. **Phone number format:**
   - What format stored in DB?
   - Need normalization?

9. **Equipment search:**
   - Is SKU field available?
   - Can search by serial number?

10. **SMS interface:**
    - Existing SMS chat interface?
    - Log endpoint needed?
    - Dealer handoff workflow?

### Low Priority (Nice to have)

11. **Additional resources:**
    - Other resource types available?
    - Video formats supported?

12. **Historical data:**
    - How far back should we fetch transactions?
    - Performance concerns with large datasets?

13. **Will Call:**
    - How is "will call" handled?
    - Special fields needed?

---

## Return Instructions

### Please Return:

1. **Completed mapping tables** (all columns filled in)
2. **Answers to all questions** (throughout document)
3. **Sample API responses** (real data with sensitive info redacted)
4. **Entity relationship diagram** (if available)
5. **Existing API documentation** (if any)
6. **Database schema** (if you can share)

### Timeline:

- **Target:** Within 1 week
- **Format:** This document with additions, or your preferred format
- **Contact:** [Your contact info]

### Next Steps After Return:

1. Review your responses
2. Create detailed API specification document
3. Schedule technical kickoff call
4. Set up staging environment
5. Begin integration testing

---

## Thank You!

This detailed mapping will help us integrate seamlessly with Lizzy and avoid misunderstandings. We appreciate your thorough completion of these tables.

**Questions while completing this document? Contact Jonathan at [contact info]**

---

**Document Version:** 1.0  
**Date:** November 17, 2025  
**Status:** Awaiting Lizzy Team Input
