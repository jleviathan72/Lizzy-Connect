# Version 1.2 - SKU/SN Selection Update Summary

**Date:** November 13, 2025  
**Update Type:** Equipment Selection Enhancement  
**Scope:** Workflow 2 (Schedule Service) and Workflow 4 (Order Parts)

-----

## Changes Implemented

### Overview
Added SKU and Serial Number selection options to workflows that begin with equipment selection, providing customers with four ways to select equipment:
1. Send Full Fleet List
2. Filter Fleet List by Type (then by Model)
3. Select Equipment by SKU
4. Select Equipment by SN

-----

## Workflow 2: Schedule Service

### Updated Menu

**Node:** `SCHEDULE_SERVICE_MENU`

**New Options:**
```
Schedule Service - Reply with a number:
1. Send Full Fleet List
2. Filter Fleet List by Type
3. Select Equipment by SKU       ← NEW
4. Select Equipment by SN        ← NEW
```

### New Nodes Added

#### 1. SELECT_BY_SKU_SERVICE
- Prompts customer to enter equipment SKU
- Searches cached equipment data (case-insensitive)
- On success: Proceeds to `SERVICE_PROBLEM_DESCRIPTION`
- On failure: "SKU not found in your equipment list. Please try again or reply MENU."

#### 2. SELECT_BY_SN_SERVICE
- Prompts customer to enter equipment serial number
- Searches cached equipment data (case-insensitive)
- On success: Proceeds to `SERVICE_PROBLEM_DESCRIPTION`
- On failure: "Serial number not found in your equipment list. Please try again or reply MENU."

-----

## Workflow 4: Order Parts

### Updated Menu

**Node:** `ORDER_PARTS_MENU`

**New Options:**
```
Order Parts - Reply with a number:
1. Send Full Fleet List
2. Filter Fleet List by Type
3. Select Equipment by SKU       ← NEW
4. Select Equipment by SN        ← NEW
```

### New Nodes Added

#### 1. SELECT_BY_SKU_PARTS
- Prompts customer to enter equipment SKU
- Searches cached equipment data (case-insensitive)
- On success: Proceeds to `SHOW_PARTS_LIST`
- On failure: "SKU not found in your equipment list. Please try again or reply MENU."

#### 2. SELECT_BY_SN_PARTS
- Prompts customer to enter equipment serial number
- Searches cached equipment data (case-insensitive)
- On success: Proceeds to `SHOW_PARTS_LIST`
- On failure: "Serial number not found in your equipment list. Please try again or reply MENU."

-----

## Workflows NOT Modified

The following workflows were reviewed and determined NOT to need SKU/SN selection:

### Workflow 1: Troubleshoot Equipment
- ✅ Already has SKU/SN selection (implemented in Version 1.2 initial release)

### Workflow 3: Service Status
- ❌ No equipment selection needed
- Shows list of equipment currently in service
- Customer selects from existing service orders

### Workflow 5: Parts Order Status
- ❌ No equipment selection needed
- Shows list of existing parts orders
- Customer selects from existing orders

### Workflow 6: View or Pay Invoice
- ❌ No equipment selection needed
- Shows list of invoices by type
- Customer selects from existing invoices

-----

## Implementation Details

### Search Logic
All SKU/SN search nodes follow the same pattern:
1. Accept free-form text input from customer
2. Search cached customer profile data (from session initialization)
3. Perform case-insensitive matching
4. Store equipment context in session if found
5. Display error message and allow retry if not found

### Session Data Stored
When equipment is successfully found via SKU or SN:
- `selectedEquipmentId`: Equipment ID for API calls
- `selectedEquipmentName`: Display name (e.g., "Honda HRX217")
- `selectedEquipmentSN`: Serial number for context display

### Error Handling
- Invalid input: "SKU not found..." or "Serial number not found..."
- Customer can retry entry
- Customer can type "MENU" to return to main menu at any time

-----

## Node Count Summary

**New Nodes Added:** 4
- `SELECT_BY_SKU_SERVICE`
- `SELECT_BY_SN_SERVICE`
- `SELECT_BY_SKU_PARTS`
- `SELECT_BY_SN_PARTS`

**Total Nodes in Version 1.2:** 15 new nodes (11 from initial release + 4 from this update)

-----

## Benefits

### User Experience
- **Faster Selection:** Customers who know their SKU or serial number can skip browsing lists
- **Power User Support:** Technical users or those with documentation can quickly access their equipment
- **Consistency:** All equipment selection workflows now offer the same 4 selection methods

### Technical
- **No Additional API Calls:** Uses cached customer profile data
- **Efficient Search:** Case-insensitive matching on cached data
- **Graceful Degradation:** Clear error messages when equipment not found

-----

## Testing Considerations

### Test Cases for Each New Node

**SKU Selection:**
1. Valid SKU (exact match)
2. Valid SKU (case variation)
3. Invalid SKU (not in customer's equipment list)
4. Invalid SKU (typo/partial match)
5. Empty input
6. Special characters in input

**Serial Number Selection:**
1. Valid SN (exact match)
2. Valid SN (case variation)
3. Invalid SN (not in customer's equipment list)
4. Invalid SN (typo/partial match)
5. Empty input
6. Special characters in input

**Cross-Workflow Testing:**
1. Verify equipment context carries forward after SKU/SN selection
2. Verify session data persistence
3. Verify error recovery path works correctly

-----

## Next Steps

1. ✅ Update requirements document
2. ✅ Add new nodes to all workflows with equipment selection
3. ⏳ Update workflow diagrams to reflect new paths
4. ⏳ Implement search logic in workflow engine
5. ⏳ Create test scripts for SKU/SN search functionality
6. ⏳ Document SKU and Serial Number field names in Lizzy API

-----

## Questions for Lizzy Team

1. What field name contains the SKU in the equipment data structure?
2. Is SKU guaranteed to be unique per customer, or globally unique?
3. Are there any special characters allowed in SKUs?
4. What field name contains the serial number in the equipment data structure?
5. Is serial number guaranteed to be unique per customer, or globally unique?
6. Should SKU/SN search be fuzzy (partial match) or exact match only?

-----

## Change Log Reference

Updated main requirements document change log:
- Version 1.2 now includes: "Added SKU/SN selection to Schedule Service and Order Parts workflows"
