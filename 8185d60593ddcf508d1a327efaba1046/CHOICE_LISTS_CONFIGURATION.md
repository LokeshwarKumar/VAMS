# VAMS Choice Lists Configuration

## Overview
This document provides the choice values for choice-type fields in VAMS tables. These choices should be configured in ServiceNow as Choice Lists.

---

## Choice Lists

### 1. Visitor Request Status (x_1917769_vams_visitor_request.status)
**Label:** Status
**Type:** Choice
**Default:** submitted

| Value | Label | Display | Order |
|-------|-------|---------|-------|
| submitted | Submitted | Submitted | 1 |
| approved | Approved | Approved | 2 |
| rejected | Rejected | Rejected | 3 |
| expired | Expired | Expired | 4 |

**Configuration in ServiceNow:**
1. Go to System Definition → Choices
2. Create new choice with:
   - Name: x_1917769_vams_visitor_request.status
   - Table: x_1917769_vams_visitor_request
   - Element: status
   - Add choices as table above

---

### 2. Visitor Pass Status (x_1917769_vams_visitor_pass.pass_status)
**Label:** Pass Status
**Type:** Choice
**Default:** active

| Value | Label | Display | Order | Color |
|-------|-------|---------|-------|-------|
| active | Active | Active | 1 | Green |
| expired | Expired | Expired | 2 | Orange |
| cancelled | Cancelled | Cancelled | 3 | Red |

**Configuration in ServiceNow:**
1. Go to System Definition → Choices
2. Create new choice with:
   - Name: x_1917769_vams_visitor_pass.pass_status
   - Table: x_1917769_vams_visitor_pass
   - Element: pass_status
   - Add choices as table above

---

### 3. Check-In Log Status (x_1917769_vams_checkin_log.status)
**Label:** Status
**Type:** Choice
**Default:** checked_in

| Value | Label | Display | Order | Color |
|-------|-------|---------|-------|-------|
| checked_in | Checked In | Checked In | 1 | Green |
| checked_out | Checked Out | Checked Out | 2 | Gray |

**Configuration in ServiceNow:**
1. Go to System Definition → Choices
2. Create new choice with:
   - Name: x_1917769_vams_checkin_log.status
   - Table: x_1917769_vams_checkin_log
   - Element: status
   - Add choices as table above

---

## Choice List Instructions

### Creating Choice List in ServiceNow UI

1. **Navigate to Choices:**
   - Path: System Definition → Choices
   - Or search "sys_choice.list"

2. **Create New Record:**
   - Click "New"
   - Fill in the details:

   ```
   Table: [Table name]
   Element: [Field name]
   Language: English
   [Add rows with Value, Label, Display, Order]
   ```

3. **Add Choice Rows:**
   - Click "Insert Row" for each choice
   - Fill in Value, Label, Display, Order
   - For colors: Add custom style or use status colors

4. **Save and Verify:**
   - Save the choice list
   - Go to a form with that field
   - Click the field → should show dropdown with choices

---

## SQL Equivalent (Direct Insert - Advanced Users Only)

If needed, you can insert choices directly using SQL (use with caution):

```sql
-- Visitor Request Status
INSERT INTO sys_choice (name, `element`, label, value, language, `order`)
VALUES 
('x_1917769_vams_visitor_request', 'status', 'Submitted', 'submitted', 'en', 1),
('x_1917769_vams_visitor_request', 'status', 'Approved', 'approved', 'en', 2),
('x_1917769_vams_visitor_request', 'status', 'Rejected', 'rejected', 'en', 3),
('x_1917769_vams_visitor_request', 'status', 'Expired', 'expired', 'en', 4);

-- Visitor Pass Status
INSERT INTO sys_choice (name, `element`, label, value, language, `order`)
VALUES 
('x_1917769_vams_visitor_pass', 'pass_status', 'Active', 'active', 'en', 1),
('x_1917769_vams_visitor_pass', 'pass_status', 'Expired', 'expired', 'en', 2),
('x_1917769_vams_visitor_pass', 'pass_status', 'Cancelled', 'cancelled', 'en', 3);

-- Check-In Log Status
INSERT INTO sys_choice (name, `element`, label, value, language, `order`)
VALUES 
('x_1917769_vams_checkin_log', 'status', 'Checked In', 'checked_in', 'en', 1),
('x_1917769_vams_checkin_log', 'status', 'Checked Out', 'checked_out', 'en', 2);
```

