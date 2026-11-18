# CrisisSync – Assignment 1, Part B: Requirements and Acceptance Tests

**Group Number:** 3  
**Team Members:** [Full name | ID number | College email for each member]
[Nofar Bulgaru | 324848969 | nofar.bulgaru@e.braude.ac.il]
[Ofir Haviv | 326397668 | ofir.haviv@e.braude.ac.il]
[Meytar Yehosha | 211924410 | meytar.yehosha@e.braude.ac.il]
[Micheal Lucia | 213768120 | micheal.lucia@e.braude.ac.il]

## 1. Scope and Use Case References

This document specifies requirements and acceptance tests for the following operations from the CrisisSync Use Case model in Part A:

- **Reserve Resource Package** (Dispatcher), including **Reserve Resources Atomically**, **Handle Reservation Conflict (Rollback)**, and **Report Task Completion** (Field Commander).
- **Produce SLA Compliance Report**, associated with **Generate monthly Reports**, **Generate Report for Selected Month**, and **View Management Reports** (System Administrator / System Clock).

The numeric NFR thresholds below are proposed measurable acceptance criteria for this assignment; the project story does not prescribe these particular numbers.

**Model verification:** The referenced Use Case names were checked against the UseCase entries in the supplied `project-ex1.vpp` file. This document does not modify the Part A model.

**Reporting assumption for acceptance-test data:** The story defines the SLA duration as event opening to completion of its last task, but does not define which month an event belongs to when it spans months. The sample test uses events whose last tasks finish within the selected month; a cross-month inclusion rule should be agreed with the instructor rather than assumed.

## 2. Functional Requirements (FR)

| ID | Requirement |
|---|---|
| FR-01 | The system shall allow a Dispatcher to reserve a complete resource package only for a task in the Ready state, including exactly one Field Commander and all other required crew and equipment resources. |
| FR-02 | The system shall reserve all resources in a package in one SQL transaction with resource-record locking, and shall roll back the entire reservation if any required resource is unavailable or already reserved; the task shall remain Ready. |
| FR-03 | The system shall allow only the Field Commander assigned to an Active task to report its completion; upon completion, the system shall mark the task Completed, release its reserved resources, and move dependent tasks to Ready only when all their predecessors are Completed. |
| FR-04 | The system shall automatically generate the monthly SLA compliance report at the end of each month and shall allow the System Administrator to generate the report on demand for a selected past month. |
| FR-05 | The system shall calculate the SLA metric as the average elapsed time from event opening until completion of the last task, grouped by event type, and shall present the results for the selected reporting month. |
| FR-06 | The system shall restrict viewing management reports to the System Administrator. |

## 3. Non-Functional Requirements (NFR)

| ID | Category | Measurable requirement |
|---|---|---|
| NFR-01 | Performance | The system shall return a successful resource-package reservation response within 3 seconds for at least 95% of requests under 20 simultaneous Dispatcher sessions on the LAN. |
| NFR-02 | Reliability | In 100 concurrent test pairs attempting to reserve the same available resource for different tasks, the system shall allow exactly one successful reservation per pair and shall leave zero partial resource packages reserved. |
| NFR-03 | Security | The system shall reject 100% of tested completion requests submitted by users other than the task's assigned Field Commander and 100% of tested management-report requests submitted by users other than the System Administrator. |
| NFR-04 | Performance | The system shall generate an SLA report for a selected month containing up to 10,000 events within 10 seconds in at least 95% of 100 measured runs. |
| NFR-05 | Usability | The system shall display a clear reservation failure message and refreshed resource availability within 2 seconds after receiving a conflict response from the server, in at least 95% of 100 trials. |
| NFR-06 | Reliability | The system shall generate exactly one scheduled SLA report for each completed calendar month in 100 tests simulating duplicate scheduler triggers, without duplicate saved reports. |

## 4. Requirements Traceability Matrix (RTM)

| Requirement ID | Use Case in Part A | Acceptance Test ID |
|---|---|---|
| FR-01 | Reserve Resource Package | AT-01 |
| FR-02 | Reserve Resources Atomically; Handle Reservation Conflict (Rollback) | AT-02 |
| FR-03 | Report Task Completion | AT-03, AT-04 |
| FR-04 | Generate monthly Reports; Generate Report for Selected Month | AT-05, AT-06 |
| FR-05 | Produce SLA Compliance Report | AT-05 |
| FR-06 | View Management Reports | AT-07 |
| NFR-01 | Reserve Resource Package | AT-08 |
| NFR-02 | Handle Reservation Conflict (Rollback) | AT-09 |
| NFR-03 | Report Task Completion; View Management Reports | AT-10 |
| NFR-04 | Produce SLA Compliance Report | AT-11 |
| NFR-05 | Handle Reservation Conflict (Rollback) | AT-12 |
| NFR-06 | Generate monthly Reports | AT-13 |

