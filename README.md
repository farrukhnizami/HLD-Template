# High-Level Design Template

**Royal Mail Group** | Processing Sites Implementation

Based on the arc42 Template (v9.0) | An HLD Template for Solution Architects across Royal Mail

---

## 📋 Document Overview

This document serves as a comprehensive **template and guide** for Solution Architects at Royal Mail Group to create a High-Level Design (HLD) for the FDS system. It is structured around the **arc42 template (v9.0)** and heavily emphasizes the **C4 model** for diagrams, **cloud (Azure)** architecture, **security (Secure by Design)**, and **data architecture**.

| Property | Value |
|----------|-------|
| Document Title | HLD - Front Line Demand & Supply (FDS) |
| Status | Draft |
| Classification | Template for Internal Use |

---

## 📚 Table of Contents

1. [Introduction and Goals](#1-introduction-and-goals)
2. [Architecture Constraints](#2-architecture-constraints)
3. [Context and Scope](#3-context-and-scope)
4. [Solution Strategy](#4-solution-strategy)
5. [Building Block View](#5-building-block-view)
6. [Runtime View](#6-runtime-view)
7. [Deployment View](#7-deployment-view)
8. [Cloud Architecture](#8-cloud-architecture-azure-focused)
9. [Security Architecture](#9-security-architecture)
10. [Data Architecture](#10-data-architecture)
11. [Cross-cutting Concepts](#11-cross-cutting-concepts)
12. [Architecture Decisions](#12-architecture-decisions)
13. [Quality Requirements](#13-quality-requirements)
14. [RAID Log and Technical Debts](#14-raid-log-and-technical-debts)
15. [Glossary](#15-glossary)

---

## 1. Introduction and Goals

Describes the relevant requirements and driving forces that software architects and development teams must consider.

### 1.1 Requirements Overview
- Short description of functional requirements in use-case format
- Link to existing requirements documents with version numbers
- Focus on business activity improvement and quality enhancement

### 1.2 Quality Goals
- Top 3-5 quality goals (maximum) with highest importance to major stakeholders
- Concrete, measurable goals - avoid buzzwords
- Table format with quality goals and scenarios, ordered by priority

### 1.3 Stakeholders
Table with role names, contacts, and expectations regarding architecture and documentation.

---

## 2. Architecture Constraints

Any requirement that constraints architects in design decisions, implementation decisions, or development process.

| Constraint Type | Description |
|----------------|-------------|
| Technical Constraints | Hardware, software, platform limitations |
| Organisational & Political Constraints | Budget, timeline, regulatory, compliance |
| Conventions | Programming, versioning, documentation, naming |

---

## 3. Context and Scope

Delimits the system from all its communication partners (neighboring systems and users).

### 3.1 Business Context
- C4-style System Context diagram
- Domain-specific inputs and outputs
- Interaction between business capabilities

### 3.2 Technical Context
- UML deployment diagram describing channels to neighboring systems
- Mapping of domain I/O to channels (protocols, hardware)
- Big Picture inclusion required

---

## 4. Solution Strategy

Short summary of fundamental decisions and solution strategies including:

- Technology decisions
- Top-level decomposition (architectural/design patterns)
- How to achieve key quality goals
- Relevant organizational decisions

> **Motivation**: These decisions form the cornerstones for your architecture.

---

## 5. Building Block View

Static decomposition of the system using **C4 model (C2 & C3)**:

### 5.1 Container Diagram (C2)
### 5.2 Component Diagram (C3) - Backend Components

Also includes:
- Integration Architecture
- Reference to ICDs (Interface Control Documents)
- Relationships between containers, components, and external systems (internal/external to RMG)

---

## 6. Runtime View

Concrete behavior and interactions of building blocks for architecturally relevant scenarios:

- System Landscape Diagram (C4 Model)
- Activity Diagrams
- Sequence Diagrams

**Scenarios to cover**:
- Important use cases/features
- Critical external interfaces
- Operation and administration (launch, startup, stop)
- Error and exception scenarios

---

## 7. Deployment View

### 7.1 Deployment Diagram
### 7.2 System Landscape Diagram
### 7.3 Infrastructure Architecture

| Environment | Purpose | Notes |
|-------------|---------|-------|
| Development | | |
| Test | | |
| Production | | |

**Infrastructure Components**:
- Platforms, Load Balancers, Physical Compute
- Databases, HA Cluster, NAS
- Network, LTM, GTM
- **Firewall Rules** (North-South / East-West) including DNAT/SNAT, FQDN-based rules, IP-based rules

---

## 8. Cloud Architecture (Azure-focused)

### 8.1 Regions
Capture Azure regions used for production and DR (Pre-Prod)

| Region | Primary | Secondary | Availability Zones |
|--------|---------|-----------|-------------------|
| North Europe | Yes | | AZ1, AZ2, AZ3 |
| UK South | | Yes | |
| Germany West Central | Yes | | Yes |
| Sweden Central | Yes | | Yes |

### 8.2 Availability Zones
- Services deployed across zones
- Zone pinning requirements
- Zonal vs zone-redundant SKUs

### 8.3 Subscriptions (Naming & Structure)
- Prod, Pre-Prod, Non-Prod (Dev/Test)
- Platform vs Workload subscriptions
- Naming standard compliance

### 8.4 SKUs Used (Per Region)
- Compute SKUs (VM sizes/VMSS)
- Storage SKUs (Standard/Premium/ZRS)
- PaaS SKUs

### 8.5 Network Security Group (NSG) Rules

| Rule Name | Source | Destination | Port | Action | Purpose |
|-----------|--------|-------------|------|--------|---------|
| Allow-App-To-DB | App Subnet | DB Subnet | 1433 | Allow | SQL access |

### 8.6 Network / Subnet Design

| Subnet | CIDR | Purpose |
|--------|------|---------|
| AzureFirewallSubnet | /26 | Firewall |
| AppSubnet | /24 | App tier |
| DBSubnet | /24 | Database |

### 8.7 Patterns / Blueprints
Consumption across: Networking, Security, Compute, Data, Integration

### 8.8 Cost Optimisation and FinOps Alignment
- Reserved capacity usage
- Right-sizing compute resources
- PaaS over IaaS where appropriate
- Costs derived from Azure Calculator
- DR costs included

---

## 9. Security Architecture

### Core Security Objectives (CIA Triad)
- **Confidentiality**: End-to-end encryption, robust IAM
- **Integrity**: Strict authorization, data classification
- **Availability**: Services remain accessible during disruptions

### 9.1 Cyber Architecture Standards Attestation
23+ questions covering:
- ISS-001: Cloud Computing Security
- ISS-002: Cryptography Standard
- ISS-003: Data Backup and Restoration
- ISS-004: Asset Management
- ISS-005: Information Classification
- ISS-007: Risk Management
- ISS-009: Network Security
- ISS-010: Vulnerability Management
- ISS-013: Authentication Standard
- ISS-014: Secure Product Development
- ISS-015: Cyber Security Testing
- ISS-017: Secret & Key Management
- ISS-020: SIEM System Monitoring
- ISS-023: Threat Intelligence

### 9.2 Data Flow Diagram / Threat Modelling
### 9.3 STRIDE Mitigation

| STRIDE | Issue | Mitigation |
|--------|-------|------------|
| Spoofing | User/Service identity impersonation | WAF, APIM Security Policies, RBAC, MQ Auth |
| Tampering | API payload/file modification | TLS Encryption, Input Validation, Checksums |
| Repudiation | User/admin denial of actions | Audit Logging, SIEM Monitoring |
| Information Disclosure | PII/HR data exposure | TLS, Encryption at Rest, Azure Key Vault, RBAC |
| Denial of Service | Traffic flooding | WAF, Rate limiting, SIEM Monitoring |

### 9.4 Identity & Access Management (IAM)
- RBAC (Role-Based Access Control)
- MFA (Multi-factor Authentication)
- User Access Matrix, Groups, Roles, Permissions
- CyberArk (Privileged Access)
- SailPoint (User Provisioning)

### 9.5 Usage of AI (If Applicable)
- Purpose of AI (Use Case)
- Approved use of AI
- Securing the AI

### 9.6 Security RAID Log
### 9.7 Application Business Criticality Rating
### 9.8-9.12: Quantum Cryptography, Active Directory Tier, Key Management

---

## 10. Data Architecture

### Core Components
- **Data Models**: Blueprints mapping data structures, relationships, business rules
- **Data Flow & Pipelines**: Ingest, process, move data
- **Data Storage**: Data Lakes, Warehouses, Lakehouses
- **Data Governance & Security**: Policies, standards, access controls

### 10.1 Data Architecture Blueprint
- Conceptual Data Models
- Data Flow Diagrams
- Storage Strategies

### 10.2 Integration and Processing
- Integration Patterns (ETL/ELT, batch, real-time streaming)
- Data Sources and Destinations
- Integration Methods

### 10.3 Ownership Model

| Interface | Owner | Steward | Consumer |
|-----------|-------|---------|----------|

### 10.4 Policies

| Data Classification | Data Categorization | Retention Policy | Compliance |
|---------------------|---------------------|------------------|------------|

### 10.5-10.12 Data Product Architecture
- Purpose / business use case
- Product owner (business + technical)
- Consumers (teams, applications)
- Value statement
- Data product boundaries
- Event-driven / batch flows

---

## 11. Cross-cutting Concepts

Examples of cross-cutting concepts to document:

- Identity, Authentication and Authorisation
- Audit and Traceability
- Integration Resilience
- Data Privacy and Retention
- Logging, Monitoring and Observability
- User-experience consistency

> **Note**: Do not attempt to cover all possible topics. Pick only the most-needed for your system.

---

## 12. Architecture Decisions

Document important, expensive, large-scale, or risky architecture decisions.

### ADR Template

| Aspect | Detail |
|--------|--------|
| Status | Accepted/Rejected/Proposed |
| Context | Decision background |
| Decision | What was decided |
| Consequences | Impact of the decision |

---

## 13. Quality Requirements

### 13.1 Quality Requirements Overview

| Category (ISO 25010) | Quality Requirement |
|----------------------|---------------------|
| Functional Suitability | |
| Performance Efficiency | |
| Compatibility | |
| Usability | |
| Reliability | |
| Security | |
| Maintainability | |

### 13.2 Quality Scenarios (selected)

**Key principle**: Quality scenarios make NFRs testable and measurable.

| NFR (Not testable) | Quality Scenario (Testable) |
|--------------------|----------------------------|
| All PII data should be encrypted at rest | All PII data should be encrypted using AES256 before storing in the database |
| All save operations should be complete in less than 5 seconds | Upon clicking Save button, record should be saved within 3 seconds and success message displayed |

---

## 14. RAID Log and Technical Debts

| ID | Type | Risk / Debt | Level | Mitigation |
|----|------|-------------|-------|-------------|
| | Risk/Issue/Debt | Description | High/Med/Low | Action plan |

> **Quote**: *"Risk management is project management for grown-ups"* — Tim Lister, Atlantic Systems Guild

---

## 15. Glossary

| Term | Definition |
|------|-------------|
| FDS | Front Line Demand & Supply |
| RMG | Royal Mail Group |
| HLD | High-Level Design |
| C4 Model | Context, Container, Component, Code model |
| STRIDE | Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege |
| WAF | Web Application Firewall |
| APIM | API Management |
| RBAC | Role-Based Access Control |
| NSG | Network Security Group |
| CIDR | Classless Inter-Domain Routing |
| DR | Disaster Recovery |
| NFR | Non-Functional Requirement |
| RAID | Risks, Assumptions, Issues, Dependencies |

---

## 📎 References

- arc42 Documentation: [https://arc42.org/](https://arc42.org/)
- C4 Model: [https://c4model.com/](https://c4model.com/)
- ISO 25010:2023 Quality Model
- Royal Mail Cyber Architecture Standards (LeanIX)

---

## 📝 Document Control

| Field | Value |
|-------|-------|
| Document Title | HLD - Front Line Demand & Supply (FDS) |
| Version | v0.1 |
| Status | Draft |
| Author | TBD |
| Document Owner | TBD |
| Classification | Template for Internal Use |
| Source Requirements | Template |