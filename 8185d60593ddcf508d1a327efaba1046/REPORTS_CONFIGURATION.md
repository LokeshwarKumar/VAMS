# VAMS Reports Configuration

## Report 1: Approved Visitor Requests
**Table:** x_1917769_vams_visitor_request
**Type:** Tabular Report

### Configuration:
- **Name:** Approved Visitor Requests
- **Filter:** status = "approved"
- **Columns:**
  1. Number
  2. Visitor Name
  3. Company Name
  4. Host Employee
  5. Visit Date
  6. Approval Date (system update date)
  7. Status
  
- **Sort:** Visit Date (Descending)
- **Group By:** Host Employee (optional)
- **Roles:** x_1917769_vams.approver, x_1917769_vams.facility_admin

### Chart Recommendations:
- Approved requests by host employee (Bar chart)
- Approved requests by visit date (Line chart)

---

## Report 2: Active Visitor Passes
**Table:** x_1917769_vams_visitor_pass
**Type:** Tabular Report

### Configuration:
- **Name:** Active Visitor Passes
- **Filter:** pass_status = "Active" AND expiry_date >= NOW()
- **Columns:**
  1. Pass Number
  2. Visitor Name
  3. Badge ID
  4. Issue Date
  5. Expiry Date
  6. Visitor Request (Link)
  7. Pass Status

- **Sort:** Expiry Date (Ascending)
- **Roles:** x_1917769_vams.receptionist, x_1917769_vams.security, x_1917769_vams.facility_admin

### Features:
- Add export to PDF functionality for printing badges
- Include a "Days Until Expiry" calculated field
- Highlight passes expiring in 1-2 days

---

## Report 3: Daily Check-In Summary
**Table:** x_1917769_vams_checkin_log
**Type:** Summary Report

### Configuration:
- **Name:** Daily Check-In Summary
- **Default Filter:** check_in_time >= TODAY()
- **Columns:**
  1. Visitor Pass (Visitor Name)
  2. Check In Time
  3. Check Out Time
  4. Duration (calculated: check_out_time - check_in_time)
  5. Checked In By
  6. Checked Out By
  7. Status

- **Group By:** 
  - Primary: Status (Checked In / Checked Out)
  - Secondary: Check In Time (by hour)

- **Aggregations:**
  - Count of check-ins
  - Count of check-outs
  - Average duration on premises
  - Current visitors (status = "Checked In")

- **Sort:** Check In Time (Descending)
- **Roles:** x_1917769_vams.security, x_1917769_vams.receptionist, x_1917769_vams.facility_admin

### Dashboard Elements:
- KPI: Total visitors today
- KPI: Currently on premises
- KPI: Average visit duration
- Table: Current visitors with check-in times
- Table: Check-outs today

---

## Additional Report Recommendations:

### Report 4: Visitor Pass Status Summary (Optional)
- Group by pass_status
- Show count and breakdown by status
- Include expired vs active passes

### Report 5: Visitor Traffic Analysis (Optional)
- Date range comparison
- Peak visit hours
- Most frequent visitors
- Most visited by host employee

### Report 6: Security Audit Log (Optional)
- All check-in/check-out activities
- Who performed the check-in/out
- Timestamp details
- Anomalies (long durations, after-hours visits)

