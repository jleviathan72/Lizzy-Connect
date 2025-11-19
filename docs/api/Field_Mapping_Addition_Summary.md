# Field Mapping Tables - Addition Summary

**Added to:** Lizzy API Requirements Package  
**Date:** November 17, 2025  
**New Document:** Lizzy_API_Field_Mapping_Tables.md

-----

## What Was Added

### New Document: Lizzy_API_Field_Mapping_Tables.md (33KB)

**Purpose:** Provide Lizzy technical team with structured tables to map our API requirements to their actual database schema.

**Format:** Comprehensive mapping tables for all 7 API endpoints with 5 columns per table:
- **Column A:** Field Name (our requirement)
- **Column B:** Lizzy App Field Name (what they call it in UI)
- **Column C:** Lizzy Navigation Path (where to find in app)
- **Column D:** Lizzy Table.Column (actual database location)
- **Column E:** Comments (notes, clarifications)

-----

## Contents Overview

### 35 Detailed Mapping Tables Created

**API 1: Get Customer Profile (6 tables)**
1. Customer Profile Root Fields (4 fields)
2. Customer Address Fields (7 fields)
3. Equipment Root Fields (10 fields) ⭐ CRITICAL
4. Equipment Resources Fields (4 fields)
5. Parts Catalog Fields (6 fields) ⭐ CRITICAL
6. Include Parameter Support (7 combinations)

**API 2: Get Customer Transactions (7 tables)**
7. Service Order Fields (7 fields)
8. Service Order Invoice Fields (6 fields)
9. Parts Order Fields (8 fields)
10. Parts Order Items Fields (4 fields)
11. Parts Order Invoice Fields (5 fields)
12. Standalone Invoices Fields (10 fields)
13. Query Parameters (5 parameters)

**API 3: Create Service Order (2 tables)**
14. Request Fields (13 fields) ⭐ CRITICAL
15. Response Fields (6 fields)

**API 4: Create Parts Order (2 tables)**
16. Request Fields (11 fields) ⭐ CRITICAL
17. Response Fields (8 fields)

**API 5: Log SMS Message (2 tables)**
18. Request Fields (6 fields)
19. Response Fields (2 fields)

**API 6: Flag Conversation (2 tables)**
20. Request Fields (7 fields)
21. Response Fields (3 fields)

**API 7: Get Conversation Status (1 table)**
22. Response Fields (4 fields)

**Reference Tables (2 tables)**
23. Data Type Reference (9 types)
24. Phone Number Format (3 formats)

**Total: 183 fields mapped across 35 tables**

-----

## Key Features

### 1. Structured Format

Every table follows consistent format:

| Field Name (Our Requirement) | Lizzy App Field Name | Lizzy Navigation Path | Lizzy Table.Column | Comments |
|------------------------------|---------------------|----------------------|-------------------|----------|
| customer_id | ← Fill this in | ← Fill this in | ← Fill this in | ← Notes here |

**Easy for Lizzy team to complete systematically**

---

### 2. Embedded Questions

**78 specific questions embedded throughout tables:**

Critical questions like:
- "How should we determine 'Top 10 common parts'?"
- "What are ALL possible status values for service orders?"
- "Is SKU field available and searchable?"
- "Can you support the 'include' parameter approach?"
- "When is invoice created - immediately or async?"

**All questions clearly marked and easy to find**

---

### 3. Priority Indicators

Questions organized by priority:

**High Priority (13 questions)** ⭐
- Critical for MVP
- Blockers if not answered
- Examples: Parts sorting, status values, unit_type field

**Medium Priority (8 questions)**
- Important but workarounds exist
- Examples: Multi-part orders, include parameter support

**Low Priority (5 questions)**
- Nice to have
- Can defer to later
- Examples: Additional resources, historical data

---

### 4. Context for Each Field

Every table includes:
- ✅ Field purpose
- ✅ Data type expected
- ✅ Required vs optional
- ✅ Where it appears in response
- ✅ Related questions
- ✅ Validation rules (where applicable)

---

### 5. Examples Throughout

