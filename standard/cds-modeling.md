# CDS Modeling

Principles for designing the CDS view stack in RAP transactional applications.

---

## Three-Layer Architecture

Every RAP app uses three CDS layers:

1. **Interface views (ZI_)** — Sit directly on the database tables. Define compositions, associations, type mappings, and calculated fields. These are the BO's data contract. They are reusable and not tied to any specific UI.

2. **Projection views (ZC_)** — Sit on the interface views. Add UI annotations (@UI), search enablement (@Search), value helps (@Consumption.valueHelp), and field visibility for the specific UI consumption. These are UI-specific.

3. **Database tables** — Raw persistence. No logic, no annotations.

Do not skip a layer. Even if the projection seems identical to the interface view, maintain the separation. Future changes (second UI, API consumption, analytics) will require it.

## Interface View Rules

- Define all compositions here: `composition [0..*] of ZI_ChildEntity as _Child`.
- Define associations to external entities here: `association [0..1] to I_Equipment as _Equipment`.
- Map all fields from the table. Use alias names in CamelCase: `key issue_uuid as IssueUuid`.
- Define the semantic key: `@ObjectModel.semanticKey: ['IssueId']`.
- Status fields should have text associations where applicable.

## Projection View Rules

- Expose only the fields the UI needs. Hide internal fields (UUIDs used only for composition, internal flags).
- Redirect associations from the interface layer: `use association _Child`.
- **Projection views do not support CASE expressions.** Any CASE-based logic (criticality mapping, derived status text, conditional calculations) must live in the interface view layer, not the projection. If you need criticality integers in the projection, either: (a) calculate them as fields in the interface view and simply expose them in the projection, or (b) use association paths to code-list CDS views that carry the criticality value. Attempting to put CASE in a projection will fail at activation.
- Add UI annotations:
  - `@UI.headerInfo` on the view for title and description fields.
  - `@UI.lineItem` for list report columns with position and importance.
  - `@UI.identification` for object page field groups.
  - `@UI.facet` for object page tab structure.
- Add search: `@Search.searchable: true` on the view, `@Search.defaultSearchElement: true` on searchable fields.
- Add value helps: `@Consumption.valueHelpDefinition` for status, severity, priority — any field with a fixed domain.

## Annotations Principles

- Annotations describe intent, not layout. The Fiori Elements framework interprets them. Do not try to micro-manage pixel placement through annotations.
- Use `@UI.lineItem.position` in increments of 10 (10, 20, 30...) to allow future insertions without renumbering.
- Use `@UI.lineItem.importance: #HIGH` for columns that must always be visible, `#MEDIUM` for columns that can collapse on narrow screens.

## Calculated and Virtual Elements

- **Calculated fields** (computed in the CDS view itself): Use for simple derivations that can be expressed in CDS syntax (CASE statements, date comparisons, concatenations). These are evaluated at the database level and are performant.
- **Virtual elements** (computed in ABAP at runtime): Use for complex calculations that require ABAP logic, external service calls, or aggregations across entities. These add runtime overhead — use sparingly and only when CDS expressions cannot cover the logic.

## Association Naming

- Associations are prefixed with underscore: `_Equipment`, `_FunctionalLocation`, `_StatusText`.
- Composition associations use the child entity's business name: `_Items`, `_Log`, `_Attachments`.

## View Naming

- Interface views: ZI_ + entity name in CamelCase (ZI_IssueReport, ZI_IssueEquipment).
- Projection views: ZC_ + entity name in CamelCase (ZC_IssueReport, ZC_IssueEquipment).
- Apply customer naming conventions for any additional prefixes or module identifiers between the standard prefix and the entity name.
