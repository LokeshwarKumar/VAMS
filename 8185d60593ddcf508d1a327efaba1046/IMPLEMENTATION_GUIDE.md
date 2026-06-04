# VAMS Complete Implementation Guide

## Project Summary
The Visitor Access Management System (VAMS) is a comprehensive ServiceNow application that manages visitor requests, generates visitor passes, and logs check-in/check-out activities with role-based access control.

## Implementation Completed ✓

### 1. Database Tables (XML Exported)
- ✓ **x_1917769_vams_visitor_request** - Core visitor request table
- ✓ **x_1917769_vams_visitor_pass** - Visitor pass management (NEW)
- ✓ **x_1917769_vams_checkin_log** - Check-in/check-out tracking (NEW)

### 2. Existing Roles
- ✓ x_1917769_vams.visitor
- ✓ x_1917769_vams.host
- ✓ x_1917769_vams.approver
- ✓ x_1917769_vams.security
- ✓ x_1917769_vams.receptionist
- ✓ x_1917769_vams.facility_admin

### 3. Application Modules (XML Exported)
- ✓ My Visitor Requests
- ✓ Pending Approvals
- ✓ Visitor Passes
- ✓ Check-In Logs

### 4. Security ACLs (XML Exported)
- ✓ Visitor Pass - Create Access
- ✓ Check-In Log - Create Access
- ✓ Role-based permissions configured

---

## Implementation Steps

### Phase 1: Import XML Files (Automated)
All XML configuration files are ready for import into your ServiceNow instance:

```
Location: 8185d60593ddcf508d1a327efaba1046/
├── dictionary/
│   ├── x_1917769_vams_visitor_pass.xml
│   └── x_1917769_vams_checkin_log.xml
└── update/
    ├── sys_dictionary_*.xml (13 new field definitions)
    ├── sys_app_module_*.xml (4 modules)
    └── sys_security_acl_*.xml (ACL rules)
```

**Import Instructions:**
1. Log into your ServiceNow instance
2. Navigate to: System Import Sets → Import a File
3. Select all XML files from the update/ folder
4. Click "Import"
5. Verify import completion and resolve any conflicts

---

### Phase 2: Flow Designer Configuration (Manual)
See: `FLOWS_CONFIGURATION.md`

**Create 3 Main Flows:**

#### Flow 1: Visitor Request Submission Notification
- Trigger: Record created in x_1917769_vams_visitor_request
- Action: Send email to approvers
- **Time to configure:** 5-10 minutes

#### Flow 2: Visitor Request Approval Processing
- Trigger: Status field changed to "Approved"
- Actions:
  - Create visitor pass record
  - Generate badge ID
  - Send approval notification
- **Time to configure:** 15-20 minutes

#### Flow 3: Visitor Request Rejection Notification
- Trigger: Status field changed to "Rejected"
- Action: Send rejection email to visitor
- **Time to configure:** 5-10 minutes

---

### Phase 3: Reports Configuration (Manual)
See: `REPORTS_CONFIGURATION.md`

**Create 3 Reports:**

1. **Approved Visitor Requests**
   - Filter by status = "approved"
   - Columns: Visitor Name, Company, Host Employee, Visit Date
   - Access: Approvers, Facility Admin
   - **Time:** 10 minutes

2. **Active Visitor Passes**
   - Filter by pass_status = "Active" AND expiry_date >= NOW()
   - Columns: Pass Number, Visitor, Badge ID, Expiry Date
   - Export to PDF capability
   - **Time:** 15 minutes

3. **Daily Check-In Summary**
   - Filter by check_in_time >= TODAY()
   - Group by status and time
   - Include KPIs: Total visitors, Current on-site
   - **Time:** 20 minutes

---

### Phase 4: Access Control Configuration (Manual)
See: `RBAC_CONFIGURATION.md`

**Configure ACLs:**

For each table (x_1917769_vams_visitor_request, x_1917769_vams_visitor_pass, x_1917769_vams_checkin_log):

1. Navigate to System Security → Access Control Lists
2. Create rules for: Create, Read, Update, Delete
3. Assign roles based on RBAC matrix
4. Configure field-level security where needed
5. **Time:** 30-40 minutes

