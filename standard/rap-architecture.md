# RAP Architecture

Principles for structuring RAP Business Objects in transactional applications.

---

## Implementation Type

- Use **managed** implementation for standard transactional apps where RAP handles persistence. This is the default for new development on S/4HANA.
- Use **unmanaged** only when you must wrap existing logic (BAPIs, function modules) or need full control over the persistence layer. This should be an explicit, justified decision — not a default.

## Draft

- Enable draft on the root entity for any application where users fill forms across multiple screens or sessions.
- Draft uses total ETag for optimistic concurrency control. The LAST_CHANGED_AT field on the root entity serves as the ETag field.
- Draft requires the admin fields (created_by, created_at, last_changed_by, last_changed_at) to be mapped in the behavior definition.
- Child entities inherit draft from the root through composition. Do not enable draft independently on child entities.

## Composition vs Association

- **Composition:** parent owns the child's lifecycle. Creating, updating, or deleting the parent affects children. Use for items, log entries, attachments — anything that has no meaning without the parent.
- **Association:** reference to an independent entity. The referenced entity has its own lifecycle. Use for lookups: equipment master, functional location, business partner.
- Every child entity in a transactional app is a composition from the root. If something is not owned by the root, it is an association, not a child.

## State Machine

- Use a status field (CHAR 2) on the root entity to drive the state machine.
- Define explicit transitions: which status can move to which, and through which action.
- Allow at most one backward transition (e.g., Resolved → Reopened) to handle real-world correction needs. Avoid free-form status changes.
- Each transition is triggered by a named action. Users do not edit the status field directly — they invoke actions.

## Feature Control

- Use **static feature control** to determine which fields are editable and which actions are available in each status.
- In early statuses, most fields are editable. As the lifecycle progresses, fewer fields are editable. In terminal statuses, nothing is editable.
- Actions are available only in the statuses they transition from. An action that transitions 01 → 02 is only visible in status 01.

## BO Composition Depth

- Root entity (the main business object) → one or more child entities (items, log, attachments).
- Avoid going deeper than 3 levels unless the domain genuinely requires it. Each level adds complexity in draft handling, lock management, and testing.

## Authorization

- Plan for authorization from the start. The behavior definition should include authorization control declarations even if the initial implementation is permissive.
- Instance-based authorization (checking whether the user can access this specific instance) is preferred over global authorization for transactional apps.
