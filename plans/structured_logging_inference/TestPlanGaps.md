---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
status: Open
gap_count: 16
last_updated: '2026-08-17'
---
# Gaps -- Structured Logging for Inference Components

## Scope & Endpoints

- **Monitoring CR schema not yet designed** -- Field structure, defaults,
  and validation CEL rules are sprint-zero work coordinated with
  RHAISTRAT-2416. No concrete field names or structure specified yet.
  Would be resolved by: ADR or design doc from sprint-zero joint session
  with RHAISTRAT-2416 owners and Platform/Observability team.

- **Collection pipeline approach unresolved (OTel vs CLO)** -- Sprint-zero
  evaluation required to determine whether to extend existing
  OTel-to-LokiStack pipeline (Path A, no CLO dependency) or add
  CLO-based collection (Path B, CLO v6). Affects infrastructure
  requirements, test configuration, and external dependencies. Would be
  resolved by: sprint-zero evaluation deliverable documenting the
  approach selection and architectural justification.

- **vLLM queue lifecycle API details missing** -- Deeper than the JSON
  formatter wrapper; requires scheduler-level changes in RHAII codebase
  intertwined with fork-based multiprocessing model. Scope and
  feasibility need early validation with RHAII. Would be resolved by:
  design doc or feature refinement from RHAII team engagement.

- **W3C traceparent propagation chain not validated end-to-end** -- Strategy
  marks this as "needs validation" and notes it "may expand if gaps are
  found" in the Gateway -> EPP -> vLLM chain. Each gap would require
  forwarding code. Would be resolved by: sprint-1 validation deliverable
  documenting propagation status and identifying specific gaps.

- **Console log-to-trace navigation requirements unclear** -- May require
  new Console plugin work for Tempo datasource configuration beyond what
  odh-observability currently provides. Would be resolved by: sprint-1
  validation deliverable from Console team.

- **GenAI semantic convention field stability unknown** -- `gen_ai.*`
  fields are in OTel experimental status; field names may change before
  stabilization. Would be resolved by: engineering monitoring of OTel
  GenAI semconv stabilization status.

- **Log delivery latency and query performance thresholds undefined** --
  No numeric thresholds defined for log delivery latency or Loki query
  response time under load. Would be resolved by: PM/Engineering
  decision based on Loki sizing PoC baseline measurements.

## Test Strategy & Risks

- **vLLM queue lifecycle instrumentation implementation approach** --
  Strategy acknowledges this is "deeper than the general JSON formatter"
  and requires "scheduler-level changes," but exact instrumentation
  points, effort, and RHAII capacity are not fully scoped. Would be
  resolved by: ADR or design doc detailing vLLM scheduler
  instrumentation approach, validated with RHAII team.

- **Design doc and ADR explicitly marked as "To be created"** -- Strategy
  calls out that design doc and ADR (if needed) are not yet written.
  Would be resolved by: design doc covering collection pipeline
  architecture, Monitoring CR schema, and cross-component integration;
  ADR for collection pipeline approach decision (Path A vs Path B).

## Environment & Infrastructure

- **CLO API version confirmation** -- If Path B is chosen, need to
  confirm CLO v5 vs v6 API version with logging team. Affects controller
  implementation and test environment setup. Would be resolved by: API
  spec or feature refinement from CLO/logging team.

- **vLLM JSON formatter prototype validation** -- Needs RHAII team
  validation that custom Python logging formatter works with current
  vLLM codebase. Affects test data structure and formatter
  configuration. Would be resolved by: design doc or prototype report
  from RHAII team sprint-zero validation.

- **Loki sizing guidance not available** -- Requires PoC deployment to
  gather realistic log volume numbers at representative RPS tiers (100,
  500, 1000 RPS) across 4 components. Estimated ~350GB/day at 1000 RPS.
  Blocks GA rollout but not sprint 1. Would be resolved by: performance
  report from Platform/Observability + Logging team PoC.

- **RHAII deployment-specific logging pipeline** -- RFE (RHAIRFE-2776)
  confirms RHAII is explicitly in-scope for chargeback use cases and
  documentation ("the solution should factor in how these use cases are
  enabled on both RHOAI and RHAII"). However, the RFE does not specify
  how the logging pipeline differs between RHOAI and RHAII. Would be
  resolved by: feature refinement clarifying RHAII vs RHOAI logging
  pipeline differences.

- **Test environment OCP version matrix** -- Strategy specifies 4.16+
  for RHOAI 3.6-ea.1 but doesn't specify if older OCP versions need
  testing for backports or compatibility. Would be resolved by: feature
  refinement specifying full OCP version support matrix.

- **Log pipeline throughput and query performance thresholds** -- No
  numeric thresholds for log delivery latency or Loki query response
  time. Loki sizing PoC should produce baseline numbers. Would be
  resolved by: performance report from Loki sizing PoC.
