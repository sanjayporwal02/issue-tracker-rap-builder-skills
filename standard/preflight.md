# Preflight Checks

Verification steps to run before starting any build phase. These prevent wasted effort from discovering infrastructure problems mid-build.

---

## System Connectivity

- Call `abap_lists_destinations` and verify the user's target system appears in the list.
- If the destination is not found, stop and report. Do not guess at alternative destination names.

## Package Existence

- Verify the specified package exists on the target system.
- If it does not exist, ask the user whether to create it or use a different package. Do not auto-create packages — package assignment may involve organizational decisions (software component, transport layer) that require user input.

## Transport Readiness

- If the user specified an existing transport request, verify it exists and is modifiable (not released).
- If the user specified "create new," note this for Phase 8. No transport is needed until then.

## Generator Availability

- Call `abap_generators-list_generators` on the target system.
- Verify that generators for tables, CDS views, behavior definitions, and service definitions are available.
- If any required generator is missing, the system version may not support the needed RAP features. Report this to the user with specifics: which generator is missing and what it implies about the system's capabilities.

### Generator Known Behaviors

**Prefix handling:** The x-ui-service generator (and most ABAP generators) automatically prepend `Z` to all generated object names. If you pass a prefix that already starts with `Z`, the generated names will have a double prefix (`ZZ`). To get standard single-Z names (e.g., `ZR_ISSUE`, `ZC_ISSUE`), set the prefix parameter to an empty string or to only the characters that follow Z. Always check the generator schema (`get_schema`) to understand how the prefix parameter is applied before generating.

**Field length collapse:** The generator may collapse custom field lengths to defaults (typically CHAR 10) regardless of what you specify. After generating tables, immediately verify every field's type and length against the Phase 0 blueprint. If fields were collapsed, fix them before proceeding to CDS views — a wrong field length in the table propagates through every layer above it.

## Standard Service Availability

- If Phase 0 identified dependencies on standard S/4 services (equipment master, functional location, BP, cost center), verify those services are published on the target system.
- Call `fetch_services` and search for each required service.
- If a required service is not found, report to the user. The application design may need to change (remove the external lookup, switch to a different data source, or request the service be activated by a Basis administrator).

## System Version Notes

- Different S/4HANA releases support different RAP features. Key checkpoints:
  - Draft with state machine requires a minimum release level.
  - Virtual elements require a minimum release level.
  - Some generator types may not be available on older releases.
- If the generators available on the system are a subset of what the design requires, report the gap and suggest simplifications.

## Report

After all preflight checks pass, summarize:
- Target system confirmed
- Package status (exists / needs creation)
- Transport status (existing / will create in Phase 8)
- Generators available
- Standard services confirmed (list each with entity count)
- Any limitations noted

If any check fails, do not proceed. Present the issue and wait for user guidance.

## ADT Tool Behaviors

**Save before activate:** When using ADT MCP tools, edits to an object must be explicitly saved before calling `activate_objects`. If you edit and activate without saving, the activation compiles the previously saved version — your changes are silently lost. The pattern is always: edit → save → activate. This applies to every object type: CDS views, behavior definitions, implementation classes, test classes.
