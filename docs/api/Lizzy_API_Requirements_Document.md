# Lizzy SMS Workflow System - API Requirements Document

**To:** Lizzy Technical Team  
**From:** Jonathan (Lizzy Agentic Commerce Project)  
**Date:** November 17, 2025  
**Version:** 1.2  
**Purpose:** API endpoint requirements for SMS workflow integration

-----

## Executive Summary

We are building an SMS-based workflow system that allows customers to interact with their dealer via text message for equipment troubleshooting, service scheduling, parts ordering, and invoice management. This system will integrate with Lizzy's existing platform via RESTful API calls.

**Key Points:**
- **7 SMS workflows** requiring real-time data access
- **Consolidated API approach** to minimize calls (7-9 endpoints total)
- **Session-based caching** to reduce API load (15-minute sessions)
- **MVP timeline:** 6 weeks from API availability
- **Expected volume:** 500-1,000 sessions/month initially

-----

## Table of Contents

1. [API Architecture Overview](#api-architecture-overview)
2. [Authentication Requirements](#authentication-requirements)
3. [Required API Endpoints](#required-api-endpoints)
4. [Data Model Requirements](#data-model-requirements)
5. [Error Handling](#error-handling)
6. [Rate Limiting](#rate-limiting)
7. [Testing & Staging](#testing--staging)
8. [Implementation Timeline](#implementation-timeline)
9. [Appendix: Sample Responses](#appendix-sample-responses)

-----

## API Architecture Overview

### Design Philosophy

We use a **consolidated API approach** to minimize API calls and improve performance:

1. **Single customer profile endpoint** returns all related data (equipment, addresses, resources, parts)
2. **Single transactions endpoint** returns all orders and invoices
3. **Session caching** stores data for 15 minutes, reducing repeated calls
4. **Transactional endpoints** for creating orders only

### Expected API Call Pattern Per Session

**Session Start:** 1-2 API calls
- GET customer profile with all related data
- GET transactions (if needed)

**During Session:** 0-2 additional API calls
- Create service order (if requested)
- Create parts order (if requested)

**Total:** 2-4 API calls per session (vs 8-12 with non-consolidated approach)

### Benefits for Lizzy Infrastructure

- ✅ **66-75% reduction** in API calls compared to traditional approach
- ✅ **Predictable load** - cached data reduces spikes
- ✅ **Efficient data transfer** - fewer round trips
- ✅ **Scalable architecture** - works at high volume

-----

## Authentication Requirements

### Preferred Method: Bearer Token

```
Authorization: Bearer {API_KEY}
```

### Questions for Lizzy Team:

1. **Authentication type?** (API Key, OAuth2, JWT?)
2. **Key rotation policy?** (How often should keys be refreshed?)
3. **Separate keys for staging/production?**
4. **Rate limiting tied to API key?**
5. **Key expiration?** (Do tokens expire?)

### Our Setup

- Keys stored as environment variables (never in code)
- Separate credentials for development/staging/production
- Secure key management following best practices

-----

## Required API Endpoints

### Summary Table

| Priority | Endpoint | Method | Purpose | Caching |
|----------|----------|--------|---------|---------|
| 🔴 Critical | `/api/customers/by-phone/{phone}` | GET | Get customer profile + equipment + parts + addresses | 15 min |
| 🔴 Critical | `/api/customers/{id}/transactions` | GET | Get all orders and invoices | 5 min |
| 🔴 Critical | `/api/service-orders` | POST | Create service order | N/A |
| 🔴 Critical | `/api/parts-orders` | POST | Create parts order | N/A |
| 🟡 Important | `/api/sms/log` | POST | Log SMS messages to Lizzy | N/A |
| 🟡 Important | `/api/sms/conversation/{phone}/flag` | POST | Flag conversation for dealer | N/A |
| 🟡 Important | `/api/sms/conversation/{phone}/status` | GET | Check if dealer has taken over | N/A |
| 🟢 Optional | `/api/orders/{id}/detail` | GET | Extended order details | 5 min |
| 🟢 Optional | `/api/invoices/{id}/detail` | GET | Extended invoice details | 5 min |

-----

## Required API Endpoints - Detailed Specifications

### 1. 🔴 Get Customer Profile (Consolidated)

**Priority:** CRITICAL - Required for MVP

**Endpoint:** `GET /api/customers/by-phone/{phone}`

**Purpose:** Single API call to retrieve complete customer profile with all related data

**Path Parameters:**
- `phone` (string, required) - Customer phone number in E.164 format
  - Example: `+12065550100`

**Query Parameters:**
- `include` (string, optional) - Comma-separated list of related data to include
  - Values: `equipment`, `addresses`, `resources`, `parts`
  - Example: `?include=equipment,addresses,resources,parts`
  - Default: `equipment` only

**Request Example:**
```http
GET /api/customers/by-phone/+12065550100?include=equipment,addresses,resources,parts
Authorization: Bearer {API_KEY}
Accept: application/json
```

**Response Format (200 OK):**
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
      "sku": "HRX217VKA",
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
          "category": "Filters",
          "popularity_rank": 1
        },
        {
          "part_number": "98079-55846",
          "description": "Spark Plug",
          "price": 8.99,
          "availability": "in_stock",
          "category": "Ignition",
          "popularity_rank": 2
        }
      ]
    }
  ]
}
```

**Error Responses:**
- `404 Not Found` - Phone number not in system
- `400 Bad Request` - Invalid phone number format
- `401 Unauthorized` - Invalid API key
- `500 Internal Server Error` - Server error

**Questions for Lizzy Team:**

1. **Field names:** Do these field names match your data model?
   - `unit_type` - Equipment category field name?
   - `sku` - SKU field available and searchable?
   - `resources.tutorial_video_url` - Field name for video links?
   - `resources.owners_manual_url` - Field name for manual PDFs?
   - `parts_catalog.popularity_rank` - Field for sorting parts?

2. **Parts catalog:** How many parts per equipment typically?
   - Need to understand payload size
   - Is pagination needed for parts_catalog?

3. **Equipment status values:** What are all possible values?
   - We've assumed: Active, In Service, Inactive
   - Are there others?

4. **Phone number format:** What format does Lizzy store?
   - E.164 (+12065550100)?
   - National (206-555-0100)?
   - Do we need normalization?

5. **Include parameter:** Is this consolidation approach feasible?
   - Or should these be separate endpoints?
   - Performance implications?

**Critical for MVP:** This is our most important endpoint. We cache this response for 15 minutes to minimize API calls.

---

### 2. 🔴 Get Customer Transactions (Consolidated)

**Priority:** CRITICAL - Required for MVP

**Endpoint:** `GET /api/customers/{customer_id}/transactions`

**Purpose:** Single API call to retrieve all current orders and invoices

**Path Parameters:**
- `customer_id` (string, required) - Customer ID from profile endpoint

**Query Parameters:**
- `include` (string, optional) - Data to include
  - Values: `service_orders`, `parts_orders`, `invoices`
  - Example: `?include=service_orders,parts_orders,invoices`
  - Default: All included
- `equipment_id` (string, optional) - Filter by specific equipment
  - Example: `?equipment_id=EQ123`
  - Use case: When customer has equipment context from previous workflow

**Request Example:**
```http
GET /api/customers/CUST123/transactions?include=service_orders,parts_orders,invoices
Authorization: Bearer {API_KEY}
Accept: application/json
```

**Response Format (200 OK):**
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
      "description": "Honda HRX217 Purchase",
      "invoice_url": "https://lizzy.com/invoice/INV999",
      "payment_url": "https://lizzy.com/pay/INV999"
    }
  ]
}
```

**Error Responses:**
- `404 Not Found` - Customer ID not found
- `401 Unauthorized` - Invalid API key
- `500 Internal Server Error` - Server error

**Questions for Lizzy Team:**

1. **Status values:** What are all possible status values for:
   - Service orders? (in_progress, complete, awaiting_parts, etc.?)
   - Parts orders? (pending, processing, shipped, delivered, will_call_ready?)
   - Invoices? (paid, unpaid, partially_paid, overdue?)

2. **Invoice URLs:** 
   - Are these publicly accessible without authentication?
   - Do they work on mobile devices?
   - Do payment URLs support Apple Pay / Google Pay?

3. **Tracking numbers:**
   - Format? (Just the number, or full tracking URL?)
   - Carrier information included?

4. **Historical data:**
   - How far back should we fetch? (90 days? 1 year? All time?)
   - Any performance concerns with large result sets?

5. **Equipment filtering:**
   - Is `equipment_id` query parameter supported?
   - Or should we filter client-side?

**Critical for MVP:** We cache this response for 5 minutes (shorter than profile due to changing status). Refreshed when accessed if stale.

---

### 3. 🔴 Create Service Order

**Priority:** CRITICAL - Required for MVP

**Endpoint:** `POST /api/service-orders`

**Purpose:** Create a new service order for equipment repair or maintenance

**Request Headers:**
```
Authorization: Bearer {API_KEY}
Content-Type: application/json
Accept: application/json
```

**Request Body:**
```json
{
  "customer_id": "CUST123",
  "equipment_id": "EQ123",
  "service_type": "repair",
  "description": "Customer reported: Engine won't start, makes clicking noise",
  "drop_off_type": "drop_off_today",
  "preferred_date": "2025-11-11",
  "pickup_requested": true,
  "pickup_date": "2025-11-15",
  "pickup_address": {
    "street": "123 Main St",
    "city": "Seattle",
    "state": "WA",
    "zip": "98101"
  }
}
```

**Field Descriptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| customer_id | string | Yes | Customer identifier |
| equipment_id | string | Yes | Equipment identifier |
| service_type | string | Yes | Type: "repair", "maintenance", "inspection" |
| description | string | Yes | Customer's problem description (10-500 chars) |
| drop_off_type | string | Yes | "drop_off_today", "drop_off_tomorrow", "schedule_pickup" |
| preferred_date | string | Yes | ISO date format (YYYY-MM-DD) |
| pickup_requested | boolean | No | Whether customer requested pickup |
| pickup_date | string | No | ISO date (if pickup_requested = true) |
| pickup_address | object | No | Pickup location (if different from default) |

**Response Format (201 Created):**
```json
{
  "order_id": "SO789",
  "invoice_id": "INV456",
  "status": "pending",
  "invoice_url": "https://lizzy.com/invoice/INV456",
  "payment_url": "https://lizzy.com/pay/INV456",
  "created_at": "2025-11-05T10:30:00Z"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid request data (with validation errors)
- `404 Not Found` - Customer or equipment not found
- `401 Unauthorized` - Invalid API key
- `500 Internal Server Error` - Server error

**Questions for Lizzy Team:**

1. **Service types:** What are all valid service_type values?
   - We've assumed: repair, maintenance, inspection
   - Are there others?

2. **Drop-off types:** Are these values acceptable?
   - drop_off_today, drop_off_tomorrow, schedule_pickup
   - Or do you use different terminology?

3. **Invoice creation:** 
   - Is invoice created immediately upon order creation?
   - Or created later in the workflow?
   - Can order be created without invoice?

4. **Order status:**
   - What's the initial status after creation?
   - What status transitions are possible?

5. **Async processing:**
   - Can this endpoint respond quickly (<2 seconds)?
   - Or should we handle async processing on our end?
   - Do you send notifications when order is fully processed?

**Critical for MVP:** This endpoint is called when customer schedules service via SMS. We handle async on our end (immediate acknowledgment to customer, follow-up when complete).

---

### 4. 🔴 Create Parts Order

**Priority:** CRITICAL - Required for MVP

**Endpoint:** `POST /api/parts-orders`

**Purpose:** Create a new parts order for equipment parts

**Request Headers:**
```
Authorization: Bearer {API_KEY}
Content-Type: application/json
Accept: application/json
```

**Request Body:**
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

**Field Descriptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| customer_id | string | Yes | Customer identifier |
| equipment_id | string | Yes | Equipment identifier |
| items | array | Yes | Array of parts to order (MVP: always 1 item) |
| items[].part_number | string | Yes | Part number from catalog |
| items[].quantity | integer | Yes | Quantity (1-99) |
| ship_to | string | Yes | "ship_to_primary", "ship_to_custom", "will_call" |
| shipping_address | object | Conditional | Required if ship_to = "ship_to_custom" |

**Response Format (201 Created):**
```json
{
  "order_id": "PO456",
  "invoice_id": "INV789",
  "total": 25.98,
  "status": "pending",
  "invoice_url": "https://lizzy.com/invoice/INV789",
  "payment_url": "https://lizzy.com/pay/INV789",
  "estimated_delivery": "2025-11-18",
  "created_at": "2025-11-10T14:20:00Z"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid request data
- `404 Not Found` - Customer, equipment, or part not found
- `409 Conflict` - Part out of stock or unavailable
- `401 Unauthorized` - Invalid API key
- `500 Internal Server Error` - Server error

**Questions for Lizzy Team:**

1. **Multiple parts:**
   - MVP only supports 1 part per order
   - Is this acceptable, or do you require multi-part support?
   - Would separate orders for multiple parts cause issues?

2. **Inventory checking:**
   - Does endpoint validate inventory availability?
   - What happens if part is out of stock?
   - Should we check availability before submitting order?

3. **Shipping options:**
   - Are ship_to values acceptable?
   - Do you handle shipping cost calculation?
   - Is shipping cost included in response?

4. **Will Call:**
   - How is will_call handled differently?
   - Any special fields needed?
   - Pickup notifications?

5. **Delivery estimates:**
   - How is estimated_delivery calculated?
   - Is it reliable for customer communication?

**Critical for MVP:** This endpoint is called when customer orders parts via SMS. Single part per order simplifies MVP.

---

### 5. 🟡 Log SMS Message

**Priority:** IMPORTANT - Needed for dealer visibility

**Endpoint:** `POST /api/sms/log`

**Purpose:** Log SMS conversation to Lizzy SMS interface so dealers can see full conversation history

**Request Headers:**
```
Authorization: Bearer {API_KEY}
Content-Type: application/json
```

**Request Body:**
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

**Field Descriptions:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| phone | string | Yes | Customer phone number (E.164) |
| customer_id | string | Yes | Customer identifier |
| direction | string | Yes | "inbound" or "outbound" |
| message | string | Yes | SMS message text |
| timestamp | string | Yes | ISO 8601 timestamp |
| source | string | Yes | "workflow_bot" or "dealer" |

**Response Format (200 OK):**
```json
{
  "message_id": "MSG123",
  "logged": true
}
```

**Questions for Lizzy Team:**

1. **Existing SMS interface:**
   - Do you currently have an SMS chat interface?
   - How do dealers view SMS conversations now?
   - Can we integrate with existing system?

2. **Message threading:**
   - How are conversations grouped?
   - By phone number? By customer ID? By time window?
   - Do you maintain conversation history?

3. **Real-time updates:**
   - Do dealers see messages in real-time?
   - Or do they need to refresh?
   - Push notifications to dealers?

4. **Message metadata:**
   - Any additional fields needed?
   - Order IDs, equipment IDs for context?
   - Workflow state information?

**Note:** If Lizzy doesn't have an SMS interface yet, we can defer this endpoint. Messages are still logged in Twilio.

---

### 6. 🟡 Flag Conversation for Dealer

**Priority:** IMPORTANT - Needed for dealer handoff

**Endpoint:** `POST /api/sms/conversation/{phone}/flag`

**Purpose:** Notify dealer that customer needs human attention (bot handoff)

**Path Parameters:**
- `phone` (string, required) - Customer phone number

**Request Headers:**
```
Authorization: Bearer {API_KEY}
Content-Type: application/json
```

**Request Body:**
```json
{
  "customer_id": "CUST123",
  "reason": "Customer requested dealer contact",
  "priority": "normal",
  "context": {
    "last_menu": "ORDER_PARTS_MENU",
    "selected_equipment": "EQ123",
    "equipment_name": "Honda HRX217"
  }
}
```

**Response Format (200 OK):**
```json
{
  "conversation_id": "CONV123",
  "flagged": true,
  "dealer_notified": true
}
```

**Questions for Lizzy Team:**

1. **Dealer notifications:**
   - How should dealers be notified?
   - Email? SMS? In-app notification?
   - Real-time alerts or periodic checks?

2. **Priority levels:**
   - Do you support priority flagging?
   - Values: urgent, normal, low?

3. **Context information:**
   - What context is helpful for dealers?
   - Equipment info? Order history? Last interaction?

**Note:** This enables customers to escalate to human dealers when bot can't help. Can be deferred if not critical for MVP.

---

### 7. 🟡 Get Conversation Status

**Priority:** IMPORTANT - Needed for dealer handoff

**Endpoint:** `GET /api/sms/conversation/{phone}/status`

**Purpose:** Check if dealer has taken over conversation (bot should pause)

**Path Parameters:**
- `phone` (string, required) - Customer phone number

**Request Headers:**
```
Authorization: Bearer {API_KEY}
Accept: application/json
```

**Response Format (200 OK):**
```json
{
  "conversation_id": "CONV123",
  "dealer_active": true,
  "last_dealer_message": "2025-11-05T10:45:00Z",
  "bot_paused": true
}
```

**Questions for Lizzy Team:**

1. **Dealer takeover mechanism:**
   - How does dealer "take over" a conversation?
   - Manual action in interface?
   - Automatic based on dealer response?

2. **Bot resume:**
   - How does bot know when to resume?
   - Dealer explicitly releases conversation?
   - Timeout after dealer inactivity?
   - Customer types "MENU" to restart?

3. **State management:**
   - Who maintains conversation state?
   - Lizzy or our system?

**Note:** This enables seamless bot-to-dealer handoff. Can be simplified for MVP.

---

### 8. 🟢 Get Order Details (Optional)

**Priority:** OPTIONAL - Only if not included in transactions endpoint

**Endpoint:** `GET /api/orders/{order_id}/detail`

**Purpose:** Get extended order details if not included in transactions endpoint

**Questions:**
1. Is this needed, or does transactions endpoint provide sufficient detail?
2. What additional information would this include?

**Note:** We can defer this if transactions endpoint provides enough detail for MVP.

---

### 9. 🟢 Get Invoice Details (Optional)

**Priority:** OPTIONAL - Only if not included in transactions endpoint

**Endpoint:** `GET /api/invoices/{invoice_id}/detail`

**Purpose:** Get extended invoice details if not included in transactions endpoint

**Questions:**
1. Is this needed, or does transactions endpoint provide sufficient detail?
2. What additional information would this include (line items, payment history)?

**Note:** We can defer this if transactions endpoint provides enough detail for MVP.

-----

## Data Model Requirements

### Equipment Object

**Required Fields:**
```json
{
  "equipment_id": "string",       // Unique identifier
  "make": "string",               // Manufacturer (e.g., "Honda")
  "model": "string",              // Model (e.g., "HRX217")
  "sku": "string",                // SKU for searching
  "unit_type": "string",          // Category: "Mower", "Chainsaw", etc.
  "serial_number": "string",      // For searching
  "status": "string"              // "Active", "In Service", "Inactive"
}
```

**Optional But Helpful:**
```json
{
  "purchase_date": "date",
  "last_service_date": "date",
  "resources": {
    "tutorial_video_url": "url",
    "owners_manual_url": "url"
  },
  "parts_catalog": [
    {
      "part_number": "string",
      "description": "string",
      "price": number,
      "availability": "string",
      "popularity_rank": number    // For Top 10 sorting
    }
  ]
}
```

**Questions:**
1. What field represents equipment category? (unit_type, category, equipment_type?)
2. Is SKU field available and searchable?
3. How should we sort parts? (popularity_rank, sales_count, alphabetical?)

### Parts Catalog Object

**Required Fields:**
```json
{
  "part_number": "string",        // Unique identifier
  "description": "string",        // Display name
  "price": number,                // Current price
  "availability": "string"        // "in_stock", "order", "out_of_stock"
}
```

**Helpful for MVP:**
```json
{
  "category": "string",           // "Filters", "Belts", "Blades"
  "popularity_rank": number       // For Top 10 common parts sorting
}
```

**Questions:**
1. How do you track "common" or "popular" parts?
2. Do you have usage statistics we could use?
3. For now, can we just display first 10 alphabetically?

### Order Status Values

**Service Orders:**
- What are all possible values?
- Suggested: "pending", "in_progress", "awaiting_parts", "complete", "ready_for_pickup"

**Parts Orders:**
- What are all possible values?
- Suggested: "pending", "processing", "shipped", "delivered", "will_call_ready"

**Invoices:**
- What are all possible values?
- Suggested: "unpaid", "partially_paid", "paid", "overdue"

-----

## Error Handling

### Standard Error Response Format

Please use consistent error format across all endpoints:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Customer with phone +12065550100 not found",
    "details": {
      "phone": "+12065550100"
    }
  }
}
```

### Standard HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 | OK | Successful GET request |
| 201 | Created | Successful POST creating resource |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Missing or invalid API key |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Business logic error (e.g., out of stock) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Temporary downtime |

### Our Error Handling

We will:
- ✅ Retry on 5xx errors (with exponential backoff)
- ✅ Display friendly messages to customers on 4xx errors
- ✅ Log all errors for debugging
- ✅ Escalate to dealer on persistent errors

-----

## Rate Limiting

### Our Expected Usage

**Per Session (typical):**
- 2-4 API calls over 15 minutes

**Overall Volume (initial):**
- 500-1,000 sessions/month
- ~35 sessions/day average
- ~2-4 calls/session = 70-140 API calls/day
- Peak: 3-5x average (holidays, promotions)

**Growth Projections:**
- Year 1: 500-1,000 sessions/month
- Year 2: 5,000-10,000 sessions/month
- Year 3: 20,000+ sessions/month

### Questions for Lizzy Team:

1. **Rate limits:**
   - What are your rate limits per API key?
   - Per second? Per minute? Per hour?
   - Different limits per endpoint?

2. **Rate limit headers:**
   - Do you provide rate limit headers?
   - `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`?

3. **Handling rate limits:**
   - 429 response when exceeded?
   - Retry-After header provided?
   - Best practices for backing off?

4. **Burst handling:**
   - Can we burst for short periods?
   - Example: 100 customers text during promotion

-----

## Testing & Staging

### Test Environment Needed

We need access to:

1. **Staging API** with:
   - Same endpoints as production
   - Test customer data we can modify
   - Test equipment we can create/update
   - Can create test orders without affecting production

2. **Test Credentials:**
   - Separate API key for staging
   - Test phone numbers that won't trigger real SMS
   - Test customer IDs we can use freely

3. **Test Data:**
   - 3-5 test customers with different scenarios:
     - Customer with 5-10 pieces of equipment
     - Customer with active service orders
     - Customer with parts orders
     - Customer with invoices
     - Customer with no equipment (error case)

### Our Testing Plan

**Phase 1: API Integration (Week 1)**
- Test all endpoints with Postman/curl
- Verify authentication works
- Validate response formats
- Test error scenarios

**Phase 2: Workflow Testing (Weeks 2-4)**
- Build SMS workflows using staging API
- Test with test phone numbers
- Verify data flow end-to-end
- Test error handling

**Phase 3: UAT (Week 5)**
- Test with 2-3 friendly dealers
- Real phone numbers, real equipment
- Monitor API performance
- Gather feedback

**Phase 4: Production Launch (Week 6)**
- Switch to production API
- Monitor carefully
- Be ready for quick fixes

### Questions for Lizzy Team:

1. **Staging environment:**
   - Do you have staging API available?
   - Same base URL with different credentials?
   - Or different URL?

2. **Test data:**
   - Can you provide test customer accounts?
   - Or can we create our own test data?
   - Data refresh frequency?

3. **Staging behavior:**
   - Does staging create real orders?
   - Or simulated/sandbox mode?
   - Can we create/delete test data?

-----

## Implementation Timeline

### Our Side (6 Weeks)

**Week 1: Setup & API Integration**
- Set up Twilio account
- Deploy Twilio Functions
- Test all Lizzy API endpoints
- Validate data formats

**Week 2-3: Core Workflows**
- Build Main Menu
- Build Troubleshoot Equipment
- Build Schedule Service
- Build Service Status

**Week 4: Additional Workflows**
- Build Parts Order Status
- Build View/Pay Invoice
- Build Order Parts (simplified)

**Week 5: Integration & Testing**
- Implement cross-workflow context
- End-to-end testing
- UAT with test dealers
- Bug fixes

**Week 6: Launch**
- Production deployment
- Soft launch with 2-3 dealers
- Monitor and refine

### What We Need From Lizzy Team

**Week 0 (Before We Start):**
- ✅ API documentation review and approval
- ✅ Staging environment access
- ✅ Test credentials
- ✅ Test customer data

**Week 1 (During Our Setup):**
- ✅ API authentication working
- ✅ Customer profile endpoint working
- ✅ Transactions endpoint working

**Week 2-3 (During Core Development):**
- ✅ Create service order endpoint working
- ✅ Create parts order endpoint working

**Week 4-5 (Nice to Have):**
- ✅ SMS logging endpoint (if available)
- ✅ Conversation flagging endpoint (if available)

**Week 6 (Production Launch):**
- ✅ Production API credentials
- ✅ Production endpoints verified
- ✅ Support contact for API issues

-----

## Appendix: Sample Responses

### Sample Customer Profile (Small Dataset)

```json
{
  "customer_id": "CUST001",
  "name": "John Smith",
  "email": "john.smith@email.com",
  "phone": "+12065551001",
  "addresses": {
    "primary": {
      "street": "456 Oak Ave",
      "city": "Seattle",
      "state": "WA",
      "zip": "98102"
    }
  },
  "equipment": [
    {
      "equipment_id": "EQ001",
      "make": "Honda",
      "model": "HRX217",
      "sku": "HRX217VKA",
      "unit_type": "Mower",
      "serial_number": "MZCG-1000001",
      "status": "Active",
      "resources": {
        "tutorial_video_url": "https://youtube.com/watch?v=example1",
        "owners_manual_url": "https://example.com/manuals/honda-hrx217.pdf"
      },
      "parts_catalog": [
        {
          "part_number": "17211-ZL8-023",
          "description": "Air Filter",
          "price": 12.99,
          "availability": "in_stock",
          "popularity_rank": 1
        },
        {
          "part_number": "98079-55846",
          "description": "Spark Plug",
          "price": 8.99,
          "availability": "in_stock",
          "popularity_rank": 2
        }
      ]
    }
  ]
}
```

### Sample Transactions Response

```json
{
  "service_orders": [
    {
      "order_id": "SO001",
      "equipment_id": "EQ001",
      "equipment_name": "Honda HRX217",
      "status": "in_progress",
      "estimated_completion": "2025-11-15",
      "created_date": "2025-11-10",
      "description": "Annual tune-up",
      "invoice": {
        "invoice_id": "INV001",
        "amount": 89.99,
        "status": "unpaid",
        "due_date": "2025-11-20",
        "invoice_url": "https://example.com/invoice/INV001",
        "payment_url": "https://example.com/pay/INV001"
      }
    }
  ],
  "parts_orders": [],
  "invoices": []
}
```

### Sample Service Order Creation Response

```json
{
  "order_id": "SO002",
  "invoice_id": "INV002",
  "status": "pending",
  "invoice_url": "https://example.com/invoice/INV002",
  "payment_url": "https://example.com/pay/INV002",
  "created_at": "2025-11-17T10:30:00Z"
}
```

-----

## Next Steps

### Immediate Actions Needed

1. **Review this document** with Lizzy technical team
2. **Answer all questions** marked throughout (48 questions total)
3. **Provide API documentation** (if more detailed specs exist)
4. **Set up staging environment** with test credentials
5. **Schedule kickoff call** to discuss integration approach

### Questions We'd Like to Discuss

1. **Authentication:** What method do you prefer?
2. **Consolidated approach:** Is the include parameter feasible?
3. **Response times:** Can endpoints respond in <2 seconds?
4. **Field names:** Do our assumed field names match your data model?
5. **Parts sorting:** How should we determine "Top 10 common parts"?
6. **SMS integration:** Do you have existing SMS interface?
7. **Timeline:** Does 6-week timeline work for your team?

### Contact Information

**Project Lead:** Jonathan  
**Project:** Lizzy Agentic Commerce SMS Workflows  
**Timeline:** 6 weeks from API availability  
**Priority:** High - Customer experience enhancement

-----

## Document Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2025-11-17 | Initial API requirements | Jonathan |

-----

**Thank you for your partnership on this project! We're excited to bring SMS workflows to Lizzy customers.**
