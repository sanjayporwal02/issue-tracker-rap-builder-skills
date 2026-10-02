# Customer Naming Conventions

> This file extends the standard naming rules defined in the standard skills (data-modeling.md, cds-modeling.md, rap-architecture.md, etc.). It adds customer-specific naming patterns. It never contradicts ABAP fundamentals — Z/Y prefix for custom objects, ZI_ for interface views, ZC_ for projection views, ZSD_ for service definitions, ZSB_ for service bindings remain mandatory.

---

## Instructions for the Customer

Fill in each section below with your organization's naming rules. Delete the placeholder examples and replace with your actual conventions. Sections left empty will default to standard naming only.

---

## Namespace

<!-- Which namespace does your organization use? -->
<!-- Example: "Z" (standard) or "/ACME/" (registered customer namespace) -->

Namespace: Z

## Module / Application Prefixes

<!-- Do you use module or functional area prefixes after the namespace? -->
<!-- Example: ZFIT_ for finance, ZMFG_ for manufacturing, ZFAC_ for facilities -->
<!-- Leave empty if you do not use module prefixes -->

| Module | Prefix |
|---|---|
| (example) Finance | ZFIT_ |
| (example) Manufacturing | ZMFG_ |
| (example) Facilities | ZFAC_ |

## Object Naming Rules

<!-- How should objects be named beyond the prefix? Describe your philosophy. -->

### Tables
<!-- Example: "Singular noun describing the business object. UPPER_SNAKE_CASE. Suffix _D for data tables, _T for text tables, _LOG for log tables." -->

Pattern: `<prefix><BUSINESS_OBJECT>[_<qualifier>]`
Example: ZFAC_ISSUE_REPORT, ZFAC_ISSUE_EQUIP, ZFAC_ISSUE_LOG

### CDS Views
<!-- Example: "CamelCase after the standard prefix. Entity name matches the table's business object name." -->

Interface: `ZI_<prefix without Z><EntityName>` → ZI_FacIssueReport
Projection: `ZC_<prefix without Z><EntityName>` → ZC_FacIssueReport

### Behavior Implementation Classes
<!-- Example: "ZCL_BP_ + root entity name" -->

Pattern: `ZCL_BP_<prefix without Z><EntityName>` → ZCL_BP_FacIssueReport

### Service Definitions and Bindings
<!-- Example: "ZSD_ + application name, ZSB_ + application name" -->

Service Definition: `ZSD_<APP_NAME>` → ZSD_FAC_ISSUE_TRACKER
Service Binding: `ZSB_<APP_NAME>` → ZSB_FAC_ISSUE_TRACKER

### Test Classes
<!-- Example: "ZCL_ + app name + _TEST" -->

Pattern: `ZCL_<APP_NAME>_TEST` → ZCL_FAC_ISSUE_TRACKER_TEST

## Field Naming Rules

<!-- How should fields be named? Describe your naming philosophy for fields. -->

### General Rules
<!-- Example: "Use full words, not abbreviations, except for approved abbreviations listed below. UPPER_SNAKE_CASE." -->

- Fields use UPPER_SNAKE_CASE
- Use full words unless the abbreviation is in the approved list below
- Boolean fields: prefix with IS_ or HAS_ (IS_UNDER_WARRANTY, HAS_ATTACHMENT)
- Status fields: qualify with entity name (ISSUE_STATUS, not STATUS)
- Foreign key fields: name must match the primary key field name in the parent table exactly

### Approved Abbreviations
<!-- List abbreviations your organization allows. Everything not listed should be spelled out. -->

| Abbreviation | Full Word |
|---|---|
| (example) ID | Identifier |
| (example) DESC | Description |
| (example) EQUIP | Equipment |
| (example) FUNC_LOC | Functional Location |
| (example) QTY | Quantity |
| (example) AMT | Amount |

### Banned Abbreviations
<!-- List abbreviations that are NOT allowed even if commonly used -->

| Banned | Use Instead |
|---|---|
| (example) NR | NUMBER |
| (example) TXT | TEXT |

## Action and Method Naming

<!-- How should actions, validations, and determinations be named? -->

- Actions: camelCase verb form (triage, startRepair, resolve, addNote)
- Validations: validate_ + what is validated (validate_severity, validate_assignment)
- Determinations: verb_ + what is determined (set_issue_id, fill_equipment_details, calculate_warranty_status)

## Length Constraints

<!-- Are there maximum length rules for object names? -->

- Maximum object name length: 30 characters
- Maximum field name length: 30 characters

## Other Naming Rules

<!-- Any additional naming rules not covered above -->

(add your rules here)
