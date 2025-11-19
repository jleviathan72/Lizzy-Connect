# Lizzy Agentic Commerce CRM - Workflow Requirements

**Version:** 1.3.0  
**Last Updated:** November 18, 2025  
**Status:** Released

**📋 MVP Implementation Note:** An MVP implementation guide using Twilio Studio (minimal coding required) is available separately. See `Twilio_Studio_MVP_Implementation_Guide.md` for a rapid prototyping approach that addresses critical limitations through strategic compromises. This MVP path enables validation before investing in the full custom Node.js engine described in this document.

-----

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.3.0 | 2025-11-18 | 8 UX improvements and API consolidation updates | Jonathan |
| 1.2.0 | 2025-11-13 | Added SKU/SN search, two-layer filtering, consolidated APIs | Jonathan |
| 1.1.0 | 2025-11-10 | Initial complete workflow specifications | Jonathan |

## Changelog (v1.3.0)

### Changed
- In-service equipment menu simplified from 7 to 5 options
- Main Menu re-sequenced (Order Parts moved to position #2)
- Service Status workflow now has single entry point only
- Key Principles simplified (removed obsolete API action framework)
- Order Parts menu message clarified

### Added
- SKU field to 6 equipment list displays (format: SKU: ###)
- BACK navigation option to Filter by Unit Type menu
- BACK navigation option to Filter by Model menu (3 workflows)

### Removed
- "Schedule Service" option from in-service equipment menu
- "Check Service Status" option from in-service equipment menu
- Troubleshoot Equipment → Service Status workflow transition
- Four-action framework from Key Principles section

-----

## Table of Contents

1. [Overview](#overview)
1. [System Architecture](#system-architecture)
1. [Main Menu](#main-menu)
1. [Workflow 1: Troubleshoot Equipment](#workflow-1-troubleshoot-equipment)
1. [Workflow 2: Schedule Service](#workflow-2-schedule-service)
1. [Workflow 3: Service Status](#workflow-3-service-status)
1. [Workflow 4: Order Parts](#workflow-4-order-parts)
1. [Workflow 5: Parts Order Status](#workflow-5-parts-order-status)
1. [Workflow 6: View or Pay Invoice](#workflow-6-view-or-pay-invoice)
1. [Common Patterns](#common-patterns)
1. [Lizzy API Requirements](#lizzy-api-requirements)
1. [Open Questions](#open-questions)

-----

## Overview

### Purpose

Facilitate dealer and customer bi-directional communications via SMS/Texting using a menu-driven workflow engine for both service and customer engagement.

### Scope (MVP)

- Menu-driven SMS workflow system (customer-facing)
- Integration with existing Lizzy SMS chat feature (dealer-facing)
- Lizzy API integrations for data retrieval and transaction creation
- No campaign management features (Phase 2)
- No custom admin dashboard (use existing Lizzy SMS interface)

### Key Principles

- **Simple numbered menu options** - Customer responds with 1, 2, 3, etc.
- **Equipment Context Display** - When a customer selects equipment in one workflow and then transitions to another workflow (e.g., from Troubleshoot to Order Parts), the equipment selection carries forward and the new workflow skips directly to the relevant step (bypassing equipment selection). All subsequent messages display equipment context using format: `[Equipment] - [Action]` (e.g., "Honda HRX217 - Schedule Service")
- **15-minute session timeout** - If customer is inactive, session expires and returns to Main Menu
- **Dealer handoff** - Customers can always choose to "Text Shop" or "Call Shop"
- **All messages logged to Lizzy** - Dealers see full conversation history in Lizzy SMS interface

### User Initiation

Any inbound text message to the dealer's phone number will trigger the Main Menu. This will also trigger two initial API calls:

1. **GET Consolidated Customer Profile** - This information will be cached and used for the duration of the session (TTL 15 min)
2. **GET Consolidated Transactions & Invoices** - This information will be cached and used for the current session (TTL 5 min)

-----

## System Architecture

### Components

1. **Twilio** - SMS gateway (send/receive)
1. **Workflow Engine** - Node.js/Express server processing menu logic
1. **Lizzy API** - Data source for equipment, orders, invoices
1. **Lizzy SMS Interface** - Dealer-facing conversation view (existing)
1. **PostgreSQL** - Session state storage
1. **DynamoDB** - Session data caching (customer profile, equipment, parts, transactions)

### Message Flow

```
Customer Phone → Twilio → Workflow Engine → Lizzy API
                   ↓                           ↓
              SMS Response              Log to Lizzy SMS
```

### Session Management

- **Session Duration:** 15 minutes from last activity
- **Session Data Stored:**
  - Phone number
  - Customer ID (from Lizzy)
  - Current workflow node
  - Selected equipment context (equipment ID, display name, serial number)
  - Temporary selections (parts, shipping options, etc.)
  - Dealer active flag (for handoff)
  - **Cached Data (from consolidated API calls):**
    - Complete customer profile (equipment, addresses, resources, parts catalogs)
    - All transactions (service orders, parts orders, invoices)
    - Cache timestamps for refresh logic
- **DynamoDB TTL:** Automatic expiration of session data after 15 minutes of inactivity

-----

## Main Menu

### Entry Points

- Any inbound SMS to dealer number
- Session timeout (after 15 min inactivity)
- Customer types "MENU" at any time
- Completion of any workflow

### Message Text

```
Welcome to [Dealer Name]! Reply with a number:
1. Troubleshoot Equipment
2. Order Parts
3. Schedule Service
4. Check Service Status
5. Parts Order Status
6. View or Pay Invoice
```

### Options

|Option|Label                 |Next Node            |
|------|----------------------|---------------------|
|1     |Troubleshoot Equipment|TROUBLESHOOT_MENU    |
|2     |Order Parts           |ORDER_PARTS_MENU     |
|3     |Schedule Service      |SCHEDULE_SERVICE_MENU|
|4     |Check Service Status  |SERVICE_STATUS_LIST  |
|5     |Parts Order Status    |PARTS_STATUS_LIST    |
|6     |View or Pay Invoice   |INVOICE_TYPE_MENU    |

### Invalid Input Handling

If customer sends anything other than 1-6:

```
Invalid option. Please reply with a number 1-6.

[Repeat Main Menu message]
```

### Required Lizzy APIs

- `GET /api/customers/by-phone/{phone}?include=equipment,addresses,resources,parts` - Consolidated customer profile lookup with all equipment, parts catalogs, addresses, and resources (cached for full session)

### Open Questions

- [x] **What happens if phone number is not found in Lizzy?**
  - **Answer:** Display error message: "You do not have an account with [Dealer Name]! Please contact [Dealer Name] to setup an account."
- [x] **Should we support "HELP" or "STOP" keywords?**
  - **Answer:** Yes.
    - **HELP:** Offer menu options to "Call Dealer" or "Text Dealer"
    - **STOP:** Display: "You previously authorized this number to be used for communications with [Dealer Name]! Are you sure you want to stop all communications from [Dealer Name]? Press 'Y' to confirm or contact the Dealer." Then display menu options to "Call Dealer" or "Text Dealer"
- [x] **Multi-language support needed?**
  - **Answer:** No, not at this time.

-----

## Workflow 1: Troubleshoot Equipment

### User Story

As a customer, I want to find troubleshooting resources for my equipment so I can try to solve issues myself before contacting the dealer.

### Entry Point

From Main Menu, customer selects option 1

### Flow Diagram

```
Main Menu (1) → Troubleshoot Menu → Equipment Selection → Equipment Resources → Exit Options
```

### Step 1: Troubleshoot Menu

**Node:** `TROUBLESHOOT_MENU`

**Message:**

```
Troubleshoot Equipment - Reply with a number:
1. Send Full Fleet List
2. Filter Fleet List by Type
3. Select Equipment by SKU
4. Select Equipment by SN
```

**Options:**

|Option|Next Node                   |
|------|----------------------------|
|1     |SHOW_FULL_FLEET_TROUBLESHOOT|
|2     |FILTER_FLEET_TROUBLESHOOT   |
|3     |SELECT_BY_SKU_TROUBLESHOOT  |
|4     |SELECT_BY_SN_TROUBLESHOOT   |

**Required Data:** None

**Note:** Options 3 and 4 (Select by SKU/SN) will require additional workflow nodes to be defined.

-----

### Step 1A: Select Equipment by SKU (NEW NODE)

**Node:** `SELECT_BY_SKU_TROUBLESHOOT`

**Message:**

```
Troubleshoot Equipment - Select by SKU

Please enter the equipment SKU:

Example: HRX217VKA
```

**Input:** Free-form text (customer types SKU)

**Input Validation:**
- Search cached equipment data for matching SKU
- Case-insensitive search
- If not found: "SKU not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SHOW_EQUIPMENT_RESOURCES` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 1B: Select Equipment by Serial Number (NEW NODE)

**Node:** `SELECT_BY_SN_TROUBLESHOOT`

**Message:**

```
Troubleshoot Equipment - Select by Serial Number

Please enter the equipment serial number:

Example: MZCG-1234567
```

**Input:** Free-form text (customer types serial number)

**Input Validation:**
- Search cached equipment data for matching serial number
- Case-insensitive search
- If not found: "Serial number not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SHOW_EQUIPMENT_RESOURCES` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 2A: Show Full Fleet List

**Node:** `SHOW_FULL_FLEET_TROUBLESHOOT`

**Message:** (Dynamic - generated from Lizzy API)

```
Your Equipment - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567 - Active
2. Husqvarna 450 - SKU: 456 - SN: 45012345 - Active
3. Echo PB-580T - SKU: 789 - SN: E12345678 - In Service
[... up to 10 items]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on equipment count

**Required Lizzy API:**

- None (uses cached customer profile data from session initialization)

**Next Node:** `SHOW_EQUIPMENT_RESOURCES`

**Open Questions:**

- [x] **What if customer has 0 equipment?**
  - **Answer:** Yes, show message "No equipment found. Contact dealer to add equipment."
- [x] **What if customer has >10 pieces of equipment?**
  - **Answer:** Yes, show first 10 with option to "Show More"
- [x] **Should we show equipment status in the list?**
  - **Answer:** Yes, show status (Active, In Service, Inactive) in the list

-----

### Step 2B: Filter by Unit Type

**Node:** `FILTER_FLEET_TROUBLESHOOT`

**Message:**

```
Filter by Unit Type - Reply with a number:
1. Mowers
2. Weed Eaters
3. Leaf Blowers
4. Chainsaws
5. Other

Reply BACK to return to previous menu
```

**Options:**

|Option|Filter Value|Next Node                    |
|------|------------|-----------------------------|
|1     |Mowers      |FILTER_BY_MODEL_TROUBLESHOOT |
|2     |Weed Eaters |FILTER_BY_MODEL_TROUBLESHOOT |
|3     |Leaf Blowers|FILTER_BY_MODEL_TROUBLESHOOT |
|4     |Chainsaws   |FILTER_BY_MODEL_TROUBLESHOOT |
|5     |Other       |SHOW_FILTERED_FLEET_TROUBLESHOOT|
|BACK  |(none)      |TROUBLESHOOT_MENU            |

**Session Data Stored:** `filterUnitType: "Mowers"`

**Open Questions:**

- [x] **How does Lizzy categorize equipment?**
  - **Answer:** Lizzy uses two-layer filtering: First by `Unit_Type` field, then by `Model` field. Need to add a second filter step after Unit_Type for filtering by Model.
- [ ] **Should categories be configurable per dealer?**
- [x] **What equipment types should be included in "Other"?**
  - **Answer:** Any equipment returned in the API call that does not have an associated value for `unit_type`

-----

### Step 2B-2: Filter by Model (NEW NODE)

**Node:** `FILTER_BY_MODEL_TROUBLESHOOT`

**Message:** (Dynamic)

```
Mowers - Filter by Model - Reply with a number:
1. HRX217
2. HRR216
3. TimeMaster
[... models within selected Unit_Type]

Reply BACK to return to unit type selection
Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on models, plus BACK

**Required Data:** Uses cached equipment data filtered by `filterUnitType`

**Next Node:** `SHOW_FILTERED_FLEET_TROUBLESHOOT` (or `FILTER_FLEET_TROUBLESHOOT` if BACK)

**Session Data Stored:** `filterModel: "HRX217"`

-----

### Step 2C: Show Filtered Fleet List

**Node:** `SHOW_FILTERED_FLEET_TROUBLESHOOT`

**Message:** (Dynamic - generated from cached data based on filter selections)

```
Mowers - HRX217 - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567 - Active
2. Honda HRX217 - SKU: 456 - SN: MZCG-9876543 - Active
[... filtered equipment]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on filtered equipment count

**Required Lizzy API:**

- None (uses cached customer profile data, filtered locally)

**Next Node:** `SHOW_EQUIPMENT_RESOURCES`

**Session Data Stored:** `selectedEquipmentId: "EQ123"`

**Special Cases:**
- If filter returns 0 equipment: "No equipment found matching your filter. Reply MENU to start over."

-----

### Step 3: Show Equipment Resources

**Node:** `SHOW_EQUIPMENT_RESOURCES`

**Message:** (Dynamic - generated based on selected equipment)

**If equipment status is NOT "In Service":**

```
Honda HRX217 - SN: MZCG-1234567

Troubleshooting Resources:
📺 Tutorial Video: [YouTube link]
📖 Owner's Manual: [PDF link]

What would you like to do? Reply with a number:
1. Schedule Service
2. Order Parts
3. Check Parts Order Status
4. View or Pay Invoice
5. Text or Call Dealer
6. Main Menu
```

**Note:** "Check Service Status" option is SUPPRESSED when equipment status is NOT "In Service"

**If equipment status is "In Service":**

```
Honda HRX217 - SN: MZCG-1234567

🔧 Currently In Service
Service Order: #SO789
Status: In Progress
Est. Completion: Nov 15, 2025

Troubleshooting Resources:
📺 Tutorial Video: [YouTube link]
📖 Owner's Manual: [PDF link]

What would you like to do? Reply with a number:
1. Order Parts
2. Check Parts Order Status
3. View or Pay Invoice
4. Text or Call Dealer
5. Main Menu
```

**Options:**

|Option|Next Node              |Notes                                                                            |
|------|-----------------------|---------------------------------------------------------------------------------|
|1     |SHOW_PARTS_LIST        |(skips ORDER_PARTS_MENU, equipment pre-selected)                                 |
|2     |PARTS_STATUS_LIST      |(equipment context provided for filtering)                                       |
|3     |INVOICE_TYPE_MENU      |(equipment context provided for filtering)                                       |
|4     |TEXT_OR_CALL_DEALER    |NEW - Combined dealer contact option                                             |
|5     |MAIN_MENU              |                                                                                 |

**Session Data Carried Forward:**

- `selectedEquipmentId`: Equipment ID for use in subsequent workflows
- `selectedEquipmentName`: Display name (e.g., "Honda HRX217")
- `selectedEquipmentSN`: Serial number for additional context

**Note:** When customer selects options 1-5, the equipment context is preserved and the next workflow skips equipment selection steps or filters by the selected equipment. Option numbering adjusts based on whether equipment is "In Service" (option 2 shown) or not (option 2 suppressed).

**Required Lizzy API:**

- None (uses cached customer profile data from session initialization for equipment resources)
- Transactions cache used to check if equipment is currently in service

**Open Questions:**

- [x] **Does Lizzy store tutorial video URLs per equipment model?**
  - **Answer:** Yes. Will need to confirm the field name where this is stored.
- [x] **Does Lizzy have owner's manual PDFs stored per equipment model?**
  - **Answer:** Yes. Will need to confirm the field name where this is stored.
- [x] **If no resources are available, what should we show?**
  - **Answer:** Display appropriate message:
    - Tutorial: "No troubleshooting tutorial videos are available for this equipment. Please contact your dealer for help with troubleshooting your equipment."
    - Manual: "No Owner's Manual is available for this equipment. Please contact your dealer for help with troubleshooting your equipment."

**Implementation Note:** When equipment is selected in this workflow and customer chooses to Schedule Service, Check Service Status, Order Parts, or Check Parts Order Status, that equipment selection carries forward automatically. The next workflow displays equipment context and skips the equipment selection step.

-----

## Workflow 2: Schedule Service

### User Story

As a customer, I want to schedule service for my equipment so I can get it repaired or maintained.

### Entry Points

- From Main Menu, customer selects option 2 → Starts at `SCHEDULE_SERVICE_MENU`
- From Troubleshoot Equipment workflow, customer selects "Schedule Service" → Skips to `SERVICE_PROBLEM_DESCRIPTION` with equipment pre-selected

### Flow Diagram

```
Main Menu (2) → Schedule Service Menu → Equipment Selection → Problem Description → Parts Order Offer → Drop-off Options → Confirmation
```

### Step 1: Schedule Service Menu

**Node:** `SCHEDULE_SERVICE_MENU`

**Message:**

```
Schedule Service - Reply with a number:
1. Send Full Fleet List
2. Filter Fleet List by Type
3. Select Equipment by SKU
4. Select Equipment by SN
```

**Options:**

|Option|Next Node                 |
|------|--------------------------|
|1     |SHOW_FULL_FLEET_SERVICE   |
|2     |FILTER_FLEET_SERVICE      |
|3     |SELECT_BY_SKU_SERVICE     |
|4     |SELECT_BY_SN_SERVICE      |

**Required Data:** None

-----

### Step 1A: Select Equipment by SKU (Service)

**Node:** `SELECT_BY_SKU_SERVICE`

**Message:**

```
Schedule Service - Select by SKU

Please enter the equipment SKU:

Example: HRX217VKA
```

**Input:** Free-form text (customer types SKU)

**Input Validation:**
- Search cached equipment data for matching SKU
- Case-insensitive search
- If not found: "SKU not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SERVICE_PROBLEM_DESCRIPTION` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 1B: Select Equipment by Serial Number (Service)

**Node:** `SELECT_BY_SN_SERVICE`

**Message:**

```
Schedule Service - Select by Serial Number

Please enter the equipment serial number:

Example: MZCG-1234567
```

**Input:** Free-form text (customer types serial number)

**Input Validation:**
- Search cached equipment data for matching serial number
- Case-insensitive search
- If not found: "Serial number not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SERVICE_PROBLEM_DESCRIPTION` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 2A: Show Full Fleet List

**Node:** `SHOW_FULL_FLEET_SERVICE`

**Message:** (Dynamic)

```
Select equipment for service - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567
2. Husqvarna 450 - SKU: 456 - SN: 45012345
3. Echo PB-580T - SKU: 789 - SN: E12345678
[... up to 10 items]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on equipment count

**Required Lizzy API:**

- None (uses cached customer profile data from session initialization)

**Next Node:** `SERVICE_PROBLEM_DESCRIPTION`

**Session Data Stored:** `selectedEquipmentId: "EQ123"`

**Note:** Flow updated to collect problem description before drop-off options.

-----

### Step 2B: Filter Fleet List

**Node:** `FILTER_FLEET_SERVICE`

**Message:**

```
Filter by equipment type - Reply with a number:
1. Mowers
2. Weed Eaters
3. Leaf Blowers
4. Chainsaws
5. Other
```

**Options:** (Same as Troubleshoot workflow - uses two-layer filtering)

|Option|Filter Value|Next Node                |
|------|------------|-------------------------|
|1     |Mowers      |FILTER_BY_MODEL_SERVICE  |
|2     |Weed Eaters |FILTER_BY_MODEL_SERVICE  |
|3     |Leaf Blowers|FILTER_BY_MODEL_SERVICE  |
|4     |Chainsaws   |FILTER_BY_MODEL_SERVICE  |
|5     |Other       |SHOW_FILTERED_FLEET_SERVICE|

**Next Node:** `FILTER_BY_MODEL_SERVICE`

-----

### Step 2B-2: Filter by Model (Service)

**Node:** `FILTER_BY_MODEL_SERVICE`

**Message:** (Dynamic)

```
Mowers - Filter by Model - Reply with a number:
1. HRX217
2. HRR216
3. TimeMaster
[... models within selected Unit_Type]

Reply BACK to return to unit type selection
Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on models, plus BACK

**Next Node:** `SHOW_FILTERED_FLEET_SERVICE` (or `FILTER_FLEET_SERVICE` if BACK)

**Session Data Stored:** `filterModel: "HRX217"`

-----

### Step 2C: Show Filtered Fleet List

**Node:** `SHOW_FILTERED_FLEET_SERVICE`

**Message:** (Dynamic)

```
Mowers - HRX217 - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567
2. Honda HRX217 - SKU: 456 - SN: MZCG-9876543
[... filtered equipment]

Reply MENU for main menu
```

**Next Node:** `SERVICE_PROBLEM_DESCRIPTION`

-----

### Step 2D: Problem Description (NEW)

**Node:** `SERVICE_PROBLEM_DESCRIPTION`

**Message:**

```
Honda HRX217 - Schedule Service

Please describe the problem or service needed:

(Type your description and press Send)
```

**Input:** Free-form text (customer types problem description)

**Input Validation:**

- Minimum 10 characters recommended
- Maximum 500 characters
- If too short: "Please provide more details about the problem (at least 10 characters)"

**Next Node:** `OFFER_PARTS_ORDER`

**Session Data Stored:** `problemDescription: "[customer's typed description]"`

**Note:** Problem description is collected before drop-off options to provide better context for the service order.

-----

### Step 2E: Offer Parts Order (NEW)

**Node:** `OFFER_PARTS_ORDER`

**Message:**

```
Honda HRX217 - Schedule Service

Problem: [customer's description]

Would you like to order parts for this service to help expedite the repair?

Reply with a number:
1. Yes, order parts now
2. No, just schedule service
3. Main Menu
```

**Options:**

|Option|Next Node              |Notes                                                   |
|------|-----------------------|--------------------------------------------------------|
|1     |SHOW_PARTS_LIST        |Jump to Order Parts workflow with equipment pre-selected|
|2     |SERVICE_DROPOFF_OPTIONS|Continue with service scheduling                        |
|3     |MAIN_MENU              |Cancel service scheduling                               |

**Session Data Stored:** `partsOrderRequested: true/false`

**Note:** If customer orders parts (option 1), after parts order completion they should return to SERVICE_DROPOFF_OPTIONS to complete service scheduling.

-----

### Step 3: Drop-off Options

**Node:** `SERVICE_DROPOFF_OPTIONS`

**Message:**

```
Honda HRX217 - Schedule Service

How would you like to drop off? Reply with a number:
1. Drop off Today
2. Drop off Tomorrow
3. Schedule Pickup
4. Main Menu
```

**Options:**

|Option|Drop-off Type    |Next Node           |
|------|-----------------|--------------------|
|1     |drop_off_today   |CREATE_SERVICE_ORDER|
|2     |drop_off_tomorrow|CREATE_SERVICE_ORDER|
|3     |schedule_pickup  |CONFIRM_PICKUP_ADDRESS|
|4     |N/A              |MAIN_MENU           |

**Session Data Stored:** `dropOffType: "drop_off_today"`

**Open Questions:**

- [x] **For "Drop off Today/Tomorrow", should we ask for a preferred time slot?**
  - **Answer:** Not at this time.
- [x] **For "Schedule Pickup", do we need customer address?**
  - **Answer:** Yes. Prompt customer to confirm if pickup should be made at the address on file (display the primary address). Provide menu option to modify pick-up location and prompt customer to provide the full pick-up location address.
- [x] **Should we ask for problem description or assume general maintenance?**
  - **Answer:** Yes, prompt the customer to provide a problem description. Then ask if they want to place a parts order for the equipment to expedite the service. If yes, jump to 'Order Parts' workflow, passing the currently selected equipment context.

-----

### Step 4A: Confirm Pickup Address (NEW)

**Node:** `CONFIRM_PICKUP_ADDRESS`

**Message:**

```
Honda HRX217 - Schedule Pickup

Pickup Address on File:
123 Main St
Seattle, WA 98101

Reply with a number:
1. Use this address
2. Enter different address
3. Main Menu
```

**Options:**

|Option|Next Node              |
|------|-----------------------|
|1     |SCHEDULE_PICKUP_DATE   |
|2     |ENTER_PICKUP_ADDRESS   |
|3     |MAIN_MENU              |

**Session Data Stored:** `useDefaultPickupAddress: true/false`

-----

### Step 4B: Enter Pickup Address (NEW)

**Node:** `ENTER_PICKUP_ADDRESS`

**Message:**

```
Honda HRX217 - Pickup Address

Enter pickup address in this format:
Street, City, State ZIP

Example: 123 Main St, Seattle, WA 98101
```

**Input:** Free-form text (customer types address)

**Input Validation:**

- Basic format checking
- If invalid: "Invalid format. Please use: Street, City, State ZIP"

**Next Node:** `SCHEDULE_PICKUP_DATE`

**Session Data Stored:** `pickupAddress: "123 Main St, Seattle, WA 98101"`

-----

### Step 4C: Schedule Pickup Date

**Node:** `SCHEDULE_PICKUP_DATE`

**Message:**

```
Honda HRX217 - Schedule Pickup

Reply with a number:
1. Tomorrow
2. In 2 days
3. In 3 days
4. Next week
5. Enter specific date
6. Call Shop to schedule
```

**Options:**

|Option|Pickup Date|Next Node           |
|------|-----------|--------------------|
|1     |+1 day     |CREATE_SERVICE_ORDER|
|2     |+2 days    |CREATE_SERVICE_ORDER|
|3     |+3 days    |CREATE_SERVICE_ORDER|
|4     |+7 days    |CREATE_SERVICE_ORDER|
|5     |Custom     |ENTER_SPECIFIC_PICKUP_DATE|
|6     |N/A        |TEXT_OR_CALL_DEALER |

**Session Data Stored:** `pickupDate: "2025-11-12"`

**Open Questions:**

- [x] **Should we allow customer to type a specific date?**
  - **Answer:** Yes, please allow the customer to type a specific date for pickup.
- [x] **What's the maximum advance scheduling allowed?**
  - **Answer:** For now, limit the advance scheduling to 3 months out from today.

-----

### Step 4D: Enter Specific Pickup Date (NEW)

**Node:** `ENTER_SPECIFIC_PICKUP_DATE`

**Message:**

```
Honda HRX217 - Schedule Pickup

Enter desired pickup date in format:
MM/DD/YYYY

Example: 11/15/2025

(Maximum 3 months from today)
```

**Input:** Free-form text (customer types date)

**Input Validation:**

- Must be valid date format (MM/DD/YYYY)
- Must be future date
- Must be within 3 months from today
- If invalid: "Invalid date. Please use format MM/DD/YYYY and select a date within the next 3 months."

**Next Node:** `CREATE_SERVICE_ORDER`

**Session Data Stored:** `pickupDate: "2025-11-15"`

-----

### Step 5: Create Service Order & Confirmation

**Node:** `CREATE_SERVICE_ORDER`

**Action:** Call Lizzy API to create service order (asynchronously)

**Required Lizzy API:**

- `POST /api/service-orders`
  
  ```json
  {
    "customer_id": "CUST123",
    "equipment_id": "EQ123",
    "service_type": "repair",
    "description": "[customer's problem description]",
    "drop_off_type": "drop_off_today",
    "preferred_date": "2025-11-11",
    "pickup_requested": true,
    "pickup_date": "2025-11-15",
    "pickup_address": "123 Main St, Seattle, WA 98101"
  }
  ```

**Immediate Message (Before API Response):**

```
Honda HRX217 - Schedule Service

Your service order has been submitted. Thank you for scheduling your service with [Dealer Name]! We will send you a copy of your order and the option to pay your invoice as soon as the order is confirmed.

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Follow-up Message (When API Completes Successfully):**

```
✅ Service Confirmed!

Order #SO789
Honda HRX217 - [Problem Description]

📄 View Invoice: [invoice_url]
💳 Pay Now: [payment_url]

Reply MENU for main menu
```

**Error Handling Message (If API Fails):**

```
Your service could not be scheduled automatically. Please contact [Dealer Name] to have your service scheduled.

Reply with a number:
1. Text Shop
2. Call Shop
3. Main Menu
```

**Options:**

|Option|Next Node|
|------|---------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU|

**Open Questions:**

- [x] **Should we send confirmation email/SMS separately?**
  - **Answer:** No. But do not wait for POST Service Order API to complete before releasing the customer. Immediately acknowledge: "Your service order has been submitted. Thank you for scheduling your service with [Dealer Name]! We will send you a copy of your order and the option to pay your invoice as soon as the order is confirmed."
- [x] **What if service order creation fails in Lizzy? Error handling?**
  - **Answer:** Provide message: "Your service could not be scheduled automatically. Please contact [Dealer Name] to have your service scheduled." Then display the option to Text or Call the dealer.
- [x] **Should we ask for problem description before creating order?**
  - **Answer:** Yes, please include a prompt to ask the customer to describe the equipment issue prior to submitting the POST create order API.

-----

## Workflow 3: Service Status

### User Story

As a customer, I want to check the status of my equipment currently being serviced so I know when it will be ready.

### Entry Point

- From Main Menu, customer selects option 4 → Starts at `SERVICE_STATUS_LIST`

### Flow Diagram

```
Main Menu (4) → Service Status List → Selected Order Details
```

### Step 1: Show Equipment in Service

**Node:** `SERVICE_STATUS_LIST`

**Message:** (Dynamic - generated from Lizzy API)

```
Equipment in Service - Reply with a number:
1. Honda HRX217 - In Service - SO789 - Est: Nov 15
2. Husqvarna 450 - In Service - SO456 - Est: Nov 18
3. Main Menu
```

**Options:** Dynamic - numbers 1-N based on active service orders

**Required Lizzy API:**

- None (uses cached transactions data, refreshes if cache is stale)

**Next Node:** `SERVICE_STATUS_DETAIL`

**Special Cases:**

- If customer has 0 equipment in service:
  
  ```
  You have no equipment currently in service.
  
  Reply with a number:
  1. Schedule Service
  2. Main Menu
  ```

**Open Questions:**

- [x] **Should we show estimated completion date in the list?**
  - **Answer:** Yes. But will need to confirm which field in Lizzy has this information.
- [x] **Should we include order number in the list?**
  - **Answer:** Yes. But will need to confirm which field in Lizzy has this information.
- [x] **Maximum number of orders to show?**
  - **Answer:** Yes, use the same pagination as Parts Orders (6 initially, up to 20 total).

-----

### Step 2: Show Service Status Details

**Node:** `SERVICE_STATUS_DETAIL`

**Message:** (Dynamic)

```
Service Order #SO789
Honda HRX217 - SN: MZCG-1234567

Status: In Progress
Estimated Completion: Nov 15, 2025

📄 View Invoice: [invoice_url]
💳 Pay Now: [payment_url]

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node|
|------|---------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU|

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] What status values are possible? (Pending, In Progress, Awaiting Parts, Complete, Ready for Pickup?)
- [ ] Should we show technician notes or problem diagnosis?
- [ ] Should we show parts that were ordered/installed?

-----

## Workflow 4: Order Parts

### User Story

As a customer, I want to order parts for my equipment so I can replace worn or broken components.

### Entry Points

- From Main Menu, customer selects option 4 → Starts at `ORDER_PARTS_MENU`
- From Troubleshoot Equipment workflow, customer selects "Order Parts" → Skips to `SHOW_PARTS_LIST` with equipment pre-selected
- From Schedule Service workflow, customer selects "Yes, order parts now" → Skips to `SHOW_PARTS_LIST` with equipment pre-selected

### Flow Diagram

```
Main Menu (4) → Order Parts Menu → Equipment Selection → Parts List → Part Details → Quantity → Shipping → Confirmation
```

### Step 1: Order Parts Menu

**Node:** `ORDER_PARTS_MENU`

**Message:**

```
Select Equipment to see list of common parts for your equipment - Reply with a number:
1. Send Full Fleet List
2. Filter Fleet List by Type
3. Select Equipment by SKU
4. Select Equipment by SN
```

**Options:**

|Option|Next Node              |
|------|-----------------------|
|1     |SHOW_FULL_FLEET_PARTS  |
|2     |FILTER_FLEET_PARTS     |
|3     |SELECT_BY_SKU_PARTS    |
|4     |SELECT_BY_SN_PARTS     |

-----

### Step 1A: Select Equipment by SKU (Parts)

**Node:** `SELECT_BY_SKU_PARTS`

**Message:**

```
Order Parts - Select by SKU

Please enter the equipment SKU:

Example: HRX217VKA
```

**Input:** Free-form text (customer types SKU)

**Input Validation:**
- Search cached equipment data for matching SKU
- Case-insensitive search
- If not found: "SKU not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SHOW_PARTS_LIST` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 1B: Select Equipment by Serial Number (Parts)

**Node:** `SELECT_BY_SN_PARTS`

**Message:**

```
Order Parts - Select by Serial Number

Please enter the equipment serial number:

Example: MZCG-1234567
```

**Input:** Free-form text (customer types serial number)

**Input Validation:**
- Search cached equipment data for matching serial number
- Case-insensitive search
- If not found: "Serial number not found in your equipment list. Please try again or reply MENU."

**Next Node:** `SHOW_PARTS_LIST` (if found)

**Session Data Stored:** `selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`

-----

### Step 2A: Show Full Fleet List

**Node:** `SHOW_FULL_FLEET_PARTS`

**Message:** (Dynamic)

```
Select equipment - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567
2. Husqvarna 450 - SKU: 456 - SN: 45012345
3. Echo PB-580T - SKU: 789 - SN: E12345678

Reply MENU for main menu
```

**Options:** Dynamic

**Required Lizzy API:**

- None (uses cached customer profile data)

**Next Node:** `SHOW_PARTS_LIST`

**Session Data Stored:** `selectedEquipmentId: "EQ123"`

-----

### Step 2B: Filter Fleet List

**Node:** `FILTER_FLEET_PARTS`

**Message:**

```
Filter by equipment type - Reply with a number:
1. Mowers
2. Weed Eaters
3. Leaf Blowers
4. Chainsaws
5. Other
```

**Options:**

|Option|Filter Value|Next Node                |
|------|------------|-------------------------|
|1     |Mowers      |FILTER_BY_MODEL_PARTS    |
|2     |Weed Eaters |FILTER_BY_MODEL_PARTS    |
|3     |Leaf Blowers|FILTER_BY_MODEL_PARTS    |
|4     |Chainsaws   |FILTER_BY_MODEL_PARTS    |
|5     |Other       |SHOW_FILTERED_FLEET_PARTS|

**Session Data Stored:** `filterUnitType: "Mowers"`

**Open Questions:**

- [x] **How does Lizzy categorize equipment?**
  - **Answer:** Uses two-layer filtering (Unit_Type → Model)
- [ ] **Should categories be configurable per dealer?**
- [x] **What equipment types should be included in "Other"?**
  - **Answer:** Equipment with no unit_type value

-----

### Step 2B-2: Filter by Model (Parts)

**Node:** `FILTER_BY_MODEL_PARTS`

**Message:** (Dynamic)

```
Mowers - Filter by Model - Reply with a number:
1. HRX217
2. HRR216
3. TimeMaster
[... models within selected Unit_Type]

Reply BACK to return to unit type selection
Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on models, plus BACK

**Next Node:** `SHOW_FILTERED_FLEET_PARTS` (or `FILTER_FLEET_PARTS` if BACK)

**Session Data Stored:** `filterModel: "HRX217"`

-----

### Step 2C: Show Filtered Fleet List

**Node:** `SHOW_FILTERED_FLEET_PARTS`

**Message:** (Dynamic - generated from Lizzy API based on filter selection)

```
Mowers - HRX217 - Reply with a number:
1. Honda HRX217 - SKU: 123 - SN: MZCG-1234567
2. Honda HRX217 - SKU: 456 - SN: MZCG-9876543
3. Toro TimeMaster - SKU: 789 - SN: 400012345
[... filtered equipment]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on filtered equipment count

**Required Lizzy API:**

- None (uses cached customer profile data, filtered locally)

**Next Node:** `SHOW_PARTS_LIST`

**Session Data Stored:** `selectedEquipmentId: "EQ123"`

**Special Cases:**
- If filter returns 0 equipment: "No equipment found matching your filter. Reply MENU to start over."

-----

### Step 3: Show Parts List

**Node:** `SHOW_PARTS_LIST`

**Message:** (Dynamic)

```
Parts for Honda HRX217 - Reply with a number:
1. Air Filter - #17211-ZL8-023
2. Spark Plug - #98079-55846
3. Blade - #72511-VH7-000
4. Oil Filter - #15400-PLM-A01
[... up to 10 items]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on parts available

**Required Lizzy API:**

- None (uses cached parts catalog from customer profile)

**Next Node:** `SHOW_PART_DETAIL`

**Session Data Stored:** `selectedPartNumber: "17211-ZL8-023"`

**Open Questions:**

- [ ] Should parts be categorized (Filters, Belts, Blades, etc.)?
- [ ] Should we show "Common Parts" vs "All Parts"?
- [ ] What if there are 100+ parts? Pagination strategy?
- [ ] Should we show part images?

-----

### Step 4: Show Part Details

**Node:** `SHOW_PART_DETAIL`

**Message:** (Dynamic)

```
Honda HRX217 - Order Part

Air Filter #17211-ZL8-023
💵 Price: $12.99
📦 Availability: In Stock

Enter quantity (1-99):
```

**Input:** Free-form number (customer types quantity)

**Input Validation:**

- Must be a number
- Must be between 1-99
- If invalid: "Please enter a quantity between 1 and 99"

**Next Node:** `PARTS_SHIPPING_OPTIONS`

**Session Data Stored:** `partQuantity: 2`

**Required Lizzy API:**

- None (uses cached parts catalog data)

**Open Questions:**

- [ ] Should we support multiple parts in one order? Or one part at a time?
- [ ] What availability statuses exist? (In Stock, Order, Out of Stock, Discontinued?)
- [ ] Should we show estimated delivery time?
- [ ] Maximum quantity limits per part?

-----

### Step 5: Shipping Options

**Node:** `PARTS_SHIPPING_OPTIONS`

**Message:** (Dynamic)

```
Honda HRX217 - Order Parts

Air Filter (Qty: 2)
Total: $25.98

Ship To: 123 Main St, Seattle, WA 98101

Reply with a number:
1. Ship to address above
2. Change Ship-To Address
3. Will Call (Pick up at shop)
4. Main Menu
```

**Options:**

|Option|Action             |Next Node              |
|------|-------------------|-----------------------|
|1     |Use primary address|CREATE_PARTS_ORDER     |
|2     |Enter new address  |CHANGE_SHIPPING_ADDRESS|
|3     |Will call          |CREATE_PARTS_ORDER     |
|4     |Cancel             |MAIN_MENU              |

**Session Data Stored:** `shippingOption: "ship_to_primary"`

**Required Lizzy API:**

- None (uses cached customer address data)

**Open Questions:**

- [ ] Should we show shipping cost estimate?
- [ ] For "Change Ship-To", how do we handle address entry via SMS? Multiple prompts?
- [ ] Should we validate addresses?
- [ ] Can customer save multiple addresses?

-----

### Step 6A: Change Shipping Address

**Node:** `CHANGE_SHIPPING_ADDRESS`

**Message:**

```
Honda HRX217 - Shipping Address

Enter shipping address in this format:
Street, City, State ZIP

Example: 123 Main St, Seattle, WA 98101
```

**Input:** Free-form text (customer types address)

**Input Validation:**

- Basic format checking
- If invalid: "Invalid format. Please use: Street, City, State ZIP"

**Next Node:** `CONFIRM_SHIPPING_ADDRESS`

**Session Data Stored:** `newShippingAddress: "123 Main St, Seattle, WA 98101"`

**Open Questions:**

- [ ] Should we validate address with USPS or other service?
- [ ] Should we show a confirmation before proceeding?
- [ ] Support for apartment/suite numbers?

-----

### Step 6B: Confirm Shipping Address

**Node:** `CONFIRM_SHIPPING_ADDRESS`

**Message:**

```
Honda HRX217 - Confirm Address

Ship Air Filter (Qty: 2) to:
123 Main St
Seattle, WA 98101

Reply with a number:
1. Yes, ship to this address
2. Re-enter address
3. Main Menu
```

**Options:**

|Option|Next Node              |
|------|-----------------------|
|1     |CREATE_PARTS_ORDER     |
|2     |CHANGE_SHIPPING_ADDRESS|
|3     |MAIN_MENU              |

-----

### Step 7: Create Parts Order & Confirmation

**Node:** `CREATE_PARTS_ORDER`

**Action:** Call Lizzy API to create parts order

**Required Lizzy API:**

- `POST /api/parts-orders`
  
  ```json
  {
    "customer_id": "CUST123",
    "equipment_id": "EQ123",
    "items": [
      {
        "part_number": "17211-ZL8-023",
        "quantity": 2
      }
    ],
    "ship_to": "ship_to_primary",
    "shipping_address": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    }
  }
  ```

**Response Expected:**

```json
{
  "order_id": "PO456",
  "invoice_id": "INV789",
  "total": 25.98,
  "invoice_url": "https://lizzy.com/invoice/INV789",
  "payment_url": "https://lizzy.com/pay/INV789",
  "estimated_delivery": "2025-11-18"
}
```

**Message:**

```
✅ Parts Order Confirmed!

Order #PO456
Honda HRX217 - Air Filter (Qty: 2)
Total: $25.98

📦 Ship to: 123 Main St, Seattle, WA
🚚 Est. Delivery: Nov 18, 2025

📄 View Invoice: [invoice_url]
💳 Pay Now: [payment_url]

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Special Case - Return to Service Scheduling:**
If customer came from `OFFER_PARTS_ORDER` workflow (partsOrderRequested = true):

```
✅ Parts Order Confirmed!

Order #PO456
Honda HRX217 - Air Filter (Qty: 2)
Total: $25.98

📦 Ship to: 123 Main St, Seattle, WA
🚚 Est. Delivery: Nov 18, 2025

Returning to service scheduling...

[Continues to SERVICE_DROPOFF_OPTIONS]
```

**Options:**

|Option|Next Node|
|------|---------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU|

**Open Questions:**

- [ ] Should we send separate order confirmation via email/SMS?
- [ ] What if parts order creation fails? Error handling strategy?
- [ ] Should we support "Add Another Part" option?
- [ ] Should payment be required before order is submitted?

-----

## Workflow 5: Parts Order Status

### User Story

As a customer, I want to check the status of my parts orders so I know when they will arrive.

### Entry Point

- From Main Menu, customer selects option 5 → Starts at `PARTS_STATUS_LIST`
- From Troubleshoot Equipment workflow, customer selects "Check Parts Order Status" → Starts at `PARTS_STATUS_LIST` with equipment context provided for filtering

### Flow Diagram

```
Main Menu (5) → Parts Order List → Selected Order Details
```

### Step 1: Show Parts Orders

**Node:** `PARTS_STATUS_LIST`

**Message:** (Dynamic)

```
Your Parts Orders - Reply with a number:
1. PO456 - Pending - Air Filter
2. PO789 - Shipped - Spark Plug
3. PO123 - Delivered - Blade
4. Show More Orders
5. Main Menu
```

**Note:** If equipment context is provided (from Troubleshoot workflow), filter parts orders to show only orders for that specific equipment.

**Options:** Dynamic - numbers 1-N based on orders (max 6 shown initially)

**Required Lizzy API:**

- None (uses cached transactions data, refreshes if stale)

**Next Node:** `PARTS_STATUS_DETAIL`

**Special Cases:**

- If customer has 0 parts orders:
  
  ```
  You have no parts orders.
  
  Reply with a number:
  1. Order Parts
  2. Main Menu
  ```

**Pagination:**

- Show first 6 orders
- Option to "Show More Orders" (shows next 14, capped at 20 total)

**Open Questions:**

- [ ] Should we only show recent orders (last 90 days)?
- [ ] Should completed/delivered orders be shown?
- [ ] How long should orders remain in the list?
- [ ] Sort by date (newest first)?

-----

### Step 2: Show Parts Order Details

**Node:** `PARTS_STATUS_DETAIL`

**Message:** (Dynamic)

```
Parts Order #PO456

Equipment: Honda HRX217
Status: Shipped
Tracking: 1Z999AA10123456784

Items:
- Air Filter (Qty: 2) - $25.98

🚚 Est. Delivery: Nov 18, 2025

📄 View Invoice: [invoice_url]
💳 Pay Now: [payment_url]

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node|
|------|---------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU|

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] What status values exist? (Pending, Processing, Shipped, Will Call Ready, Delivered?)
- [ ] Should we show tracking link for shipped orders?
- [ ] Should we show all line items or just summary?
- [ ] Should we support "Cancel Order" option for pending orders?

-----

## Workflow 6: View or Pay Invoice

### User Story

As a customer, I want to view and pay my invoices so I can keep my account current.

### Entry Point

- From Main Menu, customer selects option 6 → Starts at `INVOICE_TYPE_MENU`
- From Troubleshoot Equipment workflow, customer selects "View or Pay Invoice" → Starts at `INVOICE_TYPE_MENU` with equipment context provided for filtering

### Flow Diagram

```
Main Menu (6) → Invoice Type Menu → Invoice List → Invoice Details
```

### Step 1: Select Invoice Type

**Node:** `INVOICE_TYPE_MENU`

**Message:**

```
View or Pay Invoice - Reply with a number:
1. Unit Sale Invoice
2. Service Invoice
3. Parts Invoice
4. Call Shop
5. Text Shop
6. Main Menu
```

**Note:** If equipment context is provided (from Troubleshoot workflow), subsequent invoice lists will be filtered to show only invoices related to that specific equipment.

**Options:**

|Option|Filter Type|Next Node   |
|------|-----------|------------|
|1     |unit_sale  |INVOICE_LIST|
|2     |service    |INVOICE_LIST|
|3     |parts      |INVOICE_LIST|
|4     |N/A        |TEXT_OR_CALL_DEALER|
|5     |N/A        |TEXT_OR_CALL_DEALER|
|6     |N/A        |MAIN_MENU   |

**Session Data Stored:** `invoiceType: "service"`

**Open Questions:**

- [ ] Should we combine all invoice types into one list?
- [ ] Should we have an "All Invoices" option?
- [ ] What about other invoice types (accessories, rental, etc.)?

-----

### Step 2: Show Invoice List

**Node:** `INVOICE_LIST`

**Message:** (Dynamic - based on invoice type selected)

```
Service Invoices - Reply with a number:
1. INV789 - Unpaid - $150.00 - Nov 5
2. INV456 - Unpaid - $45.00 - Oct 28
3. INV123 - Paid - $220.00 - Oct 15
4. Show More Invoices
5. Main Menu
```

**Note:** If equipment context is provided (from Troubleshoot workflow), filter invoices to show only those related to that specific equipment.

**Options:** Dynamic - numbers 1-N (max 6 shown initially)

**Required Lizzy API:**

- None (uses cached transactions data, refreshes if stale)

**Next Node:** `INVOICE_DETAIL`

**Pagination:**

- Show first 6 invoices
- Option to "Show More" (shows next 14, capped at 20 total)

**Special Cases:**

- If customer has 0 invoices of selected type:
  
  ```
  You have no service invoices.
  
  Reply with a number:
  1. View other invoice types
  2. Main Menu
  ```

**Open Questions:**

- [ ] Should we only show unpaid invoices?
- [ ] Should we show overdue invoices separately?
- [ ] How far back should we show invoices (90 days, 1 year, all time)?
- [ ] Should we show payment status in the list?

-----

### Step 3: Show Invoice Details

**Node:** `INVOICE_DETAIL`

**Message:** (Dynamic)

```
Invoice #INV789
Service Order #SO456

Honda HRX217 - Oil Change & Tune Up

Amount: $150.00
Status: Unpaid
Due Date: Nov 15, 2025

📄 View Invoice: [invoice_url]
💳 Pay Now: [payment_url]

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node|
|------|---------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU|

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] Should we show line-item details (parts, labor, tax)?
- [ ] Should we show payment history if partially paid?
- [ ] Should we allow scheduling payment plans via SMS?
- [ ] What happens after customer clicks "Pay Now"? Do they return to SMS or stay on payment page?

-----

## Common Patterns

### Equipment Context Display Standard

**When to Display Equipment Context:**

- Any workflow step that occurs after equipment has been selected
- All transactional prompts (quantity entry, shipping options, confirmations)
- Order status and detail views

**Standard Format:**

```
Primary Format: "[Equipment] - [Action]"
Example: "Honda HRX217 - Schedule Service"

Detail Format: "[Equipment] - [Serial Number]\n[Content]"
Example: "Honda HRX217 - SN: MZCG-1234567\nTroubleshooting Resources:"

Confirmation Format: "[Action]\n[Equipment] - [Details]"
Example: "✅ Service Scheduled!\nHonda HRX217 - Drop off Today"
```

**Cross-Workflow Equipment Carry-Forward:**
When a customer selects equipment in one workflow (e.g., Troubleshoot Equipment) and then chooses an action that starts another workflow (e.g., "Schedule Service" or "Order Parts"), the system:

1. Stores equipment context in session (`selectedEquipmentId`, `selectedEquipmentName`, `selectedEquipmentSN`)
1. Skips the equipment selection step in the new workflow
1. Jumps directly to the next relevant step (e.g., `SERVICE_PROBLEM_DESCRIPTION` or `SHOW_PARTS_LIST`)
1. Displays equipment context in all subsequent messages

**Example Flow:**

```
Customer Path: Troubleshoot → Select "Honda HRX217" → Choose "Order Parts"
System Behavior: Skip ORDER_PARTS_MENU → Jump to SHOW_PARTS_LIST
First Message: "Parts for Honda HRX217 - Reply with a number:"
```

-----

### Text or Call Dealer Option (NEW NODE)

**Node:** `TEXT_OR_CALL_DEALER`

**Message:**

```
Contact [Dealer Name] - Reply with a number:
1. Text Shop
2. Call Shop
3. Main Menu
```

**Options:**

|Option|Next Node |
|------|----------|
|1     |TEXT_SHOP |
|2     |CALL_SHOP |
|3     |MAIN_MENU |

**Note:** This consolidated node replaces individual "Call Shop" and "Text Shop" options throughout workflows.

-----

### Call Shop Option

**Node:** `CALL_SHOP`

**Message:**

```
Call us at: (206) 555-0100

Hours: Mon-Fri 8am-6pm, Sat 9am-4pm

Reply MENU to return to main menu
```

**Options:**

- Customer can type "MENU" to return to Main Menu
- Otherwise, session remains active for 15 minutes

**Note:** This doesn't end the SMS session, just provides phone number

-----

### Text Shop Option (Dealer Handoff)

**Node:** `TEXT_SHOP`

**Action:** Flag conversation for dealer attention in Lizzy

**Message:**

```
A team member will respond shortly. You can continue texting at this number.
```

**Backend Actions:**

1. Set `session.dealerActive = true`
1. Call Lizzy API: `POST /api/sms/conversation/{phone}/flag`
1. Bot stops auto-responding
1. Dealer sees notification in Lizzy SMS interface
1. Dealer takes over conversation manually

**Session Behavior:**

- Bot will not respond to any further messages from this customer
- All messages are still logged to Lizzy
- Dealer types responses directly in Lizzy interface

**Reactivate Bot:**

- Dealer must manually "release" conversation back to bot in Lizzy
- OR customer types "MENU" to restart bot interaction

**Open Questions:**

- [ ] How does dealer "release" conversation back to bot?
- [ ] Should bot automatically reactivate after X hours of dealer inactivity?
- [ ] Should bot notify customer when dealer has taken over?

-----

### HELP Keyword Handler (NEW NODE)

**Node:** `HELP_MENU`

**Trigger:** Customer types "HELP" at any point

**Message:**

```
Need assistance? Reply with a number:
1. Text Shop
2. Call Shop
3. Main Menu
```

**Options:**

|Option|Next Node |
|------|----------|
|1     |TEXT_SHOP |
|2     |CALL_SHOP |
|3     |MAIN_MENU |

-----

### STOP Keyword Handler (NEW NODE)

**Node:** `STOP_CONFIRMATION`

**Trigger:** Customer types "STOP" at any point

**Message:**

```
You previously authorized this number to be used for communications with [Dealer Name]! 

Are you sure you want to stop all communications from [Dealer Name]?

Reply with a number:
1. Yes, stop all messages (Press Y)
2. Text Shop
3. Call Shop
4. Main Menu
```

**Options:**

|Option|Next Node |Action                                    |
|------|----------|------------------------------------------|
|Y     |N/A       |Unsubscribe customer, end session         |
|2     |TEXT_SHOP |                                          |
|3     |CALL_SHOP |                                          |
|4     |MAIN_MENU |                                          |

**Backend Actions (if Y selected):**
1. Call Lizzy API: `POST /api/sms/unsubscribe/{phone}`
1. End session
1. Add phone to do-not-contact list
1. Send final message: "You have been unsubscribed from [Dealer Name] messages."

-----

### Session Timeout Behavior

**Timeout:** 15 minutes of inactivity

**Behavior:**

- Session data is cleared from database
- Next message from customer triggers Main Menu
- Customer sees welcome message again

**Message After Timeout:**

```
Your session has expired.

[Shows Main Menu]
```

-----

### Invalid Input Handling

**For Numbered Menu Options:**

```
Invalid option. Please reply with a number from the menu.

[Repeat current menu]
```

**For Free-Form Input (quantity, address):**

```
Invalid format. [Specific instruction]

[Repeat prompt]
```

**After 3 Invalid Attempts:**

```
Having trouble? Reply with a number:
1. Text Shop for help
2. Call Shop
3. Main Menu
```

-----

### "MENU" Keyword

**Behavior:** Customer can type "MENU" at any point to return to Main Menu

**Message:**

```
Returning to main menu...

[Shows Main Menu]
```

**Session Handling:**

- Clears all session data except customer ID and phone
- Resets to MAIN_MENU node

-----

### SMS Character Limits

**Standard SMS:** 160 characters per message

**Strategy for Long Messages:**

1. Use link shorteners for URLs
1. Break long equipment/parts lists into multiple messages
1. Use concise wording
1. Consider MMS for messages with images (future enhancement)

**Example - Long Parts List:**

```
Message 1:
Parts for Honda HRX217:
1. Air Filter - #17211-ZL8-023
2. Spark Plug - #98079-55846
3. Blade - #72511-VH7-000

Message 2:
4. Oil Filter - #15400-PLM-A01
5. Belt - #22431-VH7-010

Reply with part # or MENU
```

-----

## Lizzy API Requirements

### API Consolidation Strategy

To optimize performance and reduce API calls, this system uses a **consolidated API approach** where related data is fetched in bulk and cached for the session duration. This reduces the total number of API calls from 16 to 7-9 per session.

-----

### Required APIs (7 Core + 2 Optional Detail APIs)

#### 1. Consolidated Customer Profile (Session Initialization)

**Endpoint:** `GET /api/customers/by-phone/{phone}?include=equipment,addresses,resources,parts`

**Purpose:** Single API call to retrieve complete customer profile with all related data at session start

**Request:**

```
GET /api/customers/by-phone/+12065550100?include=equipment,addresses,resources,parts
```

**Response:**

```json
{
  "customer_id": "CUST123",
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+12065550100",
  "addresses": {
    "primary": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    },
    "additional": []
  },
  "equipment": [
    {
      "equipment_id": "EQ123",
      "make": "Honda",
      "model": "HRX217",
      "unit_type": "Mower",
      "serial_number": "MZCG-1234567",
      "vin": "",
      "status": "Active",
      "purchase_date": "2024-05-15",
      "last_service_date": "2025-08-20",
      "resources": {
        "tutorial_video_url": "https://youtube.com/watch?v=...",
        "owners_manual_url": "https://lizzy.com/manuals/honda-hrx217.pdf",
        "parts_diagram_url": "https://lizzy.com/diagrams/honda-hrx217.pdf"
      },
      "parts_catalog": [
        {
          "part_number": "17211-ZL8-023",
          "description": "Air Filter",
          "price": 12.99,
          "availability": "in_stock",
          "category": "Filters"
        },
        {
          "part_number": "98079-55846",
          "description": "Spark Plug",
          "price": 8.99,
          "availability": "in_stock",
          "category": "Ignition"
        }
      ]
    },
    {
      "equipment_id": "EQ456",
      "make": "Husqvarna",
      "model": "450",
      "unit_type": "Chainsaw",
      "serial_number": "45012345",
      "status": "Active",
      "purchase_date": "2023-03-10",
      "last_service_date": "2025-06-15",
      "resources": {
        "tutorial_video_url": "https://youtube.com/watch?v=...",
        "owners_manual_url": "https://lizzy.com/manuals/husqvarna-450.pdf"
      },
      "parts_catalog": [
        {
          "part_number": "544287401",
          "description": "Chain",
          "price": 24.99,
          "availability": "in_stock",
          "category": "Cutting"
        }
      ]
    }
  ]
}
```

**Cache Duration:** Full 15-minute session

**Used In:** Session initialization when customer first texts

**Data Fields Explained:**

- `unit_type`: Equipment category for filtering (Mower, Chainsaw, Weed Eater, Leaf Blower, etc.)
- `model`: Specific model identifier (used for display and filtering)
- `status`: Current status (Active, In Service, Inactive)
- `resources`: Links to tutorials and manuals
- `parts_catalog`: Complete list of available parts for this equipment

-----

#### 2. Consolidated Transactions & Invoices

**Endpoint:** `GET /api/customers/{customer_id}/transactions?include=service_orders,parts_orders,invoices`

**Purpose:** Single API call to retrieve all current orders and invoices

**Request:**

```
GET /api/customers/CUST123/transactions?include=service_orders,parts_orders,invoices
```

**Response:**

```json
{
  "service_orders": [
    {
      "order_id": "SO789",
      "equipment_id": "EQ123",
      "equipment_name": "Honda HRX217",
      "status": "in_progress",
      "estimated_completion": "2025-11-12",
      "created_date": "2025-11-05",
      "description": "Oil change and tune up",
      "invoice": {
        "invoice_id": "INV456",
        "amount": 150.00,
        "status": "unpaid",
        "due_date": "2025-11-15",
        "invoice_url": "https://lizzy.com/invoice/INV456",
        "payment_url": "https://lizzy.com/pay/INV456"
      }
    }
  ],
  "parts_orders": [
    {
      "order_id": "PO456",
      "equipment_id": "EQ123",
      "equipment_name": "Honda HRX217",
      "status": "shipped",
      "tracking_number": "1Z999AA10123456784",
      "estimated_delivery": "2025-11-18",
      "created_date": "2025-11-10",
      "items": [
        {
          "part_number": "17211-ZL8-023",
          "description": "Air Filter",
          "quantity": 2,
          "price": 25.98
        }
      ],
      "invoice": {
        "invoice_id": "INV789",
        "amount": 25.98,
        "status": "unpaid",
        "invoice_url": "https://lizzy.com/invoice/INV789",
        "payment_url": "https://lizzy.com/pay/INV789"
      }
    }
  ],
  "invoices": [
    {
      "invoice_id": "INV999",
      "type": "unit_sale",
      "equipment_id": "EQ123",
      "amount": 1200.00,
      "status": "unpaid",
      "due_date": "2025-11-20",
      "created_date": "2025-10-15",
      "invoice_url": "https://lizzy.com/invoice/INV999",
      "payment_url": "https://lizzy.com/pay/INV999"
    }
  ]
}
```

**Cache Duration:** 5 minutes (refresh when accessed if cache is stale)

**Used In:** Service Status, Parts Order Status, View or Pay Invoice workflows

**Note:** Backend can filter by equipment_id when equipment context is provided from cross-workflow transitions

-----

#### 3. Create Service Order

**Endpoint:** `POST /api/service-orders`

**Purpose:** Create new service order

**Request:**

```json
{
  "customer_id": "CUST123",
  "equipment_id": "EQ123",
  "service_type": "maintenance",
  "description": "Customer reported: Engine won't start",
  "drop_off_type": "drop_off_today",
  "preferred_date": "2025-11-11",
  "pickup_requested": false,
  "pickup_date": null,
  "pickup_address": null
}
```

**Response:**

```json
{
  "order_id": "SO789",
  "invoice_id": "INV456",
  "status": "pending",
  "invoice_url": "https://lizzy.com/invoice/INV456",
  "payment_url": "https://lizzy.com/pay/INV456"
}
```

**Cache Invalidation:** After creating service order, invalidate transactions cache to force refresh on next access

-----

#### 4. Create Parts Order

**Endpoint:** `POST /api/parts-orders`

**Purpose:** Create new parts order

**Request:**

```json
{
  "customer_id": "CUST123",
  "equipment_id": "EQ123",
  "items": [
    {
      "part_number": "17211-ZL8-023",
      "quantity": 2
    }
  ],
  "ship_to": "ship_to_primary",
  "shipping_address": {
    "street": "123 Main St",
    "city": "Seattle",
    "state": "WA",
    "zip": "98101"
  }
}
```

**Response:**

```json
{
  "order_id": "PO456",
  "invoice_id": "INV789",
  "total": 25.98,
  "status": "pending",
  "invoice_url": "https://lizzy.com/invoice/INV789",
  "payment_url": "https://lizzy.com/pay/INV789",
  "estimated_delivery": "2025-11-18"
}
```

**Cache Invalidation:** After creating parts order, invalidate transactions cache to force refresh on next access

-----

#### 5. Log SMS Message

**Endpoint:** `POST /api/sms/log`

**Purpose:** Log SMS message to Lizzy SMS chat interface

**Request:**

```json
{
  "phone": "+12065550100",
  "customer_id": "CUST123",
  "direction": "inbound",
  "message": "I want to order parts",
  "timestamp": "2025-11-05T10:30:00Z",
  "source": "workflow_bot"
}
```

**Response:**

```json
{
  "message_id": "MSG123",
  "logged": true
}
```

-----

#### 6. Flag Conversation for Dealer

**Endpoint:** `POST /api/sms/conversation/{phone}/flag`

**Purpose:** Notify dealer that customer needs attention

**Request:**

```json
{
  "customer_id": "CUST123",
  "reason": "Customer requested dealer contact",
  "priority": "normal",
  "context": {
    "last_menu": "ORDER_PARTS_MENU",
    "selected_equipment": "EQ123"
  }
}
```

**Response:**

```json
{
  "conversation_id": "CONV123",
  "flagged": true,
  "dealer_notified": true
}
```

-----

#### 7. Get Conversation Status

**Endpoint:** `GET /api/sms/conversation/{phone}/status`

**Purpose:** Check if dealer has taken over conversation

**Request:**

```
GET /api/sms/conversation/+12065550100/status
```

**Response:**

```json
{
  "conversation_id": "CONV123",
  "dealer_active": true,
  "last_dealer_message": "2025-11-05T10:45:00Z",
  "bot_paused": true
}
```

-----

### Optional Detail APIs (If Not Included in Consolidated Transactions)

#### 8. Get Order Details (Optional)

**Endpoint:** `GET /api/orders/{order_id}/detail`

**Purpose:** Get extended order details with line items and technician notes

**Note:** Only needed if consolidated transactions API doesn't include full details

-----

#### 9. Get Invoice Details (Optional)

**Endpoint:** `GET /api/invoices/{invoice_id}/detail`

**Purpose:** Get extended invoice details with line item breakdown

**Note:** Only needed if consolidated transactions API doesn't include full details

-----

### API Call Pattern Per Session

**Session Start:**

1. Customer texts dealer → **1 API call** (consolidated customer profile)

**During Session:**

- Browse equipment/parts/resources → **0 additional calls** (all cached)
- Filter equipment by type → **0 additional calls** (filter cached data)
- Check service/parts status → **0-1 API calls** (use cache if <5 min old, else refresh)
- View invoices → **0-1 API calls** (use cache if <5 min old, else refresh)
- Create service order → **1 API call** (POST service-order)
- Create parts order → **1 API call** (POST parts-order)
- View order details → **0-1 API calls** (depends if included in transactions)

**Typical Session:** 2-4 API calls total (vs 8-12 with non-consolidated approach)

-----

### Backend Implementation Requirements

#### DynamoDB Session Table Structure

**Table Name:** `LizzySMSSessions`

**Primary Key:**
- Partition Key: `phone` (String) - Customer phone number in E.164 format
- Sort Key: Not required for this use case

**Attributes:**

```json
{
  "phone": "+12065550100",
  "customerId": "CUST123",
  "currentNode": "MAIN_MENU",
  "cachedData": {
    "customerProfile": { /* full profile from API 1 */ },
    "profileCacheTime": "2025-11-11T10:30:00Z",
    "transactions": { /* from API 2 */ },
    "transactionsCacheTime": "2025-11-11T10:35:00Z"
  },
  "sessionData": {
    "selectedEquipmentId": "EQ123",
    "selectedEquipmentName": "Honda HRX217",
    "selectedEquipmentSN": "MZCG-1234567",
    "filterUnitType": "Mowers",
    "filterModel": "HRX217",
    "problemDescription": "Engine won't start",
    "partsOrderRequested": false
  },
  "lastActivity": "2025-11-11T10:40:00Z",
  "ttl": 1731325500
}
```

**TTL Configuration:**
- Enable TTL on the `ttl` attribute
- Set to current timestamp + 900 seconds (15 minutes)
- DynamoDB automatically deletes expired items
- Update TTL on each customer interaction

**Indexes:**
- No secondary indexes required for MVP
- Consider adding GSI on `customerId` for future dealer dashboard features

**Capacity Mode:**
- **Recommended:** On-Demand (pay per request)
  - Best for unpredictable traffic patterns
  - No capacity planning needed
  - Automatic scaling
- **Alternative:** Provisioned (if traffic is predictable)
  - Set RCU/WCU based on expected load
  - Enable auto-scaling
```

#### Cache Refresh Logic

```javascript
// DynamoDB session operations using AWS SDK v3
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, GetCommand, PutCommand, UpdateCommand } = require("@aws-sdk/lib-dynamodb");

const client = new DynamoDBClient({ region: "us-east-1" });
const docClient = DynamoDBDocumentClient.from(client);

// Get session from DynamoDB
async function getSession(phone) {
  const command = new GetCommand({
    TableName: "LizzySMSSessions",
    Key: { phone }
  });
  const response = await docClient.send(command);
  return response.Item;
}

// Save/Update session in DynamoDB
async function saveSession(session) {
  const ttl = Math.floor(Date.now() / 1000) + 900; // 15 minutes from now
  const command = new PutCommand({
    TableName: "LizzySMSSessions",
    Item: {
      ...session,
      lastActivity: new Date().toISOString(),
      ttl
    }
  });
  await docClient.send(command);
}

// Refresh transactions cache if older than 5 minutes
async function getTransactions(session) {
  const cacheAge = Date.now() - new Date(session.cachedData.transactionsCacheTime);
  const fiveMinutes = 5 * 60 * 1000;
  
  if (!session.cachedData.transactions || cacheAge > fiveMinutes) {
    const fresh = await lizzyAPI.getTransactions(session.customerId);
    session.cachedData.transactions = fresh;
    session.cachedData.transactionsCacheTime = new Date().toISOString();
    await saveSession(session);
  }
  
  return session.cachedData.transactions;
}
```

#### Local Data Filtering

```javascript
// Filter equipment by unit_type without API call
function filterEquipment(session, unitType) {
  return session.cachedData.customerProfile.equipment
    .filter(e => e.unit_type === unitType);
}

// Filter equipment by model within unit_type
function filterByModel(session, unitType, model) {
  return session.cachedData.customerProfile.equipment
    .filter(e => e.unit_type === unitType && e.model === model);
}

// Get parts for equipment without API call
function getPartsForEquipment(session, equipmentId) {
  const equipment = session.cachedData.customerProfile.equipment
    .find(e => e.equipment_id === equipmentId);
  return equipment?.parts_catalog || [];
}

// Filter transactions by equipment
function getEquipmentOrders(session, equipmentId) {
  const txns = session.cachedData.transactions;
  return {
    serviceOrders: txns.service_orders.filter(o => o.equipment_id === equipmentId),
    partsOrders: txns.parts_orders.filter(o => o.equipment_id === equipmentId),
    invoices: txns.invoices.filter(i => i.equipment_id === equipmentId)
  };
}
```

-----

### Benefits of Consolidated API Approach

1. **Performance:** 2-4 API calls per session vs 8-12 with non-consolidated approach
1. **Reduced Load:** Fewer requests to Lizzy servers
1. **Better UX:** Instant responses when browsing equipment/parts (no loading delays)
1. **Resilience:** If Lizzy API is temporarily slow, cached data keeps system responsive
1. **Simplified Code:** Backend filtering vs complex API orchestration

-----

### Trade-offs & Considerations

#### Payload Size

- Initial customer profile call returns more data (potentially 50-100KB for customer with 10+ pieces of equipment)
- Acceptable for SMS workflow where we need this data anyway
- Consider pagination if customers regularly have 50+ pieces of equipment

#### Data Freshness

- **Equipment/Parts/Resources:** Static enough for 15-min session cache (rarely change during conversation)
- **Prices:** Could change, but acceptable risk for session duration
- **Order Statuses:** 5-min cache with refresh - balances freshness with performance
- **Addresses:** Static enough for session cache

#### Cache Invalidation

- After creating service/parts order, invalidate transactions cache
- On session timeout, all cache is cleared
- If customer reports stale data, manual refresh available

-----

## Open Questions

### High Priority

1. **Dealer SMS Integration**
- [ ] Does Lizzy currently use Twilio or different SMS provider?
- [ ] Do dealers already have SMS numbers in Lizzy?
- [ ] How does the existing SMS chat interface work?
1. **Lizzy API Authentication**
- [ ] How does authentication work for Lizzy APIs?
- [ ] API keys? OAuth? JWT?
- [ ] Rate limiting considerations?
1. **Phone Number Format**
- [ ] What format does Lizzy store phone numbers? (E.164: +12065550100?)
- [ ] Do we need to normalize incoming phone numbers?
1. **Multi-Dealer Support**
- [ ] Is this a single-dealer deployment or multi-dealer platform?
- [ ] How do we identify which dealer a customer belongs to?
- [ ] Separate phone numbers per dealer?
1. **Equipment Categorization**
- [ ] Should categories be configurable per dealer?

### Medium Priority

1. **Payment Processing**
- [ ] What payment gateway does Lizzy use?
- [ ] Do payment links support Apple Pay / Google Pay?
- [ ] Can customers save payment methods?
1. **Error Handling**
- [ ] What should happen if Lizzy API is down?
- [ ] Should we queue failed API calls and retry?
- [ ] How to communicate errors to customer?
1. **Parts Ordering**
- [ ] Should we support multiple parts in one order?
- [ ] Should we show related/recommended parts?
- [ ] Minimum order amounts?
1. **Inventory Management**
- [ ] Real-time inventory checking for parts?
- [ ] What if part goes out of stock during order process?
- [ ] Backorder handling?
1. **Service Status Details**
- [ ] What status values are possible? (Pending, In Progress, Awaiting Parts, Complete, Ready for Pickup?)
- [ ] Should we show technician notes or problem diagnosis?
- [ ] Should we show parts that were ordered/installed?

### Low Priority

1. **Business Hours**
- [ ] Should bot behavior change outside business hours?
- [ ] Different messages after hours?
1. **Analytics & Reporting**
- [ ] What metrics should we track?
- [ ] Conversion rates, drop-off points, popular flows?
1. **A/B Testing**
- [ ] Should we support testing different message wording?
- [ ] Different menu structures?

-----

## Change Log

|Date      |Version|Changes                                                                                                                                       |Author  |
|----------|-------|----------------------------------------------------------------------------------------------------------------------------------------------|--------|
|2025-11-11|1.0    |Initial draft created                                                                                                                         |Jonathan|
|2025-11-13|1.1    |Consolidated API approach - reduced from 16 to 7-9 APIs. Added `unit_type` and `model` fields to equipment data. Added DynamoDB caching strategy.|Jonathan|
|2025-11-13|1.2    |Answered 20+ open questions. Added SKU/SN search, two-layer filtering, conditional menu options, service workflow enhancements (problem description, pickup address, parts order integration), HELP/STOP keyword support. Added SKU/SN selection to Schedule Service and Order Parts workflows. Replaced Redis with DynamoDB for session caching.|Jonathan|

-----

## Next Steps

1. **Review this document** and add comments/questions
1. **Work with Lizzy team** to confirm API availability and field names
1. **Create workflow flowcharts** (Mermaid diagrams)
1. **Build workflow engine** (JavaScript implementation)
1. **Set up Twilio** account and test environment
1. **Create test scripts** for each workflow

-----

## Appendix

### Glossary

- **Workflow Node:** A specific point in the conversation flow with a message and options
- **Session:** Temporary storage of customer's current position and data in the workflow
- **Equipment Context:** The selected equipment information (ID, name, serial number) that carries forward through multi-step workflows
- **Cross-Workflow Transition:** When a customer moves from one workflow to another with equipment pre-selected, skipping redundant selection steps
- **Dealer Handoff:** When bot stops responding and human dealer takes over conversation
- **Will Call:** Customer picks up parts/equipment at shop instead of shipping
- **Two-Layer Filtering:** First filter by Unit_Type, then by Model within that type

### Assumptions

- Customer has smartphone capable of receiving SMS
- Customer phone number is registered in Lizzy system
- Dealer has trained staff monitoring Lizzy SMS interface
- Payment links work on mobile devices
- Links in SMS are clickable on customer's device

### Success Metrics (Future)

- % of customers who complete a workflow
- Average time to complete parts order
- % of customers who use "Text Shop" vs completing workflow
- Customer satisfaction scores
- Reduction in phone call volume to dealer
