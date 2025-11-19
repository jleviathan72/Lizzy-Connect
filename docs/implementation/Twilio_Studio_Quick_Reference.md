# Twilio Studio Quick Reference Card

**Version:** 1.2 MVP  
**Purpose:** Fast reference while building flows

-----

## Widget Quick Reference

| What You Need | Studio Widget | Key Settings |
|---------------|---------------|-------------|
| Show menu, wait for reply | **Send & Wait for Reply** | Message body, 15 min timeout |
| Show message, continue | **Send Message** | Message body only |
| Call Lizzy API | **HTTP Request** | URL, method, headers, save response |
| Branch on response | **Split Based On** | Variable to check, conditions |
| Custom logic | **Run Function** | Function name, parameters |
| Store data | **Set Variables** | Variable name, value |

-----

## Variable Cheat Sheet

### Access Customer Input
```
{{widgets.WIDGET_NAME.inbound.Body}}
```

### Access API Response
```
{{widgets.HTTP_WIDGET_NAME.parsed.FIELD_NAME}}
```

### Access Cross-Flow Context
```
{{trigger.execution.EquipmentId}}
{{trigger.execution.EquipmentName}}
{{trigger.execution.ReturnToService}}
```

### Store in Flow Variable
```
{{flow.variables.VARIABLE_NAME}}
```

### Customer Phone Number
```
{{trigger.message.From}}
```

-----

## Common Widget Configurations

### Send & Wait for Reply (Menu)
```
Widget Name: main_menu
Message Body: 
  Welcome! Reply with a number:
  1. Option One
  2. Option Two
  3. Option Three
```

### Split Based On (Check Response)
```
Widget Name: menu_choice
Variable: {{widgets.main_menu.inbound.Body}}
Conditions:
  - Equals "1" → go to option_1_widget
  - Equals "2" → go to option_2_widget  
  - Equals "3" → go to option_3_widget
  - No Condition Match → go to invalid_input
```

### HTTP Request (GET Customer)
```
Widget Name: get_customer
Request Method: GET
Request URL: {{env.LIZZY_API_URL}}/api/customers/by-phone/{{trigger.message.From}}
Headers:
  - Authorization: Bearer {{env.LIZZY_API_KEY}}
Parse JSON Response: Checked
Save Response As: customerProfile
```

### HTTP Request (POST Order)
```
Widget Name: create_order
Request Method: POST
Request URL: {{env.LIZZY_API_URL}}/api/service-orders
Headers:
  - Authorization: Bearer {{env.LIZZY_API_KEY}}
  - Content-Type: application/json
Request Body:
{
  "customer_id": "{{flow.variables.customerId}}",
  "equipment_id": "{{flow.variables.selectedEquipmentId}}",
  "description": "{{flow.variables.problemDescription}}"
}
Parse JSON Response: Checked
Save Response As: orderResult
```

### Run Function (Search Equipment)
```
Widget Name: search_sku
Function URL: /searchEquipmentBySKU
Function Parameters:
  - sku: {{widgets.sku_input.inbound.Body}}
  - customerProfile: {{flow.variables.customerProfile}}
Save Result As: searchResult
```

### Run Function (Trigger Other Flow)
```
Widget Name: trigger_service
Function URL: /triggerFlowWithContext
Function Parameters:
  - targetFlow: Schedule_Service
  - customerPhone: {{trigger.message.From}}
  - equipmentId: {{flow.variables.selectedEquipmentId}}
  - equipmentName: {{flow.variables.selectedEquipmentName}}
  - equipmentSN: {{flow.variables.selectedEquipmentSN}}
  - returnToService: false
Save Result As: triggerResult
```

-----

## Liquid Template Examples

### Display Equipment List
```liquid
Your Equipment - Reply with a number:
{% for equipment in flow.variables.customerProfile.equipment limit:10 %}
{{forloop.index}}. {{equipment.make}} {{equipment.model}} - SN: {{equipment.serial_number}}
{% endfor %}
Reply MENU for main menu
```

### Display with Equipment Context
```liquid
{{flow.variables.selectedEquipmentName}} - Schedule Service

Please describe the problem:
```

### Filter and Display
```liquid
{% assign inServiceOrders = flow.variables.transactions.service_orders | where: "status", "in_progress" %}
Equipment in Service:
{% for order in inServiceOrders limit:10 %}
{{forloop.index}}. {{order.equipment_name}} - #{{order.order_id}}
{% endfor %}
```

### Conditional Display
```liquid
{{flow.variables.selectedEquipmentName}}

{% if flow.variables.equipmentStatus == "In Service" %}
🔧 Currently In Service
Service Order: #{{flow.variables.currentOrderId}}
{% endif %}

Troubleshooting Resources:
📺 Tutorial: {{flow.variables.tutorialUrl}}
```

-----

## Environment Variables to Set

In Twilio Console → Functions & Assets → Configuration:

