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