Real examples provided:
- Equipment: "Honda HRX217"
- Status values: "in_progress", "shipped", "paid"
- Phone formats: "+12065550100", "(206) 555-0100"
- Dates: "2025-11-17T10:30:00Z"

---

### 6. Return Instructions

Clear instructions for Lizzy team:

**What to return:**
1. Completed mapping tables
2. Answers to all questions
3. Sample API responses
4. ER diagram (if available)
5. Existing API docs (if any)

**Timeline:** Within 1 week

**Format:** This document with additions or their preferred format

-----

## Critical Tables to Review

### Table 1.3: Equipment Root Fields ⭐⭐⭐

**Most important table - 10 fields mapped:**
- equipment_id, make, model, SKU
- **unit_type** (critical for filtering)
- serial_number (critical for search)
- status (critical for workflow logic)
- purchase_date, last_service_date

**4 critical questions embedded:**
1. What do you call unit_type? Complete list of values?
2. Is SKU field distinct from model? Searchable?
3. What are ALL possible status values?
4. Is serial_number always populated and searchable?

---

### Table 1.5: Parts Catalog Fields ⭐⭐⭐

**Critical for MVP - 6 fields mapped:**
- part_number, description, price
- availability, category
- **popularity_rank** (for Top 10 sorting)

**Most critical question in entire document:**
> "How should we determine 'Top 10 common parts'?
> - Do you track sales frequency? Order count? Replacement interval?
> - Can you provide a sort field?
> - Or should we just show first 10 alphabetically for MVP?"

**This answer determines if MVP is feasible as planned**

---

### Table 2.1: Service Order Fields ⭐⭐

**7 fields with critical status question:**

> "What are ALL possible status values for service orders?
> We need to filter for 'in_progress' or 'in_service'"

**Suggested values provided:**
- pending, in_progress, awaiting_parts, ready_for_pickup, complete, cancelled

**Need complete list with definitions**

---

### Table 3.1: Create Service Order Request ⭐⭐

**13 request fields mapped including:**
- service_type, drop_off_type, description
- pickup_requested, pickup_date, pickup_address

**3 critical questions:**
1. Valid values for service_type?
2. Valid values for drop_off_type?
3. How is pickup handled in your workflow?

---

### Table 4.1: Create Parts Order Request ⭐⭐

**11 request fields with MVP-critical question:**

> "MVP: We only support 1 part per order.
> Is this acceptable?
> Or must we support multiple parts in single order?"

**This answer affects MVP scope**

-----

## How This Helps

### For Lizzy Technical Team

**Before:** Vague requirements like "we need equipment data"

**After:** Exact table with:
```
| equipment_id | ? | ? | customers.equipment_id | Primary key |
| make | ? | ? | equipment.manufacturer | Varchar(50) |
| model | ? | ? | equipment.model_name | Varchar(100) |
```

**Result:** They know exactly what to provide

---

### For You

**Before:** Assumptions about field names and data structure

**After:** Verified mapping from their actual database

**Result:** No surprises during integration

---

### For Integration

**Before:** Trial and error to find correct field names

**After:** Complete field mapping document

**Result:** Smooth integration, no guesswork

-----

## Usage Instructions

### Step 1: Send to Lizzy Team

**Include:**
1. Original API Requirements Document
2. **NEW:** Field Mapping Tables Document
3. Cover email explaining purpose

**Email template:**
```
Subject: API Integration - Field Mapping Tables

Hi [Lizzy Technical Team],

In addition to the API Requirements Document, I've created detailed 
field mapping tables to help us align on data structure.

Please complete the mapping tables by filling in columns B-E for each 
field. This will ensure we're calling the correct database fields and 
using the right field names.

Critical sections:
- Table 1.3: Equipment fields (especially unit_type)
- Table 1.5: Parts catalog (especially popularity_rank)
- Table 2.1: Service order status values
- Table 3.1: Service order creation fields

Target: Complete within 1 week
Questions: Contact Jonathan

Attached:
1. Lizzy_API_Requirements_Document.md (original)
2. Lizzy_API_Field_Mapping_Tables.md (NEW - fill this out)

Thanks!
```

---

### Step 2: Review Their Response