---

### Phase 5: Related Lists & Forms (Manual)
See: `RELATED_LISTS_CONFIGURATION.md`

**Configure Related Lists:**

1. **Visitor Request Form:**
   - Add "Visitor Passes" related list
   - Add "Check-In Activity" related list

2. **Visitor Pass Form:**
   - Add "Check-In/Out History" related list

3. **Check-In Log Form:**
   - Reference links to parent records

**Time:** 15-20 minutes

---

## File Structure Summary

```
VAMS/
├── 8185d60593ddcf508d1a327efaba1046/
│   ├── dictionary/
│   │   ├── x_1917769_vams_visitor_request.xml (existing)
│   │   ├── x_1917769_vams_visitor_pass.xml (NEW)
│   │   └── x_1917769_vams_checkin_log.xml (NEW)
│   │
│   └── update/
│       ├── Tables (3 total)
│       │   ├── sys_dictionary_x_1917769_vams_visitor_pass_*.xml (7 fields)
│       │   └── sys_dictionary_x_1917769_vams_checkin_log_*.xml (6 fields)
│       │
│       ├── Modules (4 total)
│       │   ├── sys_app_module_my_visitor_requests.xml
│       │   ├── sys_app_module_pending_approvals.xml
│       │   ├── sys_app_module_visitor_passes.xml
│       │   └── sys_app_module_checkin_logs.xml
│       │
│       └── Security (5 total)
│           ├── sys_security_acl_visitor_pass_create.xml
│           ├── sys_security_acl_role_visitor_pass_create.xml
│           ├── sys_security_acl_checkin_log_create.xml
│           ├── sys_security_acl_role_checkin_log_create_security.xml
│           └── sys_security_acl_role_checkin_log_create_receptionist.xml
│
├── FLOWS_CONFIGURATION.md (Flow Designer setup)
├── REPORTS_CONFIGURATION.md (Reports setup)
├── RBAC_CONFIGURATION.md (Access control)
└── RELATED_LISTS_CONFIGURATION.md (Data relationships)
```

---

## Field Definitions Summary

### Visitor Pass Table
| Field | Type | Mandatory | Unique | Notes |
|-------|------|-----------|--------|-------|
| pass_number | Auto Number | Yes | No | Auto-generated |
| visitor_request | Reference | Yes | No | Links to visitor request |
| visitor_name | String | Yes | No | Copied from request |
| issue_date | Date/Time | Yes | No | Auto-set on creation |
| expiry_date | Date/Time | Yes | No | Based on visit date |
| badge_id | String | Yes | Yes | Unique identifier |
| pass_status | Choice | Yes | No | Active/Expired/Cancelled |

### Check-In Log Table
| Field | Type | Mandatory | Notes |
|-------|------|-----------|-------|
| visitor_pass | Reference | Yes | Links to visitor pass |
| check_in_time | Date/Time | Yes | When visitor arrives |
| check_out_time | Date/Time | No | When visitor leaves |
| checked_in_by | Reference (sys_user) | Yes | Who recorded check-in |
| checked_out_by | Reference (sys_user) | No | Who recorded check-out |
| status | Choice | Yes | Checked In/Checked Out |

---

## Workflow & Integration Points

### Record Lifecycle

```
Visitor Request Created
    ↓
[Flow 1] Send Approval Notification
    ↓
Approver Reviews & Approves
    ↓
[Flow 2] Create Visitor Pass & Generate Badge
    ↓
Visitor Receives Pass
    ↓
Visitor Arrives (Check-In)
    ↓
Security Logs Check-In → Create Check-In Log
    ↓
Visitor On-Premises
    ↓
Visitor Leaves (Check-Out)
    ↓
Security Logs Check-Out → Update Check-In Log
    ↓
Complete
```

### Alternative Path (Rejection)
```
Visitor Request Created
    ↓
Approver Reviews & Rejects
    ↓
[Flow 3] Send Rejection Notification
    ↓
End
```

---

## Testing Checklist

