# ATC Compliance

Principles for using the ABAP Test Cockpit as a quality gate.

---

## When to Run ATC

- After Phase 3 (Behavior) and Phase 4 (Test) are complete — this is the primary quality gate.
- After Phase 7 (Extend) if extension artifacts were added.
- As a final check in Phase 8 (Release) across the full transport scope.

## Acceptable Finding Levels

- **Errors:** Zero. No errors in the final state. Every error must be resolved.
- **Warnings:** Zero. Warnings often indicate real issues (missing exception handling, unused variables, risky type conversions). Resolve them.
- **Informational:** Acceptable. Review them for relevance but they do not block release.

## Fix Strategy

Apply fixes in this order:

1. **Deterministic quick fixes first.** These are automated, predictable fixes for naming conventions, missing pragmas, simple code style issues. They are safe to apply in batch.

2. **AI fixes second.** These address more complex findings: error handling improvements, exception class usage, code restructuring. Apply them one category at a time and review what changed.

3. **Manual analysis last.** If a finding cannot be fixed by either method, analyze it. If it is a false positive (the check does not apply to this context), document it. If it is a genuine issue, fix it manually.

## After Every Fix Round

- Re-run unit tests immediately. A fix that breaks behavior is worse than the finding it resolved.
- Re-run ATC to confirm the findings are resolved. Some fixes can introduce new findings.

## Suppression

- Do not suppress findings with pragmas unless the finding is genuinely a false positive and you can explain why.
- Never suppress to "clean up" the ATC report. Suppression hides problems — it does not solve them.

## Reporting

- After the final ATC pass, report to the user: total findings found, how many were fixed deterministically, how many were fixed by AI, how many remain (should be zero errors/warnings), and any informational findings worth noting.
