# Lizzy Agentic Commerce - Twilio Studio MVP Implementation Guide

**Version:** 1.2 MVP  
**Date:** November 17, 2025  
**Platform:** Twilio Studio + Twilio Functions  
**Status:** Implementation Ready

-----

## Table of Contents

1. [MVP Scope & Decisions](#mvp-scope--decisions)
1. [Architecture Overview](#architecture-overview)
1. [Implementation Strategy](#implementation-strategy)
1. [Workflow Implementation Details](#workflow-implementation-details)
1. [Twilio Functions Required](#twilio-functions-required)
1. [Cross-Workflow Context Management](#cross-workflow-context-management)
1. [Setup Instructions](#setup-instructions)
1. [Cost Estimates](#cost-estimates)
1. [Migration Path to Custom Engine](#migration-path-to-custom-engine)

-----

## MVP Scope & Decisions

### ✅ Accepted Limitations & Workarounds

#### Critical Limitations - RESOLVED

**1. Order Parts - Large Parts Catalog**
- **Decision**: Limit to "Top 10 Common Parts" per equipment
- **Implementation**: API call returns sorted list, display first 10
- **Sorting Logic**: TBD - Will determine based on usage data
- **User Message**: "Top 10 Common Parts for [Equipment]"
- **Fallback**: Option to "Text Shop for Other Parts"

**2. Multi-Part Cart Management**
- **Decision**: Single part per order only
- **Implementation**: Complete order after one part selected
- **User Message**: "To order multiple parts, complete this order and start a new one"

#### Moderate Limitations - SOLUTIONS DEFINED

**3. Cross-Workflow Equipment Context** ⭐ CRITICAL FEATURE
- **Decision**: MUST implement with workarounds
- **Solution**: REST API triggers between Studio Flows with context passing
- **Implementation**: Detailed in [Cross-Workflow Context Management](#cross-workflow-context-management)
- **Complexity**: Moderate - Requires careful variable mapping

**4. Async Order Processing**
- **Decision**: Accept coding requirement
- **Solution**: Twilio Function for async processing
- **Implementation**: Function returns immediately, processes in background, sends follow-up SMS
- **Complexity**: Moderate - Requires one custom Function

**5. Search Functionality (SKU/SN)**
- **Decision**: Use Twilio Functions
- **Solution**: Create search Functions for equipment lookup
- **Implementation**: 2 Functions (searchBySKU, searchBySN)
- **Complexity**: Low - Simple search logic

**6. Complex Validations**
- **Decision**: Rely on clear UX instructions, minimal validation
- **Solution**: Clear format examples in messages
- **Implementation**: Studio's built-in validation only
- **Examples**:
  - "Enter address: Street, City, State ZIP"
  - "Enter date: MM/DD/YYYY (e.g., 12/25/2025)"
  - "Describe problem (10+ characters):"

#### Minor Limitations - ACCEPTED

**7. Pagination "Show More"**
- **Decision**: Limit lists to 10 items initially
- **Workaround**: Multiple messages if needed
- **Impact**: Acceptable for MVP

**8. Two-Layer Filtering**
- **Decision**: Accept multiple Studio steps
- **Implementation**: Filter by Unit_Type → Filter by Model
- **Impact**: Acceptable

-----

## Architecture Overview

### Components

```
Customer Phone Number
        ↓
    Twilio SMS
        ↓
┌───────────────────────────────────────┐
│      Twilio Studio Flows              │
│  - Main Menu Flow                     │
│  - Troubleshoot Equipment Flow        │
│  - Schedule Service Flow              │
│  - Service Status Flow                │
│  - Order Parts Flow                   │
│  - Parts Status Flow                  │
│  - Invoice Flow                       │
└───────────────────────────────────────┘
        ↓
┌───────────────────────────────────────┐
│      Twilio Functions                 │
│  - searchEquipmentBySKU()             │
│  - searchEquipmentBySN()              │
│  - createServiceOrderAsync()          │
│  - createPartsOrderAsync()            │
│  - triggerFlowWithContext()           │
└───────────────────────────────────────┘
        ↓
┌───────────────────────────────────────┐
│      Lizzy API                        │
│  - GET /customers/by-phone            │
│  - GET /transactions                  │
│  - POST /service-orders               │
│  - POST /parts-orders                 │
└───────────────────────────────────────┘
        ↓
    Lizzy SMS Interface
```

### Data Flow

**Session Management:**
- Twilio Studio maintains session variables (automatic)
- Session duration: 15 minutes of inactivity
- Variables persist across widgets within same Flow
- Cross-Flow context requires REST API trigger

**Caching Strategy:**
- Initial API call caches customer profile in Studio variables
- Profile includes: equipment list, addresses, parts catalogs
- Valid for Flow duration (max 15 min)
- Transactions fetched on-demand (not cached)

-----

## Implementation Strategy

### Phase 1: Core Flows (Week 1-2)

**Priority 1 Flows:**
1. Main Menu Flow
2. Troubleshoot Equipment Flow
3. Schedule Service Flow (without parts integration)

**Deliverables:**
- 3 working Studio Flows
- Basic Lizzy API integration
- Session variable structure defined

### Phase 2: Status & Invoice Flows (Week 2-3)

**Priority 2 Flows:**
4. Service Status Flow
5. Parts Order Status Flow
6. View/Pay Invoice Flow

**Deliverables:**
- 3 additional Studio Flows
- Transaction data display working

### Phase 3: Advanced Features (Week 3-4)

**Priority 3 Features:**
7. Order Parts Flow (simplified)
8. Cross-workflow context passing
9. Twilio Functions implementation
10. Async order processing

**Deliverables:**
- All 7 Flows complete
- 5 Twilio Functions deployed
- Full system integration working

### Phase 4: Testing & Refinement (Week 4)

**Testing:**
- End-to-end workflow testing
- Error handling verification
- User acceptance testing
- Documentation

-----

## Workflow Implementation Details

### Workflow 1: Troubleshoot Equipment

**Studio Flow Name:** `Troubleshoot_Equipment`

**Widgets Required:**
1. **Send & Wait for Reply** - Main troubleshooting menu (4 options)
2. **Split Based On** - Route based on selection (1-4)
3. **HTTP Request** - Get customer equipment list from Lizzy API
4. **Send & Wait for Reply** - Display equipment list OR filter menu
5. **Run Function** - Search by SKU/SN (if options 3/4 selected)
6. **HTTP Request** - Get equipment resources
7. **Send Message** - Display resources and exit options
8. **Split Based On** - Route to other Flows or end

**Key Variables:**
- `flow.variables.customerProfile` - Full customer data from API
- `flow.variables.selectedEquipmentId` - Selected equipment ID
- `flow.variables.selectedEquipmentName` - Equipment display name
- `flow.variables.selectedEquipmentSN` - Serial number

**HTTP Request Configuration:**

```javascript
// Get Customer Profile
URL: {{env.LIZZY_API_URL}}/api/customers/by-phone/{{trigger.message.From}}?include=equipment,addresses,resources,parts
Method: GET
Headers: 
  Authorization: Bearer {{env.LIZZY_API_KEY}}

// Store response in: flow.variables.customerProfile
```

**Equipment List Display Widget:**

```
Message Body:
Your Equipment - Reply with a number:
{% for equipment in flow.variables.customerProfile.equipment | slice(0, 10) %}
{{loop.index}}. {{equipment.make}} {{equipment.model}} - SN: {{equipment.serial_number}} - {{equipment.status}}
{% endfor %}

Reply MENU for main menu
```

**Exit Options Widget** (includes cross-workflow triggers):

```
Message Body:
{{flow.variables.selectedEquipmentName}} - SN: {{flow.variables.selectedEquipmentSN}}

Troubleshooting Resources:
📺 Tutorial: {{flow.variables.equipmentResources.tutorial_video_url}}
📖 Manual: {{flow.variables.equipmentResources.owners_manual_url}}

What would you like to do? Reply with a number:
1. Schedule Service
2. Order Parts
3. Check Service Status (only if status="In Service")
4. Check Parts Order Status
5. View or Pay Invoice
6. Text or Call Dealer
7. Main Menu

// User selects 1 or 2 → Trigger REST API to call other Flow with equipment context
```

**Cross-Workflow Trigger (for options 1 & 2):**

Use **Run Function** widget to call `triggerFlowWithContext` Function:
- Input: Selected option (1 or 2), equipment context variables
- Function triggers target Flow via REST API
- Passes equipment context as execution parameters

**Implementation Notes:**
- Limit equipment list to first 10 items
- Use Twilio Functions for SKU/SN search (options 3 & 4)
- Store equipment context in Flow variables for potential handoff

---

### Workflow 2: Schedule Service

**Studio Flow Name:** `Schedule_Service`

**Entry Points:**
1. From Main Menu (standard flow)
2. From Troubleshoot Equipment (with equipment context via REST API trigger)

**Widgets Required:**
1. **Trigger** - Check if equipment context provided (from REST API)
2. **Split Based On** - If context exists, skip to Problem Description, else show menu
3. **Send & Wait for Reply** - Equipment selection menu (if no context)
4. **HTTP Request** - Get customer equipment (if no context)
5. **Send & Wait for Reply** - Equipment list display
6. **Send & Wait for Reply** - Problem description (free text)
7. **Send & Wait for Reply** - Offer parts order
8. **Split Based On** - If Yes to parts, trigger Order Parts Flow
9. **Send & Wait for Reply** - Drop-off options
10. **Send & Wait for Reply** - Pickup address confirmation (if pickup selected)
11. **Send & Wait for Reply** - Pickup date selection
12. **Run Function** - Create service order (async)
13. **Send Message** - Immediate confirmation
14. **Send Message** - Follow-up with invoice (sent by Function when ready)

**Key Variables:**
- `trigger.execution.EquipmentId` - Equipment ID from REST API trigger (if provided)
- `trigger.execution.EquipmentName` - Equipment name from REST API trigger
- `trigger.execution.EquipmentSN` - Serial number from REST API trigger
- `flow.variables.selectedEquipmentId` - Equipment ID (if selected in this Flow)
- `flow.variables.problemDescription` - Customer's problem description
- `flow.variables.partsOrderRequested` - Boolean flag
- `flow.variables.dropOffType` - Type selected (today, tomorrow, pickup)
- `flow.variables.pickupDate` - Scheduled pickup date
- `flow.variables.pickupAddress` - Pickup address (if different from default)

**Context Detection Widget (First widget after Trigger):**

```javascript
Split Based On: {{trigger.execution.EquipmentId}}

Conditions:
- If trigger.execution.EquipmentId exists → Jump to Problem Description widget
- Otherwise → Continue to Equipment Selection menu
```

**Equipment Context Display (when context provided):**

```
Message Body:
{{trigger.execution.EquipmentName}} - Schedule Service

[Continue with problem description prompt]
```

**Problem Description Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Service

Please describe the problem or service needed (be as detailed as possible):

Format: Explain what's wrong or what service you need

Examples:
- "Engine won't start, makes clicking noise"
- "Need oil change and blade sharpening"
- "Equipment runs but loses power after 10 minutes"
```

**Offer Parts Order Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Service

Problem: {{flow.variables.problemDescription}}

Would you like to order parts for this service to help expedite the repair?

Reply with a number:
1. Yes, order parts now
2. No, just schedule service
3. Main Menu
```

**Cross-Flow Trigger to Order Parts** (if option 1 selected):

Use **Run Function** widget to call `triggerFlowWithContext`:
- Target Flow: `Order_Parts`
- Context: Equipment ID, Name, SN
- Return context: `flow.variables.returnToServiceScheduling = true`

**Drop-off Options Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Service

How would you like to drop off? Reply with a number:
1. Drop off Today
2. Drop off Tomorrow
3. Schedule Pickup
4. Main Menu
```

**Pickup Address Confirmation Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Pickup

Pickup Address on File:
{{flow.variables.customerProfile.addresses.primary.street}}
{{flow.variables.customerProfile.addresses.primary.city}}, {{flow.variables.customerProfile.addresses.primary.state}} {{flow.variables.customerProfile.addresses.primary.zip}}

Reply with a number:
1. Use this address
2. Enter different address
3. Main Menu

// If option 2: Send & Wait for Reply widget for new address
// Message: "Enter pickup address: Street, City, State ZIP"
```

**Pickup Date Selection Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Pickup

Reply with a number:
1. Tomorrow
2. In 2 days
3. In 3 days
4. Next week
5. Enter specific date
6. Call Shop to schedule

// If option 5 selected:
// Next widget: "Enter date in format MM/DD/YYYY (within 3 months)"
// Example: 12/25/2025
```

**Create Service Order - Async Function Call:**

Use **Run Function** widget to call `createServiceOrderAsync`:

```javascript
Function: createServiceOrderAsync
Parameters:
  customerId: {{flow.variables.customerProfile.customer_id}}
  equipmentId: {{flow.variables.selectedEquipmentId}}
  description: {{flow.variables.problemDescription}}
  dropOffType: {{flow.variables.dropOffType}}
  pickupRequested: {{flow.variables.pickupRequested}}
  pickupDate: {{flow.variables.pickupDate}}
  pickupAddress: {{flow.variables.pickupAddress}}
  customerPhone: {{trigger.message.From}}

// Function returns immediately with: { submitted: true, message: "Processing..." }
// Function continues processing in background
// Function sends follow-up SMS when API completes
```

**Immediate Confirmation Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Service

Your service order has been submitted. Thank you for scheduling your service with [Dealer Name]! We will send you a copy of your order and the option to pay your invoice as soon as the order is confirmed.

What would you like to do? Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Follow-up SMS** (sent by Function when API completes):

```
// Sent by createServiceOrderAsync Function after API responds
Message Body:
✅ Service Confirmed!

Order #{{orderId}}
{{equipmentName}} - {{problemDescription}}

📄 View Invoice: {{invoiceUrl}}
💳 Pay Now: {{paymentUrl}}

Reply MENU for main menu
```

**Implementation Notes:**
- Equipment context from Troubleshoot workflow detected via `trigger.execution.*`
- Problem description has no validation (rely on clear instructions)
- Async Function handles order creation and follow-up SMS
- All date validation done via clear examples, no complex parsing

---

### Workflow 3: Service Status

**Studio Flow Name:** `Service_Status`

**Widgets Required:**
1. **HTTP Request** - Get customer transactions from Lizzy API
2. **Send & Wait for Reply** - Display equipment in service (max 10)
3. **Split Based On** - Route to selected order details
4. **Send Message** - Display service order details
5. **Send & Wait for Reply** - Exit options

**Key Variables:**
- `flow.variables.transactions` - All customer transactions from API
- `flow.variables.selectedOrderId` - Selected service order ID

**HTTP Request Configuration:**

```javascript
URL: {{env.LIZZY_API_URL}}/api/customers/{{flow.variables.customerId}}/transactions?include=service_orders
Method: GET
Headers: 
  Authorization: Bearer {{env.LIZZY_API_KEY}}

// Store response in: flow.variables.transactions
```

**Service Orders List Widget:**

```
Message Body:
Equipment in Service - Reply with a number:
{% assign inServiceOrders = flow.variables.transactions.service_orders | where: "status", "in_progress" | slice(0, 10) %}
{% for order in inServiceOrders %}
{{loop.index}}. {{order.equipment_name}} - {{order.status}} - #{{order.order_id}} - Est: {{order.estimated_completion | date: "%b %d"}}
{% endfor %}
{{inServiceOrders.size | plus: 1}}. Main Menu

// If no orders:
// Message: "You have no equipment currently in service.\n\n1. Schedule Service\n2. Main Menu"
```

**Order Details Widget:**

```
Message Body:
Service Order #{{flow.variables.selectedOrder.order_id}}
{{flow.variables.selectedOrder.equipment_name}} - SN: {{flow.variables.selectedOrder.equipment_serial}}

Status: {{flow.variables.selectedOrder.status}}
Estimated Completion: {{flow.variables.selectedOrder.estimated_completion | date: "%B %d, %Y"}}

📄 View Invoice: {{flow.variables.selectedOrder.invoice.invoice_url}}
💳 Pay Now: {{flow.variables.selectedOrder.invoice.payment_url}}

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Implementation Notes:**
- Filter transactions to show only service orders with status "in_progress" or "in_service"
- Limit display to first 10 orders
- Order details pulled from cached transaction data

---

### Workflow 4: Order Parts ⭐ MVP SIMPLIFIED VERSION

**Studio Flow Name:** `Order_Parts`

**Entry Points:**
1. From Main Menu (standard flow)
2. From Troubleshoot Equipment (with equipment context via REST API trigger)
3. From Schedule Service (with equipment context + return flag)

**Widgets Required:**
1. **Trigger** - Check if equipment context provided
2. **Split Based On** - If context exists, skip to parts list, else show menu
3. **Send & Wait for Reply** - Equipment selection menu (if no context)
4. **HTTP Request** - Get customer equipment (if no context)
5. **Send & Wait for Reply** - Equipment list display
6. **Send Message** - Display Top 10 Common Parts
7. **Send & Wait for Reply** - Part selection
8. **Send Message** - Display part details
9. **Send & Wait for Reply** - Quantity entry
10. **Send & Wait for Reply** - Shipping options
11. **Run Function** - Create parts order (async if needed)
12. **Send Message** - Order confirmation
13. **Split Based On** - If returning to Service, trigger Service Flow

**Key Variables:**
- `trigger.execution.EquipmentId` - Equipment context from REST API trigger
- `trigger.execution.ReturnToService` - Boolean flag to return to Service Flow
- `flow.variables.selectedEquipmentId`
- `flow.variables.topParts` - Top 10 common parts for selected equipment
- `flow.variables.selectedPartNumber`
- `flow.variables.partQuantity`
- `flow.variables.shippingOption`

**Parts List Display Widget** ⭐ MVP VERSION:

```
Message Body:
Top 10 Common Parts for {{flow.variables.selectedEquipmentName}}

Reply with a number:
{% assign topParts = flow.variables.customerProfile.equipment | where: "equipment_id", flow.variables.selectedEquipmentId | first %}
{% assign sortedParts = topParts.parts_catalog | sort: "popularity" | slice(0, 10) %}
{% for part in sortedParts %}
{{loop.index}}. {{part.description}} - #{{part.part_number}} - ${{part.price}}
{% endfor %}
11. Text Shop for other parts
12. Main Menu

// Note: "popularity" sort field TBD - will determine logic later
```

**Part Details Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Order Part

{{flow.variables.selectedPart.description}} #{{flow.variables.selectedPart.part_number}}
💵 Price: ${{flow.variables.selectedPart.price}}
📦 Availability: {{flow.variables.selectedPart.availability}}

Enter quantity (1-99):

// Note: No validation beyond Studio's number check
// Clear instruction is sufficient per MVP decisions
```

**Shipping Options Widget:**

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Order Parts

{{flow.variables.selectedPart.description}} (Qty: {{flow.variables.partQuantity}})
Total: ${{flow.variables.orderTotal}}

Ship To: {{flow.variables.customerProfile.addresses.primary.street}}, {{flow.variables.customerProfile.addresses.primary.city}}, {{flow.variables.customerProfile.addresses.primary.state}} {{flow.variables.customerProfile.addresses.primary.zip}}

Reply with a number:
1. Ship to address above
2. Change Ship-To Address
3. Will Call (Pick up at shop)
4. Main Menu

// If option 2 selected:
// Next widget: "Enter shipping address: Street, City, State ZIP"
// Example: 123 Main St, Seattle, WA 98101
```

**Create Parts Order Widget:**

```javascript
// HTTP Request OR Run Function (depending on async needs)

// If sync (simple):
HTTP Request:
URL: {{env.LIZZY_API_URL}}/api/parts-orders
Method: POST
Headers: Authorization: Bearer {{env.LIZZY_API_KEY}}
Body: {
  "customer_id": "{{flow.variables.customerProfile.customer_id}}",
  "equipment_id": "{{flow.variables.selectedEquipmentId}}",
  "items": [{
    "part_number": "{{flow.variables.selectedPartNumber}}",
    "quantity": {{flow.variables.partQuantity}}
  }],
  "ship_to": "{{flow.variables.shippingOption}}",
  "shipping_address": {{flow.variables.shippingAddress}}
}

// If async (like service orders):
Function: createPartsOrderAsync
// Same pattern as service orders
```

**Order Confirmation Widget:**

```
Message Body:
✅ Parts Order Confirmed!

Order #{{flow.variables.partsOrder.order_id}}
{{flow.variables.selectedEquipmentName}} - {{flow.variables.selectedPart.description}} (Qty: {{flow.variables.partQuantity}})
Total: ${{flow.variables.partsOrder.total}}

📦 Ship to: {{flow.variables.shippingAddressDisplay}}
🚚 Est. Delivery: {{flow.variables.partsOrder.estimated_delivery | date: "%b %d, %Y"}}

📄 View Invoice: {{flow.variables.partsOrder.invoice_url}}
💳 Pay Now: {{flow.variables.partsOrder.payment_url}}

{% if trigger.execution.ReturnToService %}
Returning to service scheduling...
{% else %}
Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
{% endif %}
```

**Return to Service Flow** (if flag set):

```javascript
// Use Run Function widget to trigger Service Flow
Function: triggerFlowWithContext
Parameters:
  targetFlow: "Schedule_Service"
  executionSid: {{trigger.execution.Sid}}
  equipmentId: {{flow.variables.selectedEquipmentId}}
  equipmentName: {{flow.variables.selectedEquipmentName}}
  returnFromParts: true

// Service Flow picks up at Drop-off Options widget
```

**Implementation Notes:**
- ⭐ ONLY show top 10 parts - sorted by "popularity" (logic TBD)
- ⭐ Single part per order - complete order immediately
- Option 11: "Text Shop for other parts" escalates to human
- Equipment context handled same as Schedule Service
- Return to Service workflow using REST API trigger

---

### Workflow 5: Parts Order Status

**Studio Flow Name:** `Parts_Order_Status`

**Widgets Required:**
1. **HTTP Request** - Get customer transactions
2. **Send & Wait for Reply** - Display parts orders (max 10)
3. **Split Based On** - Route to selected order
4. **Send Message** - Display parts order details
5. **Send & Wait for Reply** - Exit options

**Key Variables:**
- `flow.variables.transactions`
- `flow.variables.selectedOrderId`

**Parts Orders List Widget:**

```
Message Body:
Your Parts Orders - Reply with a number:
{% assign partsOrders = flow.variables.transactions.parts_orders | slice(0, 10) %}
{% for order in partsOrders %}
{{loop.index}}. #{{order.order_id}} - {{order.status}} - {{order.items[0].description}}{% if order.items.size > 1 %} (+{{order.items.size | minus: 1}} more){% endif %}
{% endfor %}
{% if partsOrders.size > 10 %}
{{partsOrders.size | plus: 1}}. Show More Orders
{% endif %}
{{partsOrders.size | plus: 2}}. Main Menu

// If no orders:
// Message: "You have no parts orders.\n\n1. Order Parts\n2. Main Menu"
```

**Order Details Widget:**

```
Message Body:
Parts Order #{{flow.variables.selectedOrder.order_id}}

Equipment: {{flow.variables.selectedOrder.equipment_name}}
Status: {{flow.variables.selectedOrder.status}}
{% if flow.variables.selectedOrder.tracking_number %}
Tracking: {{flow.variables.selectedOrder.tracking_number}}
{% endif %}

Items:
{% for item in flow.variables.selectedOrder.items %}
- {{item.description}} (Qty: {{item.quantity}}) - ${{item.price}}
{% endfor %}

🚚 Est. Delivery: {{flow.variables.selectedOrder.estimated_delivery | date: "%B %d, %Y"}}

📄 View Invoice: {{flow.variables.selectedOrder.invoice.invoice_url}}
💳 Pay Now: {{flow.variables.selectedOrder.invoice.payment_url}}

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Implementation Notes:**
- Show all parts orders (pending, shipped, delivered)
- Limit to 10 initially, with "Show More" option
- Single part per order makes display simple

---

### Workflow 6: View or Pay Invoice

**Studio Flow Name:** `View_Pay_Invoice`

**Widgets Required:**
1. **Send & Wait for Reply** - Invoice type selection
2. **HTTP Request** - Get customer transactions
3. **Send & Wait for Reply** - Display invoices of selected type (max 10)
4. **Split Based On** - Route to selected invoice
5. **Send Message** - Display invoice details
6. **Send & Wait for Reply** - Exit options

**Key Variables:**
- `flow.variables.invoiceType` - Selected type (unit_sale, service, parts)
- `flow.variables.transactions`
- `flow.variables.selectedInvoiceId`

**Invoice Type Menu Widget:**

```
Message Body:
View or Pay Invoice - Reply with a number:
1. Unit Sale Invoice
2. Service Invoice
3. Parts Invoice
4. Call Shop
5. Text Shop
6. Main Menu
```

**Invoice List Widget:**

```
Message Body:
{{flow.variables.invoiceTypeDisplay}} Invoices - Reply with a number:
{% assign filteredInvoices = flow.variables.transactions.invoices | where: "type", flow.variables.invoiceType | slice(0, 10) %}
{% for invoice in filteredInvoices %}
{{loop.index}}. #{{invoice.invoice_id}} - {{invoice.status}} - ${{invoice.amount}} - {{invoice.created_date | date: "%b %d"}}
{% endfor %}
{% if filteredInvoices.size > 10 %}
{{filteredInvoices.size | plus: 1}}. Show More Invoices
{% endif %}
{{filteredInvoices.size | plus: 2}}. Main Menu

// If no invoices:
// Message: "You have no {{invoiceTypeDisplay}} invoices.\n\n1. View other invoice types\n2. Main Menu"
```

**Invoice Details Widget:**

```
Message Body:
Invoice #{{flow.variables.selectedInvoice.invoice_id}}
{% if flow.variables.selectedInvoice.order_id %}
Order #{{flow.variables.selectedInvoice.order_id}}
{% endif %}

{% if flow.variables.selectedInvoice.equipment_name %}
{{flow.variables.selectedInvoice.equipment_name}} - {{flow.variables.selectedInvoice.description}}
{% else %}
{{flow.variables.selectedInvoice.description}}
{% endif %}

Amount: ${{flow.variables.selectedInvoice.amount}}
Status: {{flow.variables.selectedInvoice.status}}
Due Date: {{flow.variables.selectedInvoice.due_date | date: "%B %d, %Y"}}

📄 View Invoice: {{flow.variables.selectedInvoice.invoice_url}}
💳 Pay Now: {{flow.variables.selectedInvoice.payment_url}}

Reply with a number:
1. Call Shop
2. Text Shop
3. Main Menu
```

**Implementation Notes:**
- Filter by invoice type as selected
- Display all statuses (paid, unpaid, overdue)
- Limit to 10 per page

---

## Twilio Functions Required

### Function 1: searchEquipmentBySKU

**Purpose:** Search customer's equipment list by SKU

**File:** `functions/searchEquipmentBySKU.js`

```javascript
exports.handler = function(context, event, callback) {
  const client = context.getTwilioClient();
  
  // Input parameters
  const sku = event.sku.toLowerCase();
  const customerProfile = JSON.parse(event.customerProfile);
  
  // Search equipment list
  const found = customerProfile.equipment.find(eq => 
    eq.sku && eq.sku.toLowerCase() === sku
  );
  
  if (found) {
    return callback(null, {
      success: true,
      equipment: {
        id: found.equipment_id,
        name: `${found.make} ${found.model}`,
        serialNumber: found.serial_number
      }
    });
  } else {
    return callback(null, {
      success: false,
      message: "SKU not found in your equipment list. Please try again or reply MENU."
    });
  }
};
```

**Studio Integration:**
- Widget: Run Function
- Input: `{{widgets.sku_input.inbound.Body}}`, `{{flow.variables.customerProfile}}`
- Output: Store in `flow.variables.searchResult`

---

### Function 2: searchEquipmentBySN

**Purpose:** Search customer's equipment list by Serial Number

**File:** `functions/searchEquipmentBySN.js`

```javascript
exports.handler = function(context, event, callback) {
  const client = context.getTwilioClient();
  
  // Input parameters
  const serialNumber = event.serialNumber.toLowerCase();
  const customerProfile = JSON.parse(event.customerProfile);
  
  // Search equipment list
  const found = customerProfile.equipment.find(eq => 
    eq.serial_number && eq.serial_number.toLowerCase() === serialNumber
  );
  
  if (found) {
    return callback(null, {
      success: true,
      equipment: {
        id: found.equipment_id,
        name: `${found.make} ${found.model}`,
        serialNumber: found.serial_number
      }
    });
  } else {
    return callback(null, {
      success: false,
      message: "Serial number not found in your equipment list. Please try again or reply MENU."
    });
  }
};
```

**Studio Integration:**
- Widget: Run Function
- Input: `{{widgets.sn_input.inbound.Body}}`, `{{flow.variables.customerProfile}}`
- Output: Store in `flow.variables.searchResult`

---

### Function 3: createServiceOrderAsync

**Purpose:** Create service order asynchronously, send follow-up SMS when complete

**File:** `functions/createServiceOrderAsync.js`

```javascript
const axios = require('axios');

exports.handler = async function(context, event, callback) {
  const client = context.getTwilioClient();
  
  // Input parameters
  const {
    customerId,
    equipmentId,
    description,
    dropOffType,
    pickupRequested,
    pickupDate,
    pickupAddress,
    customerPhone
  } = event;
  
  // Return immediately to Studio
  callback(null, {
    submitted: true,
    message: "Your service order has been submitted..."
  });
  
  // Continue processing in background
  try {
    // Call Lizzy API to create service order
    const response = await axios.post(
      `${context.LIZZY_API_URL}/api/service-orders`,
      {
        customer_id: customerId,
        equipment_id: equipmentId,
        service_type: 'repair',
        description: description,
        drop_off_type: dropOffType,
        pickup_requested: pickupRequested === 'true',
        pickup_date: pickupDate,
        pickup_address: pickupAddress
      },
      {
        headers: {
          'Authorization': `Bearer ${context.LIZZY_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    // Send follow-up SMS with order confirmation
    await client.messages.create({
      body: `✅ Service Confirmed!\n\nOrder #${response.data.order_id}\n${event.equipmentName} - ${description}\n\n📄 View Invoice: ${response.data.invoice_url}\n💳 Pay Now: ${response.data.payment_url}\n\nReply MENU for main menu`,
      from: context.TWILIO_PHONE_NUMBER,
      to: customerPhone
    });
    
  } catch (error) {
    // Send error message to customer
    await client.messages.create({
      body: `Your service could not be scheduled automatically. Please contact [Dealer Name] to have your service scheduled.\n\nReply with:\n1. Text Shop\n2. Call Shop\n3. Main Menu`,
      from: context.TWILIO_PHONE_NUMBER,
      to: customerPhone
    });
    
    console.error('Service order creation failed:', error);
  }
};
```

**Environment Variables Needed:**
- `LIZZY_API_URL`
- `LIZZY_API_KEY`
- `TWILIO_PHONE_NUMBER`

**Dependencies:**
- Add `axios` package to Twilio Functions

**Studio Integration:**
- Widget: Run Function
- Parameters: All service order data
- Response: Immediate acknowledgment
- Follow-up: Function sends SMS directly when API completes

---

### Function 4: createPartsOrderAsync

**Purpose:** Create parts order (async if needed), similar pattern to service orders

**File:** `functions/createPartsOrderAsync.js`

```javascript
const axios = require('axios');

exports.handler = async function(context, event, callback) {
  const client = context.getTwilioClient();
  
  // Input parameters
  const {
    customerId,
    equipmentId,
    partNumber,
    quantity,
    shippingOption,
    shippingAddress,
    customerPhone,
    equipmentName
  } = event;
  
  try {
    // Call Lizzy API to create parts order
    const response = await axios.post(
      `${context.LIZZY_API_URL}/api/parts-orders`,
      {
        customer_id: customerId,
        equipment_id: equipmentId,
        items: [{
          part_number: partNumber,
          quantity: parseInt(quantity)
        }],
        ship_to: shippingOption,
        shipping_address: JSON.parse(shippingAddress)
      },
      {
        headers: {
          'Authorization': `Bearer ${context.LIZZY_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    // Return order details to Studio
    return callback(null, {
      success: true,
      orderId: response.data.order_id,
      total: response.data.total,
      invoiceUrl: response.data.invoice_url,
      paymentUrl: response.data.payment_url,
      estimatedDelivery: response.data.estimated_delivery
    });
    
  } catch (error) {
    console.error('Parts order creation failed:', error);
    
    return callback(null, {
      success: false,
      message: "Parts order could not be created. Please contact dealer."
    });
  }
};
```

**Studio Integration:**
- Widget: Run Function
- Parameters: All parts order data
- Response: Sync response with order details
- Display: Confirmation message in Studio

**Note:** Can be made async like service orders if Lizzy API is slow

---

### Function 5: triggerFlowWithContext ⭐ CRITICAL FOR CROSS-WORKFLOW

**Purpose:** Trigger another Studio Flow via REST API, passing equipment context

**File:** `functions/triggerFlowWithContext.js`

```javascript
const axios = require('axios');

exports.handler = async function(context, event, callback) {
  const client = context.getTwilioClient();
  
  // Input parameters
  const {
    targetFlow,
    customerPhone,
    equipmentId,
    equipmentName,
    equipmentSN,
    returnToService,
    returnFromParts
  } = event;
  
  // Map target flow names to Flow SIDs
  const flowMapping = {
    'Schedule_Service': context.SCHEDULE_SERVICE_FLOW_SID,
    'Order_Parts': context.ORDER_PARTS_FLOW_SID,
    'Service_Status': context.SERVICE_STATUS_FLOW_SID
  };
  
  const flowSid = flowMapping[targetFlow];
  
  if (!flowSid) {
    return callback(new Error(`Unknown target flow: ${targetFlow}`));
  }
  
  try {
    // Trigger the target Flow via Twilio REST API
    const execution = await client.studio.v2
      .flows(flowSid)
      .executions
      .create({
        to: customerPhone,
        from: context.TWILIO_PHONE_NUMBER,
        parameters: JSON.stringify({
          EquipmentId: equipmentId,
          EquipmentName: equipmentName,
          EquipmentSN: equipmentSN,
          ReturnToService: returnToService === 'true',
          ReturnFromParts: returnFromParts === 'true'
        })
      });
    
    return callback(null, {
      success: true,
      executionSid: execution.sid,
      message: `Triggered ${targetFlow} with equipment context`
    });
    
  } catch (error) {
    console.error('Flow trigger failed:', error);
    
    return callback(null, {
      success: false,
      message: "Could not switch workflows. Please try again or reply MENU."
    });
  }
};
```

**Environment Variables Needed:**
- `SCHEDULE_SERVICE_FLOW_SID` - Flow SID for Schedule Service
- `ORDER_PARTS_FLOW_SID` - Flow SID for Order Parts
- `SERVICE_STATUS_FLOW_SID` - Flow SID for Service Status
- `TWILIO_PHONE_NUMBER`

**Studio Integration:**
- Widget: Run Function
- Input: Target flow name, equipment context variables
- Output: Triggers new Flow execution with context
- Current Flow: Ends after triggering

**How It Works:**
1. Customer in Troubleshoot Equipment selects "Schedule Service"
2. Troubleshoot Flow calls `triggerFlowWithContext` Function
3. Function triggers Schedule Service Flow via REST API
4. Schedule Service Flow receives equipment context in `trigger.execution.*`
5. Schedule Service Flow detects context and skips equipment selection

---

## Cross-Workflow Context Management

### Overview

Cross-workflow equipment context is CRITICAL for MVP success. This section details the implementation strategy.

### The Challenge

Twilio Studio Flows are isolated - variables don't automatically pass between Flows. When a customer selects equipment in Troubleshoot and then chooses "Schedule Service," we need to preserve that equipment selection.

### The Solution: REST API Trigger

Use Twilio Functions to trigger target Flows via REST API, passing equipment context as execution parameters.

### Implementation Pattern

**Step 1: Source Flow (e.g., Troubleshoot Equipment)**

When customer selects an action that requires another Flow:

```
Widget: Run Function
Function: triggerFlowWithContext
Parameters:
  targetFlow: "Schedule_Service"
  customerPhone: {{trigger.message.From}}
  equipmentId: {{flow.variables.selectedEquipmentId}}
  equipmentName: {{flow.variables.selectedEquipmentName}}
  equipmentSN: {{flow.variables.selectedEquipmentSN}}
  returnToService: false
  returnFromParts: false

Next: End Flow (or show confirmation message then end)
```

**Step 2: Target Flow (e.g., Schedule Service)**

First widget after Trigger checks for context:

```
Widget: Split Based On
Variable: {{trigger.execution.EquipmentId}}

Conditions:
1. If EquipmentId exists:
   - Store in flow.variables.selectedEquipmentId
   - Store EquipmentName in flow.variables.selectedEquipmentName
   - Store EquipmentSN in flow.variables.selectedEquipmentSN
   - Jump to Problem Description widget

2. Otherwise:
   - Continue to Equipment Selection Menu (standard flow)
```

**Step 3: Display Equipment Context**

All subsequent messages show equipment context:

```
Message Body:
{{flow.variables.selectedEquipmentName}} - Schedule Service

[Continue with workflow]
```

### Context Data Structure

Equipment context passed between Flows:

```json
{
  "EquipmentId": "EQ123",
  "EquipmentName": "Honda HRX217",
  "EquipmentSN": "MZCG-1234567",
  "ReturnToService": false,
  "ReturnFromParts": false
}
```

### Context Flow Map

```
Troubleshoot Equipment → Schedule Service
  Context: Equipment ID, Name, SN
  Return: No

Troubleshoot Equipment → Order Parts
  Context: Equipment ID, Name, SN
  Return: No

Troubleshoot Equipment → Service Status
  Context: Equipment ID (for filtering)
  Return: No

Schedule Service → Order Parts
  Context: Equipment ID, Name, SN
  Return: Yes (ReturnToService = true)

Order Parts → Schedule Service (return)
  Context: Equipment ID, Name, SN
  Return: No (ReturnFromParts = true)
```

### Return Flow Pattern

When Order Parts needs to return to Schedule Service:

**Order Parts Flow:**
```
After order confirmation:

Widget: Split Based On
Variable: {{trigger.execution.ReturnToService}}

If true:
  Widget: Send Message
  Body: "Returning to service scheduling..."
  
  Widget: Run Function
  Function: triggerFlowWithContext
  Parameters:
    targetFlow: "Schedule_Service"
    equipmentId: {{flow.variables.selectedEquipmentId}}
    returnFromParts: true
  
  Next: End Flow

Otherwise:
  Continue to standard exit options
```

**Schedule Service Flow:**
```
After problem description (or at return point):

Widget: Split Based On
Variable: {{trigger.execution.ReturnFromParts}}

If true:
  Jump directly to Drop-off Options widget
  (Skip problem description and parts order offer)

Otherwise:
  Continue standard flow
```

### Testing Context Passing

**Test Scenario 1: Troubleshoot → Schedule Service**
1. Customer starts Troubleshoot Equipment
2. Selects "Honda HRX217"
3. Views resources
4. Selects "Schedule Service"
5. ✅ Verify: Schedule Service shows "Honda HRX217 - Schedule Service"
6. ✅ Verify: No equipment selection menu appears
7. ✅ Verify: Goes directly to problem description

**Test Scenario 2: Schedule Service → Order Parts → Return**
1. Customer starts Schedule Service
2. Selects equipment
3. Enters problem description
4. Selects "Yes, order parts now"
5. ✅ Verify: Order Parts shows correct equipment
6. ✅ Verify: No equipment selection menu in Order Parts
7. Customer completes parts order
8. ✅ Verify: "Returning to service scheduling..." message appears
9. ✅ Verify: Returns to Schedule Service at drop-off options
10. ✅ Verify: Equipment context still present

**Test Scenario 3: Context NOT Present**
1. Customer texts "2" to Main Menu (Schedule Service)
2. ✅ Verify: Equipment selection menu appears
3. ✅ Verify: Standard flow works normally

### Troubleshooting Context Issues

**Issue: Context not passing**
- Check Function is returning success
- Verify Flow SID environment variables are correct
- Check parameters are JSON stringified
- Verify target Flow's first widget checks trigger.execution

**Issue: Context passing but not displaying**
- Check variable names match exactly (case-sensitive)
- Verify Split Based On conditions are correct
- Check Liquid template syntax in messages

**Issue: Return flow not working**
- Verify ReturnToService flag is set correctly
- Check target Flow has condition to detect return
- Verify jump to correct widget on return

---

## Setup Instructions

### Prerequisites

1. **Twilio Account**
   - Sign up at twilio.com
   - Verify your account
   - Purchase a phone number with SMS capability

2. **Lizzy API Access**
   - API base URL
   - API authentication key
   - Test credentials for development

3. **Technical Skills Needed (Minimal)**
   - Copy/paste text
   - Navigate Twilio console
   - Follow step-by-step instructions

### Phase 1: Twilio Configuration (30 minutes)

**Step 1: Purchase Phone Number**
1. Log into Twilio Console
2. Go to Phone Numbers → Buy a Number
3. Select country (United States)
4. Search for number with SMS capability
5. Purchase number (~$1-2/month)
6. Note your phone number for later

**Step 2: Create Studio Flows**
1. Go to Studio → Flows
2. Click "Create new Flow"
3. Name: "Main_Menu"
4. Click "Next"
5. Select "Start from scratch"
6. You'll build this in Phase 2

**Step 3: Set Up Environment Variables**
1. Go to Functions & Assets → Configuration
2. Add Environment Variables:
   - `LIZZY_API_URL` = [Lizzy API base URL]
   - `LIZZY_API_KEY` = [Your Lizzy API key]
   - `TWILIO_PHONE_NUMBER` = [Your Twilio phone number]
3. Click "Deploy All" (takes 1-2 minutes)

**Step 4: Create Twilio Functions**
1. Go to Functions & Assets → Services
2. Click "Create Service"
3. Name: "Lizzy-SMS-Functions"
4. Click "Next"
5. Add 5 Functions (copy code from sections above):
   - searchEquipmentBySKU.js
   - searchEquipmentBySN.js
   - createServiceOrderAsync.js
   - createPartsOrderAsync.js
   - triggerFlowWithContext.js
6. Add Dependencies:
   - Package: `axios`, Version: `latest`
7. Click "Deploy All"

### Phase 2: Build Studio Flows (2-4 weeks)

Follow the detailed workflow implementations above. Build in this order:

**Week 1:**
1. Main Menu Flow
2. Troubleshoot Equipment Flow
3. Test end-to-end

**Week 2:**
4. Schedule Service Flow
5. Test with Main Menu
6. Test cross-workflow context from Troubleshoot

**Week 3:**
7. Service Status Flow
8. Parts Order Status Flow
9. View/Pay Invoice Flow
10. Test all status flows

**Week 4:**
11. Order Parts Flow
12. Test parts ordering
13. Test return to Service Flow
14. Full system testing

### Phase 3: Connect to Lizzy API (1 week)

**Step 1: Test API Endpoints**
1. Use Postman or curl to test each Lizzy API endpoint
2. Verify authentication works
3. Verify response formats match expectations
4. Document any discrepancies

**Step 2: Update Studio HTTP Requests**
1. Configure correct URLs
2. Add authentication headers
3. Test with real data
4. Verify error handling

**Step 3: Test Data Flow**
1. Real phone number → Twilio → Studio → Lizzy API
2. Verify customer lookup works
3. Verify order creation works
4. Verify invoice links work

### Phase 4: Testing & Launch (1 week)

**Test Scenarios:**
1. Happy path for each workflow
2. Invalid inputs (wrong phone number, non-existent equipment)
3. API failures (simulate Lizzy API down)
4. Cross-workflow context passing
5. Timeout scenarios (15-minute inactivity)
6. Multiple concurrent users

**Launch Checklist:**
- [ ] All 7 Studio Flows working
- [ ] All 5 Twilio Functions deployed
- [ ] Lizzy API integration tested
- [ ] Error messages are user-friendly
- [ ] Phone number configured correctly
- [ ] Documentation complete
- [ ] Training materials for dealers prepared

---

## Cost Estimates

### Monthly Operating Costs (MVP)

**Twilio Costs:**

| Item | Unit Cost | Volume (estimated) | Monthly Cost |
|------|-----------|-------------------|--------------|
| Studio Executions | $0.01/step | 500 sessions × 8 steps avg = 4,000 steps | $40.00 |
| Twilio Functions | $0.0001/invoke | 2,000 invokes | $0.20 |
| SMS (Outbound) | $0.0079/msg | 2,000 messages | $15.80 |
| SMS (Inbound) | Free | - | $0.00 |
| Phone Number | $1.15/month | 1 number | $1.15 |

**Total Twilio: ~$57/month**

**Lizzy API Costs:**
- Assumed to be existing service (no additional cost)

**Total MVP Operating Cost: ~$57/month**

### Cost Comparison

| Solution | Development Cost | Monthly Operating | Break-even |
|----------|-----------------|-------------------|------------|
| **MVP - Twilio Studio** | $0 (DIY) | $57/month | N/A |
| **Custom Node.js Engine** | $5-10K (hired dev) | $25/month | 156-312 months |
| **Hybrid Approach** | $0 → $5K later | $57 → $25/month | After migration |

### Cost at Scale

**At 5,000 sessions/month:**
- Twilio Studio: ~$285/month
- Custom Engine: ~$30/month
- **Savings: $255/month ($3,060/year)**

**Break-even on custom engine development:**
- Savings: $255/month
- Development cost: $7,500
- Break-even: 29 months

---

## Migration Path to Custom Engine

### When to Migrate

Migrate from Studio to custom Node.js engine when ANY of these occur:

1. **Volume threshold:** >1,000 sessions/month (cost becomes significant)
2. **Feature limitations:** Need multi-part ordering, advanced search, or other unsupported features
3. **Debugging needs:** Studio logs insufficient for troubleshooting
4. **Maintenance burden:** Workflow changes too complex in Studio
5. **Integration complexity:** Adding more external systems becomes unwieldy

### Migration Strategy

**Phase 1: Preparation (Weeks 1-2)**
1. Document all Studio Flow logic
2. Export Flow configurations (JSON)
3. Document all Twilio Functions
4. Create requirements doc for developer
5. Set up GitHub repository

**Phase 2: Build Custom Engine (Weeks 3-8)**
1. Hire developer or development team
2. Build Node.js/Express workflow engine
3. Implement DynamoDB session management
4. Migrate all workflow logic
5. Set up testing environment

**Phase 3: Parallel Operation (Weeks 9-10)**
1. Deploy custom engine to staging
2. Route 10% of traffic to new engine
3. Compare behavior with Studio
4. Fix bugs and discrepancies
5. Gradually increase traffic percentage

**Phase 4: Cutover (Week 11)**
1. Route 100% of traffic to custom engine
2. Keep Studio Flows as backup for 1 week
3. Monitor for issues
4. Decommission Studio Flows when stable

**Phase 5: Optimization (Week 12+)**
1. Add features not possible in Studio
2. Optimize performance
3. Reduce costs
4. Implement advanced monitoring

### Migration Artifacts to Preserve

From Studio MVP, preserve:
1. All workflow logic and message templates
2. Error message copy
3. User experience insights from usage data
4. Dealer feedback on workflows
5. Twilio Functions (reuse in custom engine)

These artifacts will significantly reduce custom engine development time and cost.

---

## Next Steps

### Immediate Actions (This Week)

1. ✅ Review this implementation guide
2. ⏳ Set up Twilio account and purchase phone number
3. ⏳ Get Lizzy API credentials and test access
4. ⏳ Create environment variables in Twilio
5. ⏳ Deploy 5 Twilio Functions

### Week 1: Build Foundation

1. Create Main Menu Flow
2. Create Troubleshoot Equipment Flow
3. Test basic SMS conversation flow
4. Verify Lizzy API integration (customer lookup)

### Week 2-3: Core Workflows

1. Build Schedule Service Flow
2. Build Service Status Flow
3. Test cross-workflow context passing
4. Test async order processing

### Week 4: Complete MVP

1. Build Order Parts Flow (simplified)
2. Build Parts Status and Invoice Flows
3. Complete end-to-end testing
4. Prepare for launch

### Week 5: Launch & Monitor

1. Soft launch with 2-3 dealers
2. Gather feedback
3. Fix bugs
4. Prepare for broader rollout

---

## Summary

This MVP implementation guide provides:

✅ **Clear scope** - Defined limitations and workarounds
✅ **Detailed workflows** - Widget-by-widget implementation
✅ **Working code** - 5 Twilio Functions ready to deploy
✅ **Cross-workflow solution** - Context passing via REST API
✅ **Cost transparency** - $57/month operating cost
✅ **Migration path** - Clear transition to custom engine when needed

**Key Success Factors:**

1. **Equipment Context Passing** - Critical feature preserved via REST API triggers
2. **Simplified Parts Ordering** - Top 10 parts makes Studio workable
3. **Clear UX Instructions** - Minimal validation, clear format examples
4. **Async Processing** - Professional user experience despite Studio limitations
5. **Pragmatic Scope** - MVP limitations accepted for faster time-to-market

**Expected Timeline:**
- Week 1-2: Setup and foundation
- Week 3-4: Core workflows
- Week 5: Complete and test
- Week 6: Launch with initial dealers

**Next Document:** Detailed Studio Flow diagrams and step-by-step widget configuration guide.

---

**Questions or clarifications needed?** This guide is ready for implementation. The next step is to begin building the Studio Flows following the detailed specifications above.
