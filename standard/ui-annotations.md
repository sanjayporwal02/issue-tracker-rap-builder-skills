# UI Annotations & Fiori UX

This skill governs how generated RAP applications present themselves in Fiori Elements. Every decision here affects what the user sees and how they interact with the app. A technically correct BO with poor UX is not a finished application.

These are general principles. They apply regardless of business domain.

---

## Input Control Selection

Every field must use the input control that matches its data nature. The agent determines the right control during Phase 0 when designing the blueprint, and enforces it through CDS annotations in Phase 3.

**Decision matrix:**

| Field Nature | Control | Implementation |
|---|---|---|
| Fixed finite values (3–10 options) | Dropdown / value help | `@Consumption.valueHelpDefinition` pointing to a code-list CDS or domain fixed values |
| Fixed finite values (2–3 mutually exclusive options) | Segmented button or radio group | `@UI.dataField` with `#WITH_URL` or small value help. Consider `@UI.textArrangement: #TEXT_ONLY` |
| Boolean (yes/no, on/off, true/false) | Checkbox or toggle | Use `abap_boolean` type. Fiori renders checkboxes for boolean fields automatically. Add `@EndUserText.label` for clarity |
| Numeric quantity | Number input | Use appropriate numeric ABAP type. `@Semantics.quantity.unitOfMeasure` if unit applies |
| Currency amount | Currency input | `@Semantics.amount.currencyCode` bound to a currency field |
| Date | Date picker | Use `DATS` or date type. Fiori renders date picker automatically |
| Date + time | DateTime picker | Use `UTCLONG` or timestamp type |
| Short free text (name, title, summary) | Text input | `CHAR` or `SSTRING` with appropriate max length |
| Long free text (description, notes, comments) | Text area (multi-line) | `STRING` type. Annotate with `@UI.multiLineText: true` |
| Reference to master data (equipment, customer, material) | Value help with search and filter | `@Consumption.valueHelpDefinition` pointing to the master data CDS with `additionalBinding` for dependent fields |
| Email, phone, URL | Semantic text input | `@Semantics.email`, `@Semantics.telephone`, `@Semantics.url` — Fiori renders appropriate links and input validation |
| Percentage | Slider or number with range | Numeric type with validation constraints |

**The guiding principle:** if the user should not invent a value, do not give them a blank input. If the set of possible values is known, present those values for selection.

---

## Navigation

Users must always know where they are and how to get back. Fiori Elements provides navigation infrastructure, but the CDS and manifest configuration must enable it.

**Back navigation:**
- Fiori shell provides a back button automatically when the app uses standard Fiori Elements floorplans (List Report + Object Page). Do not build custom back logic.
- Ensure the app is configured as a List Report Object Page template — this gives back navigation for free.
- If using Flexible Column Layout, back button behavior is handled by the column collapse. The user clicks the list item again or uses the shell back.

**Breadcrumbs:**
- Fiori Elements Object Page renders breadcrumbs automatically when navigating from list → detail → sub-detail.
- For this to work, composition navigations must be annotated correctly in CDS: the child entity's `@UI.facet` must use `type: #LINEITEM_REFERENCE` pointing to the composition.
- Breadcrumb text comes from `@UI.headerInfo.typeName` — make this meaningful ("Equipment Issue", "Repair Action"), not technical ("ZZC_EQUIPMENTISSUE").

**List-to-detail navigation:**
- Annotate the semantic key or identifier field in the list with `@UI.lineItem: [{ type: #WITH_NAVIGATION_PATH }]` or rely on the default Fiori Elements behavior where clicking a row navigates to the object page.
- The object page route is configured automatically when the service binding uses a standard Fiori Elements template.

**Detail-to-child navigation (drill-down):**
- Child entities displayed as table facets should allow inline navigation to a sub-object page if the child has enough fields to warrant its own detail view.
- For simple children (3–5 fields, like a repair log entry), inline display within the parent object page is sufficient — no separate sub-page needed.
- For complex children (10+ fields, their own lifecycle), configure a sub-object page via `@UI.facet` with `type: #LINEITEM_REFERENCE` and ensure the child has its own `@UI.headerInfo`.