## 5. BDD Acceptance Tests (Given–When–Then)

### AT-01 – Successful Resource Package Reservation (FR-01)
**Given** a Dispatcher is logged in, a task is Ready, and its complete required package (including exactly one Field Commander) is available  
**When** the Dispatcher reserves the package  
**Then** the system shall reserve every required resource and change the task status to Active.

### AT-02 – Reservation Conflict and Full Rollback (FR-02)
**Given** a Ready task requires a package containing a resource already reserved for another task  
**When** the Dispatcher attempts to reserve that package  
**Then** the system shall reject the reservation, roll back every change made for this request, leave the task Ready, and show the current availability.

### AT-03 – Successful Task Completion (FR-03)
**Given** an Active task has an assigned Field Commander and a dependent Blocked task whose predecessors will all be Completed  
**When** the assigned Field Commander reports completion  
**Then** the system shall mark the active task Completed, release its resources, and change the dependent task to Ready.

### AT-04 – Unauthorized Task Completion (FR-03)
**Given** an Active task is assigned to Commander A  
**When** Commander B attempts to report its completion  
**Then** the system shall reject the action and preserve the task status and all resource reservations.

### AT-05 – Generate SLA Report for a Past Month (FR-04, FR-05)
**Given** the System Administrator is logged in and the selected past month contains two Fire events, both with their last tasks completed in that month: Event A opened at 08:00 and its last task completed at 10:00 (2 hours); Event B opened at 09:00 and its last task completed at 13:00 (4 hours)  
**When** the administrator requests the SLA compliance report for that month  
**Then** the system shall display **3 hours** as the average time for the Fire event type ((2 + 4) / 2), and identify the selected month.

### AT-06 – Automatic Monthly SLA Report (FR-04)
**Given** the system clock reaches the end-of-month reporting trigger  
**When** the scheduled monthly report process runs  
**Then** the system shall generate the SLA compliance report for the completed month without an administrator request.

### AT-07 – Unauthorized Report Access (FR-06)
**Given** a Dispatcher is logged in  
**When** the Dispatcher attempts to open a management SLA report  
**Then** the system shall deny access and shall not expose report data.

### AT-08 – Reservation Response Time (NFR-01)
**Given** 20 simultaneous Dispatcher sessions are connected over the LAN and eligible tasks have available resource packages  
**When** at least 100 successful reservations are measured  
**Then** at least 95% of reservation responses shall complete within 3 seconds.

### AT-09 – Concurrent Reservation Reliability (NFR-02)
**Given** two Dispatchers simultaneously attempt to reserve the same available resource for different Ready tasks  
**When** this test is repeated 100 times with the initial state reset before each repetition  
**Then** exactly one reservation shall succeed in each repetition and no partial package shall remain reserved.

### AT-10 – Role-Based Access Protection (NFR-03)
**Given** test accounts include users other than an assigned Field Commander and users other than the System Administrator  
**When** unauthorized task-completion and management-report requests are submitted  
**Then** 100% of those requests shall be denied without modifying tasks or exposing report data.

### AT-11 – SLA Report Generation Performance (NFR-04)
**Given** a selected month contains 10,000 events with sufficient data to calculate SLA averages  
**When** the SLA report is generated 100 times under the same test conditions  
**Then** at least 95 report generations shall finish within 10 seconds.

### AT-12 – Conflict Recovery Feedback (NFR-05)
**Given** a Dispatcher submits a reservation request that conflicts with an existing reservation  
**When** the server returns the conflict result  
**Then** the client shall display a clear failure message and refreshed resource availability within 2 seconds in at least 95 of 100 trials.

### AT-13 – Duplicate Monthly Trigger Recovery (NFR-06)
**Given** the scheduled report for a completed month has already been generated  
**When** the scheduler triggers the same month's report generation again  
**Then** the system shall retain exactly one saved scheduled SLA report for that month, in each of 100 repetitions.
