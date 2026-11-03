# Test Case Index — Inference Request Log Collection

[Parent test plan](../TestPlan.md) · [Current strategy](https://redhat.atlassian.net/browse/RHAISTRAT-2413)

**7 test cases: 4 P0, 2 P1, 1 P2.** All replaced cases target the five acceptance
criteria in the Jira description revised August 26, 2026; the original 51 cases for
structured JSON formatting and controller logging have been retired.

## E2E System Testing

| Test Case | Title | Priority | Strategy |
|-----------|-------|----------|----------|
| [TC-E2E-001](TC-E2E-001.md) | Native inference request logs reach Loki | P0 | AC #1 |
| [TC-E2E-002](TC-E2E-002.md) | Missing Loki dependency surfaces status | P0 | AC #2; resilience |
| [TC-E2E-003](TC-E2E-003.md) | Disabled logging creates no targeting resources | P0 | AC #3 |
| [TC-E2E-004](TC-E2E-004.md) | Namespace-scoped queries isolate tenants | P1 | AC #4 |
| [TC-E2E-005](TC-E2E-005.md) | Controller reconciliation logs excluded | P1 | AC #5 |
| [TC-E2E-006](TC-E2E-006.md) | Unreachable logging dependency does not block deployment | P0 | AC #2; resilience |

## Non-functional Testing

| Test Case | Title | Priority | Strategy |
|-----------|-------|----------|----------|
| [TC-NFR-001](TC-NFR-001.md) | Collection overhead and best-effort delivery baseline | P2 | Performance; delivery |

## Execution gate

Before executing the cases, resolve the Monitoring CR toggle/status contract,
selected pipeline resource kinds, native request-log identifiers, and Loki tenant
query/RBAC design documented in [TestPlanGaps.md](../TestPlanGaps.md). Case steps do
not assume field names, URLs, operators, or numeric thresholds absent from the strategy.
