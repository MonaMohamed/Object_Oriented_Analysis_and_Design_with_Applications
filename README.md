# Vacation Tracking System — Requirements Summary

## 1. Vision
A Vacation Tracking System (VTS) will provide individual employees with the capability to manage their own vacation time, sick leave, and personal time off, without needing to be an expert in company or location policies. 

The system must be **easy to use**. 

Goal / motivation:
- streamline HR operations
- reduce non‑core management activities
- empower employees

## 2. Functional Requirements (Key Features)
- flexible rules‑based validation engine
- optional manager approval workflow
- view past 6 months and request up to 18 months ahead
- email notification for request status change
- extend existing intranet portal (SSO)
- activity logging of all transactions
- HR + admin override
- managers can award extra comp time (with limits)
- web service for other systems to query summary
- integrate with HR legacy data

## 3. Non‑Functional Requirements
- usability: must be intuitive / easy‑to‑use is primary
- reliability: notifications must not be lost
- security: SSO authentication + role based rights
- scalability: works for large enterprise (thousands employees)
- performance: UI pages must load acceptably via corporate LAN
- maintainability: HR can maintain rules without programmers

## 4. Constraints
- must run on existing intranet portal infrastructure
- must reuse existing hardware + middleware
- HR rules are maintained by HR, not developers
- approval may be optional depending employee level
- use email infrastructure already in place

---

## 5. Domain — Define the Problem
Employees request time off → request must satisfy rules (company + location) → request may require manager approval → request becomes Approved / Rejected / Canceled / Withdrawn.

Rules vary by type of time + location + coworker coverage + holiday adjacency, etc.

---

## 6. Actors
| Actor | Description |
|---|---|
| Employee | creates, views, withdraws, cancels vacation requests |
| Manager | approves/denies + can award comp time |
| HR Clerk | manages employee / category / location / override |
| System Admin | manages logs + infra |

---

## 7. Use Case: Manage Time

### 7.1 Entities (analysis level data model)
- Employee
- Manager (is-a Employee)
- Request (pending/approved/rejected/canceled/withdrawn)
- Category (vacation type)
- Grant (balance allocation per employee/category)
- Location
- Restriction (abstract validation rule)

### 7.2 Flowchart (Main flow)
<img width="500" height="1000" alt="Untitled diagram-2025-11-10-124335" src="https://github.com/user-attachments/assets/c0fe0837-848a-45c4-8d86-3f12f49b3b50" />

### 7.3 Sequence Diagram
<img width="1000" height="500" alt="Untitled diagram-2025-11-10-125526" src="https://github.com/user-attachments/assets/37dd00d7-237a-4bb1-82f8-2a851e9153bf" />


### 7.4 Pseudocode (manage time – main flow)

```
authenticate(employee)
summary = load_request_summary(employee)

if employee.action == "new_request":
    req = build_request_form()
    if validate(req) == false:
        show_errors()
    else:
        save(req)
        if req.requires_manager_approval:
            send_email(manager)
        show_home()
```

### 7.5 UI Sketch — Requests Pages

#### Employee Home Screen

```
+---------------------------------------------------------------+
| VTS : My Vacation Requests                                   |
| Logged in as: <Employee>                                     |
+---------------------------------------------------------------+

Balances:
   Vacation..............  48 hrs
   Sick..................  24 hrs
   Personal..............  16 hrs

---------------------------------------------------------------
My Requests (Last 6 months → + 18 months future)
---------------------------------------------------------------
| Dates       | Cat | Hrs | Title              | Status          |
|-------------|-----|-----|--------------------|-----------------|
| 12 Mar→14   | VAC | 24h | Trip to Cairo      | PendingApproval |
| 01 Feb      | SIC |  8h | Fever day          | Approved        |
| 19 Jan      | PER |  4h | Kids meeting       | Cancelled       |
---------------------------------------------------------------

Actions vary by state:
- New Request
- Edit / Withdraw (PendingApproval)
- Cancel (Approved + future/recent past)

[ Create New Request ]
```

#### Manager Home Screen

```
+---------------------------------------------------------------+
| VTS : Manager Dashboard                                       |
| Logged in as: <Manager>                                       |
+---------------------------------------------------------------+

My Own Balances:
   Vacation..............  64 hrs
   Sick..................  32 hrs
   Personal..............  16 hrs

---------------------------------------------------------------
My Own Requests (same as Employee)
---------------------------------------------------------------

===============================================================
Pending Approvals (direct reports)
===============================================================
| Emp Name   | Dates     | Cat | Hrs | Title         | Actions        |
|------------|-----------|-----|-----|---------------|----------------|
| John Emp   | 12→14 Mar | VAC | 24h | Trip to Cairo | Approve Reject |
| Mary Emp   | 28 Feb    | PER |  8h | Family event  | Approve Reject |
---------------------------------------------------------------

Reject requires comment entry (modal)
```

