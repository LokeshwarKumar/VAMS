# VAMS Role-Based Access Control (RBAC) Configuration

## Role Summary
| Role | Description | Permissions |
|------|-------------|-------------|
| x_1917769_vams.visitor | Visitor/Guest | Create own requests, view own data |
| x_1917769_vams.host | Host Employee | Create requests, manage own visitors |
| x_1917769_vams.approver | Approval Authority | Approve/reject requests |
| x_1917769_vams.security | Security Personnel | Manage check-in/check-out |
| x_1917769_vams.receptionist | Reception Staff | View passes, manage visitor info |
| x_1917769_vams.facility_admin | System Administrator | Full access to all records |

---

## Detailed Access Rules

### Visitor Request Table (x_1917769_vams_visitor_request)

#### Create Access:
- **x_1917769_vams.visitor:** YES
- **x_1917769_vams.host:** YES
- **x_1917769_vams.approver:** NO
- **x_1917769_vams.security:** NO
- **x_1917769_vams.receptionist:** NO
- **x_1917769_vams.facility_admin:** YES

#### Read Access:
- **x_1917769_vams.visitor:** Own records only
- **x_1917769_vams.host:** Own records + managed visitors
- **x_1917769_vams.approver:** All records
- **x_1917769_vams.security:** All records
- **x_1917769_vams.receptionist:** All approved records
- **x_1917769_vams.facility_admin:** All records

#### Update Access:
- **x_1917769_vams.visitor:** Own records (until approved)
- **x_1917769_vams.host:** Own records (until approved)
- **x_1917769_vams.approver:** Approval status, comments
- **x_1917769_vams.security:** No update (read-only)
- **x_1917769_vams.receptionist:** No update (read-only)
- **x_1917769_vams.facility_admin:** All fields

#### Delete Access:
- **x_1917769_vams.visitor:** NO
- **x_1917769_vams.host:** NO
- **x_1917769_vams.approver:** NO
- **x_1917769_vams.security:** NO
- **x_1917769_vams.receptionist:** NO
- **x_1917769_vams.facility_admin:** YES

#### Field-Level Security:
| Field | Visitor | Host | Approver | Security | Reception | Admin |
|-------|---------|------|----------|----------|-----------|-------|
| Visitor Name | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Company Name | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Email | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Phone Number | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Host Employee | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Visit Date | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Purpose of Visit | Read/Write | Read/Write | Read | Read | Read | Read/Write |
| Status | Read | Read | Read/Write | Read | Read | Read/Write |
| Check-In | - | - | - | Read | - | Read/Write |
| Check-Out | - | - | - | Read | - | Read/Write |
| Approval Comments | Read | Read | Read/Write | - | - | Read/Write |

---

### Visitor Pass Table (x_1917769_vams_visitor_pass)

#### Create Access:
- **x_1917769_vams.visitor:** NO
- **x_1917769_vams.approver:** System flow only
- **x_1917769_vams.security:** NO
- **x_1917769_vams.receptionist:** NO
- **x_1917769_vams.facility_admin:** YES

#### Read Access:
- **x_1917769_vams.visitor:** Own passes only
- **x_1917769_vams.approver:** All passes
- **x_1917769_vams.security:** All active passes
- **x_1917769_vams.receptionist:** All passes (for printing)
- **x_1917769_vams.facility_admin:** All passes

#### Update Access:
- **x_1917769_vams.security:** pass_status only (mark as expired/cancelled)
- **x_1917769_vams.receptionist:** NO
- **x_1917769_vams.facility_admin:** All fields

#### Delete Access:
- **x_1917769_vams.facility_admin:** YES (only)

---

### Check-In Log Table (x_1917769_vams_checkin_log)

#### Create Access:
- **x_1917769_vams.security:** YES
- **x_1917769_vams.receptionist:** YES
- **x_1917769_vams.facility_admin:** YES

#### Read Access:
- **x_1917769_vams.security:** All logs
- **x_1917769_vams.receptionist:** All logs
- **x_1917769_vams.facility_admin:** All logs

#### Update Access:
- **x_1917769_vams.security:** All fields
- **x_1917769_vams.receptionist:** Limited fields (notes only)
- **x_1917769_vams.facility_admin:** All fields

#### Delete Access:
- **x_1917769_vams.facility_admin:** YES (only)

---

## ACL Configuration Checklist

### To be implemented in ServiceNow UI:

#### Visitor Request ACLs:
- [ ] Create: visitor, host, facility_admin
- [ ] Read: visitor (own), host (own/managed), approver (all), security (all), reception (approved), admin (all)
- [ ] Update: visitor (own/draft), host (own/draft), approver (status), admin (all)
- [ ] Delete: admin only
- [ ] Field: status - approver/admin write, others read
- [ ] Field: approval_comments - approver/admin write, others read

#### Visitor Pass ACLs:
- [ ] Create: facility_admin, system flows only
- [ ] Read: visitor (own), approver (all), security (all), reception (all), admin (all)
- [ ] Update: security (pass_status), admin (all)
- [ ] Delete: admin only
- [ ] Field: pass_status - security/admin write, others read

#### Check-In Log ACLs:
- [ ] Create: security, reception, admin
- [ ] Read: security (all), reception (all), admin (all)
- [ ] Update: security (all), reception (limited), admin (all)
- [ ] Delete: admin only

---

## Script Inclusions for Custom Logic

### Field Rule: Visitor Request - Auto-assign Approver
When a visitor request is created, automatically assign to the primary approver role.

### Workflow: Status Change Handler
Trigger related workflows when status changes:
- Approved → Create Pass Flow
- Rejected → Rejection Notification Flow

### Business Rule: Validate Visit Date
Ensure visit date is not in the past.

### Business Rule: Check Duplicate Requests
Prevent duplicate requests from same visitor for same date.

