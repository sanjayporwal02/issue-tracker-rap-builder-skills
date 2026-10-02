# Behavior Design

Principles for implementing validations, determinations, actions, and feature control in RAP.

---

## Validations

- A validation checks that data meets a business rule. It runs on save or on a specific action trigger.
- Validations must never modify data. They only read and report errors. If you need to change data based on a condition, that is a determination.
- Every validation must return a clear, user-readable message on failure. Messages should state what is wrong and what the user should do. Avoid generic messages like "Validation failed."
- Validate one concern per validation method. Do not bundle unrelated checks into a single validation. This keeps error messages specific and makes testing straightforward.
- Trigger-on-save validations run every time the BO is saved. Trigger-on-action validations run only when a specific action is invoked. Use action-triggered validations for rules that only apply at a specific lifecycle point (e.g., "assignment must be filled before starting repair").

## Determinations

- A determination automatically sets field values based on triggers (create, save, field change).
- On-create determinations: set defaults and auto-generated values (sequential IDs, reported_by, reported_on).
- On-save determinations: compute derived fields (is_under_warranty based on warranty_end vs today, fill descriptions from external service lookups).
- A determination that reads from an external service must handle the case where the external service is unavailable or returns no data. Set the fields to empty and log a warning — do not crash the BO.
- Determinations run before validations in the RAP save sequence. Design accordingly: a determination can set a field that a validation then checks.

## Actions

- An action represents a user-initiated operation that changes the BO state. Every state machine transition is an action.
- Actions are defined as instance-bound (they operate on a specific instance of the BO).
- An action that transitions state should: validate preconditions (via linked validations), update the status field, and trigger any side effects (e.g., creating a log entry via a determination).
- Non-state-changing actions (like "add note") are also valid. They perform an operation without changing the lifecycle status.
- Actions return the updated instance data so the UI refreshes correctly.

## Feature Control

- Static feature control maps status values to field editability and action availability.
- Define a clear matrix: for each status, which fields are read-only and which actions are enabled.
- In the implementation, use a CASE statement on the status field. This is the most readable and maintainable pattern.
- Global features (fields that are always read-only, like UUID, created_by, created_at) should be set outside the CASE statement.

## External Service Consumption

- When the BO needs data from a standard S/4 service (equipment master, BP, cost center), consume it within a determination or validation — not in an action.
- Wrap the service call in a TRY-CATCH block. External services can fail. The BO must remain functional even when an external service is down.
- Cache results where possible. If the same equipment ID appears multiple times in a single save, call the service once and reuse the result.

## Log Pattern

- Many transactional apps include an activity/resolution log as a child entity. This log records what happened: status changes, notes, assignments.
- Log entries are created by determinations that trigger on status change or by dedicated "add note" actions.
- Log entries are typically immutable — no update, no delete. Enforce this in the behavior definition by omitting update and delete for the log entity.
