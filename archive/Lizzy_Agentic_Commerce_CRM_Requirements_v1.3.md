# Lizzy Agentic Commerce CRM - Workflow Requirements

**Version:** 1.3.0  
**Last Updated:** November 18, 2025  
**Status:** Active - Shared with Stakeholders

**📋 Implementation Note:** An MVP implementation guide using Twilio Studio (minimal coding required) is available separately. See `Twilio_Studio_MVP_Implementation_Guide.md` for a rapid prototyping approach that addresses critical limitations through strategic compromises. This MVP path enables validation before investing in the full custom Node.js engine described in this document.

-----

## Document Version History

| Version | Date | Author | Changes | Status |
|---------|------|--------|---------|--------|
| 1.3.0 | 2025-11-18 | Jonathan | 8 UX improvements (see CHANGELOG.md) | **Current** |
| 1.2.0 | 2025-11-13 | Jonathan | Added SKU/SN search capabilities | Superseded |
| 1.1.0 | 2025-11-10 | Jonathan | Added two-layer filtering | Superseded |
| 1.0.0 | 2025-11-05 | Jonathan | Initial requirements | Superseded |

**Quick Links:**
- [View Complete Changelog](CHANGELOG.md)
- [View v1.2 → v1.3 Changes](Requirements_Update_Summary_Nov18.md)

-----

## What's New in v1.3

**Summary:** 8 user experience improvements focused on simplifying navigation and clarifying messaging.

