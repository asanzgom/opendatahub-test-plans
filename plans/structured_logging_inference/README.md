# Inference Request Log Collection

Test plan for [RHAISTRAT-2413](https://redhat.atlassian.net/browse/RHAISTRAT-2413),
using the Jira strategy description revised **August 26, 2026**. This scope collects
native stdout/stderr logs from active vLLM, EPP, and MaaS inference requests into Loki.
Structured formatting and controller lifecycle logging are expressly out of scope.

- [Test plan](TestPlan.md) (v1.2.0, In Review)
- [Test case index](test_cases/INDEX.md) (7 cases: 4 P0, 2 P1, 1 P2)
- [Quality review](TestPlanReview.md)
- [Open gaps](TestPlanGaps.md)

## Changelog

### v1.2.0 — 2026-11-03

- Rebased the plan and acceptance-criterion traceability on the current five-AC
  strategy; removed stale 20-AC, structured-logging and controller-lifecycle coverage.
- Replaced 51 cases with seven focused E2E/NFR cases; execution is blocked on
  documented collection-path, CR schema, and tenant-query implementation contracts.

### v1.0.0 — 2026-08-17

- Initial plan for the earlier structured JSON logging strategy (superseded).
