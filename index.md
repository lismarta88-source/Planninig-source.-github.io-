# PPM — Module Overview

---

## Table of Contents
- [About This Document](#about-this-document)
- [Overview: Function and Business Value](#overview-function-and-business-value)
- [Information Model: What Data Lives Here and How It's Structured](#information-model-what-data-lives-here-and-how-its-structured)
- [Access and Usage: User Roles, Scenarios, and Workflows](#access-and-usage-user-roles-scenarios-and-workflows)
- [Integration Layer: How Data Moves In and Out](#integration-layer-how-data-moves-in-and-out)
- [System Design: Implementation Patterns and Rationale](#system-design-implementation-patterns-and-rationale)
- [Configuration Landscape: Design Choices and Customizations](#configuration-landscape-design-choices-and-customizations)
- [Key Takeaways & Related Documents](#key-takeaways--related-documents)

---

## About This Document

This document is a **comprehensive technical and functional reference** for PPM (Portfolio and Project Management) in the SAP S/4HANA investment management landscape. It is designed for multiple audiences:

- **End Users & Power Users:** Business perspective on what PPM does, how to use it, and where it fits in planning and approval workflows
- **Executive Leadership:** Portfolio governance, approval workflows, and strategic alignment capabilities

The content is intentionally **non-prescriptive**: it explains concepts, objects, data flows, and design patterns without providing step-by-step user instructions or configuration procedures. All statements trace to authoritative source documents (FS Functional Specifications, CS Configuration Specifications, DS Design Specifications).

**Key Rule:** This system overview page is always linked FROM process landing pages, not used as a standalone entry point.

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Figure: Document Structure and Audience Map -->
<!-- Purpose: Diagram showing which sections are relevant to which audience (end users, admins, technical teams, executives) and recommended reading paths -->
<!-- /IMAGE PLACEHOLDER -->

---

## Overview: Function and Business Value

**What PPM Is:** PPM is the central planning and governance system for investment projects at Bayer. It serves as the **single source of truth** for portfolio structure, investment planning data, approval workflows, and project master data throughout the planning and pre-execution phases.

### Business Perspective

From a business perspective, PPM is where investment ideas are born, evaluated, refined through progressive elaboration across FEL gates, and ultimately approved for execution. PPM enables:

- **Strategic Portfolio Planning:** View and balance investments across the entire organization (divisions, functions, sites) aligned with corporate strategy
- **Progressive Elaboration:** Refine investment scope, budgets, and resource plans as understanding deepens through FEL phases
- **Centralized Approval Governance:** Route investment decisions through appropriate approval bodies (DIV-IC, ExCo, BoM) based on value and risk
- **Real-Time Financial Visibility:** Monitor investment budgets, forecasts, and spending against plans through integrated dashboards (CPM, SAC)
- **Seamless Execution Handoff:** Pass approved investments to PS with complete master data and financial baseline for project execution

### Technical Perspective

From a technical perspective, PPM manages:

- **Hierarchical Portfolio Structure:** Five-level organizational hierarchy (Division → Function → Sub-Function → Site + Legal Entity → Project) with flexible collections and classifications
- **Multi-Version Portfolio Items:** Support scenario planning and stage-gate progression through portfolio item versioning
- **Comprehensive Planning Data:** Budgets, forecasts, prefunds, supplements, resource allocations, and milestone tracking
- **Automated Approval Workflows:** Event-driven routing of authorization requests based on investment characteristics and governance thresholds
- **System Integration:** Real-time/batch data synchronization with PS (execution), CPM (reporting), SAC (analytics), and Master Data systems

### Value Proposition

PPM's value comes from:
- **Single System of Record:** All portfolio information in one location, eliminating spreadsheets and manual consolidation
- **Consistency:** All investments follow consistent approval workflows, data quality rules, and governance standards
- **Speed:** Automated workflows and integrated planning eliminate manual approvals and information requests
- **Clarity:** Portfolio managers, approvers, and executives see consistent, current information reflected across all systems (PPM, CPM, SAC)
- **Traceability:** Complete audit trail of portfolio item creation, modification, planning decisions, and approvals

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Figure: PPM Business Value Proposition -->
<!-- Purpose: Visual representation showing PPM's role from idea inception (FEL0) through execution handoff (PS transition) with key benefits at each stage -->
<!-- /IMAGE PLACEHOLDER -->

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Diagram: Investment Management Landscape Overview -->
<!-- Purpose: High-level system architecture showing PPM at center connected to PS, CPM, SAC, and Master Data systems, with data flow directions and business purposes -->
<!-- /IMAGE PLACEHOLDER -->

---

## Information Model: What Data Lives Here and How It's Structured

The PPM data model is built on five fundamental object types that work together to represent the investment portfolio: **Portfolio Types, Portfolio Items, Buckets, Collections, and Classification Hierarchies**. Each object type serves a distinct purpose and relates to others through well-defined cardinality rules.

### Core Objects & Relationships

#### Portfolio Type Y001 "BAYER Portfolio"

A single container for all investment-related portfolio items at Bayer. This design decision simplifies governance by ensuring all investments follow consistent rules and appear in a unified portfolio view.

| Attribute | Value | Purpose |
|-----------|-------|---------|
| **Portfolio Type Code** | Y001 | Unique identifier |
| **Portfolio Name** | BAYER Portfolio | Business name |
| **Scope** | All CapEx, OPEX, and strategic initiatives | Portfolio coverage |
| **Item Types Supported** | Z001–Z007 | Investment categorization (see below) |
| **Status Flow** | Created → Planned → Approved → Executing → Closed | Lifecycle progression |

[Reference: CS_A2R-IM-10-10, Section "Portfolio Type Configuration"]

#### Portfolio Item Types (Z001–Z007)

Seven Portfolio Item Types differentiate investments by their nature, approval requirements, and execution characteristics:

| Code | Type Name | Description | PS Link | FEL Gates | Approval Type | Use Case |
|------|-----------|-------------|---------|-----------|---------------|----------|
| **Z001** | Investment Project | Full-lifecycle capital investment | Yes (Project) | FEL0, 1, 2, 3 | Full authorization | Major CapEx initiatives requiring detailed planning |
| **Z002** | Investment w/o SAP PS | Capital investment at non-SAP sites | No | FEL0, 1, 2, 3 | Full authorization | Investments at facilities without PS instance |
| **Z003** | Lumpsum Budget | Budget placeholder for aggregated planning | Yes (Project) | FEL0, 1, 2, 3 | Full authorization | Grouped smaller projects tracked as single budget line |
| **Z004** | Lumpsum Project | Small project with simplified approval | Yes (Project) | FEL0, 3 only | Simplified (skip FEL1/2) | Quick turnaround projects with low complexity |
| **Z005** | B&C Project | Business & Concept phase work | Yes (Project) | FEL0, 1, 2 | Concept approval | Requirements exploration and solution design |
| **Z006** | Admin Project | Internal delta adjustments | No | N/A | Administrative | Budget reconciliation and accounting adjustments |
| **Z007** | Expense Project | OPEX-only initiatives | Yes (Project) | FEL0, 3 | Simplified (no req docs) | Streamlined operational expense initiatives |

[Reference: FS_A2R-IM-10-10, Section "Portfolio Item Types"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Diagram: Portfolio Item Type Decision Tree -->
<!-- Purpose: Flowchart showing how to select correct item type based on investment characteristics (CapEx vs. OPEX, SAP site vs. non-SAP, complexity level, scope) -->
<!-- /IMAGE PLACEHOLDER -->

#### Portfolio Items

Individual investments, proposals, projects, or concepts being tracked and managed. Each portfolio item belongs to exactly one bucket in the organizational hierarchy and can have multiple versions representing different planning scenarios or approval stages.

**Key Characteristics:**
- **Portfolio Item ID:** Generated to match PS Project ID for Z001, Z003, Z004, Z005, Z007 item types (enables seamless PPM-PS communication)
- **Portfolio Item Versions:** Support progressive elaboration and scenario planning (Version 0 = concept, Version 1 = refined plan, etc.)
- **Status:** Created → In Planning → Submitted → Approved → In Execution → Closed
- **Data Elements:** Name, description, scope, budget, resource plan, milestones, classifications, collections, approval status
- **Ownership:** Portfolio Item owning organization (bucket) established at creation; typically the Site + Legal Entity level

#### Organizational Hierarchy (Buckets): Five Levels

The standard organizational hierarchy with five nested levels, where each portfolio item occupies exactly one bucket slot:

```
Division (e.g., CS Crop Science)
  ├─ Function (e.g., R&D, Production, Supply Chain)
  │  ├─ Sub-Function (e.g., PH R&D Vaccines)
  │  │  ├─ Site + Legal Entity (e.g., Germany Site / Bayer AG)
  │  │  │  ├─ Project Bucket 1 → Portfolio Item Z001
  │  │  │  ├─ Project Bucket 2 → Portfolio Item Z003
  │  │  │  └─ Project Bucket 3 → Portfolio Item Z002
```

**Hierarchy Rules:**
- Each portfolio item occupies exactly one Project bucket (1:1 relationship)
- Bucket relationships are mandatory and immutable after establishment
- Portfolio items inherit organizational attributes from parent buckets
- Buckets serve as planning containers: targets/forecasts can be set at any level and rolled up or down

**Cardinality:**
- Division → 1:many Functions
- Function → 1:many Sub-Functions
- Sub-Function → many:many Sites/Legal Entities
- Site + Legal Entity → 1:many Projects

[Reference: CS_A2R-IM-10-10, Section "Portfolio Structure Configuration"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Figure: Portfolio Structure and Hierarchy Levels -->
<!-- Purpose: Diagram showing complete bucket hierarchy with example Bayer organizational assignments (CS/PH/CH divisions, functions, sites) and cardinality relationships (1:n) -->
<!-- /IMAGE PLACEHOLDER -->

#### Collections & Classifications

Two complementary grouping mechanisms that operate orthogonally to the standard hierarchy:

**Collections:** Strategic initiatives or programs that span organizational boundaries
- Many-to-many relationship: One collection contains items from multiple buckets; one item can belong to multiple collections
- Enables cross-organizational program reporting and initiative tracking
- Examples: "Site Modernization Program 2025," "Digital Transformation Initiative," "Cost Reduction Program"

**Classification Hierarchies:** Alternative portfolio views organized by strategic theme or business outcome
- Portfolio items classified by themes such as "IT Investment," "Strategic Investment," "Grow & Optimize," "Secure & Sustain," "Site Investment"
- Same many-to-many flexibility as collections
- Enables executive decision-making: "What is our strategic spending mix?" "Are we investing in growth or optimization?"

| Grouping Type | Relationship | Use Case | Reporting Purpose |
|---------------|--------------|----------|-------------------|
| **Standard Hierarchy (Buckets)** | 1:1 (item to bucket) | Organizational planning and ownership | Departmental responsibility and accountability |
| **Collections** | Many:many | Multi-project programs and initiatives | Program-level monitoring and cross-functional visibility |
| **Classifications** | Many:many | Strategic theme grouping | Executive portfolio balancing and strategic alignment |

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Diagram: Portfolio Grouping Mechanisms (Hierarchy, Collections, Classifications) -->
<!-- Purpose: Visual showing standard hierarchy (vertical stack), collections (horizontal program groupings), and classifications (theme-based groupings) with example assignments -->
<!-- /IMAGE PLACEHOLDER -->

### Portfolio Item Lifecycle & Status Flow

Portfolio items transition through lifecycle states that govern what actions are permitted and what data is required:

```
Created
  ↓ (User enters basic info)
In Planning (Data entry allowed)
  ↓ (User submits for approval)
Submitted for Approval (In approval workflow)
  ↓ (Approval decision received)
Approved / In Execution (PS project active or created)
  ↓ (Execution milestones reached)
Completed / Closed (Project finished, financial closure)
  ↓
Closed (Archive state)
```

**Status Transition Triggers:**
- User actions (submit authorization request, enter planning data)
- System events (approval decision received from workflow, PS project creation confirmation, PS project closure notification)
- Business rules (data completeness requirements vary by gate: FEL2 requires detailed budgets, FEL3 requires authorization documentation)

[Reference: DS_A2R-IM-10-10-10, Section "Portfolio Item Status Management"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Diagram: Portfolio Item Lifecycle and Status Flow -->
<!-- Purpose: State diagram showing transitions between Created, In Planning, Submitted, Approved, In Execution, Completed, Closed with transition triggers and business rule requirements -->
<!-- /IMAGE PLACEHOLDER -->

### Planning Data Objects

PPM manages several layers of planning data:

| Data Type | Scope | Purpose | Lifecycle |
|-----------|-------|---------|-----------|
| **Budget (Plan)** | Annual fiscal period | Approved investment amount per year | Set at FEL2/FEL3, locked after approval |
| **Forecast** | Rolling annual | Current expected cost/spending | Updated regularly post-approval to reflect execution reality |
| **Prefund** | Specific authorization | Partial funding for design/engineering work | Issued at FEL2 before main investment approval |
| **Supplement** | Incremental | Additional funding for scope changes or cost overruns | Submitted during execution when changes approved |
| **Actual** | Posted cost | Real expenses charged to PS project | Flows from PS execution system post-project closeout |

[Reference: FS_A2R-IM-10-10, Section "Financial Planning Concepts"]

---

## Access and Usage: User Roles, Scenarios, and Workflows

### User Roles & Responsibilities

| Role | Primary Responsibilities | System Access | Key Workflows |
|------|--------------------------|---------------|----|
| **Portfolio Manager** | Portfolio planning, budget planning, organizational assignment, business case development | View/Edit all portfolio items in assigned organizational scope; Submit authorization requests | Create portfolio items, enter planning data, submit for approval, respond to approval requests |
| **Project Manager** | Project-level planning, resource allocation, detailed scope definition, milestone tracking | View/Edit own project portfolio items; Monitor related items in organizational hierarchy | Update project plans, manage versions, coordinate with approvers |
| **Financial Controller** | Financial planning oversight, budget control, variance monitoring, reporting accuracy | Read-only across full portfolio; Write access to financial data elements | Monitor budgets vs. actuals, create forecasts, validate financial completeness |
| **Approver (DIV-IC / ExCo / BoM)** | Authorization request review, approval decision, risk assessment | View authorization requests routed to them; Access supporting portfolio data | Review business cases, make approval decisions, provide feedback |
| **Executive Leadership** | Strategic portfolio oversight, resource allocation decisions, strategic alignment | Read-only across full portfolio; Portfolio dashboards and analytics | Monitor portfolio health, review strategic alignment, analyze portfolio mix |
| **System Administrator** | User access management, security, system configuration, integration monitoring | Full administrative access | Maintain user roles, monitor system health, troubleshoot integration issues |

### Key Usage Scenarios

#### Scenario 1: New Investment Registration (FEL 0)

**Who:** Portfolio Manager in R&D Division  
**When:** Q1, during annual planning cycle  
**Steps:**
1. Create new portfolio item (Z001 – Investment Project)
2. Assign to organizational bucket (Division: PH, Function: R&D, Sub-Function: Vaccines, Site: Germany)
3. Enter high-level scope: "Vaccine Production Capacity Expansion"
4. Assign to collections ("Site Modernization Program 2025")
5. Assign to classifications ("Strategic Investment," "Capacity Expansion")
6. Set rough budget estimate (±30% accuracy)
7. Portfolio item status: **Created** → **In Planning**

**Result:** Portfolio item visible in Division portfolio view, collection dashboards, and strategic classification reports.

<!-- IMAGE PLACEHOLDER (SOURCE UNKNOWN – NEEDS HUMAN) -->
<!-- Purpose: Screenshot or diagram showing PPM user interface for portfolio item creation with field mappings and default values -->
<!-- /IMAGE PLACEHOLDER -->

#### Scenario 2: Detailed Planning & Prefund Authorization (FEL 2)

**Who:** Project Manager in PH R&D  
**When:** Q2, after FEL1 concept approval  
**Steps:**
1. Create Version 2 of portfolio item (progressive elaboration)
2. Enter detailed budget breakdown by cost element (equipment, labor, materials, contingency)
3. Assign budget by fiscal year (e.g., €2M Year 1, €3M Year 2, €1M Year 3)
4. Add resource plan (headcount, skill requirements, contractor support)
5. Define key milestones and decision points
6. Submit Prefund Authorization Request
7. PPM automatically routes request to DIV-IC based on prefund amount (e.g., <€1M threshold)
8. Portfolio item status: **Submitted for Approval**

**Approver Actions:**
- Review business case and financial details in PPM
- Access related dashboards in CPM for context
- Make approval decision (Approve/Reject/Request Changes)
- Prefund Authorization approved

**System Reaction:**
- PPM updates portfolio item status to **Approved** (Prefund stage)
- PPM automatically triggers PS project creation with prefund budget
- Portfolio Item ID automatically matches generated PS Project ID
- Portfolio Manager receives notification: "PS Project X-12345 created; proceed with detailed planning"

**Result:** Design and engineering work can proceed in PS; project budget is locked in both PPM and PS.

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-20 | Diagram: Prefund Authorization Workflow -->
<!-- Purpose: Swimlane diagram showing Portfolio Manager (submit), PPM (route), Approver (review/decide), PS (project creation), Database (sync) interactions -->
<!-- /IMAGE PLACEHOLDER -->

#### Scenario 3: Final Investment Approval (FEL 3)

**Who:** Portfolio Manager, Financial Controller, Approvers (DIV-IC/ExCo/BoM)  
**When:** Q3, after detailed planning completion  
**Steps:**
1. Portfolio Manager creates final Version 3 with all planning details
2. Financial Controller reviews and validates budget completeness, cost estimates
3. Portfolio Manager submits Main Authorization Request
4. PPM evaluates investment characteristics (amount €7M, Strategic Classification, Site location) → Routes to **ExCo** level
5. Request sits in ExCo approval workflow for 2 weeks
6. ExCo reviews: business case, financial ROI, resource availability, portfolio fit
7. Approval decision: **Approved**
8. PPM status transitions to **In Execution**
9. If Prefund already created PS project: PPM confirms project; if no Prefund: PPM creates new PS project
10. Portfolio Item ID matches PS Project ID; master data synced

**Result:** Project is now active in PS for execution. Portfolio Manager transitions to monitor mode; Project Manager takes over day-to-day execution in PS.

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-20 | Diagram: FEL 3 Authorization Workflow (Full Investment Approval) -->
<!-- Purpose: Swimlane diagram showing Portfolio Manager, Financial Controller, PPM workflow engine, Approvers (DIV-IC/ExCo/BoM), and PS with all interactions and approval decision points -->
<!-- /IMAGE PLACEHOLDER -->

#### Scenario 4: In-Execution Supplement Request

**Who:** Project Manager in PS; initiated in PPM  
**When:** Mid-execution, when scope change is approved  
**Steps:**
1. PS Project Manager identifies cost overrun due to requirements change
2. Returns to PPM to submit Supplement Authorization Request (€500K additional budget)
3. PPM routes to Approver (threshold €500K → DIV-IC level)
4. Approver reviews: scope change justification, revised timeline, impact on portfolio
5. Approval decision: **Approved**
6. PPM adds €500K supplement to budget; total now €7.5M
7. Budget increase flows to PS project
8. Project can resume without stopping for funding approval

**Result:** Investment continues with approved additional budget. Forecast vs. plan variance explained and approved in governance process.

#### Scenario 5: Executive Portfolio Review

**Who:** Chief Financial Officer, Executive Team  
**When:** Monthly portfolio review meeting  
**Steps:**
1. CFO opens SAC dashboard (fed by PPM data)
2. Views portfolio composition by strategic classification:
   - 40% "Strategic Investment" = €50M
   - 35% "Growth & Optimize" = €44M
   - 25% "Secure & Sustain" = €31M
3. Drills down: which divisions are driving strategic spend? (PH = 60%, CS = 30%, CH = 10%)
4. Checks FEL gate distribution: 50% projects at FEL0/1, 40% at FEL2, 10% at FEL3
5. Identifies portfolio imbalance: "Too much strategic, not enough operational resilience"
6. Decision: Shift €10M from Strategic to Secure & Sustain; return to Portfolio Managers to replan
7. Portfolio Managers update classifications; new mix appears in next monthly refresh

**Result:** Portfolio rebalanced to align with corporate strategic direction.

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Dashboard Mock-Up: Executive Portfolio View (SAC) -->
<!-- Purpose: Example dashboard showing portfolio composition, strategic alignment, FEL distribution, and drill-down interactivity -->
<!-- /IMAGE PLACEHOLDER -->

### Workflow Lifecycle: From Idea to Execution

```
Quarter 1                    Quarter 2                   Quarter 3
┌─────────────────┐      ┌─────────────────┐      ┌──────────────────┐
│ FEL 0: IDENTIFY │      │ FEL 2: DEFINE   │      │ FEL 3: EXECUTE   │
├─────────────────┤      ├─────────────────┤      ├──────────────────┤
│ PM creates item │      │ PM adds details │      │ PM finalizes     │
│ Z001 Portfolio  │  →   │ Submits Prefund │  →   │ Submits Main AR  │
│ Item            │      │ Auth Request    │      │ Auth Request     │
│                 │      │ (routes to      │      │ (routes to ExCo) │
│ Status: Created │      │ DIV-IC)         │      │                  │
└────────┬────────┘      └────────┬────────┘      └────────┬─────────┘
         │                        │                         │
         │ DIV decision:          │ DIV-IC approves         │ ExCo approves
         │ FEL1 planning          │ Prefund: €500K          │ Main AR: €7M
         │ approved               │ PS Project created      │
         │                        │                         │ Status: Approved
         ↓                        ↓                         ↓
┌─────────────────┐      ┌─────────────────┐      ┌──────────────────┐
│ FEL 1: CONCEPT  │      │ FEL 2: PREFUND  │      │ EXECUTION (PS)   │
├─────────────────┤      ├─────────────────┤      ├──────────────────┤
│ PM refines plan │      │ Design/Eng work │      │ PM manages WBS   │
│ Business case   │      │ Detailed budget │      │ Cost collection  │
│ Created         │      │ (locked in PS)  │      │ Progress tracking│
│ In Planning     │      │ Approved        │      │ In Execution     │
└─────────────────┘      └─────────────────┘      └──────────────────┘
```

[Reference: FS_A2R-IM-10-10, Section "FEL Process Flow"; FS_A2R-IM-10-20, Section "Authorization Request Workflows"]

---

## Integration Layer: How Data Moves In and Out

PPM integrates with four major system groups: PS (execution), CPM (reporting), SAC (analytics), and Master Data systems (reference data). Each integration serves a distinct purpose and operates on different timing patterns.

### Integration Patterns

| Integration | Direction | Timing | Data Exchanged | Business Purpose | Frequency |
|---|---|---|---|---|---|
| **PPM ↔ PS** | Bidirectional | PPM→PS: event-driven; PS→PPM: scheduled | Portfolio Item ID → PS Project ID; master data (scope, budget, org); status updates | Execution project creation and status sync | Real-time (creation); Daily batch (status) |
| **PPM → CPM** | Read-only | Real-time pull | Portfolio structure, budgets, forecasts, classifications, status, milestones | Portfolio reporting and monitoring dashboards | Real-time (users see current data) |
| **PPM → SAC** | Read-time pull | Scheduled batch | Portfolio data, financials, metrics, organizational roll-ups | Executive dashboards, strategic analytics, portfolio balancing | Daily or weekly refresh |
| **MD ↔ PPM** | Inbound | Scheduled batch | Organizational structures, cost centers, user authorizations, ref data | Validation lists, user permissions, budget allocation rules | Daily or weekly |

### PPM → PS Integration: Approval-Triggered Project Creation

**What Happens:**
When a Prefund (FEL2) or Main (FEL3) authorization request is approved in PPM, the system automatically creates or confirms the execution project in PS for portfolio item types Z001, Z003, Z004, Z005, and Z007.

**Data Flow:**
```
PPM: Authorization Approved
      ↓
PPM: Validate data completeness
      ↓ (if valid)
PPM: Generate project master data (scope, budget, org assignment, codes)
      ↓
PPM: Send create-project request to PS
      ↓
PS: Create project with WBS structure
      ↓
PS: Initialize budget (locked to PPM prefund amount)
      ↓
PS: Confirm creation back to PPM
      ↓
PPM: Update portfolio item status to "In Execution"
      ↓
PPM: Notify Portfolio Manager: "Project X-12345 ready in PS"
```

**Key Design Pattern: Portfolio Item ID = PS Project ID**
- PPM generates Portfolio Item IDs using legal-entity-specific coding masks that align with SAP standard project numbering
- Result: The same ID appears in both PPM and PS
- Benefit: Users reference "Project P-98765" in both systems; no confusion from separate ID schemes
- Example: Portfolio Item "P-98765" created in PPM; PS Project automatically named "P-98765"

**Error Handling:**
If project creation fails (data quality issues, PS system down, authorization problem):
- PPM logs error and blocks approval workflow completion
- Notification sent to Portfolio Manager with error details
- Example: "Project creation failed: Cost center 'XX999' not found in Master Data. Correct organizational assignment and resubmit."

[Reference: DS_A2R-IM-10-40-10, Section "PPM-PS Project Creation Interface"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-40-10 | Diagram: PPM→PS Approval-Triggered Project Creation Sequence -->
<!-- Purpose: Sequence diagram showing approval decision → validation → request → PS response → status update → notification, with error handling branch -->
<!-- /IMAGE PLACEHOLDER -->

### PS → PPM Integration: Execution Status Synchronization

**What Happens:**
PS sends status updates back to PPM when major execution events occur: technical project closure, financial closure, or project cancellation. These events trigger automatic portfolio item status transitions in PPM.

**Data Flow:**
```
PS: Project milestone reached / closure initiated
      ↓
PS: Send status update (e.g., "Technically closed," "Financially closed")
      ↓
PPM: Receive status update in scheduled batch job
      ↓
PPM: Validate update (project exists, status valid)
      ↓
PPM: Update portfolio item status (e.g., "In Execution" → "Completed")
      ↓
PPM: Apply business rules (e.g., prevent new supplements on closed projects)
      ↓
PPM: Update related reports and dashboards (CPM, SAC next refresh)
      ↓
PPM: Notify Portfolio Manager: "Project closure confirmed; portfolio item closed"
```

**Timing:**
- Scheduled batch process (typically daily or weekly, not real-time)
- PS closures are not time-critical for portfolio planning; daily sync is sufficient
- Asynchronous approach reduces system load and coupling between systems

[Reference: DS_A2R-IM-10-40-20, Section "PS-PPM Status Sync"]

### PPM → CPM Integration: Real-Time Portfolio Data Feed

**What Happens:**
CPM reads portfolio data directly from PPM in real-time to generate portfolio monitoring dashboards, reports, and status views.

**Data Exchanged:**
- Portfolio structure (bucket hierarchy, assignments)
- Financial planning data (budgets, forecasts, prefunds, supplements, plan vs. actual)
- Portfolio item status and lifecycle info
- Classifications and collection assignments
- Milestones and key dates

**User Experience:**
- Portfolio Manager enters budget in PPM
- CPM dashboard refreshes immediately (or within minutes)
- No export/import; data is live and consistent
- Single source of truth: one entry point (PPM), multiple consumption points (CPM, SAC)

**Technical Approach:**
- CPM reads from PPM database tables directly (or via API/data services layer)
- Real-time or near-real-time data synchronization
- CPM is read-only; changes flow only PPM → CPM, never the reverse

[Reference: FS_A2R-IM-10-40, Section "CPM Integration Architecture"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-40 | Diagram: Data Flow from PPM to CPM Reporting -->
<!-- Purpose: Visual showing PPM data (portfolio, budgets, status) flowing to CPM dashboards and reports; illustrate real-time nature and single-source principle -->
<!-- /IMAGE PLACEHOLDER -->

### PPM → SAC Integration: Portfolio Analytics & Executive Dashboards

**What Happens:**
SAC pulls portfolio data from PPM via scheduled batch processes to create executive dashboards, strategic analytics, and portfolio balancing views.

**Key Metrics & Dimensions:**
- Portfolio size by division, function, site
- Budget composition by strategic classification (Strategic vs. Operational vs. Growth vs. Secure)
- Investment pipeline by FEL stage (FEL0/1: early-stage ideas; FEL2: in design; FEL3: approved; Executing: active projects)
- Budget variance (Plan vs. Forecast vs. Actual)
- Portfolio risk profile by item type and complexity

**Example Executive Insights:**
- "40% of portfolio is IT strategic investments; 35% is operational maintenance; 25% is growth"
- "50 projects in pipeline at FEL0/1 (high attrition risk); 10 projects in FEL3 ready to launch this month"
- "PH Division investing heavily in strategic initiatives; CS and CH more focused on operational resilience"
- "Year-to-date forecast vs. plan variance is +5% (overrun); which projects are driving variance?"

**Technical Approach:**
- SAC data models are fed by scheduled extract/transform/load (ETL) jobs
- Typically daily or weekly refresh cycle (not real-time)
- SAC aggregates and reshapes data for multi-dimensional analysis and visualization

[Reference: FS_A2R-IM-10-40, Section "SAC Integration Architecture"]

### Master Data → PPM Integration: Organizational Reference Data

**What Happens:**
PPM receives organizational structures, cost centers, reference data, and user authorizations from central SAP master data systems (MDG, HR, Finance).

**Data Received:**
- Organization structure (divisions, functions, sites, legal entities, cost centers)
- User directory and authorization assignments
- Fiscal calendars, currencies, exchange rates
- Reference codes and validation lists

**User Experience:**
- When Portfolio Manager selects organizational bucket, dropdown shows current list from master data
- Organizational changes flow automatically; no manual configuration in PPM
- Example: New site added to master data → Site + Legal Entity bucket appears in PPM within 24 hours

**Technical Approach:**
- Scheduled batch synchronization (daily or weekly)
- Prevents data inconsistencies between PPM and corporate master data
- Users always work with current, validated organizational context

### Data Consistency & Error Management

**Consistency Mechanisms:**
- **Synchronous validation:** PPM validates data completeness before sending PS project creation request; if invalid, request fails with clear error message
- **Asynchronous reconciliation:** PS status updates reconcile execution reality with planning records daily; discrepancies logged and escalated
- **Scheduled refresh:** CPM/SAC pull latest data on regular intervals; reporting is current but not real-time

**Monitoring & Escalation:**
- Integration failures are logged with timestamp, error code, affected records
- Technical support team monitors logs and receives alerts for critical failures
- Example failures and resolutions:
  - "PS Project creation failed: Cost center not in Master Data" → Fix: Update cost center in organizational master
  - "Status sync failed: Portfolio Item not found" → Fix: Verify portfolio item ID matches PS project ID
  - "CPM data refresh failed" → Fix: Check SAP data services connectivity

[Reference: DS_A2R-IM-10-40, Section "Integration Error Handling"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-40 | Diagram: System Integration Architecture (PPM Central) -->
<!-- Purpose: High-level architecture diagram showing PPM connected to PS (bidirectional), CPM (PPM→CPM read), SAC (PPM→SAC batch), MD (MD→PPM batch) with data flow directions, timing (real-time/batch), and integration patterns -->
<!-- /IMAGE PLACEHOLDER -->

---

## System Design: Implementation Patterns and Rationale

PPM's implementation reflects core architectural decisions that balance business requirements, governance models, and technical sustainability. This section explains the design patterns that underpin PPM's functionality.

### Core Design Philosophy: Portfolio-Based Planning

**Pattern:** Portfolio-centric data model, not project-centric

**Rationale:**
PPM is fundamentally a portfolio management system, not a project management system. The organizing principle is the portfolio hierarchy (buckets containing items), not individual projects. This design decision enables:
- **Multi-level planning:** Set targets at Division level, roll down or aggregate across organizational boundaries
- **Consistent governance:** All investments follow same approval rules and processes, regardless of type
- **Strategic visibility:** Executive team sees complete portfolio composition, not just approved projects
- **Flexibility:** Collections and classifications provide cross-organizational views without restructuring core hierarchy

**Implication for Users:** Portfolio Managers think in terms of "which bucket does this investment belong to?" and "how does this fit in the overall division/function portfolio?" rather than "which project am I managing?"

[Reference: DS_A2R-IM-10-10-10, Section "Portfolio-Centric Architecture"]

### Design Pattern 1: Single Portfolio Type (Y001)

**Pattern:** One Portfolio Type ("BAYER Portfolio") contains all investment-related items; differentiation via Item Types (Z001–Z007)

**Alternative Considered:** Multiple portfolio types (e.g., "CapEx Portfolio," "OPEX Portfolio," "Strategic Portfolio") for different investment categories

**Rationale for Selection:**
- **Simplified Governance:** Single approval workflow for all investments; rules don't vary by portfolio type
- **Consolidated Reporting:** Executive dashboards show complete portfolio picture without having to union multiple portfolios
- **Consistency:** Risk of siloed portfolios (separate reporting, inconsistent processes) is eliminated
- **Maintainability:** One portfolio configuration is simpler to maintain and upgrade than multiple variants

**Trade-off:** Item Types (Z001–Z007) provide the necessary differentiation (CapEx vs. OPEX, full approval vs. simplified, SAP vs. non-SAP sites) without fragmenting portfolio views

**Design Consequence:** Portfolio Items of all types appear in the same portfolio hierarchy, enabling portfolio balancing and strategic allocation decisions across investment categories

[Reference: CS_A2R-IM-10-10, Section "Single Portfolio Type Rationale"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-10-10 | Diagram: Single Portfolio Type Design (vs. Multiple Portfolio Types Alternative) -->
<!-- Purpose: Visual comparison showing Y001 "BAYER Portfolio" with Z001-Z007 item types; contrasted with alternative of multiple portfolio types (would result in siloed views) -->
<!-- /IMAGE PLACEHOLDER -->

### Design Pattern 2: Portfolio Item ID = PS Project ID Matching

**Pattern:** Portfolio Item IDs are generated using legal-entity-specific coding masks that align with SAP standard project numbering, ensuring Portfolio Item ID exactly matches PS Project ID

**Example Flow:**
```
Portfolio Manager creates Portfolio Item in PPM
  ↓
PPM applies coding mask for Legal Entity (DE/Germany) → Generates ID "P-98765"
  ↓
FEL2 Prefund approved → PPM sends to PS
  ↓
PS creates project using ID "P-98765"
  ↓
Result: Same ID in both systems
```

**Rationale:**
- **User Clarity:** Portfolio Manager and Project Manager reference the same project ID in both PPM and PS
- **Troubleshooting:** When issues arise, teams can quickly locate the same project in both systems using one ID
- **Integration Simplicity:** No need for mapping tables or ID translation logic; PPM and PS speak the same language
- **Auditability:** Complete audit trail uses single project ID across systems

**Alternative Considered:** Separate ID schemes with ID mapping table (e.g., PPM Item "PPI-1001" maps to PS Project "P-98765")
- **Rejected because:** Creates cognitive overhead, complicates troubleshooting, requires mapping table maintenance

**Technical Implementation:**
- Coding masks are defined per legal entity and applied during portfolio item creation
- Format: [Prefix]-[Sequential Number] (e.g., "P-" + 5-digit sequential = "P-98765")
- Masks must align with SAP standard project numbering rules to avoid conflicts

[Reference: DS_A2R-IM-10-10-10, Section "Portfolio Item ID and Coding Mask Logic"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-10-10 | Diagram: Portfolio Item ID = PS Project ID Mapping (by Legal Entity) -->
<!-- Purpose: Visual showing coding mask application by legal entity (DE → P-xxxxx, US → PA-xxxxx, etc.) and result in PPM/PS synchronization -->
<!-- /IMAGE PLACEHOLDER -->

### Design Pattern 3: Five-Level Organizational Hierarchy

**Pattern:** Bucket hierarchy with levels: Division → Function → Sub-Function → Site + Legal Entity → Project

**Rationale:**
- **Reflects Business Reality:** Hierarchy aligns with how Bayer is organized; each level represents a meaningful planning/reporting boundary
- **Enables Multi-Level Planning:** Portfolio targets can be set at Division level (strategic) or Project level (tactical); intermediate levels aggregate or roll down
- **Supports Multiple Views:** One investment can be viewed by Division (for C-suite), Function (for business unit management), or Site (for location management)
- **Clear Ownership:** Each investment "lives" in a specific bucket; ownership and accountability are unambiguous

**Design Consequences:**
- Each portfolio item occupies exactly one Project bucket (1:1 relationship at Project level)
- Each Project bucket can contain only one portfolio item
- Portfolio items inherit organizational attributes from their bucket assignment (e.g., division, cost center, business unit)
- Bucket assignments are immutable after portfolio item creation to prevent orphaned data and reporting inconsistencies

**Flexibility:** Collections and Classifications provide cross-organizational groupings without disrupting the standard hierarchy structure

[Reference: CS_A2R-IM-10-10, Section "Organizational Hierarchy Design"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: FS_A2R-IM-10-10 | Figure: Five-Level Bucket Hierarchy with Example Bayer Org Structure -->
<!-- Purpose: Visual representation of Division/Function/Sub-Function/Site+LE/Project levels with example assignments (CS R&D / Germany / Vaccines Project) -->
<!-- /IMAGE PLACEHOLDER -->

### Design Pattern 4: Event-Driven Authorization to Project Creation

**Pattern:** FEL2 or FEL3 approval of authorization request automatically triggers PS project creation; project creation is synchronous and part of approval workflow

**Process Flow:**
```
Portfolio Manager submits Authorization Request
  ↓
PPM workflow routes to Approver (DIV-IC/ExCo/BoM based on thresholds)
  ↓
Approver reviews and makes decision: Approve/Reject/Request Changes
  ↓
If APPROVED:
  ↓
  PPM validates portfolio item data (budget, org, scope)
  ↓
  PPM sends project creation request to PS (synchronous)
  ↓
  PS creates project and returns confirmation
  ↓
  PPM updates portfolio item status to "Approved" / "In Execution"
  ↓
  Portfolio Manager notified: "Project P-98765 created in PS; ready for detailed planning"
  ↓
If REJECTED or CHANGES REQUESTED:
  ↓
  Workflow ends; portfolio item stays in "Submitted" status
  ↓
  Portfolio Manager resubmits after making requested changes
```

**Rationale:**
- **Single Approval:** One approval decision triggers both PPM update and PS project creation; no manual follow-up steps
- **Data Consistency:** PPM and PS are synchronized immediately after approval; no lag or dual-maintenance period
- **Error Prevention:** If project creation fails, approval workflow is blocked until issue is resolved; prevents orphaned portfolio items without execution projects
- **User Experience:** Project Manager sees project in PS immediately after approval; can start detailed planning without waiting for manual setup

**Technical Implementation:**
- Workflow step includes synchronous call to PS project creation API
- If PS call fails: approval workflow rollback (authorization status stays "Submitted"), error notification sent to Portfolio Manager
- Retry logic handles transient failures (e.g., temporary PS unavailability)

[Reference: DS_A2R-IM-10-40-10, Section "Approval-Triggered Project Creation"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-40-10 | Diagram: Synchronous Authorization Approval → PS Project Creation Workflow -->
<!-- Purpose: Swimlane diagram showing approval decision → validation → sync PS call → creation confirmation → status update with error/retry handling -->
<!-- /IMAGE PLACEHOLDER -->

### Design Pattern 5: Threshold-Based Approval Routing

**Pattern:** Authorization requests are routed to different approval bodies (DIV-IC, ExCo, BoM) based on investment value, organizational assignment, and strategic classification

**Decision Logic:**
```
IF Authorization Request (AR) Amount ≤ €500K 
  AND Portfolio Item Type NOT in (Strategic, Board-Required)
  THEN Route to Division Investment Committee (DIV-IC)

ELSE IF Amount ≤ €2M 
  AND Portfolio Item Type = Strategic
  THEN Route to Executive Committee (ExCo)

ELSE IF Amount > €2M 
  OR Portfolio Item Type = Board-Required
  THEN Route to Board of Management (BoM)

ELSE IF Item Type = Lumpsum / Expense / Admin
  THEN Route to Finance Approval (simplified)
```

**Configuration Examples:**

| Investment Type | Value Range | Approver | Review Criteria |
|---|---|---|---|
| **Z001 – Investment Project (Strategic)** | €0–500K | DIV-IC | Business case, resource fit, cost estimate |
| | €500K–2M | ExCo | Strategic alignment, portfolio fit, risk profile |
| | >€2M | BoM | Strategic importance, board-level governance |
| **Z004 – Lumpsum Project** | Any | Finance | Budget sufficiency, compliance |
| **Z007 – Expense Project** | Any | Finance | OPEX budget availability |

**Rationale:**
- **Risk-Based Governance:** Larger investments receive higher-level review; spending authority matches organizational hierarchy
- **Efficiency:** Small investments approved quickly at DIV-IC; time of executive team reserved for material decisions
- **Flexibility:** Thresholds can be adjusted per division or strategic priority without changing core workflow logic
- **Consistency:** Same decision criteria applied across all divisions; fair, predictable approval process

**Technical Implementation:**
- Decision table stored in SAP configuration (IMG or custom table)
- Workflow engine evaluates decision table during authorization request submission
- Routing is automatic; no manual assignment of approvers

[Reference: DS_A2R-IM-10-10-20, Section "Approval Routing Configuration"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-10-20 | Diagram: Approval Routing Decision Tree (by Value & Type) -->
<!-- Purpose: Flowchart showing how PPM evaluates investment amount, type, and strategic classification to route to appropriate approval body -->
<!-- /IMAGE PLACEHOLDER -->

### Design Pattern 6: Portfolio Versioning for Progressive Elaboration

**Pattern:** Portfolio items support multiple versions; each version represents a planning stage (concept, refined plan, approved baseline)

**Versioning Lifecycle:**
```
Version 0 (FEL0): Concept
  - Rough scope
  - Order-of-magnitude budget (±30%)
  - Strategic classification only

Version 1 (FEL1): Refined Plan
  - Detailed scope description
  - Preliminary budget breakdown (±15%)
  - Resource estimates
  - Supporting documentation

Version 2 (FEL2 Prefund): Design Budget
  - Detailed cost breakdown by element
  - Resource plan finalized
  - WBS structure drafted
  - Prefund AR submitted with this version

Version 3 (FEL3): Approved Plan
  - Final scope definition
  - Approved budget (locked)
  - Detailed resource commitments
  - Main AR approved; project created in PS
```

**Rationale:**
- **Scenario Planning:** Multiple versions enable "what-if" analysis (e.g., fast-track vs. standard timeline) before commitment
- **Progressive Refinement:** Budget estimates improve as planning progresses (±30% → ±15% → ±5%)
- **Audit Trail:** Each version is retained; complete history of planning decisions available for review
- **Version Control:** No need to choose "which version is current"; PPM tracks versions explicitly

**User Experience:** Portfolio Manager works in "Draft" version until ready to submit for approval; submitted version becomes "official" for that gate; new version can be created for next FEL stage

[Reference: DS_A2R-IM-10-10-10, Section "Portfolio Item Versioning"]

<!-- IMAGE PLACEHOLDER (SOURCE UNKNOWN – NEEDS HUMAN) -->
<!-- Purpose: Timeline or diagram showing portfolio item progression through versions from FEL0 (Version 0) through FEL3 (Version 3), with accuracy improvement and detail enrichment at each stage -->
<!-- /IMAGE PLACEHOLDER -->

---

## Configuration Landscape: Design Choices and Customizations

Bayer's PPM configuration reflects strategic choices that shape how the system operates. This section describes what has been configured and why.

### Portfolio Structure Configuration

**Organizational Hierarchy Setup:**

| Level | Scope | Organizational Mapping | Planning Use |
|-------|-------|------------------------|---|
| **Division** | Strategic business area | CS (Crop Science), PH (Pharma), CH (ChemSpecialties) | Top-level budget targets, strategic direction |
| **Function** | Capability or department | R&D, Production, Supply Chain, Finance, IT, etc. | Business unit planning, organizational responsibility |
| **Sub-Function** | Operational area within function | "PH R&D – Vaccines," "Supply Chain – Manufacturing," "IT – Infrastructure" | Detailed planning, cost allocation |
| **Site + Legal Entity** | Geographic/legal location | Germany (Bayer AG), USA (Bayer Inc.), India (Bayer India Ltd), etc. | Project ownership, resource availability, local governance |
| **Project** | Individual portfolio item bucket | "Vaccines Capacity Expansion," "IT Data Center Upgrade," etc. | Individual investment tracking, approval ownership |

[Reference: CS_A2R-IM-10-10, Section "Organizational Hierarchy Configuration"]

### Portfolio Item Type Configuration

All seven Portfolio Item Types (Z001–Z007) are configured and active in the BAYER Portfolio (Y001):

| Item Type Code | Type Name | Lifecycle | PS Project Creation | Approval Type | Active |
|---|---|---|---|---|---|
| Z001 | Investment Project | Full FEL0→1→2→3 | Yes | Full (3 levels) | ✓ Yes |
| Z002 | Investment w/o SAP PS | Full FEL0→1→2→3 | No | Full (3 levels) | ✓ Yes |
| Z003 | Lumpsum Budget | Full FEL0→1→2→3 | Yes | Full (3 levels) | ✓ Yes |
| Z004 | Lumpsum Project | Simplified FEL0→3 (skip FEL1/2) | Yes | Simplified | ✓ Yes |
| Z005 | B&C Project | FEL0→1→2 (concept only) | Yes | Simplified | ✓ Yes |
| Z006 | Admin Project | N/A (administrative) | No | Administrative | ✓ Yes |
| Z007 | Expense Project | Simplified FEL0→3 (no docs) | Yes | Simplified | ✓ Yes |

[Reference: CS_A2R-IM-10-10, Section "Portfolio Item Type Configuration"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: CS_A2R-IM-10-10 | Reference Table: Portfolio Item Type Configuration Details -->
<!-- Purpose: Detailed configuration table showing all Z-codes with FEL gates, approval workflows, validation rules, and PS integration triggers -->
<!-- /IMAGE PLACEHOLDER -->

### Approval Workflow Configuration

**Approval Routing Rules (DIV-IC vs. ExCo vs. BoM):**

```
Decision Logic for Authorization Request Routing:

IF Portfolio Item Type = Z006 (Admin) or Z007 (Expense)
  THEN Route to Finance Approval
       (No threshold evaluation; simplified review)

ELSE IF Amount ≤ €500K
  THEN Route to Division Investment Committee (DIV-IC)
       Review: Business case fit, resource availability, cost reasonableness

ELSE IF Amount ≤ €2M
  THEN Route to Executive Committee (ExCo)
       Review: Strategic alignment, portfolio impact, risk profile

ELSE IF Amount > €2M
  THEN Route to Board of Management (BoM)
       Review: Strategic importance, board-level governance, external impact

ELSE IF Strategic Classification = "Strategic Initiative"
  THEN Route to Executive Committee (ExCo)
       (Bypass normal thresholds; strategic items always reviewed at ExCo level)
```

**Configuration Parameters:**

| Parameter | Value | Purpose |
|---|---|---|
| DIV-IC Threshold | €500K | Small/medium investments approved at division level |
| ExCo Threshold | €2M | Medium/large investments reviewed by executive team |
| BoM Threshold | >€2M | Large investments require board-level governance |
| Strategic Bypass | Enabled | Strategic investments automatically escalated to ExCo |
| Prefund Threshold | €1M | Investments ≥€1M get separate prefund authorization at FEL2 |

[Reference: DS_A2R-IM-10-10-20, Section "Approval Routing Configuration"]

### Collection and Classification Configuration

**Collections (Strategic Programs):**
- Site Modernization Program 2025
- Digital Transformation Initiative
- Cost Reduction Program
- Supply Chain Resilience Program
- ESG / Sustainability Investments

**Classification Hierarchies (Strategic Themes):**
- Strategic Investment (long-term, transformational projects)
- Growth & Optimize (revenue growth or efficiency)
- Secure & Sustain (continuity, resilience, compliance)
- IT Investment (digital infrastructure, software systems)
- Site Investment (location-specific projects)

Each classification can be refined further (e.g., "Growth & Optimize" may split into "Organic Growth," "M&A Integration," "Cost Efficiency," "Quality Improvement").

[Reference: CS_A2R-IM-10-10, Section "Collections and Classifications Setup"]

### Data Validation Rules

**FEL2 (Prefund Gate) Data Requirements:**

| Data Element | Required | Validation |
|---|---|---|
| Portfolio Item Name | Yes | Non-empty, <100 characters |
| Organizational Assignment (Bucket) | Yes | Must be active bucket at Site + Legal Entity level |
| Item Type | Yes | Must be valid Z-code (Z001–Z007) |
| Scope Description | Yes | Minimum 50 characters |
| Budget (by fiscal period) | Yes | ≥ €0, specified for minimum 2 fiscal years |
| Resource Plan | Yes | Headcount or FTE commitment specified |
| Classification | Yes | At least one strategic classification assigned |
| Supporting Documentation | Yes | Business case document attached |
| Risk Assessment | Yes | Risk level (Low/Medium/High) + mitigation plan |

**FEL3 (Main AR) Data Requirements:**

Same as FEL2, plus:

| Data Element | Required | Validation |
|---|---|---|
| Detailed Cost Breakdown | Yes | Costs allocated to cost elements (Labor, Equipment, Materials, Contingency) |
| Milestone Schedule | Yes | Key milestones with planned dates |
| Success Criteria | Yes | Measurable success metrics for project completion |
| Financial ROI / Business Case | Yes | Expected return or business justification |

If data is incomplete, system prevents submission of authorization request with error message indicating missing fields.

[Reference: CS_A2R-IM-10-10, Section "Data Validation Rules by FEL Gate"]

### Custom vs. Standard Functionality

**Standard SAP Functionality Used:**
- Portfolio and portfolio item object model
- Workflow orchestration and approval routing
- Financial planning (budgets, forecasts, actuals)
- Organizational hierarchy
- Authorization and role-based access control
- Standard reports and analytics

**Custom Enhancements Implemented:**

| Enhancement | Business Purpose | Implementation |
|---|---|---|
| **Portfolio Item Type-Specific Logic** | Z004/Z007 skip FEL1/FEL2; Z002 doesn't create PS project | Custom ABAP logic in portfolio item creation/submission |
| **Coding Mask for ID Matching** | Ensure Portfolio Item ID = PS Project ID | Custom function module applying legal-entity-specific masks |
| **Approval Threshold Decision Table** | Route to DIV-IC/ExCo/BoM based on amount & classification | Custom decision table in IMG / Z-table |
| **Pre-Integration Validation** | Check data completeness before PS project creation | Custom ABAP in approval workflow step |
| **CPM/SAC Integration Extract** | Feed portfolio data to reporting systems | Custom ETL jobs (via SAP Data Services or SAP Analytics Cloud connectors) |
| **Strategic Classification Hierarchy** | Portfolio views by strategic theme (Strategic/Growth/Secure) | Custom hierarchy configuration in PPM |
| **Collections for Cross-Org Programs** | Multi-project program tracking | Custom collection setup and dashboards |

**Standard Functionality NOT Used:**
- PPM's embedded time management (Bayer uses PS for work schedule)
- Embedded resource management (resource planning done in PS or HR systems)
- Internal order allocation (CapEx invoicing handled by Finance module)

[Reference: DS_A2R-IM-10-10-30, Section "Custom Development Summary"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-10-30 | Diagram: Standard vs. Custom Functionality in PPM -->
<!-- Purpose: Visual showing which PPM capabilities are standard SAP (colored blue) and which are Bayer custom (colored green), with integration touchpoints -->
<!-- /IMAGE PLACEHOLDER -->

### Authorization & Security Model

**Role-Based Access Control (RBAC):**

| Role | Responsibilities | Access Scope | Data Permissions |
|---|---|---|---|
| **Portfolio Manager (PM)** | Create items, plan budgets, submit ARs | Own organizational scope (e.g., Division or Function) | View/Edit all portfolio items in assigned scope; Submit ARs; View own ARs' approval status |
| **Project Manager (Proj Mgr)** | Manage specific portfolio item | Single portfolio item or related items | View/Edit assigned portfolio items; Monitor related items; Submit version updates |
| **Financial Controller (FC)** | Budget oversight, variance analysis | Full portfolio (read-only for most data) | Read portfolio items, budgets, forecasts; Write access to financial elements only; Approve financial completeness |
| **Approver (DIV-IC / ExCo / BoM)** | Authorize investment decisions | ARs routed to them | View authorization requests assigned to them; Access supporting portfolio data; Make approval decision |
| **Executive Leadership (CxO)** | Strategic oversight | Full portfolio (read-only) | View full portfolio; Access dashboards, strategic classification views; No edit permissions |
| **System Admin (SysAdmin)** | Maintain system, users, security | Full technical access | Manage user roles, monitor integrations, configure business rules, troubleshoot |

**Data Privacy:**
- Portfolio Managers cannot access portfolio items outside their organizational scope unless explicitly granted
- Example: PH R&D Portfolio Manager cannot see CS Supply Chain portfolio items
- Exception: Executive Leadership has cross-organizational read access for strategic oversight

**Authorization Workflow Participation:**
- Authorization requests are visible only to:
  - Original submitter (Portfolio Manager)
  - Current approver (routing rule determines who)
  - Escalation approver (if request re-routed to higher level)
  - System Administrator (for troubleshooting)
- Rejected or completed ARs become read-only; cannot be modified

**Audit & Compliance:**
- All portfolio item create/edit/delete actions logged with: user ID, timestamp, field-level changes, business reason
- Authorization request decisions logged with: approver ID, timestamp, decision (Approve/Reject), comments/rationale
- Audit logs retained for minimum 7 years per company policy

[Reference: CS_A2R-IM-10-10, Section "Authorization and Security Configuration"]

### Integration Configuration Points

**PPM ↔ PS Integration:**
- Portfolio item ID coding mask (ensures ID matching)
- Pre-approval data validation (completeness checks before PS creation)
- Synchronous project creation during approval workflow
- Scheduled status sync job (daily/weekly to reconcile closures)

**PPM → CPM:**
- Real-time or near-real-time data replication
- Portfolio structure sync (buckets, items, versions)
- Financial data sync (budgets, forecasts)
- Status and lifecycle sync

**PPM → SAC:**
- Scheduled ETL job to extract portfolio snapshots
- Data model mapping (portfolio structure to analytics dimensions)
- Refresh frequency (typically daily)

**Master Data → PPM:**
- Scheduled org structure sync (daily/weekly)
- Cost center and GL account sync
- User directory sync for authorization assignments

[Reference: DS_A2R-IM-10-40, Section "Integration Configuration"]

<!-- IMAGE PLACEHOLDER -->
<!-- Source: DS_A2R-IM-10-40 | Diagram: Integration Configuration Points (Data Models, Sync Jobs, APIs) -->
<!-- Purpose: Technical architecture diagram showing PPM interfaces, replication/ETL jobs, scheduling, and data object mappings -->
<!-- /IMAGE PLACEHOLDER -->

---

## Key Takeaways & Related Documents

### Key Takeaways

**PPM as Investment Governance Foundation:**
PPM is the central hub for investment planning, approval, and governance at Bayer. It serves as the single source of truth for portfolio structure, investment planning, and authorization decision-making from idea inception (FEL0) through execution handoff and beyond.

**Business Value:**
- **Unified Portfolio View:** Single portfolio (Y001) with differentiated item types (Z001–Z007) enables consistent governance and strategic balancing without reporting silos
- **Automated Approval Governance:** Threshold-based routing to DIV-IC/ExCo/BoM ensures risk-appropriate approval levels; reduces manual assignment overhead
- **Seamless Execution Handoff:** Matching Portfolio Item ID to PS Project ID eliminates confusion; event-driven project creation keeps PPM and PS synchronized
- **Real-Time Strategic Visibility:** Integrated data feed to CPM/SAC provides portfolio dashboards for portfolio managers and executive analytics for decision-making

**Technical Strengths:**
- **Portfolio-Centric Architecture:** Five-level organizational hierarchy supports multi-level planning and reporting
- **Flexible Grouping:** Collections and classifications enable cross-organizational programs and strategic theme analysis without disrupting organizational structure
- **Event-Driven Integration:** Approval decisions trigger automatic PS project creation; status changes sync bidirectionally to maintain consistency
- **Extensible Design:** Custom item types, approval logic, and integration enhancements support Bayer-specific requirements while leveraging SAP standard foundations

**For Your Role:**

**If you are a Portfolio Manager:**
- PPM is your primary planning tool; use it to register investments, develop budgets, and submit authorization requests
- Coordinate with Project Managers on detailed planning; watch for prefund and main authorization approval notifications
- Monitor forecasts and supplements during execution

**If you are a Project Manager:**
- You receive PPM portfolio item data at approval (prefund or main); use it as your project baseline in PS
- Portfolio Item ID in PPM = PS Project ID; reference is consistent across systems
- Return to PPM during execution to submit supplements when changes require additional funding

**If you are a Financial Controller:**
- Use PPM for financial planning oversight; validate budget completeness at approval gates
- Access budget vs. forecast vs. actual dashboards in CPM for portfolio-level financial monitoring
- Enforce financial validation rules during authorization request approval

**If you are an Approver (DIV-IC / ExCo / BoM):**
- You receive authorization requests routed to your level based on investment value and classification
- Review supporting business case documentation in PPM; access CPM dashboards for portfolio context
- Make approval decision (Approve / Reject / Request Changes); decision triggers downstream workflow (PS project creation or resubmission)

**If you are Executive Leadership:**
- Access SAC dashboards for strategic portfolio analytics (composition by theme, FEL distribution, variance analysis)
- Drill into CPM views for portfolio details; decisions on portfolio rebalancing flow back to Portfolio Managers for action

---

### Related Documents & Resources

**Process Documentation:**
- [A2R-IM-10-10] Portfolio Planning Process (Portfolio Manager workflows, item registration, planning steps)
- [A2R-IM-10-20] Capacity Planning & Authorization Requests (How to submit ARs, approval workflow details)
- [A2R-IM-10-30] Budget Supplements & Forecast Updates (Post-approval planning adjustments)

**Technical Documentation:**
- [FS_A2R-IM-10-10] PPM Functional Specification (Complete feature list, business requirements)
- [CS_A2R-IM-10-10] PPM Configuration Specification (Portfolio setup, item type config, approval rules)
- [DS_A2R-IM-10-10-10] PPM Object Model & Portfolio Item ID Design (Data structures, coding masks)
- [DS_A2R-IM-10-10-20] Approval Workflow Design (Decision logic, threshold routing, escalation)
- [DS_A2R-IM-10-40] System Integration Specifications (PPM-PS, PPM-CPM, PPM-SAC, Master Data sync)

**System Overview Pages:**
- [PS — Project System Overview] Execution platform; where approved projects are managed
- [CPM — Portfolio Reporting Overview] Real-time portfolio monitoring dashboards (fed by PPM)
- [SAC — Analytics Cloud Overview] Executive portfolio analytics and strategic dashboards
- [Master Data Management] Organizational structures and reference data used by PPM

**Troubleshooting & Support:**
- PPM Issues: Contact Investment Management Helpdesk (IM-Support@bayer.com)
- PS Integration Problems: Contact Systems Support team
- Dashboard/Analytics Questions: Contact Portfolio Analytics team
- Access/Authorization Issues: Contact your HR Business Partner or System Administrator

---

### Document Quality & Approach

This document provides:
- ✓ **Conceptual Foundation:** What PPM is, what data it manages, how it fits in the investment landscape
- ✓ **User Perspective:** How different roles interact with PPM; typical workflows and scenarios
- ✓ **Technical Architecture:** System design patterns, integration patterns, object relationships
- ✓ **Configuration Context:** What has been configured and why; governance model rationale
- ✗ **NOT Included:** Step-by-step user instructions (see process guides), configuration procedures (see technical specs), or UI navigation paths

**For procedural guidance,** consult the process-specific user guides listed above.  
**For technical implementation details,** refer to FS/CS/DS specifications.  
**For urgent support,** contact the Investment Management helpdesk or your team lead.

---

**Last Updated:** January 12, 2026  
**Document Type:** SharePoint System Overview (Technical Reference)  
**Related Systems:** PS (Project System), CPM (Commercial Project Management), SAC (Analytics Cloud), Master Data Management  
**Source Documents:** FS_A2R-IM-10-10, FS_A2R-IM-10-20, FS_A2R-IM-10-40, CS_A2R-IM-10-10, DS_A2R-IM-10-10-10, DS_A2R-IM-10-10-20, DS_A2R-IM-10-40-10, DS_A2R-IM-10-40-20  
**Governance:** This document traces to FACT SOURCES (FS/CS/DS specifications). All claims have documented references. See section headers for [Reference] citations.
