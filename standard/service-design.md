# Service Design

Principles for exposing RAP Business Objects as OData services.

---

## Service Definition

- The service definition names which projection views are exposed. It is the API contract.
- Expose all projection views that the UI needs to navigate: the root entity and all child entities.
- Name the service definition ZSD_ + application name (e.g., ZSD_ISSUE_TRACKER). Apply customer naming conventions for additional prefixes.
- Do not expose interface views directly. Always go through the projection layer.

## Service Binding

- The service binding connects the service definition to an OData protocol and a binding type.
- Use **OData V4** for new development. V2 only if there is a specific integration requirement with a V2-only consumer.
- Use **UI binding type** for Fiori Elements consumption. Use **Web API** binding type for headless/API-only consumption.
- Name the service binding ZSB_ + application name (e.g., ZSB_ISSUE_TRACKER).

## Verification After Exposure

After activating the service definition and binding, always verify the exposed service using the business service MCP tools:

1. Confirm all expected entity sets are present.
2. Confirm compositions produce navigable associations in the OData metadata.
3. Confirm all actions from the behavior definition are visible as bound operations.
4. Confirm draft operations (Edit, Activate, Discard, Prepare) are present if draft is enabled.

If anything is missing, the most common causes are:
- Action not exposed in the projection behavior definition.
- Composition not redirected in the projection view using `use association`.
- Child entity not listed in the service definition.

Fix at the source (projection layer or service definition), not through workarounds.

## Projection Behavior

- The projection behavior definition must explicitly expose every action and draft operation that should be available to consumers.
- If an action exists in the interface behavior but is not mentioned in the projection behavior, it will not appear in the service. This is a common oversight.