### 7.6 Pseudo-Code — Employee & Manager Request Screens

#### Employee Requests Screen

```
WHEN Employee opens VTS home
    identify employee (SSO)
    load balances
    load all requests (−6 months .. +18 months)
    display table + balances

IF Employee clicks "New Request"
    show New-Request form

    WHEN Submit pressed
        collect category, dates, hours, title, notes
        IF required fields missing → show validation errors → STOP

        run rules engine
        IF rule failure → show errors → STOP

        save Request (status = PendingSubmission)

        IF employee level requires approval
            update status → PendingApproval
            send email to manager(s)
        ELSE
            update status → Approved

        redirect to Home (summary refreshed)
END IF

IF Employee selects an Approved (future) request → Cancel flow
    show “Confirm Cancel?”
    IF Confirm
        update status → Cancelled
        restore hours back to balances
        send email to manager
        return to Home

IF Employee selects a PendingApproval request → Edit / Withdraw
    IF Withdraw chosen
        confirm
        IF Confirm → status = Withdrawn → return Home
    ELSE
        show edit form
        on Submit → validate → if ok save → return Home
```

#### Manager Requests Screen
```
WHEN Manager opens VTS home
    identify manager (SSO)
    load manager’s own balances + requests
    load all pending approval requests for direct reports
    display 2 sections:
        My Requests
        Pending Approvals

FOR each row in Pending Approvals
    showing [Approve] [Reject]

IF Manager presses Approve
    confirm
    change request status → Approved
    adjust balances for employee
    send email to employee
    refresh both lists

IF Manager presses Reject
    prompt for comment (required)
    change request status → Rejected
    send email to employee (include comment)
    refresh both lists
```
### 7.6 State Machine — Request
<img width="500" height="500" alt="Untitled diagram-2025-11-10-140217" src="https://github.com/user-attachments/assets/9e28fad5-2eae-4524-87f7-8d2cb76ec201" />

---
## 8. Use Case: Cancel Request
### 8.1 Actor:
Employee

### 8.2 Goal:
Cancel an Approved request that is in the future or recent past (previous 5 business days).

### 8.3 Preconditions:
Employee is authenticated (via portal SSO). A target request exists in state Approved and within allowed window.

### 8.4 Post Conditions:
Request becomes Cancelled; time allowance returned; manager notified.

### 8.5 Flowchart
<img width="800" height="800" alt="Untitled diagram-2025-11-10-140217" src="https://github.com/user-attachments/assets/b9e0eec4-e2c4-4745-ae7c-c562529a96ef" />

### 8.6 Sequence Diagram
<img width="702" height="623" alt="Screenshot 2025-11-10 163013" src="https://github.com/user-attachments/assets/0956401e-295b-4d1b-96f1-b90b03b1508d" />

---
## 9. Use Case: Edit Pending Request
### 9.1 Actor:
Employee

### 9.2 Goal:
Edit title/comments/dates of a PendingApproval request; or Withdraw it.

### 9.3 Preconditions:
Employee authenticated; target request is PendingApproval.

### 9.4 Post Conditions:
If edits pass validation → saved; if withdraw → state Withdrawn; home refreshed.

### 9.5 Flowchart
<img width="481" height="651" alt="Screenshot 2025-11-10 164831" src="https://github.com/user-attachments/assets/6bc5d4cc-2210-4dd4-bfd3-4f29f1e29dbd" />

### 9.6 Sequence Diagram
<img width="567" height="758" alt="Screenshot 2025-11-10 165148" src="https://github.com/user-attachments/assets/9874f072-ab93-47ef-8986-7c78c20d29c0" />

---
## 10. DB Change Check — Summary (table)
| Flow            | Table         | Change                                                        |
|-----------------|---------------|----------------------------------------------------------------|
| Cancel Approved | request       | update `state = 'Cancelled'`                                   |
| Cancel Approved | grant_balance | add cancelled hours back into employee’s allowance            |
| Cancel Approved | request_hist  | insert row (from Approved → Cancelled)                        |
| Edit Pending    | request       | modify fields: dates / hours / title / notes                  |
| Edit Pending    | request_hist  | insert row (old version snapshot → new version snapshot)      |