**Check for completeness:**
- [ ] All columns filled in?
- [ ] All questions answered?
- [ ] Sample responses provided?
- [ ] Status value lists complete?

**Identify gaps:**
- Missing field names?
- Unclear answers?
- Conflicting information?

---

### Step 3: Follow-up Call

**Schedule 30-60 minute call to:**
1. Clarify any unclear answers
2. Discuss critical questions (Top 10 parts, status values)
3. Review sample API responses
4. Align on timeline
5. Set up staging environment

---

### Step 4: Create Final Spec

**Using their responses:**
1. Update API Requirements with actual field names
2. Update Implementation Guide with correct paths
3. Update Flow Diagrams with actual status values
4. Create API testing checklist

-----

## Most Important Questions

### Top 10 Critical Questions for Lizzy Team

1. **How to determine "Top 10 common parts"?**
   - Table 1.5, Question 3
   - Make or break for MVP parts ordering

2. **Complete list of service order status values?**
   - Table 2.1, Question 1
   - Needed for filtering "in service" equipment

3. **Complete list of parts order status values?**
   - Table 2.3, Question 1
   - Needed for status display logic

4. **Complete list of invoice status values?**
   - Table 2.6, Question 2
   - Needed for payment status display

5. **What is unit_type field called? All possible values?**
   - Table 1.3, Question 1
   - Critical for equipment filtering

6. **Is SKU field available and searchable?**
   - Table 1.3, Question 2
   - Needed for SKU search feature

7. **Can you support 'include' parameter?**
   - Table 1.6, Question 1
   - Determines if we use consolidated or separate endpoints

8. **Valid values for service_type?**
   - Table 3.1, Question 1
   - Needed for service order creation

9. **Valid values for drop_off_type?**
   - Table 3.1, Question 2
   - Needed for drop-off options

10. **Must support multiple parts per order?**
    - Table 4.1, Question 1
    - Affects MVP scope (we planned single part only)

**These 10 answers determine if MVP is feasible as designed**

-----

## Next Steps After Lizzy Response

### Week 1: Validation
1. Review completed tables
2. Verify all questions answered
3. Test sample API calls
4. Validate data formats

### Week 2: Update Documentation
5. Update API Requirements with actual fields
6. Update Implementation Guide
7. Update Quick Reference
8. Update Flow Diagrams (if needed)

### Week 3: Begin Integration
9. Configure Twilio Functions with correct fields
10. Build first workflow (Main Menu)
11. Test customer profile retrieval
12. Verify equipment list display

### Week 4-6: Complete MVP
13. Build remaining workflows
14. Test all API integrations
15. Validate status filtering
16. Launch with test dealers

-----

## Summary

### What You're Sending Lizzy Team

**Two Documents:**

1. **Lizzy_API_Requirements_Document.md (45KB)**
   - High-level API requirements
   - Endpoint specifications
   - Sample requests/responses
   - 48 embedded questions

2. **Lizzy_API_Field_Mapping_Tables.md (33KB)** ⭐ NEW
   - 35 detailed mapping tables
   - 183 fields to map
   - 78 specific questions
   - Clear return instructions

**Total: 78KB of detailed API documentation**

### Expected Response

**Completed tables showing:**
- Actual field names in Lizzy
- Navigation paths in app
- Database table.column names
- Validation rules
- Sample data

**Plus:**
- Answers to all 78 questions
- Complete status value lists
- Sample API responses
- ER diagram (if available)

### Result

**Perfect alignment between:**
- Your requirements → Their implementation
- Your field names → Their field names
- Your assumptions → Their reality
- Your MVP scope → Their capabilities

**No surprises during integration! 🎯**

-----

## Files Updated

### New File Created
- ✅ `Lizzy_API_Field_Mapping_Tables.md` (33KB)

### Updated Documentation Index
- ✅ Added reference to new mapping document
- ✅ Updated file count to 12 documents
- ✅ Updated total size to 335KB

### Total Documentation Package
**12 files, 335KB, ~780 pages**

-----

**The field mapping tables are ready to send to Lizzy technical team. This structured approach will ensure smooth API integration with no surprises! 📋**
