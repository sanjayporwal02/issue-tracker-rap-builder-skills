# Disclaimer
This project is a learning resource created as part of Devtoberfest 2026. The agent definition and skill files represent generic development patterns and are not official guidance, recommendations, or best practices from SAP. Use at your own discretion.

# issue-tracker-rap-builder-skills
Sample skills, agent definition and prompts for generating ABAP RAP based issue tracker applications using ADT MCP tools. Supports different customer contexts — factory floor, HVAC, fleet — from natural language requirements.

## What This Is

A custom agent (`rap-it-generator.md`) that takes natural language business requirements and produces a fully working, tested, transport-ready RAP application on S/4HANA — using the ADT MCP Server for all development operations.

The agent is guided by skill files that encode technical standards and customer-specific conventions. No pre-built domain spec — the agent derives the data model, behavior, CDS views, service layer, and test cases from the user's requirements.

## How It Works

```
User provides:
  - Target system, package, transport
  - Business requirements in natural language

Agent reads:
  - Customer naming conventions and coding standards
  - Standard technical skills (data modeling, RAP architecture, CDS, testing, etc.)

Agent executes 9 phases (4 with human checkpoints):
  0. Translate requirements → technical blueprint (user confirms)  ← HUMAN
  1. Preflight & discover standard services (user confirms if blockers)
  2. Generate data model (tables + CDS views)
  3. Build behavior, UI annotations, feature control (user reviews)  ← HUMAN
  4. Create and run ALL unit tests (count matches blueprint)
  5. ATC quality gate (user reviews if warnings remain)
  6. Expose as OData service (user previews Fiori app)  ← HUMAN
  7. Extend (if requirements include post-core features)
  8. Package transport, present for release  ← HUMAN
```

## Repository Structure

```
rap-it-generator.md                        # The agent — orchestrator and workflow
skills/
  standard/                             #  standard skills
    data-modeling.md                    #   Table design, keys, field types, relationships
    rap-architecture.md                 #   BO structure, draft, state machine, composition
    behavior-design.md                  #   Validations, determinations, actions, feature control
    cds-modeling.md                     #   CDS view stack, annotations, projections
    ui-annotations.md                   #   Fiori UX: value helps, criticality, facets, actions
    service-design.md                   #   Service definition, binding, OData exposure
    testing.md                          #   Unit test design, coverage, structure
    atc-compliance.md                   #   ATC quality gate, fix strategy
    transport-management.md             #   Package design, transport layering, release
    preflight.md                        #   System verification before build
  customer specific/                    # Customer-specific skills (templates)
    naming-conventions.md               #   Customer naming philosophy (extends  standards)
    coding-standards.md                 #   Customer coding policies (extends  standards)
```

## Usage

### 1. Fill in customer skills

Copy the templates in `skills/customer specific/` and fill in your organization's naming conventions and coding standards. These extend the standards — they never override the fundamentals.

### 2. Provide requirements

```
System:      BTP_DEV
Package:     ZFAC_ISSUE_TRACKER
Transport:   create new

Requirements:
Field technicians report machine breakdowns from mobile devices.
Each report covers one or more machines on the shop floor.
We need severity levels (critical, urgent, normal), assignment to
repair crews, and a full repair log. Machines under warranty should
be flagged automatically from the equipment master. If the same
machine is reported 3+ times in 90 days, escalate to plant manager.
```

### 3. Agent builds the app

The agent translates your requirements into a technical blueprint, presents it for confirmation, then executes all 8 phases — generating tables, CDS views, behavior, tests, running ATC, exposing the service, and packaging the transport.

## Example Customer Contexts

The same agent handles different variations of equipment issue tracking:

**Factory floor** — Machine breakdowns, repair crews assigned by shift, parts tracking, recurring failure escalation.

**Commercial real estate** — HVAC and elevator issues across buildings, tenant-submitted complaints, SLA tracking against maintenance contracts.

**Vehicle fleet** — Truck breakdowns, GPS location capture, roadside vs depot classification, towing request automation, missed-maintenance detection.

Each uses the same agent and standard skills. Only the natural language requirements and customer skills differ.

## Skill Design Principles

**Standard skills** contain principles, not code snippets. They guide decisions — when to use managed vs unmanaged, how to structure CDS layers, what to test. They trust the LLM's knowledge of ABAP syntax while enforcing architectural correctness.

**Customer skills** extend the standards with organization-specific naming philosophy, coding policies, error handling patterns, and authorization rules. They are templates — fill them in per customer engagement.

**The agent never contradicts the fundamentals.** Z-prefix for custom objects, UUID keys for managed RAP, draft ETag requirements — these are non-negotiable regardless of customer preferences.

## Prerequisites

- S/4HANA system with ADT MCP Server enabled
- Access to ADT MCP tools (object creation, generators, activation, ATC, unit tests, transport, business services)
- VS Code or compatible editor with MCP client support