**Cancel and discard (draft navigation):**
- Draft-enabled apps show a "Discard Draft" action. Fiori Elements handles this automatically.
- When the user cancels editing, the draft is discarded and the app navigates back to display mode or the list page.
- Ensure `with draft` is properly configured in the behavior definition — Fiori Elements derives all draft navigation from this.

**Flexible Column Layout (optional):**
- For apps where users frequently switch between list items, consider a 2-column or 3-column layout.
- List stays visible on the left, object page on the right. No separate page navigation needed.
- Configure in the app manifest via `"flexibleColumnLayout"` settings. Not driven by CDS annotations.
- Default to full-page layout unless the use case clearly benefits from side-by-side viewing.

---

## Criticality and Visual Indicators

Any field that carries a qualitative meaning (status, severity, priority, health, risk level) must communicate that meaning visually, not just through text.

**Implementation:** Add a calculated element (e.g., `StatusCriticality`) to the CDS projection using a CASE expression that maps each value to a criticality integer:

| Criticality Value | Color | Typical Use |
|---|---|---|
| 0 | Neutral (grey) | Default, unknown, draft, not started |
| 1 | Negative (red) | Error, critical, overdue, blocked, breached |
| 2 | Warning (orange) | At risk, high severity, approaching deadline |
| 3 | Positive (green) | Completed, resolved, on-track, normal |
| 5 | Information (blue) | In progress, assigned, informational |

Reference it in annotations: `@UI.lineItem: [{ criticality: 'StatusCriticality' }]`

**Rules:**
- Every status-like field and every severity/priority field MUST have a criticality element.
- Boolean flags that indicate a problem (overdue, breached, recurring) should map `true` → criticality 1 (red) and `false` → criticality 0 (neutral).
- Do not display status as plain uncolored text. If the field has meaning, show the meaning visually.

---

## List Page Design

The list page is the first screen. Column selection and order determine whether the app feels useful or cluttered.

**Column selection (5–7 columns):**
- First: the semantic identifier (issue number, request ID, order number).
- Second: the most important descriptive field (summary, title, name).
- Include status and severity/priority — with criticality coloring.
- Include the assigned/responsible person if relevant.
- Include one timestamp (last changed or created).
- Do NOT include: UUID keys, technical fields, admin fields (created_by), or internal codes.

**Filter bar (3–5 selection fields):**
- Status — almost always the primary filter.
- Priority/severity — if the app has it.
- Assigned/responsible person.
- Date range.
- The primary reference object (equipment, customer, building).

**Search:**
- `@Search.searchable: true` on the root entity.
- `@Search.defaultSearchElement: true` on 2–4 fields: semantic ID, summary/title, key description field.

**Column importance for responsive layout:**
- `@UI.importance: #HIGH` on 3–4 essential columns. These survive when the screen is narrow.
- Secondary columns get `#MEDIUM` — visible on desktop, collapsed on mobile.

---

## Object Page Design

Structure the detail page with facets that group related information logically.

**Header area:**
- `@UI.headerInfo`: typeName (what this object is, in business language), title from semantic ID or summary, description from a secondary field.
- Show status with criticality in the header as a badge.
- Show severity/priority with criticality if applicable.

**Facet structure:**
- Group related fields into named sections using collection facets and field groups.
- Typical sections: General Information, [Reference Object] Details, Assignment/Ownership, [Child Entity] (as table).
- Each section should have 3–8 fields. If a section has 15+ fields, split it.
- Child entity tables are shown as `#LINEITEM_REFERENCE` facets within the object page.

**Field arrangement within sections:**
- Read-only derived fields (auto-populated from master data) grouped separately from editable fields.
- Mandatory fields visually together at the top of their section.
- Optional/secondary fields lower in the section.

**Never:** dump all fields in one flat form. If the object page has no facet structure, the UX is wrong.

---

## Action Placement

**Header toolbar:** Actions that change the overall object state (Assign, Approve, Reject, Resolve, Cancel, Reopen). These are the primary workflow actions. Annotate with `@UI.identification: [{ type: #FOR_ACTION }]`.

**Section toolbar:** Actions that operate on child entities (Add Entry, Add Note). These appear in the table toolbar of the relevant facet.

**Inline row actions:** Rare. Use only when an action applies to a specific child row (e.g., "Remove Item" in an editable table).