**Major Changes:**
- ⭐ Main Menu re-sequenced (Order Parts promoted to #2)
- ⭐ In-service equipment menu simplified (5 options vs 7)
- ⭐ BACK navigation added to filter menus
- ⭐ SKU field added to all equipment lists
- ⭐ Key Principles simplified (removed obsolete API framework)

**See:** [Full v1.3 Change Summary](Requirements_Update_Summary_Nov18.md)

-----

## Table of Contents

1. [Overview](#overview)
1. [Key Principles](#key-principles)
1. [Main Menu](#main-menu)
1. [Workflow 1: Troubleshoot Equipment](#workflow-1-troubleshoot-equipment)
1. [Workflow 2: Schedule Service](#workflow-2-schedule-service)
1. [Workflow 3: Service Status](#workflow-3-service-status)
1. [Workflow 4: Order Parts](#workflow-4-order-parts)
1. [Workflow 5: Parts Order Status](#workflow-5-parts-order-status)
1. [Workflow 6: View or Pay Invoice](#workflow-6-view-or-pay-invoice)
1. [Workflow 7: Text or Call Dealer](#workflow-7-text-or-call-dealer)
1. [Cross-Workflow Equipment Context](#cross-workflow-equipment-context)
1. [Lizzy API Requirements Summary](#lizzy-api-requirements-summary)
1. [Open Questions & Assumptions](#open-questions--assumptions)

-----

## Overview

### Purpose

Enable Lizzy customers to interact with their dealer via SMS text messaging for:
- Equipment troubleshooting and resource access
- Service scheduling and status checking
- Parts ordering and order tracking
- Invoice viewing and payment
- Direct dealer communication

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

**Session Initialization:**
1. Customer sends any text to dealer number
2. System creates session (15 min TTL)
3. API Call 1: GET consolidated customer profile (cached 15 min)
4. API Call 2: GET consolidated transactions (cached 5 min)
5. Display Main Menu

**Session Persistence:**
- Session data stored in PostgreSQL
- Customer data cached in DynamoDB
- Session expires after 15 min inactivity
- Customer typing "MENU" returns to Main Menu without new session

**Session Termination:**
- After 15 min inactivity → Auto-expire, next message creates new session
- Customer completes workflow → Option to return to Main Menu or end
- Customer types "STOP" → Unsubscribe (Twilio handles)

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
Invalid option. Please reply with a number 1-6, or reply MENU for the main menu.
```

Then re-display Main Menu.

### Required Lizzy API

- **GET** `/customers/by-phone/{phone}` - Look up customer by phone number

**If customer not found:**

```
You do not have an account with [Dealer Name]! Please contact [Dealer Name] to setup an account.

Reply with a number:
1. Text Shop
2. Call Shop
3. Return to Menu
```

|Option|Action                                       |
|------|---------------------------------------------|
|1     |Route to TEXT_OR_CALL_DEALER (Text selected)|
|2     |Route to TEXT_OR_CALL_DEALER (Call selected)|
|3     |Return to MAIN_MENU                          |

-----

## Workflow 1: Troubleshoot Equipment

### User Story

As a customer, I want to troubleshoot my equipment by accessing tutorial videos and owner's manuals so I can attempt to fix issues myself before scheduling service.

### Entry Point

- From Main Menu, customer selects option 1 → Starts at `TROUBLESHOOT_MENU`

### Flow Diagram

```
Main Menu (1) → Equipment Selection → Equipment Resources Display → Action Menu
```

### Step 1: Equipment Selection Method

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

|Option|Label                     |Next Node                      |
|------|--------------------------|-------------------------------|
|1     |Send Full Fleet List      |SHOW_FULL_FLEET_TROUBLESHOOT   |
|2     |Filter Fleet List by Type |FILTER_FLEET_TROUBLESHOOT      |
|3     |Select Equipment by SKU   |SELECT_BY_SKU_TROUBLESHOOT     |
|4     |Select Equipment by SN    |SELECT_BY_SN_TROUBLESHOOT      |

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

**Options:**

|Option|Next Node              |Notes                                                           |
|------|-----------------------|----------------------------------------------------------------|
|1     |SERVICE_PROBLEM_DESCRIPTION|(skips SCHEDULE_SERVICE_MENU, equipment pre-selected)           |
|2     |SHOW_PARTS_LIST        |(skips ORDER_PARTS_MENU, equipment pre-selected)                |
|3     |PARTS_STATUS_LIST      |(equipment context provided for filtering)                      |
|4     |INVOICE_TYPE_MENU      |(equipment context provided for filtering)                      |
|5     |TEXT_OR_CALL_DEALER    |NEW - Combined dealer contact option                            |
|6     |MAIN_MENU              |                                                                |

-----

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

When customer selects options 1-5, the following data is carried to the next workflow:
- `selectedEquipmentId: "EQ123"`
- `selectedEquipmentName: "Honda HRX217"`
- `selectedEquipmentSN: "MZCG-1234567"`

This allows the subsequent workflows to skip equipment selection and jump directly to their primary function.

**Required Lizzy API:**

- None (equipment data already cached from session init)
- Resources (video/manual URLs) should be included in equipment data

**Open Questions:**

- [x] **Should we track which resources customers access?**
  - **Answer:** Not required for MVP, but log clicks for future analytics
- [x] **What if resources don't exist for a piece of equipment?**
  - **Answer:** Show "Resources not available. Contact dealer for assistance."
- [ ] **Do tutorial videos need to be hosted on Lizzy or can they link to YouTube?**

-----

## Workflow 2: Schedule Service

### User Story

As a customer, I want to schedule service for my equipment so my dealer knows when to expect my equipment for repair or maintenance.

### Entry Point

- From Main Menu, customer selects option 3 → Starts at `SCHEDULE_SERVICE_MENU`
- From Troubleshoot Equipment workflow, customer selects "Schedule Service" → Skips to `SERVICE_PROBLEM_DESCRIPTION` with equipment pre-selected

### Flow Diagram

```
Main Menu (3) → Equipment Selection → Problem Description → Drop-off/Pickup Selection → Confirmation
```

### Step 1: Equipment Selection Method

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

|Option|Label                     |Next Node                   |
|------|--------------------------|----------------------------|
|1     |Send Full Fleet List      |SHOW_FULL_FLEET_SERVICE     |
|2     |Filter Fleet List by Type |FILTER_FLEET_SERVICE        |
|3     |Select Equipment by SKU   |SELECT_BY_SKU_SERVICE       |
|4     |Select Equipment by SN    |SELECT_BY_SN_SERVICE        |

**Special Case:** If entering from Troubleshoot Equipment workflow with equipment already selected, skip this node and go directly to `SERVICE_PROBLEM_DESCRIPTION`

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

-----

### Step 2B: Filter by Unit Type (Service)

**Node:** `FILTER_FLEET_SERVICE`

**Message:**

```
Filter by Unit Type - Reply with a number:
1. Mowers
2. Weed Eaters
3. Leaf Blowers
4. Chainsaws
5. Other
```

**Options:**

|Option|Filter Value|Next Node                 |
|------|------------|--------------------------|
|1     |Mowers      |FILTER_BY_MODEL_SERVICE   |
|2     |Weed Eaters |FILTER_BY_MODEL_SERVICE   |
|3     |Leaf Blowers|FILTER_BY_MODEL_SERVICE   |
|4     |Chainsaws   |FILTER_BY_MODEL_SERVICE   |
|5     |Other       |SHOW_FILTERED_FLEET_SERVICE|

**Session Data Stored:** `filterUnitType: "Mowers"`

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

- Minimum length: 10 characters
- Maximum length: 500 characters
- If too short: "Please provide more detail (at least 10 characters)"
- If too long: "Description too long. Please limit to 500 characters."

**Next Node:** `SERVICE_DROPOFF_PICKUP`

**Session Data Stored:** `problemDescription: "Engine won't start"`

**Open Questions:**

- [x] **Should we offer pre-defined problem categories?**
  - **Answer:** No, free-form text provides better information for dealer
- [ ] **Should we use AI to categorize the problem?**

-----

### Step 3: Drop-off or Pickup Selection

**Node:** `SERVICE_DROPOFF_PICKUP`

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

|Option|Label            |Next Node                   |Notes                                       |
|------|-----------------|----------------------------|--------------------------------------------|
|1     |Drop off Today   |SERVICE_CONFIRMATION        |Create service order with today's date      |
|2     |Drop off Tomorrow|SERVICE_CONFIRMATION        |Create service order with tomorrow's date   |
|3     |Schedule Pickup  |SERVICE_PICKUP_ADDRESS      |Request pickup address and date             |
|4     |Main Menu        |MAIN_MENU                   |Abandon service scheduling                  |

**Session Data Stored:** `dropOffType: "today"|"tomorrow"|"pickup"`

-----

### Step 3A: Pickup Address Confirmation

**Node:** `SERVICE_PICKUP_ADDRESS`

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

|Option|Label                 |Next Node                        |
|------|----------------------|---------------------------------|
|1     |Use this address      |SERVICE_PICKUP_DATE              |
|2     |Enter different address|SERVICE_PICKUP_ADDRESS_ENTRY    |
|3     |Main Menu             |MAIN_MENU                        |

**Required Lizzy API:**

- None (address from cached customer profile)

-----

### Step 3B: Pickup Address Entry

**Node:** `SERVICE_PICKUP_ADDRESS_ENTRY`

**Message:**

```
Honda HRX217 - Schedule Pickup

Please enter the pickup address:

Example: 456 Oak Ave, Seattle, WA 98102
```

**Input:** Free-form text (customer types address)

**Input Validation:**

- Basic validation: Must contain street, city, state, zip
- If invalid: "Invalid address format. Please include street, city, state, and ZIP code."

**Next Node:** `SERVICE_PICKUP_DATE`

**Session Data Stored:** `pickupAddress: "456 Oak Ave, Seattle, WA 98102"`

**Open Questions:**

- [ ] **Should we validate address against USPS API?**
- [ ] **What if address is outside service area?**

-----

### Step 3C: Pickup Date Selection

**Node:** `SERVICE_PICKUP_DATE`

**Message:**

```
Honda HRX217 - Schedule Pickup

When would you like pickup? Reply with a number:
1. Tomorrow
2. In 2 days
3. In 3 days
4. Next week
5. Enter specific date
6. Call Shop to schedule
```

**Options:**

|Option|Label             |Next Node           |Notes                           |
|------|------------------|--------------------|--------------------------------|
|1     |Tomorrow          |SERVICE_CONFIRMATION|Calculate tomorrow's date       |
|2     |In 2 days         |SERVICE_CONFIRMATION|Calculate date                  |
|3     |In 3 days         |SERVICE_CONFIRMATION|Calculate date                  |
|4     |Next week         |SERVICE_CONFIRMATION|Calculate date 7 days from now  |
|5     |Enter specific date|SERVICE_PICKUP_DATE_ENTRY|Allow custom date entry   |
|6     |Call Shop         |TEXT_OR_CALL_DEALER |Transfer to dealer              |

**Session Data Stored:** `pickupDate: "2025-11-20"`

-----

### Step 3D: Custom Pickup Date Entry

**Node:** `SERVICE_PICKUP_DATE_ENTRY`

**Message:**

```
Honda HRX217 - Schedule Pickup

Please enter the pickup date (MM/DD/YYYY):

Example: 11/25/2025
```

**Input:** Date in MM/DD/YYYY format

**Input Validation:**

- Must be valid date format
- Must be future date (not past)
- Must be within 90 days
- If invalid: "Invalid date. Please use MM/DD/YYYY format (e.g., 11/25/2025)"

**Next Node:** `SERVICE_CONFIRMATION`

**Session Data Stored:** `pickupDate: "2025-11-25"`

-----

### Step 4: Service Order Confirmation

**Node:** `SERVICE_CONFIRMATION`

**Message:**

```
Honda HRX217 - Schedule Service

Service Details:
Problem: [customer's description]
Drop-off: [Today / Tomorrow / Pickup on MM/DD/YYYY]
[If pickup: Pickup Address: [address]]

Creating your service order...
```

**Required Lizzy API:**

- **POST** `/service-orders`
  - Request Body:
    ```json
    {
      "customer_id": "CUST123",
      "equipment_id": "EQ123",
      "service_type": "repair",
      "description": "[customer's problem description]",
      "drop_off_type": "today"|"tomorrow"|"pickup",
      "pickup_date": "2025-11-20",
      "pickup_address": "[if different from default]"
    }
    ```
  - Response:
    ```json
    {
      "order_id": "SO789",
      "invoice_id": "INV456",
      "invoice_url": "https://lizzy.com/invoice/INV456",
      "payment_url": "https://lizzy.com/pay/INV456"
    }
    ```

**Next Node:** `SERVICE_COMPLETE`

-----

### Step 5: Service Order Complete

**Node:** `SERVICE_COMPLETE`

**Message:**

```
✅ Service Scheduled!

Order #SO789
Honda HRX217
Drop-off: [Today / Tomorrow / Pickup on MM/DD/YYYY]

Thank you for scheduling your service with [Dealer Name]!

📄 View Invoice: [URL]
💳 Pay Invoice: [URL]

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node          |
|------|-------------------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU          |

**Open Questions:**

- [x] **Should invoice be created immediately upon service order creation?**
  - **Answer:** Yes, invoice should be generated so customer can pre-pay if desired
- [ ] **Should we send confirmation via email as well?**

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
1. Honda HRX217 - #SO789 - In Progress - Est: Nov 15
2. Husqvarna 450 - #SO456 - Awaiting Parts - Est: Nov 20
[... all equipment currently in service]

Reply MENU for main menu
```

**If no equipment in service:**

```
You have no equipment currently in service.

Would you like to:
1. Schedule Service
2. Main Menu
```

**Options (if equipment in service):**

Dynamic - numbers 1-N based on equipment in service count

**Options (if no equipment in service):**

|Option|Next Node            |
|------|---------------------|
|1     |SCHEDULE_SERVICE_MENU|
|2     |MAIN_MENU            |

**Required Lizzy API:**

- None (uses cached transactions data from session initialization)
- Filter: `status = "in_service"` or `status = "in_progress"`

**Next Node:** `SERVICE_STATUS_DETAIL`

**Session Data Stored:** `selectedOrderId: "SO789"`

**Open Questions:**

- [x] **What statuses indicate "in service"?**
  - **Answer:** Need complete list from Lizzy team. Assume: "in_progress", "awaiting_parts", "ready_for_pickup"
- [ ] **Should we show completed service in past 30 days?**

-----

### Step 2: Service Order Details

**Node:** `SERVICE_STATUS_DETAIL`

**Message:** (Dynamic)

```
Service Order #SO789
Honda HRX217 - SN: MZCG-1234567

Status: In Progress
Estimated Completion: Nov 15, 2025

Problem: Engine won't start
Drop-off Date: Nov 10, 2025

📄 View Invoice: [URL]
💳 Pay Invoice: [URL]

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node          |
|------|-------------------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU          |

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] **Should we allow customers to cancel service orders via SMS?**

-----

## Workflow 4: Order Parts

### User Story

As a customer, I want to order parts for my equipment so I can perform repairs myself or have parts ready for my dealer to install.

### Entry Point

- From Main Menu, customer selects option 2 → Starts at `ORDER_PARTS_MENU`
- From Troubleshoot Equipment workflow, customer selects "Order Parts" → Skips to `SHOW_PARTS_LIST` with equipment pre-selected

### Flow Diagram

```
Main Menu (2) → Equipment Selection → Parts List → Quantity → Address → Confirmation
```

### Step 1: Equipment Selection Method

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

### Step 2B: Filter by Unit Type (Parts)

**Node:** `FILTER_FLEET_PARTS`

**Message:**

```
Filter by Unit Type - Reply with a number:
1. Mowers
2. Weed Eaters
3. Leaf Blowers
4. Chainsaws
5. Other
```

**Options:**

|Option|Filter Value|Next Node            |
|------|------------|---------------------|
|1     |Mowers      |FILTER_BY_MODEL_PARTS|
|2     |Weed Eaters |FILTER_BY_MODEL_PARTS|
|3     |Leaf Blowers|FILTER_BY_MODEL_PARTS|
|4     |Chainsaws   |FILTER_BY_MODEL_PARTS|
|5     |Other       |SHOW_FILTERED_FLEET_PARTS|

**Options:** Dynamic

**Next Node:** `FILTER_BY_MODEL_PARTS`

**Session Data Stored:** `filterUnitType: "Mowers"`

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

**Options:** Dynamic

**Next Node:** `SHOW_PARTS_LIST`

-----

### Step 3: Show Parts List

**Node:** `SHOW_PARTS_LIST`

**Message:** (Dynamic - generated from Lizzy API)

```
Honda HRX217 - Order Parts

Reply with a number:
1. Air Filter - $12.99
2. Spark Plug - $8.99
3. Blade - $45.99
4. Drive Belt - $25.99
5. Oil (Quart) - $9.99
[... up to 10 parts]

Reply MENU for main menu
```

**Options:** Dynamic - numbers 1-N based on available parts

**Required Lizzy API:**

- None (parts catalog included in cached customer profile for each equipment)

**Next Node:** `PARTS_QUANTITY`

**Session Data Stored:** `selectedPartNumber: "17211-ZL8-023"`

**Open Questions:**

- [x] **What if equipment has >10 parts?**
  - **Answer:** Show "common parts" (top 10 most frequently ordered), with option to "Text Shop for Other Parts"
- [x] **How should parts be sorted?**
  - **Answer:** By frequency of orders (most common first)
- [ ] **Should we show part numbers?**
- [ ] **Should we show stock status (in stock / out of stock)?**

-----

### Step 4: Part Quantity

**Node:** `PARTS_QUANTITY`

**Message:**

```
Honda HRX217 - Air Filter

Price: $12.99 ea
Part #: 17211-ZL8-023

How many would you like to order? Reply with a number:

(Or reply MENU for main menu)
```

**Input:** Number (1-99)

**Input Validation:**

- Must be integer 1-99
- If invalid: "Please enter a quantity between 1 and 99"

**Next Node:** `PARTS_ADDRESS`

**Session Data Stored:** `partQuantity: 2`

-----

### Step 5: Shipping Address

**Node:** `PARTS_ADDRESS`

**Message:**

```
Honda HRX217 - Air Filter (Qty: 2)
Total: $25.98

Ship To:
123 Main St
Seattle, WA 98101

Reply with a number:
1. Ship to this address
2. Change ship-to address
3. Will Call (Pick up at shop)
4. Main Menu
```

**Options:**

|Option|Label                   |Next Node                |
|------|------------------------|-------------------------|
|1     |Ship to this address    |PARTS_CONFIRMATION       |
|2     |Change ship-to address  |PARTS_ADDRESS_ENTRY      |
|3     |Will Call               |PARTS_CONFIRMATION       |
|4     |Main Menu               |MAIN_MENU                |

**Required Lizzy API:**

- None (address from cached customer profile)

**Session Data Stored:** `shippingOption: "primary"|"custom"|"will_call"`

-----

### Step 5A: Custom Shipping Address Entry

**Node:** `PARTS_ADDRESS_ENTRY`

**Message:**

```
Honda HRX217 - Air Filter (Qty: 2)

Please enter the shipping address:

Example: 456 Oak Ave, Seattle, WA 98102
```

**Input:** Free-form text (customer types address)

**Input Validation:**

- Basic validation: Must contain street, city, state, zip
- If invalid: "Invalid address format. Please include street, city, state, and ZIP code."

**Next Node:** `PARTS_CONFIRMATION`

**Session Data Stored:** `shippingAddress: "456 Oak Ave, Seattle, WA 98102"`

-----

### Step 6: Parts Order Confirmation

**Node:** `PARTS_CONFIRMATION`

**Message:**

```
Honda HRX217 - Air Filter (Qty: 2)
Total: $25.98

Ship To: [address or "Will Call - Pickup at shop"]

Creating your parts order...
```

**Required Lizzy API:**

- **POST** `/parts-orders`
  - Request Body:
    ```json
    {
      "customer_id": "CUST123",
      "equipment_id": "EQ123",
      "parts": [
        {
          "part_number": "17211-ZL8-023",
          "quantity": 2
        }
      ],
      "ship_to": "primary"|"custom"|"will_call",
      "shipping_address": "[if ship_to = custom]"
    }
    ```
  - Response:
    ```json
    {
      "order_id": "PO456",
      "invoice_id": "INV789",
      "total": 25.98,
      "invoice_url": "https://lizzy.com/invoice/INV789",
      "payment_url": "https://lizzy.com/pay/INV789"
    }
    ```

**Next Node:** `PARTS_COMPLETE`

-----

### Step 7: Parts Order Complete

**Node:** `PARTS_COMPLETE`

**Message:**

```
✅ Parts Order Placed!

Order #PO456
Honda HRX217 - Air Filter (Qty: 2)
Total: $25.98

Ship To: [address or "Will Call - Pickup at shop"]

📄 View Invoice: [URL]
💳 Pay Invoice: [URL]

Thank you for your order!

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node          |
|------|-------------------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU          |

**Open Questions:**

- [ ] **Should we allow ordering multiple parts in one order?**
- [ ] **Should we provide tracking numbers?**
- [ ] **Should we send order confirmation via email?**

-----

## Workflow 5: Parts Order Status

### User Story

As a customer, I want to check the status of my parts orders so I know when they will arrive or be ready for pickup.

### Entry Point

- From Main Menu, customer selects option 5 → Starts at `PARTS_STATUS_LIST`
- From Troubleshoot Equipment workflow, customer selects "Check Parts Order Status" → Can filter by equipment

### Flow Diagram

```
Main Menu (5) → Parts Orders List → Selected Order Details
```

### Step 1: Show Parts Orders

**Node:** `PARTS_STATUS_LIST`

**Message:** (Dynamic - generated from Lizzy API)

```
Your Parts Orders - Reply with a number:
1. #PO456 - Honda HRX217 Air Filter - Shipped - Est: Nov 12
2. #PO789 - Husqvarna Chain - Pending - Est: Nov 18
[... all parts orders]

Reply MENU for main menu
```

**If no parts orders:**

```
You have no parts orders.

Would you like to:
1. Order Parts
2. Main Menu
```

**Options (if parts orders exist):**

Dynamic - numbers 1-N based on parts order count

**Options (if no parts orders):**

|Option|Next Node       |
|------|----------------|
|1     |ORDER_PARTS_MENU|
|2     |MAIN_MENU       |

**Required Lizzy API:**

- None (uses cached transactions data from session initialization)

**Next Node:** `PARTS_STATUS_DETAIL`

**Session Data Stored:** `selectedOrderId: "PO456"`

**Open Questions:**

- [x] **Should we show all orders or only recent (e.g., last 90 days)?**
  - **Answer:** Show all active orders (pending, shipped), completed orders from last 30 days
- [ ] **Should orders be sorted by date or status?**

-----

### Step 2: Parts Order Details

**Node:** `PARTS_STATUS_DETAIL`

**Message:** (Dynamic)

```
Parts Order #PO456
Honda HRX217

Items:
- Air Filter (Qty: 2) - $25.98

Status: Shipped
Tracking: 1Z999AA10123456784
Estimated Delivery: Nov 12, 2025

Ship To:
123 Main St
Seattle, WA 98101

📄 View Invoice: [URL]
💳 Pay Invoice: [URL]

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node          |
|------|-------------------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU          |

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] **Should we provide tracking links?**
- [ ] **Should we allow customers to cancel parts orders?**

-----

## Workflow 6: View or Pay Invoice

### User Story

As a customer, I want to view and pay my invoices so I can settle my account with my dealer.

### Entry Point

- From Main Menu, customer selects option 6 → Starts at `INVOICE_TYPE_MENU`
- From Troubleshoot Equipment or other workflows, customer selects "View or Pay Invoice" → Can filter by equipment

### Flow Diagram

```
Main Menu (6) → Invoice Type Selection → Invoice List → Invoice Details → Payment
```

### Step 1: Invoice Type Selection

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

**Options:**

|Option|Label             |Next Node               |
|------|------------------|------------------------|
|1     |Unit Sale Invoice |INVOICE_LIST (filter: unit_sale)|
|2     |Service Invoice   |INVOICE_LIST (filter: service)|
|3     |Parts Invoice     |INVOICE_LIST (filter: parts)|
|4     |Call Shop         |TEXT_OR_CALL_DEALER     |
|5     |Text Shop         |TEXT_OR_CALL_DEALER     |
|6     |Main Menu         |MAIN_MENU               |

**Session Data Stored:** `invoiceType: "unit_sale"|"service"|"parts"`

-----

### Step 2: Invoice List

**Node:** `INVOICE_LIST`

**Message:** (Dynamic - generated from Lizzy API based on selected type)

```
Service Invoices - Reply with a number:
1. #INV456 - $150.00 - Due Nov 15 - Unpaid
2. #INV789 - $89.99 - Due Nov 20 - Unpaid
3. #INV123 - $220.00 - Paid Oct 15
[... all invoices of selected type]

Reply MENU for main menu
```

**If no invoices of selected type:**

```
You have no [unit sale / service / parts] invoices.

Would you like to:
1. View other invoice types
2. Main Menu
```

**Options (if invoices exist):**

Dynamic - numbers 1-N based on invoice count

**Options (if no invoices):**

|Option|Next Node        |
|------|-----------------|
|1     |INVOICE_TYPE_MENU|
|2     |MAIN_MENU        |

**Required Lizzy API:**

- None (uses cached transactions data, filtered by invoice type)

**Next Node:** `INVOICE_DETAIL`

**Session Data Stored:** `selectedInvoiceId: "INV456"`

-----

### Step 3: Invoice Details

**Node:** `INVOICE_DETAIL`

**Message:** (Dynamic)

```
Invoice #INV456
Service Order #SO789
Honda HRX217

Amount: $150.00
Status: Unpaid
Due Date: Nov 15, 2025

Description: Oil change and tune up

📄 View Invoice: [URL]
💳 Pay Now: [URL]

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Options:**

|Option|Next Node          |
|------|-------------------|
|1     |TEXT_OR_CALL_DEALER|
|2     |TEXT_OR_CALL_DEALER|
|3     |MAIN_MENU          |

**Required Lizzy API:**

- None (uses cached transactions data)

**Open Questions:**

- [ ] **Should customers be able to pay invoices directly via SMS?**
  - Could integrate with payment gateway (Stripe, PayPal, etc.)
  - Security concerns with payment info via SMS?
- [ ] **Should we send payment confirmation via email?**

-----

## Workflow 7: Text or Call Dealer

### User Story

As a customer, I want to communicate directly with my dealer when the automated system can't help me.

### Entry Point

- Available from all workflows as an exit option
- Customer can type "HELP" at any time

### Node: TEXT_OR_CALL_DEALER

**Message:**

```
Contact [Dealer Name]

Reply with a number:
1. Text Shop
2. Call Shop
3. Main Menu
```

**Options:**

|Option|Label     |Next Node             |
|------|----------|----------------------|
|1     |Text Shop |DEALER_TEXT_HANDOFF   |
|2     |Call Shop |DEALER_CALL_INFO      |
|3     |Main Menu |MAIN_MENU             |

-----

### Text Shop (Dealer Handoff)

**Node:** `DEALER_TEXT_HANDOFF`

**Message:**

```
A member of our team will respond shortly. Please describe what you need help with:

(Type your message and press Send)
```

**System Behavior:**

1. Flag conversation in Lizzy SMS interface for dealer attention
2. All subsequent messages from customer go directly to dealer (bypass automation)
3. Dealer can respond directly in Lizzy SMS interface
4. Customer can type "MENU" to return to automated system

**Required Lizzy API:**

- **POST** `/sms/conversation/{phone}/flag` - Flag conversation for dealer attention
- **POST** `/sms/log` - Log all messages to Lizzy

**Open Questions:**

- [x] **How does dealer "take over" the conversation?**
  - **Answer:** Conversation flagged in Lizzy SMS interface, dealer responds directly
- [x] **How does customer return to automated system?**
  - **Answer:** Customer types "MENU" or dealer marks conversation as resolved
- [ ] **Should there be business hours enforcement?**

-----

### Call Shop

**Node:** `DEALER_CALL_INFO`

**Message:**

```
[Dealer Name]
Phone: (206) 555-0100

Business Hours:
Mon-Fri: 8am-6pm
Sat: 9am-4pm
Sun: Closed

Tap the number above to call on mobile devices.

Reply MENU to return to main menu.
```

**System Behavior:**

On mobile devices, phone number should be clickable to initiate call.

-----

## Cross-Workflow Equipment Context

### Overview

When a customer selects equipment in one workflow (e.g., Troubleshoot Equipment) and then navigates to another workflow (e.g., Order Parts or Schedule Service), the equipment selection should carry forward. This eliminates the need for customers to re-select their equipment.

### Implementation

**Session Data:**

When equipment is selected in any workflow, store in session:
- `selectedEquipmentId: "EQ123"`
- `selectedEquipmentName: "Honda HRX217"`
- `selectedEquipmentSN: "MZCG-1234567"`

**Workflow Entry Detection:**

When entering a new workflow, check if `selectedEquipmentId` exists in session:

- **If exists:** Skip equipment selection step, jump directly to workflow's primary function
- **If not exists:** Display normal equipment selection menu

### Workflow-Specific Behavior

**Troubleshoot Equipment → Schedule Service:**
System Behavior: Skip SCHEDULE_SERVICE_MENU → Jump to SERVICE_PROBLEM_DESCRIPTION
Message: "Honda HRX217 - Schedule Service"

**Troubleshoot Equipment → Order Parts:**
System Behavior: Skip ORDER_PARTS_MENU → Jump to SHOW_PARTS_LIST
Message: "Honda HRX217 - Order Parts"

**Troubleshoot Equipment → Service Status:**
System Behavior: Skip SERVICE_STATUS_LIST → Jump to SERVICE_STATUS_DETAIL (if equipment is in service)
Message: "Honda HRX217 - Service Status"
Special Case: If equipment is NOT in service, show "Honda HRX217 is not currently in service. Reply MENU."

**Troubleshoot Equipment → Parts Status:**
System Behavior: Go to PARTS_STATUS_LIST but filter by selected equipment
Message: "Parts Orders for Honda HRX217"

**Troubleshoot Equipment → Invoices:**
System Behavior: Go to INVOICE_TYPE_MENU but provide equipment context for filtering
Message: "Invoices for Honda HRX217"

### Message Format with Context

All messages in subsequent workflows should display equipment context:

```
[Equipment Name] - [Workflow Action]

[Rest of message]
```

Example:
```
Honda HRX217 - Schedule Service

Please describe the problem or service needed:
```

-----

## Lizzy API Requirements Summary

### Required API Endpoints (Consolidated Approach)

#### GET `/api/customers/by-phone/{phone}`

**Purpose:** Get complete customer profile including equipment, addresses, resources, and parts catalogs

**Query Parameters:**
- `include` (optional): Comma-separated list of related data to include
  - Values: `equipment`, `addresses`, `resources`, `parts`
  - Example: `?include=equipment,addresses,resources,parts`

**Response:** Complete customer profile with all related data (see API Requirements document for full specification)

**Caching:** 15 minutes (session duration)

#### GET `/api/customers/{customer_id}/transactions`

**Purpose:** Get all customer transactions including service orders, parts orders, and invoices

**Query Parameters:**
- `include` (optional): Filter response sections
  - Values: `service_orders`, `parts_orders`, `invoices`
  - Example: `?include=service_orders,parts_orders,invoices`

**Response:** Complete transaction history (see API Requirements document for full specification)

**Caching:** 5 minutes (shorter TTL due to changing status)

#### POST `/api/service-orders`

**Purpose:** Create new service order

**Request Body:** See Workflow 2, Step 4 for specification

**Response:** Order ID, invoice ID, invoice URLs

#### POST `/api/parts-orders`

**Purpose:** Create new parts order

**Request Body:** See Workflow 4, Step 6 for specification

**Response:** Order ID, invoice ID, total, invoice URLs

#### POST `/api/sms/log`

**Purpose:** Log SMS messages to Lizzy for dealer visibility

**Request Body:**
```json
{
  "phone": "+12065550100",
  "customer_id": "CUST123",
  "direction": "inbound"|"outbound",
  "message": "Customer message text",
  "timestamp": "2025-11-10T10:30:00Z"
}
```

#### POST `/api/sms/conversation/{phone}/flag`

**Purpose:** Flag conversation for dealer attention (handoff from bot to human)

**Request Body:**
```json
{
  "customer_id": "CUST123",
  "reason": "Customer requested dealer contact",
  "context": {
    "last_menu": "ORDER_PARTS_MENU",
    "selected_equipment": "EQ123"
  }
}
```

### API Call Strategy

**Session Initialization (2 calls):**
1. GET customer profile with all related data → Cache 15 min
2. GET customer transactions → Cache 5 min

**During Session (0-2 additional calls):**
- POST service order (if customer schedules service)
- POST parts order (if customer orders parts)
- POST flag conversation (if customer requests dealer)

**Total API calls per session:** 2-4 calls (60-75% reduction vs traditional approach)

**See Also:** `Lizzy_API_Requirements_Document.md` for complete API specifications

-----

## Open Questions & Assumptions

### Questions for Lizzy Team

**Data Model:**
- [ ] What is the exact field name for equipment category? (`unit_type`, `category`, `equipment_type`?)
- [ ] What are ALL possible values for equipment status?
- [ ] What are ALL possible values for service order status?
- [ ] What are ALL possible values for parts order status?
- [ ] What are ALL possible values for invoice status?
- [ ] Is SKU field distinct from model number and searchable?
- [ ] How should "common parts" be determined? (frequency, sales count, dealer preference?)

**API Capabilities:**
- [ ] Can you support the consolidated `include` parameter approach for customer profile and transactions?
- [ ] Are tutorial videos hosted on Lizzy or can they link to YouTube?
- [ ] Are invoice and payment URLs publicly accessible without authentication?
- [ ] Do payment URLs support mobile payment methods (Apple Pay, Google Pay)?

**Business Rules:**
- [ ] Should there be business hours enforcement for dealer handoff?
- [ ] Should we allow customers to cancel service orders via SMS?
- [ ] Should we allow customers to cancel parts orders via SMS?
- [ ] Should we provide tracking links for parts orders?
- [ ] Should we send confirmation emails for service orders and parts orders?

### Current Assumptions

**Equipment:**
- Customer profile includes all equipment owned
- Equipment has status field (Active, In Service, Inactive)
- Tutorial videos and owner's manuals are linked in equipment data
- Parts catalog is available for each piece of equipment

**Service Orders:**
- Invoices are created immediately upon service order creation
- Service orders can be scheduled for same day, next day, or pickup
- Pickup requires address and date selection

**Parts Orders:**
- Parts can be shipped to customer address or picked up at dealer (will call)
- Parts orders create invoices immediately
- Single part per order (MVP - may expand later)

**Invoices:**
- Invoices have view URL and payment URL
- Payment URLs work on mobile devices
- Invoices are categorized by type (unit sale, service, parts)

**Session Management:**
- 15-minute session timeout
- Customer profile cached for session duration (15 min)
- Transaction data cached for 5 minutes
- Customer can type "MENU" at any time to return to main menu

-----

## Document Maintenance

**This is a living document.** As implementation progresses and questions are answered, this document will be updated to reflect:
- Confirmed API field names and structures
- Resolved business logic questions
- New features or requirements
- Changes based on user testing feedback

**Version Control:**
- Major changes increment the minor version (e.g., 1.3 → 1.4)
- Small clarifications increment the patch version (e.g., 1.3.0 → 1.3.1)
- All changes documented in CHANGELOG.md

**Last Updated:** November 18, 2025  
**Next Review:** December 1, 2025

-----

## Related Documents

- [CHANGELOG.md](CHANGELOG.md) - Complete version history
- [Lizzy_API_Requirements_Document.md](Lizzy_API_Requirements_Document.md) - Detailed API specifications
- [Lizzy_API_Field_Mapping_Tables.md](Lizzy_API_Field_Mapping_Tables.md) - Field-by-field mapping for Lizzy team
- [Twilio_Studio_MVP_Implementation_Guide.md](Twilio_Studio_MVP_Implementation_Guide.md) - MVP implementation approach
- [Lizzy_SMS_Complete_Flow_Diagrams.md](Lizzy_SMS_Complete_Flow_Diagrams.md) - Visual workflow diagrams

-----

**End of Requirements Document v1.3.0**
