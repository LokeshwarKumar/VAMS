# VAMS Related Lists & Data Relationships Configuration

## Related Lists Setup

### 1. Visitor Request → Visitor Pass (One-to-Many)

#### On Visitor Request Form:
- **Related List Name:** "Visitor Passes"
- **Table:** x_1917769_vams_visitor_pass
- **Filter:** visitor_request = [current request ID]
- **Display Columns:**
  - Pass Number
  - Badge ID
  - Issue Date
  - Expiry Date
  - Pass Status

#### Actions Available:
- View Pass details
- Print Badge
- Mark as Expired/Cancelled

#### Display Options:
- Show count badge
- Sort by Issue Date (Descending)
- Max rows: 10

---

### 2. Visitor Request → Check-In Logs (Indirect - via Visitor Pass)

#### On Visitor Request Form:
- **Related List Name:** "Check-In Activity"
- **Table:** x_1917769_vams_checkin_log
- **Filter:** visitor_pass.visitor_request = [current request ID]
- **Display Columns:**
  - Pass Number (from visitor_pass)
  - Check In Time
  - Check Out Time
  - Duration (calculated)
  - Checked In By
  - Status

#### Actions Available:
- View log details
- Export log data

---

### 3. Visitor Pass → Check-In Logs (One-to-Many)

#### On Visitor Pass Form:
- **Related List Name:** "Check-In/Out History"
- **Table:** x_1917769_vams_checkin_log
- **Filter:** visitor_pass = [current pass ID]
- **Display Columns:**
  - Check In Time
  - Check Out Time
  - Checked In By
  - Checked Out By
  - Status
  - Duration (calculated)

#### Actions Available:
- Create new check-in log
- View/update log details
- Export history

#### Display Options:
- Sort by Check In Time (Descending)
- Show only active (checked in) visitors
- Highlight long durations (> 8 hours)

---

### 4. Visitor Pass → Visitor Request (Reference Link)

#### On Visitor Pass Form:
- **Parent Link:** Click to view original visitor request
- **Display:** Request number, status, visitor details
- **Breadcrumb Navigation:** Show context

---

### 5. Check-In Log → Visitor Pass (Reference Link)

#### On Check-In Log Form:
- **Parent Link:** Click to view visitor pass details
- **Display:** Pass number, badge ID, visitor name
- **Inline Details:** Show pass status and expiry date

---

## Business Rules for Relationships

### Rule 1: Prevent Pass Creation for Non-Approved Requests
```javascript
if (current.visitor_request.status != 'approved') {
    gs.addErrorMessage('Cannot create pass for non-approved requests');
    action.setAbortAction(true);
}
```

### Rule 2: Prevent Check-In for Expired Passes
```javascript
var pass = current.visitor_pass;
if (pass.pass_status == 'Expired' || pass.expiry_date < gs.now()) {
    gs.addErrorMessage('Cannot check-in with expired pass');
    action.setAbortAction(true);
}
```

### Rule 3: Auto-Calculate Check-Out Duration
```javascript
if (current.check_out_time != null && current.check_in_time != null) {
    var duration = (current.check_out_time - current.check_in_time) / 1000 / 60; // minutes
    current.duration_minutes = duration;
}
```

### Rule 4: Auto-Update Check-In Log Status
```javascript
if (current.check_out_time != null) {
    current.status = 'Checked Out';
}
```

---

## Data Synchronization Rules

### Sync Rule 1: Copy Visitor Name to Pass
When Visitor Request is approved:
- Copy visitor_name from request → visitor_pass.visitor_name
- Copy company_name to pass record for easy reference

### Sync Rule 2: Update Visitor Request with Pass Number
When Visitor Pass is created:
- Update visitor_request.pass_id with new pass number (optional field)

---

## Reference Qualifiers

### Visitor Request → Host Employee
- **Reference:** sys_user
- **Qualifier:** HAS ROLE: x_1917769_vams.host

### Check-In Log → Checked In By
- **Reference:** sys_user  
- **Qualifier:** HAS ROLE: x_1917769_vams.security OR x_1917769_vams.receptionist

### Check-In Log → Checked Out By
- **Reference:** sys_user
- **Qualifier:** HAS ROLE: x_1917769_vams.security OR x_1917769_vams.receptionist

---

## Related List Configuration Checklist

- [ ] Add Visitor Pass related list to Visitor Request form
- [ ] Add Check-In Logs related list to Visitor Pass form
- [ ] Configure column display for each related list
- [ ] Set up sorting and filtering
- [ ] Create custom display scripts if needed
- [ ] Add action buttons for common operations
- [ ] Configure related list actions (create, delete, etc.)
- [ ] Test related list functionality

---

## Form Layout Recommendations

### Visitor Request Form Layout:
```
[Header Section]
- Request Number, Status, Created Date

[Visitor Information]
- Visitor Name, Company, Email, Phone

[Visit Details]
- Visit Date, Host Employee, Purpose

[Approval Section]
- Status, Approval Comments, Approver

[Related Lists]
- Visitor Passes (tab)
- Check-In Activity (tab)

[Activity Log]
- Comments, Updates, Approvals
```

### Visitor Pass Form Layout:
```
[Header Section]
- Pass Number, Badge ID, Status

[Pass Details]
- Visitor Name, Issue Date, Expiry Date

[Reference Section]
- Visitor Request (link to original)

[Related Lists]
- Check-In/Out History (tab)

[Activity Log]
- Status changes, Expiry notifications
```

### Check-In Log Form Layout:
```
[Header Section]
- Record ID, Status

[Visitor Information]
- Visitor Pass (reference), Visitor Name

[Check-In Details]
- Check In Time, Checked In By

[Check-Out Details]
- Check Out Time, Checked Out By, Duration

[Notes]
- Additional remarks field
```

