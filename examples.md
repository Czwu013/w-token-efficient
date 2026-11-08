---
when: you want a worked example of applying the rules end to end
topics: example, CSV export bug, closing report
---

# Example: fix a CSV export date bug and get CI passing

1. Map: list the file tree excluding `node_modules` and `dist`; find the export entry point.
2. Scoped read: open only `export/csv.ts` and the date helper it calls.
3. Plan check: 1-2 files → fix directly; a dozen call sites → write a change list first.
4. Subagents: a single-point fix needs none. If three unrelated export formats must also be checked, use 2-3 subagents, one per format, returning conclusion + file path only.
5. Bounded test run: at most 3 attempts; same test failing 3 times for the same reason → stop and report instead of editing the assertion.
6. Compact if the session is long; start the next task in a new session.

Closing report:

```
Fixed the CSV export date format (export/csv.ts, now ISO 8601); all tests pass.
Next step: add the same date tests to the other export formats.
```