### Unit Testing
- [ ] Visitor request creation with required fields
- [ ] Visitor pass generation on approval
- [ ] Badge ID generation (unique, proper format)
- [ ] Check-in log creation
- [ ] Check-out log update
- [ ] Status transitions validate correctly
- [ ] Duplicate request prevention works
- [ ] Past date validation prevents invalid visits

### Access Control Testing
- [ ] Visitors can create own requests only
- [ ] Approvers can approve/reject
- [ ] Security can check-in/out
- [ ] Receptionists can view passes
- [ ] Facility admin has full access
- [ ] Field-level security enforces restrictions
- [ ] Unauthorized users get access denied

### Flow Testing
- [ ] Submission flow sends notification to approvers
- [ ] Approval flow creates pass, generates badge, sends notification
- [ ] Rejection flow sends rejection email
- [ ] Email templates render correctly with variables
- [ ] Notifications are sent to correct recipients
- [ ] Badge ID is unique and persistent

### Report Testing
- [ ] Approved Visitor Requests shows only approved records
- [ ] Active Visitor Passes filters correctly
- [ ] Daily Check-In Summary shows current date data
- [ ] Reports export to PDF
- [ ] Charts display correctly
- [ ] Filters work as expected

---

## Post-Implementation Tasks

### After Import & Configuration

1. **Data Migration** (if applicable)
   - Migrate existing visitor data
   - Create historical check-in records
   - Map legacy systems to new structure

2. **User Training**
   - Train visitors on request submission
   - Train approvers on approval workflow
   - Train security on check-in/out procedures
   - Train reception on pass management

3. **Documentation**
   - Create user guides for each role
   - Document badge printing procedures
   - Create troubleshooting guide
   - Prepare FAQ document

4. **Customization** (Optional)
   - Add custom fields if needed
   - Extend flows with additional logic
   - Create additional reports
   - Integrate with other ServiceNow modules (CMDB, etc.)

5. **Monitoring**
   - Set up log retention policies
   - Configure audit trail
   - Create daily summary dashboard
   - Set up alerts for high-risk activities

---

## Support & Maintenance

### Regular Tasks
- Daily: Review check-in logs for anomalies
- Weekly: Generate Check-In Summary report
- Monthly: Review access patterns, audit approvals
- Quarterly: Review and update security policies

### Performance Optimization
- Create indexes on frequently filtered fields (visit_date, status)
- Archive old check-in logs (>1 year)
- Optimize Flow Designer for performance
- Monitor table growth

### Troubleshooting
- Check flow execution logs if notifications not sent
- Review ACL configuration if access denied errors
- Verify email settings if emails not received
- Validate date/time fields if filtering issues

---

## Quick Reference Commands

### Import XML Files (Command Line - Optional)
```bash
# Navigate to the update folder
cd "8185d60593ddcf508d1a327efaba1046\update"

# Use ServiceNow CLI if available
sn import --file <filename> --instance <instance>
```

### Revert Changes (if needed)
```
System Update Sets → Submitted Updates
Find VAMS updates → Rollback
```

---

## Additional Resources

### ServiceNow Documentation
- Flow Designer: https://docs.servicenow.com/bundle/platform-user-interface/page/flow-designer/flow-designer-overview.html
- Reports: https://docs.servicenow.com/bundle/vancouver-platform-administration/page/administer/reports/concept_report_overview.html
- Access Control Lists: https://docs.servicenow.com/bundle/vancouver-platform-administration/page/administer/security/concept_access_control_list_acl.html

### VAMS Specific
- Main README: See README.md
- Configuration Files: All .md files in this directory
- XML Exports: All .xml files for version control

---

## Project Statistics

| Item | Count |
|------|-------|
| Tables Created | 2 (new) + 1 (existing) |
| Fields Added | 13 |
| Roles Configured | 6 |
| Application Modules | 4 |
| Security ACLs | 5 |
| Flows to Create | 3 |
| Reports to Create | 3 |
| Related Lists | 5 |
| Documentation Files | 5 |
| Total XML Files | 28 |
| Estimated Setup Time | 2-3 hours |

---

## Support Contact
For issues or questions regarding VAMS implementation, contact your ServiceNow administrator or refer to the configuration documentation files.

