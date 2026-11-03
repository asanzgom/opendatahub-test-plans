---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
status: Open
gap_count: 9
last_updated: '2026-11-03'
---
# Gaps — Inference Request Log Collection

The current Jira strategy was revised on August 26, 2026. These are outstanding
implementation and execution details, not permission to reinstate the descoped
structured-logging requirements. Seven current-scope cases are drafted, but their
cluster-specific steps cannot be made fully executable without the contracts below.

## Blocking test execution

1. **Collection architecture and selectors** — Sprint-zero must choose existing OTel
   Path A or CLO Path B and document collection resource kinds, namespace/pod selection
   rules and resources that should not exist when logging is disabled (AC #1, #3, #5).
   Resolve with an Observability/Logging design decision and deployed configuration.
2. **Monitoring CR contract** — Logging toggle field/default, reconciliation completion
   signal, dependency condition type/reasons and recovery state are unspecified (AC #2,
   #3). Resolve with the Monitoring CR API/schema coordinated with RHAISTRAT-2416.
3. **Loki tenancy/RBAC contract** — Namespace-to-Loki tenant mapping, synthetic user
   roles and permissions, query authentication, and authorization behavior are
   unspecified (AC #4). Resolve with the selected pipeline's security design; do not
   test as cluster admin in place of a tenant.
4. **Native request-log evidence** — Specific native vLLM, EPP and MaaS messages and
   testable inference request entry points depend on the deployed build; no new JSON
   fields or request identifiers are required (AC #1, #5). Resolve with request-path
   test fixtures and log samples on the target release.
5. **Failure-injection/recovery procedure** — No approved procedure for simulating a
   missing and unreachable logging backend or selected collector while preserving
   inference traffic is documented (AC #2; Dependency Resilience NFR). Resolve with
   a platform-owned reversible test procedure on a dedicated cluster.

## Non-blocking scope / measurement decisions

1. **Version matrix** — Current strategy sets no OCP, RHOAI, collector or Loki version
   floor. Resolve exact target builds with release owner before execution.
2. **Best-effort and performance gates** — No numerical overhead, collector backlog,
   delivery window or acceptable loss threshold is specified. TC-NFR-001 records
   comparative evidence only; PM/engineering must set a threshold if a CI gate is wanted.
3. **Runtime opt-in content logging** — P1 high-level requirement has no acceptance
   criterion, flag name or expected default/format. Keep it out of the seven AC-traced
   cases unless a runtime-specific contract and criterion are supplied.
4. **Optional query/correlation behavior** — Console filters and correlation via an
   already-present `trace_id` are described without a separate AC; no new trace fields
   or Console-to-Tempo navigation is required. Define any extra validation explicitly
   before adding cases.

## Superseded gaps

The prior 16-gap list concerned the older 20-AC strategy. JSON formatter,
GenAI schema, vLLM queue instrumentation, mandatory trace propagation and KServe
controller-log delivery gaps were retired because those behaviors are now **out of scope**.
Unresolved collection-path, CR, Loki sizing and version details were carried over above.
