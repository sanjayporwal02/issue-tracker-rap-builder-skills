# Transport Management

Principles for packaging and releasing ABAP development objects.

---

## Package Design

- One development package per application. All objects for the Equipment Issue Tracker (or equivalent) live in a single package.
- Package name follows customer naming conventions. Typical pattern: Z + module prefix + application abbreviation (e.g., ZFAC_ISSUE_TRACKER).
- Do not split an application across multiple packages unless the application is genuinely modular (shared library package consumed by multiple apps). For a single transactional app, one package is correct.

## Transport Layering

All objects for one application go into one transport request. Within that transport:

- Tables and data elements are foundational — they must activate first.
- CDS interface views depend on tables.
- Behavior definition depends on CDS interface views.
- Behavior implementation depends on behavior definition.
- CDS projection views depend on interface views and behavior.
- Service definition and binding depend on projection views.
- Test classes depend on all of the above.

The transport system handles activation order within a single transport. However, if objects fail to activate in the target system, check this dependency chain.

## Transport Naming

- Transport description should clearly identify the application and purpose: "ZFAC_ISSUE_TRACKER — Initial development" or "ZFAC_ISSUE_TRACKER — Phase 7 extension."
- Apply customer conventions for transport description format if specified.

## What Must Be in the Transport

Before release, verify the transport contains:
- All database tables and data elements
- All CDS interface views
- All CDS projection views
- Behavior definition (interface and projection)
- Behavior implementation class
- Service definition
- Service binding
- Test class(es)
- Any custom exception classes, message classes, or number range objects

A missing object causes activation failures in the target system. Check the transport object list against your package contents.

## Release

- The agent does not release transports. It prepares the transport, generates the unified diff for review, runs final ATC and tests, and presents everything to the user.
- The user decides when and whether to release. This is a governance decision, not an automation decision.
