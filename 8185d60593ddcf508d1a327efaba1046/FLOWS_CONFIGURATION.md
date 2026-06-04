# VAMS Flow Designer Flows Configuration

## Flow 1: Visitor Request Approval Notification
**Trigger:** When a Visitor Request is submitted
**Description:** Sends approval request to the Approver role

### Flow Steps:
1. **Trigger Event:** Visitor Request record created
2. **Get Approver Users:** Query sys_user table for users with x_1917769_vams.approver role
3. **Send Email Notification:**
   - To: Approvers
   - Subject: "New Visitor Request Pending Approval - [Visitor Name]"
   - Body: "A new visitor request from {visitor_name} has been submitted for approval. Request Details: Company: {company_name}, Visit Date: {visit_date}, Purpose: {purpose_of_visit}"
4. **Create Activity:**
   - Log activity: "Approval notification sent"

### Email Template Recommendation:
```
Subject: New Visitor Request Pending Approval - {visitor_name}
Body:
A new visitor request has been submitted and requires your approval.

Visitor Name: {visitor_name}
Company: {company_name}
Email: {email}
Phone: {phone_number}
Host Employee: {host_employee}
Visit Date: {visit_date}
Purpose: {purpose_of_visit}

Please review and approve/reject this request.

View Request: [link to record]
```

---

## Flow 2: Visitor Request Approval Processing
**Trigger:** When Visitor Request status changes to "Approved"
**Description:** Creates visitor pass, generates badge ID, and sends approval notification

### Flow Steps:
1. **Trigger Event:** Visitor Request status changes to "Approved"
2. **Create Visitor Pass Record:**
   - Table: x_1917769_vams_visitor_pass
   - Fields:
     - visitor_request: [Reference to current request]
     - visitor_name: [From request.visitor_name]
     - issue_date: [Current date/time]
     - expiry_date: [visit_date + configured days, default 7 days]
     - pass_status: "Active"
     - badge_id: [Auto-generated using script task]

3. **Generate Badge ID** (Script Task):
   ```javascript
   var passNum = workflow.scratchpad.pass_number;
   var timestamp = new Date().getTime().toString().slice(-6);
   var badgeId = "VP-" + passNum + "-" + timestamp;
   workflow.scratchpad.badge_id = badgeId;
   ```

4. **Update Visitor Pass** with generated badge ID

5. **Send Approval Notification Email:**
   - To: Visitor
   - Subject: "Your Visitor Request Has Been Approved"
   - Body: Include pass details and badge information

6. **Send Notification to Receptionist:**
   - Notify: Users with x_1917769_vams.receptionist role
   - Message: "New visitor pass ready for pickup - {visitor_name}"

### Email Template:
```
Subject: Your Visitor Request Has Been Approved - Badge {badge_id}
Body:
Your visitor request has been approved!

Visitor Pass Details:
Pass Number: {pass_number}
Badge ID: {badge_id}
Issue Date: {issue_date}
Expiry Date: {expiry_date}
Status: Active

Please present your badge at the facility entrance.
For questions, contact the reception desk.
```

---

## Flow 3: Visitor Request Rejection Notification
**Trigger:** When Visitor Request status changes to "Rejected"
**Description:** Sends rejection notification to visitor

### Flow Steps:
1. **Trigger Event:** Visitor Request status changes to "Rejected"
2. **Send Rejection Email:**
   - To: Visitor email
   - Subject: "Your Visitor Request Has Been Rejected"
   - Body: Include rejection reason and contact information

3. **Log Activity:**
   - Log: "Rejection notification sent to {visitor_name}"

### Email Template:
```
Subject: Visitor Request Status Update - Rejected
Body:
We regret to inform you that your visitor request has been rejected.

Original Request Details:
Visitor Name: {visitor_name}
Requested Visit Date: {visit_date}
Purpose: {purpose_of_visit}

Reason: {approval_comments}

Please contact the facility management for further information.
Contact: [facility contact info]
```

---

## Additional Flow Recommendations:

### Flow 4: Visitor Pass Expiry Check (Optional - Scheduled)
**Trigger:** Scheduled to run daily
**Description:** Mark expired passes and notify security

### Flow 5: Check-In Reminder (Optional)
**Trigger:** When visitor pass is created
**Description:** Send reminder email on day before visit date

### Flow 6: Visitor Check-Out Timeout (Optional - Scheduled)
**Trigger:** Scheduled to run every hour
**Description:** Auto-checkout visitors who haven't checked out after configurable hours (e.g., 8 hours)