---

## Field Validation

### Visitor Request Status Field
**Name:** status
**Field Type:** Choice
**Choices to Add:**
1. submitted → "Submitted" (Default)
2. approved → "Approved"
3. rejected → "Rejected"
4. expired → "Expired"

**Validation Rules:**
- Can only transition: submitted → approved/rejected → (nothing)
- Once approved/rejected, cannot be changed by visitors
- Only approvers and admins can change status

---

### Visitor Pass Status Field
**Name:** pass_status
**Field Type:** Choice
**Choices to Add:**
1. active → "Active" (Default)
2. expired → "Expired"
3. cancelled → "Cancelled"

**Validation Rules:**
- Set to "Active" when pass is created
- Only security/admin can change status
- Once cancelled, cannot be reactivated
- Auto-mark as "Expired" when expiry_date passes

---

### Check-In Log Status Field
**Name:** status
**Field Type:** Choice
**Choices to Add:**
1. checked_in → "Checked In" (Default)
2. checked_out → "Checked Out"

**Validation Rules:**
- Created as "Checked In"
- Can only transition: checked_in → checked_out
- Once checked_out, cannot revert
- check_out_time must be after check_in_time

---

## Dynamic Choice Lists (Optional)

### Reference Choices for Host Employee
Instead of manually maintaining a choice list for host employee, use the reference field which automatically shows users with the x_1917769_vams.host role.

**Field:** host_employee
**Type:** Reference
**Reference Table:** sys_user
**Dynamic Reference Qualifier:** 
```
user has_role x_1917769_vams.host
```

---

## Testing Choice Lists

After creating choice lists, test them:

1. **UI Test:**
   - Open a form with the choice field
   - Click the field
   - Verify all choices appear in dropdown
   - Select a choice → verify it saves

2. **Script Test:**
   - In browser console, run:
   ```javascript
   // Get choice list values
   GlideChoiceList.getChoicesByTable('x_1917769_vams_visitor_pass', 'pass_status');
   ```

3. **Report Test:**
   - Create a report filtering by one of the choice fields
   - Verify filter dropdown shows all choices
   - Filter by each choice → verify results

---

## Localization (i18n)

To support multiple languages, configure message definitions:

1. Navigate to: System Localization → Messages
2. Create message keys for each choice:
   ```
   x_1917769_vams_visitor_request.status.submitted
   x_1917769_vams_visitor_request.status.approved
   ... etc
   ```

3. Add translations for each language

---

## Migration from Legacy System

If migrating choice values from a legacy system:

1. **Mapping Document:**
   ```
   Legacy System → VAMS
   Draft → submitted
   Pending → submitted  
   Authorized → approved
   Denied → rejected
   Lapsed → expired
   ```

2. **Data Migration Script:**
   - Use GlideRecord to read legacy data
   - Map values using mapping document
   - Insert into new VAMS tables

3. **Validation:**
   - Verify all records mapped correctly
   - Check for orphaned or invalid values
   - Audit data integrity

---

## Troubleshooting Choice Lists

### Issue: Choices not appearing in dropdown
**Solution:**
1. Verify sys_choice records exist
2. Check table and element names match exactly
3. Clear browser cache (Ctrl+Shift+Delete)
4. Reload the form

### Issue: Choice value not saving
**Solution:**
1. Verify field type is "Choice"
2. Check for field validation rules blocking saves
3. Verify user has write permission to field
4. Review browser console for JavaScript errors

### Issue: Wrong choice displaying
**Solution:**
1. Verify choice value in database (Table x_1917769_vams_*)
2. Check for duplicate choice definitions
3. Verify language is set correctly (should be 'en')
4. Review table_name.field_name configuration

---

## Choice List Reference Template

Use this template for documenting custom choices:

```
Choice List: [Name]
Table: [Table Name]
Field: [Field Name]
Type: [Choice/Reference]
Default: [Default Value]

Values:
| Value | Label | Order | Notes |
|-------|-------|-------|-------|
| | | | |
| | | | |

Validation Rules:
- 
- 

Usage:
- Used in: [Flows/Reports/etc]
- Modified by: [Roles]
```

