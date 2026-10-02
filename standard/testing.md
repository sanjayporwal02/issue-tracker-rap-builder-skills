# Testing

Principles for unit testing RAP Business Objects.

---

## What to Test

For every artifact type, there is a minimum test set:

- **Each validation:** One test where the validation passes (valid input). One test where the validation fails (invalid input). Verify the correct error message is returned on failure.
- **Each determination:** One test verifying the field is correctly auto-populated. One test verifying behavior when the source data is missing or unavailable (e.g., external service returns nothing).
- **Each action (state transition):** One test executing the action from the correct source status. One test verifying the action is rejected from an incorrect status.
- **Each feature control state:** One test per status verifying which fields are editable and which actions are available. At minimum, test the first status (most editable) and the last status (least editable).
- **Full lifecycle:** One end-to-end test that walks the BO through every state from creation to terminal status. This catches issues in the transition sequence that individual tests miss.

## Test Structure

- One test class per root entity. Name it per customer naming conventions (typically ZCL_ + app name + _TEST).
- Use test methods named to describe the scenario: what is being tested, under what condition, and what is expected. Readability matters more than brevity.
- Use the given-when-then structure within each method: set up data (given), perform the action (when), assert the result (then).

## Test Data

- Create test data using EML (Entity Manipulation Language) within the test method. Do not rely on pre-existing data in the system — tests must be self-contained.
- For external service dependencies (equipment master lookups, functional location validation), use test doubles or conditional logic in the determination/validation that can handle missing data gracefully. The test environment may not have the same master data as production.

## Test Execution

- Run all tests after every code change in the behavior implementation. Do not batch test runs to the end.
- When a test fails, fix the implementation — not the test. A test represents a business requirement. Changing the test means changing the requirement.
- Track which tests failed and what was fixed. This log is useful for the user's review.

## Coverage

- Aim for every validation, determination, and action to be covered. Zero-coverage behavior methods are unacceptable.
- Do not write tests for framework behavior (draft save, draft activate) unless you have customized it. Trust the RAP framework for standard draft mechanics.

## Draft-Enabled BO Testing

Testing draft-enabled BOs has specific requirements that differ from non-draft BOs:

**%tky and %is_draft:** When reading or modifying entities via EML in tests, use `%tky` (total key) which includes `%is_draft`. For active instances, always set `%is_draft = if_abap_behv=>mk-off`. Omitting `%is_draft` causes ambiguous key resolution and test failures. Example: `MODIFY ENTITIES ... ENTITY Issue EXECUTE Assign FROM VALUE #( ( %tky = VALUE #( %is_draft = if_abap_behv=>mk-off IssueUuid = lv_uuid ) ... ) )`.

**OSQL test doubles and CDS reads:** OSQL test doubles intercept direct SQL reads but do NOT intercept CDS-based reads performed by the RAP framework during BO operations. If your determinations or validations read data through CDS views (which they do in managed RAP), the test double data will not be visible to them. Workaround: use real database persistence for BO-owned tables (INSERT into the table, run the test, DELETE in teardown) and use OSQL test doubles only for master data tables that are read via direct SQL SELECT.

**Test isolation:** Each test method must clean up its own data. Use the `teardown` or `cleanup` method to DELETE all records inserted during the test. Draft-enabled BOs may leave draft table entries — clean those too.
