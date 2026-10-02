# Data Modeling

Principles for designing database tables in RAP transactional applications.

---

## Keys

- Every root entity table uses a UUID primary key (RAW 16, data element SYSUUID_X16). This is non-negotiable for managed RAP with draft.
- Child entity tables use their own UUID primary key plus the parent's UUID as a foreign key field.
- Semantic keys (human-readable identifiers like issue numbers, request IDs) are separate fields, not the technical primary key. They are populated by determinations (number range or sequential logic).

## Admin Fields

Every table includes four administrative fields at the end of the structure:
- CREATED_BY (SYUNAME, CHAR 12)
- CREATED_AT (TIMESTAMPL, DEC 21,7)
- LAST_CHANGED_BY (SYUNAME, CHAR 12)
- LAST_CHANGED_AT (TIMESTAMPL, DEC 21,7)

These are mapped to RAP's administrative field annotations in the behavior definition. Do not omit them.

## Field Design

- Use existing data elements for standard concepts: EQUNR for equipment, TPLNR for functional location, KOSTL for cost center, BUKRS for company code, MATNR for material.
- Create custom data elements only for domain-specific fields. Name them per the customer naming conventions.
- Status fields use CHAR(2) to allow two-digit status codes (01, 02, 03...). This gives room for up to 99 states without structure changes.
- Boolean fields use ABAP_BOOLEAN (CHAR 1). Name them with IS_ or HAS_ prefix to make their meaning unambiguous.
- Description/text fields: use STRING for long text, CHAR with appropriate length for short text. Avoid CHAR(255) as a lazy default — choose a length that reflects the actual data.
- Date fields use DATS. Timestamp fields use TIMESTAMPL.

## Table Relationships

- Parent-child relationships are expressed through the child table containing the parent's UUID as a foreign key field. The field name in the child must match the primary key field name in the parent exactly.
- Do not create deep hierarchies without justification. Most transactional apps need 2-3 levels maximum: root → items → log/attachments.

## Table Naming

- Tables follow the Z-prefix convention (or customer namespace) plus a descriptive name.
- Apply customer naming conventions for abbreviations, module prefixes, and semantic naming rules.
- Table names are in UPPER_SNAKE_CASE.

## Post-Generation Verification

After generating tables from a blueprint, immediately verify the generated structure against the design:

- **Field types and lengths:** Generators may collapse custom field lengths to defaults (e.g., all CHAR fields become CHAR 10). Compare every field in the generated table to the blueprint. If a field was specified as CHAR(40) for a description but the generator created CHAR(10), fix it before proceeding. A wrong field length in the table propagates through every CDS layer above it.
- **Field completeness:** Verify every field from the blueprint exists in the generated table. Generators may silently drop fields they cannot interpret.
- **Key design:** Confirm UUID primary key (RAW 16) and parent foreign keys are present and correctly typed.

Do not proceed to CDS view generation until the tables match the blueprint exactly.
