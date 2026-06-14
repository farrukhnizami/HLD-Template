# Frontline Demand & Supply (FDS)

**Resource Planning and Duty Management System**

| | |
|---|---|
| **Project Managers** | Nilgul Perk, Jamie Dobson |
| **Solution Architects** | Rashu Khanna, Kunle Olagbegi |

---

## Table of Contents

1. [Overview](#overview)
2. [Actors & Users](#actors--users)
3. [System Context (C1)](#system-context-c1)
4. [Container Architecture (C2)](#container-architecture-c2)
5. [Backend Components (C3)](#backend-components-c3)
6. [Integration Architecture](#integration-architecture)
7. [Key Use Cases](#key-use-cases)
8. [Technology Stack](#technology-stack)
9. [Data & Storage](#data--storage)
10. [Architecture Diagrams](#architecture-diagrams)

---

## Overview

FDS (Frontline Demand & Supply) is an enterprise resource planning and duty management system used by frontline operations. It covers the full lifecycle of workforce duty management including:

- Attendance tracking via barcode scanners (PDA / Console)
- Attendance plan creation and modification
- Duty revision submission and approval workflows
- Overtime, scheduled attendance (SA), and agency shift management and approval
- Integration with HR, payroll, fleet, and reporting platforms

---

## Actors & Users

| Actor | Role | Interactions |
|---|---|---|
| **OPG** | Frontline Workers | Scan in/out via PDA or Console; view approved overtime in People App |
| **PSM / COM / PCOM / CCOM / Bookroom** | Duty Managers | Manage duties in Delivery and Processing Units via FDS Web UI |
| **COM / PCOM / CCOM** | Attendance Plan Managers | Create/update attendance plans; approve or reject overtime, SA and agency shifts |
| **Delivery Assurance / OPL / Finance Business Partner** | Approvers | Approve or reject duty change submissions |

---

## System Context (C1)

FDS operates within a broader ecosystem of internal platforms and external systems.

### External Systems

| System | Type | Description | Integration |
|---|---|---|---|
| **PSP (SAP)** | Platform | Sends absences, HR master data, and overtime shift status | IBM MQ, FTPS |
| **JoinedUp** | External SaaS | Agency worker requisition and payments | REST API |
| **M5 Fleet (AssetWorks)** | External System | Provides fleet data | FTPS |
| **Winpak (Honeywell)** | External System | Provides ID card / access control data | FTPS |
| **MDM (Teradata)** | Platform | Sends master data | FTPS |
| **Barcode Scanner** | External System | PDA and Console devices that record employee attendance | IBM MQ |
| **Delivery System Apps** | System Group | Legacy delivery planning tools (DDS Legacy, DDS MARS, MDS, Desktop Planning Tool) | Azure APIM |
| **Email Provider** | External SaaS | Microsoft Exchange for sending approval notification emails | REST |
| **Data Platform (GCP)** | Platform | Receives operational data for analytics and reporting | FTPS, IBM MQ |
| **Reporting Platform (Qlik)** | Platform | Business intelligence and reporting dashboards | Data Platform |
| **People App** | Mobile App | Displays approved overtime shifts to frontline workers | Azure APIM |

### Integration Middleware — BIG

**BIG** (Backbone Integration Gateway) is the central integration layer composed of:

| Component | Technology | Purpose |
|---|---|---|
| **FTPS** | Secured FTP | File-based data exchange for HR, fleet, ID card, and master data |
| **MQ** | IBM MQ | Event-driven messaging for attendance, absences, overtime status, and approved shifts |
| **APIM** | Azure API Management | API gateway used by delivery system apps and People App |

---

## Container Architecture (C2)

FDS is composed of four containers:

```
┌─────────────────────────────────────────────────────────┐
│                    FDS System                           │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Frontend   │  │     APIs     │  │   Backend    │  │
│  │  (Angular)   │  │   (.NET)     │  │   (.NET /    │  │
│  │              │  │              │  │    SSIS)     │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
│         │                 │                  │          │
│         └─────────────────┴──────────────────┘          │
│                           │                             │
│                  ┌────────▼────────┐                    │
│                  │   Databases     │                    │
│                  │  (SQL Server)   │                    │
│                  └─────────────────┘                    │
└─────────────────────────────────────────────────────────┘
```

---

### Frontend Web App

**Technology:** Angular.js / TypeScript  
**Shape:** Browser Web Application

The FDS web interface provides four functional modules:

| Module | Description | API Dependency |
|---|---|---|
| **Duty Manager** | Add, delete, modify duties and submit for approval | Duty Manager Service |
| **Approval Workflow** | View submitted duties and approve or reject duty changes | Approval Workflow Service |
| **Attendance Plan** | Create/modify attendance plans, create overtime shifts, assign OPGs to duties | Attendance Plan Service |
| **Timesheet Approval** | View overtime and agency shifts, approve or reject | Timesheet Backend (direct) |

All frontend modules send user activity data to the **Action Log Service** for auditing.

---

### APIs Container

**Technology:** .NET REST APIs

Seven REST API services act as the interface layer between the frontend and backend processing logic:

| Service | Description | Downstream |
|---|---|---|
| **Email Service** | Sends notification emails using templates | Microsoft Exchange |
| **Agency Worker Service** | Requests agency workers from JoinedUp; sends approved shifts for payment | JoinedUp |
| **Action Log Service** | Audits all user activities | FDS DB |
| **Duty Manager Service** | Reads/writes duty data | Backend: Duty Manager |
| **Timesheet Service** | Reads/writes timesheet data | Backend: Timesheet Management |
| **Attendance Plan Service** | Reads/writes attendance plan data | Backend: Attendance Plan |
| **Approval Workflow Service** | Reads/writes approval workflow status | FDS DB |

---

### Backend Container

**Technology:** .NET (business logic), SSIS (ETL)

Five backend components handle core business processing:

| Component | Technology | Description |
|---|---|---|
| **Data Import** | SSIS | Ingests data from FTPS and MQ into Staging DB, then validates and loads into FDS DB |
| **Data Export** | SSIS | Exports data to Data Platform (GCP), IBM MQ (PSP payments), Agency Service, and DelSys DB |
| **Approval Workflow** | .NET | Manages duty revision approval workflow state changes |
| **Attendance Plan** | .NET | Reads and writes attendance plans |
| **Timesheet Management** | .NET | Reads and writes timesheet approvals for overtime and agency shifts |

---

### Databases Container

**Technology:** SQL Server 2019

| Database | Description | Retention |
|---|---|---|
| **FDS SQL DB** | Main transactional database for all FDS operational data | 7 years |
| **Staging DB** | Holds imported data as-is before validation and transformation | — |
| **DelSys DB** | Stores delivery system data for reporting and downstream integration | — |

**FDS SQL DB Backup Policy:**
- Full backup: Daily, 14-day retention
- Differential backup: Hourly, 24-hour retention

---

## Backend Components (C3)

The backend components interact as follows:

```
FTPS / MQ
    │
    ▼
Data Import (SSIS)
    ├──► Staging DB ──► FDS SQL DB
    └──► FDS SQL DB (directly for validated data)

FDS SQL DB
    │
    ├──► Data Export (SSIS)
    │       ├──► Data Platform (GCP)
    │       ├──► IBM MQ (PSP payments)
    │       ├──► Agency Worker Service
    │       └──► DelSys DB
    │
    ├──► Approval Workflow (.NET)
    ├──► Attendance Plan (.NET)
    └──► Timesheet Management (.NET)
```

---

## Integration Architecture

FDS uses a **hybrid integration pattern** — event-driven messaging via IBM MQ combined with file-based exchange via FTPS:

```
External Sources          BIG Middleware          FDS
──────────────           ───────────────        ──────
PSP (SAP)   ──MQ──►┐
Barcode     ──MQ──►├──► IBM MQ ──────────────► Data Import (SSIS)
Scanner            │
                   │
M5 Fleet    ──────►┐
Winpak      ──────►├──► FTPS ───────────────► Data Import (SSIS)
MDM         ──────►┘

FDS ──────────────────► IBM MQ ──────────────► PSP (payments)
FDS ──────────────────► FTPS ────────────────► Data Platform (GCP)
FDS ──────────────────► APIM ────────────────► Delivery System Apps
FDS ──────────────────► APIM ────────────────► People App
```

---

## Key Use Cases

### UC-01: Approving Overtime Shifts

1. COM Manager opens the **Timesheet Approval** screen in FDS Web UI
2. Frontend fetches all unapproved overtime and SA shifts from **Timesheet Service API** → **FDS DB**
3. Manager reviews and approves shifts
4. Approved shifts are sent via `POST /timesheet/approve` to **Timesheet Service API**
5. Timesheet Service **publishes to IBM MQ** topic `approved-shifts`
6. IBM MQ fans out **in parallel**:
   - **PSP (SAP)** consumes approved shifts for automated payroll processing
   - **Data Platform (GCP)** consumes for reporting and analytics → Qlik dashboards
7. PSP sends an acknowledgment back to MQ
8. MQ triggers **Data Import (SSIS)** to update shift statuses in FDS DB
9. Updated statuses flow back to the manager's Timesheet screen

### UC-02: Approving Duty Revisions

Duty revision submissions follow the **Approval Workflow**:

1. PSM/Bookroom submits a duty revision via the **Approval Workflow** UI
2. **Delivery Assurance / OPL / Finance Business Partner** receives notification via the **Email Service** (Microsoft Exchange)
3. Approver logs in and approves or rejects the submission via the **Approval Workflow** UI
4. Workflow status is updated via **Approval Workflow Service** → **Approval Workflow backend** → **FDS DB**

---

## Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Angular.js, TypeScript |
| Backend APIs | .NET (REST APIs) |
| Backend Processing | .NET, SSIS (SQL Server Integration Services) |
| Databases | SQL Server 2019 |
| Messaging | IBM MQ |
| File Transfer | FTPS (Secured FTP) |
| API Gateway | Azure API Management |
| HR / Payroll | SAP (PSP) |
| Agency Workforce | JoinedUp (SaaS) |
| Fleet Management | AssetWorks (M5) |
| Access Control | Honeywell (Winpak) |
| Master Data | Teradata (MDM) |
| Data Platform | Google Cloud Platform (GCP) |
| Reporting / BI | Qlik Sense |
| Email | Microsoft Exchange |
| Architecture Modeling | LikeC4 (C4 DSL) |

---

## Data & Storage

### Data Flows In

| Source | Transport | Data |
|---|---|---|
| PSP (SAP) | IBM MQ | Absences, overtime shift status |
| PSP (SAP) | FTPS | HR master data |
| Barcode Scanners (PDA/Console) | IBM MQ | Employee attendance records |
| M5 Fleet | FTPS | Fleet/vehicle data |
| Winpak | FTPS | ID card / access control data |
| MDM | FTPS | Master data |
| JoinedUp | REST API | Agency worker information |

### Data Flows Out

| Destination | Transport | Data |
|---|---|---|
| PSP (SAP) | IBM MQ | Approved overtime and SA shifts (for payroll) |
| Data Platform (GCP) | IBM MQ / FTPS | Operational data for analytics |
| Reporting (Qlik) | via Data Platform | Dashboards and reports |
| Delivery System Apps | Azure APIM | Duty data (DDS Legacy, DDS MARS, MDS) |
| People App | Azure APIM | Approved overtime shifts |
| JoinedUp | REST API | Approved agency shifts (for payment) |

---

## Architecture Diagrams

Diagrams are generated from the LikeC4 model source files:

| File | Purpose |
|---|---|
| [model.c4](model.c4) | Full architecture model — all elements and relationships |
| [views.c4](views.c4) | View definitions (Context, Container, Components, Landscape) |
| [elements-specs.c4](elements-specs.c4) | Element type styles and notations |
| [relationships-specs.c4](relationships-specs.c4) | Relationship type definitions |
| [use-cases/usecase.01-approve-overtime.c4](use-cases/usecase.01-approve-overtime.c4) | Dynamic sequence view: Approve Overtime |

### Rendered Diagram Images

| Diagram | Description |
|---|---|
| [context.png](diagrams/context.png) | System Context (C1) |
| [container.png](diagrams/container.png) | Container Diagram (C2) |
| [backendcomponents.png](diagrams/backendcomponents.png) | Backend Components (C3) |
| [index.png](diagrams/index.png) | Full System Landscape |
| [fds.png](diagrams/fds.png) | FDS Container Detail |

To regenerate diagrams, run:

```bash
npm install
npx likec4 build
```
