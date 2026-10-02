# Customer Coding Standards

> This file extends the coding principles defined in the standard skills (behavior-design.md, coding-standards.md equivalents within the standard skills). It adds customer-specific coding policies. It never contradicts clean ABAP or RAP framework requirements.

---

## Instructions for the Customer

Fill in each section below with your organization's coding rules. Delete the placeholder examples and replace with your actual standards. Sections left empty will default to standard practices only.

---

## Exception Handling

<!-- How does your organization handle exceptions? -->
<!-- Example: "All custom exceptions inherit from a root exception class ZCX_<module>_ROOT. Each application has its own exception class inheriting from the module root." -->

Exception class hierarchy:
- Root: (example) ZCX_FAC_ROOT
- Per application: (example) ZCX_FAC_ISSUE_TRACKER inheriting from ZCX_FAC_ROOT
- All RAP validation messages use the application exception class with message constants

## Message Handling

<!-- How should messages be structured? -->
<!-- Example: "Use a dedicated message class per application. Message IDs are three-digit numeric. Severity levels follow: E for validation errors, W for warnings, I for informational, S for success." -->

- Message class per application: (example) ZFAC_ISSUE
- Message numbering: (example) 001-099 for validations, 100-199 for determinations, 200-299 for actions, 900-999 for technical errors

## Logging

<!-- Does your organization require application logging beyond RAP's built-in change tracking? -->
<!-- Example: "All external service calls must be logged to the Application Log (BAL). Log object: Z_FAC, sub-object per application." -->

(describe your logging requirements here)

## Error Handling for External Service Calls

<!-- What should happen when an external service call fails? -->
<!-- Example: "Log the error to application log. Set the dependent fields to empty. Add a warning message to the BO messages. Never crash the save operation due to an external service failure." -->

(describe your external service error handling policy here)

## Authorization

<!-- What authorization pattern does your organization use? -->
<!-- Example: "All custom transactional apps use a custom authorization object Z_FAC_AUTH with fields for action type (CREATE, CHANGE, DISPLAY) and organizational scope (plant, company code). Instance authorization checks plant assignment of the functional location." -->

Authorization object: (example) Z_FAC_AUTH
Fields: (example) ACTVT (activity), WERKS (plant)
Instance check basis: (example) Plant derived from functional location

## Code Documentation

<!-- What level of inline documentation is expected? -->
<!-- Example: "Method-level comments for every validation, determination, and action explaining the business rule in one sentence. No line-by-line comments. Class header comment describing the purpose and BO it implements." -->

(describe your documentation requirements here)

## Performance Rules

<!-- Any customer-specific performance rules? -->
<!-- Example: "No SELECT inside a loop. All database reads must be bulk-collected. External service calls must be deduplicated — same input parameters should not trigger multiple calls within a single save sequence." -->

(describe your performance rules here)

## Code Review Checklist

<!-- What does your team check in code reviews? This helps the agent proactively address review concerns. -->
<!-- Example: "1. Every validation returns a user-readable message. 2. No hard-coded values — use constants or configuration. 3. External service calls are wrapped in TRY-CATCH. 4. Feature control matrix is complete for all statuses." -->

(describe your review criteria here)

## Other Coding Rules

<!-- Any additional coding standards not covered above -->

(add your rules here)