**Ordering:** Place the most common next-step action first. If the object is in "reported" state, "Assign" should be the first visible action, not "Cancel".

**Confirmation:** Irreversible or destructive actions (Cancel, Delete, Close) should require confirmation. Use `with additional implementation` in the behavior definition if a confirmation dialog or parameter input is needed.

**Overflow:** If more than 3–4 header actions exist, less-used ones move to the "..." overflow menu automatically. Design for the 2–3 most common actions to be directly visible.

---

## Side Effects

When one field changes, dependent fields must refresh immediately without a full page reload.

**Declare side effects in the behavior definition:**

```
side effects { field [SourceField] affects field [DependentField1], field [DependentField2]; }
```

**When to use:**
- A lookup field changes → derived description/detail fields refresh (e.g., equipment number → equipment description, location).
- A classifying field changes → a calculated field refreshes (e.g., severity changes → priority recalculates).
- An action executes → status, feature control, and dependent fields refresh (actions handle this automatically in Fiori Elements).

**Rule:** Every determination that auto-populates fields based on a user-editable input field MUST have a corresponding side effect. Without it, the user changes the input and the derived fields show stale data until page reload.

---

## Boolean and Flag Fields

Booleans should communicate meaning, not show raw values.

- Fiori Elements renders boolean fields as checkboxes automatically — this is acceptable for editable booleans (e.g., "Is Urgent").
- For read-only indicator flags (e.g., "Has Recurring Issues", "SLA Breached"), use criticality to show as a colored badge/icon rather than a disabled checkbox.
- Always provide a meaningful `@EndUserText.label`: "Under Warranty" not "IS_WARRANTY", "Recurring Problem" not "HAS_MULTI_OPEN".
- Position indicator flags in the header or at the top of a relevant section — not buried at the bottom of a form.

---

## Field Labels and Text

Every user-visible field MUST have a meaningful, business-language label.

- Use `@EndUserText.label` if the data element label is too technical.
- Labels: 2–3 words, business language. "Equipment Number" not "Equipment Master Data Identifier".
- For code fields with text: show the human-readable value ("Reported", "Critical"), not the internal code ("01", "C"). Use `@ObjectModel.text.element` or `@UI.textArrangement: #TEXT_ONLY`.
- `@UI.headerInfo.typeName` should be human-readable: "Equipment Issue", not "ZZC_EQUIPMENTISSUE".

---

## Child Entity Tables

Composition children displayed as table facets:

- Sort by the most relevant field: timestamp descending for logs/histories, sequence ascending for ordered items.
- Show 4–6 columns. Hide UUID keys, parent references, and technical fields.
- For immutable children (logs, audit trails): suppress Edit and Delete at the projection behavior level. Show only the "Create" action ("Add Entry").
- For editable children (line items): allow inline editing if the child has few fields, or navigate to a sub-object page if complex.
- Enable pagination if the child can have many records (20+).

---

## Error Messages and Validation Feedback

How errors are presented matters as much as whether they're caught.

- Validation errors must be field-level: attach the message to the specific field using `reported` with the field path in the behavior implementation. The error should appear next to the field, not as a generic page-level message.
- Use `severity` appropriately: `if_abap_behv_message=>severity-error` for blocking issues, `severity-warning` for non-blocking concerns, `severity-information` for informational notes.
- Error text should tell the user what to do, not just what's wrong: "Enter a severity before assigning" not "Severity is initial".
- Side effect validation (validate on field change, not just on save) gives immediate feedback. Wire critical validations to `draft determine action Prepare` so they fire during save and also during side-effect revalidation.

---

## What Must Never Happen

- A field with a known finite set of values presented as a blank text input.
- A status or severity displayed without criticality coloring.
- All fields in one flat section with no facet structure.
- UUID technical keys visible anywhere in the UI.
- Admin fields (created_by, last_changed_at) shown as editable.
- An object page with no way to navigate back to the list.
- A child entity table showing parent UUID or technical key columns.
- Actions placed where they are not discoverable (e.g., root-level workflow action buried in a child section).
- A list page with 12+ columns and no filters.
- Error messages that name the internal field ("ISSUE_STATUS is initial") instead of the business label ("Enter a status").
- A header with no type name or a type name that shows the technical CDS entity name.