```
LIZZY_API_URL = https://api.lizzy.com
LIZZY_API_KEY = your_api_key_here
TWILIO_PHONE_NUMBER = +12065550100
SCHEDULE_SERVICE_FLOW_SID = FWxxxx...
ORDER_PARTS_FLOW_SID = FWxxxx...
SERVICE_STATUS_FLOW_SID = FWxxxx...
```

-----

## Cross-Flow Context Pattern

### Source Flow (e.g., Troubleshoot)
```
Widget: Run Function
Function: triggerFlowWithContext
Parameters:
  - targetFlow: "Schedule_Service"
  - customerPhone: {{trigger.message.From}}
  - equipmentId: {{flow.variables.selectedEquipmentId}}
  - equipmentName: {{flow.variables.selectedEquipmentName}}
  - equipmentSN: {{flow.variables.selectedEquipmentSN}}

Next: End flow or show confirmation
```

### Target Flow (e.g., Schedule Service)
```
FIRST WIDGET after Trigger:

Widget: Split Based On
Variable: {{trigger.execution.EquipmentId}}
Conditions:
  - Is Not Null → store context & skip equipment selection
  - No Condition Match → show equipment selection menu
```

### Store Context (if found)
```
Widget: Set Variables
Variables:
  - selectedEquipmentId = {{trigger.execution.EquipmentId}}
  - selectedEquipmentName = {{trigger.execution.EquipmentName}}
  - selectedEquipmentSN = {{trigger.execution.EquipmentSN}}

Next: Jump to problem description (skip equipment selection)
```

-----

## Testing Checklist

### For Each Flow:
- [ ] Happy path works (1 → 2 → 3 → complete)
- [ ] Invalid input handled (shows error, loops back)
- [ ] "MENU" keyword returns to Main Menu
- [ ] All decision branches lead somewhere
- [ ] No dead ends
- [ ] Equipment context displays correctly
- [ ] API calls return expected data
- [ ] Variables store correctly
- [ ] Messages fit in SMS (160 chars)

### For Cross-Flow:
- [ ] Context passes correctly
- [ ] Target flow detects context
- [ ] Equipment selection skipped when context present
- [ ] Equipment name displays in all messages
- [ ] Return flow works (Parts → Service)

-----

## Common Mistakes to Avoid

❌ **Forgetting to Save Response**
- Always set "Save Response As" for HTTP Request and Run Function

❌ **Wrong Variable Path**
- Use `{{widgets.NAME.inbound.Body}}` not `{{widgets.NAME.Body}}`
- Use `{{flow.variables.NAME}}` not `{{variables.NAME}}`

❌ **Missing Split Conditions**
- Always add "No Condition Match" path for invalid input

❌ **Hardcoded Values**
- Use environment variables for API URLs and keys
- Use variables for dynamic content

❌ **Not Testing All Paths**
- Test every menu option
- Test every error condition
- Test cross-flow transitions

❌ **Forgetting Message Length**
- Keep messages under 160 characters when possible
- Long lists will split into multiple messages

-----

## Widget Connection Tips

### Connect Widgets:
1. Click the red dot on source widget
2. Drag to target widget
3. Release to connect
4. Label connection (for Split Based On conditions)

### Organize Canvas:
- Left to right flow
- Group related widgets
- Use vertical space for branches
- Keep it readable

### Naming Convention:
- Use lowercase with underscores: `get_customer`
- Be descriptive: `search_equipment_by_sku`
- Match diagram names when possible

-----

## Troubleshooting

### Widget Not Executing
- Check transitions are connected
- Verify condition syntax in Split Based On
- Check logs in Execution Logs

### Variable Not Found
- Verify variable name spelling
- Check if widget executed before access
- Use correct path (widgets. vs flow.variables.)

### API Call Failing
- Check URL includes https://
- Verify headers set correctly
- Check API key in environment variables
- Test endpoint with Postman first

### Function Not Working
- Verify function is deployed
- Check function has correct parameters
- View function logs for errors
- Test function in Functions console

-----

## Build Order Reminder

1. Main Menu (test customer lookup)
2. Troubleshoot Equipment (test equipment display)
3. Schedule Service (test order creation)
4. Service Status (test transaction display)
5. Parts Order Status (similar to service)
6. View/Pay Invoice (similar to status)
7. Order Parts (test simplified parts)
8. Cross-flow triggers (test context passing)

**Build one at a time. Test thoroughly before moving to next.**

-----

## Key Success Factors

✅ **Start Simple** - Build Main Menu first
✅ **Test Often** - Test each widget after adding
✅ **Follow Diagrams** - Use as blueprint
✅ **Name Consistently** - Match documentation
✅ **Document As You Go** - Note any changes
✅ **One Flow at a Time** - Don't rush
✅ **Cross-Flow Last** - Get basics working first

-----

**Keep this card handy while building in Twilio Studio!**
