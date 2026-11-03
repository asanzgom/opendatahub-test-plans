---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
source_type: strat
status: In Review
author: AI Core Platform
components:
- AI Core Platform
- llm-d
- Model and Agent Observability
additional_docs: []
last_updated: '2026-11-03'
version: 1.2.0
reviewers: []
---
# Inference Request Log Collection Test Plan

**AI Core Platform / Model and Agent Observability** – **request-path log collection**

**Strategy**: [RHAISTRAT-2413](https://redhat.atlassian.net/browse/RHAISTRAT-2413)

> Based on the Jira description revised 2026-08-26, not the 2026-08-17 strategy attachment.
> The original plan and 51 cases covered an earlier, broader structured-logging strategy.

---

## 1. Executive Summary

### 1.1 Purpose

Validate that `odh-observability` collects native stdout/stderr logs produced while vLLM,
llm-d EPP, and MaaS process active inference requests in targeted namespaces, and forwards
those entries to an admin-installed Loki backend. Validate dependency reporting, disabled
collection, tenant isolation, and exclusion of controller reconciliation logs.

Log formatting, runtime instrumentation, and non-request lifecycle or administrative logs
are **not** deliverables. The collection path (existing OTel pipeline or conditional CLO)
and Monitoring CR schema require implementation design before path-specific tests can run.

### 1.2 Scope

#### In Scope (AI Core Platform / Observability Responsibilities)

- Collect native container stdout/stderr from active inference requests in targeted
  inference namespaces and query the captured entries in Loki. (Objective: #1)
- Report a specific missing or unreachable logging dependency through Monitoring CR
  status without interrupting inference or deployment. (Objective: #2)
- With logging disabled, create no collection resources targeting inference namespaces.
  (Objective: #3)
- Prevent a namespace-scoped tenant from reading another tenant's inference request logs.
  (Objective: #4)
- Exclude controller reconciliation/lifecycle logs while collecting request logs.
  (Objective: #5)
- Observe inference hot-path overhead without changing runtime log serialization.
  (Objective: #6)
- Treat stdout/stderr delivery as best effort, not lossless or guaranteed. (Objective: #7)
- Keep inference and model deployment functional when logging dependencies fail.
  (Objective: #8)

#### Out of Scope (Other Teams)

- Structured JSON or OTel Logs Data Model formatting, GenAI fields, new trace/span
  propagation, formatter changes, and vLLM scheduler instrumentation.
- KServe and `odh-model-controller` reconciliation, status, and lifecycle logs;
  non-request events from runtime/EPP/MaaS pods and MaaS administrative/quota logs.
- Vendor gateway (Envoy, Authorino, Limitador), DCGM/node, and `kserve-agent` logs.
- Automatic installation or lifecycle management of Loki or CLO operators.
- Mandatory Console-to-Tempo navigation or new log-to-trace correlation fields.

### 1.3 Test Objectives

1. Show active completions/request-time stdout/stderr entries in the target namespace's
   Loki query results. (AC: #1 — request logs captured and forwarded)
2. Inspect Monitoring CR status for a specific missing/unreachable logging dependency.
   (AC: #2 — dependency status condition)
3. Inspect collection resources after reconciliation with logging disabled and find none
   targeting inference namespaces. (AC: #3 — disabled means no collection resources)
4. Query as two distinct tenants and prevent Tenant A from viewing Tenant B's request
   logs. (AC: #4 — namespace-scoped Loki isolation)
5. Trigger controller reconciliation during an inference request and confirm its logs are
   absent from the inference-request collection results. (AC: #5 — controller exclusion)
6. Compare request processing with and without collection, without introducing runtime
   logging-library/serialization changes. (NFR: Performance — no hot-path CPU overhead)
7. Validate stdout/stderr collection on a best-effort basis, without asserting a
   lossless-delivery guarantee. (NFR: Delivery Guarantee — best effort)
8. While a dependency is unavailable, verify inference and model deployment continue.
   (NFR: Dependency Resilience — workloads continue despite logging failure)

---

## 2. Test Strategy

### 2.1 Test Levels

- **E2E System Testing** — Send real requests to deployed inference workloads, inspect
  Monitoring CR and collection resources, then query Loki as authorized tenants.

### 2.2 Test Types

- **Positive** — Request logs from vLLM, EPP and MaaS reach Loki while logging is enabled.
- **Negative** — Missing/unreachable dependencies, disabled logging, cross-tenant queries,
  and exclusion of background controller reconciliation output.
- **Non-functional** — Record request latency and CPU with and without collection; do not
  impose an unstated threshold or demand guaranteed delivery.

### 2.3 Test Priorities

- **P0 (Critical)** — Request-log delivery, dependency status, and disabled logging
  correspond to the strategy's P0 requirements. (Objective: #1)
- **P1 (High)** — Namespace isolation and exclusion of controller reconciliation logs.
  (Objective: #4)
- **P2 (Medium)** — Performance baseline and best-effort delivery observations, without
  a strategy-specified numeric gate. (Objective: #6)

---

## 3. Test Environment

### 3.1 Infrastructure & Configuration

- OpenShift cluster running RHOAI inference workloads with `odh-observability` enabled,
  including a deployed vLLM runtime and request paths through EPP and MaaS. Target
  release is RHOAI 3.6 per the original test-plan context; confirm exact builds with
  the release owner before execution (the current strategy sets no version floor).
- Two distinct user inference namespaces containing workloads capable of producing
  recognizable request-time stdout/stderr entries; a separate controller workload
  that can be reconciled while those requests run.
- Healthy, admin-installed Loki backend and the selected collection pipeline. CLO is
  required **only** if sprint-zero selects Path B; Path A uses existing OTel-to-LokiStack.
- Accessible `services.platform.opendatahub.io/v1alpha1` Monitoring CR for setting
  logging on/off and reading status. The actual toggle and condition schema must be
  taken from RHAISTRAT-2416 design; do not use an assumed field name or reason.
- Deliberately isolated test environment for temporarily making logging dependencies
  unavailable. Obtain an approved reversible failure-injection procedure before use.

### 3.2 Test Data Requirements

- Synthetic completions calls to vLLM; capture namespace, pod, UTC request window and
  a unique non-sensitive request marker (e.g., `qe-log-2413-a-001`) **only if the native
  request log includes it**. Otherwise identify the entry by known request-path text,
  pod and narrow time range; do not require a new log schema or echoed prompt.
- EPP request dispatch and MaaS request proxy traffic with independently identifiable
  native log entries. Determine representative message patterns from installed builds.
- Separate synthetic requests and log-query identities for tenants A and B; query each
  tenant's namespace and the other tenant's namespace in both directions.
- Controller reconciliation output emitted during the same bounded time window acts as
  an exclusion sentinel; identify by controller pod and native message, not assumed JSON.
- Healthy, missing and unreachable dependency states; preserve before/after Monitoring
  CR status snapshots and evidence that inference requests still complete.

### 3.3 Test Users

- Test-only cluster administrator: may update/read the Monitoring CR and list the selected
  collection resources wherever the implementation deploys them.
- Tenant A and Tenant B: distinct synthetic namespace-scoped query identities, permitted
  to read their own namespace's Loki inference logs and denied cross-namespace query.
  The exact Loki roles/resources and Kubernetes-to-Loki mapping are a blocking design
  gap; the tenant isolation test cannot run until this permission contract is documented.
- Collection-pipeline service account: permissions to read targeted inference container
  logs and write to Loki; actual bindings depend on the selected pipeline implementation.

### 3.4 Test Tools

- `oc` to inspect Monitoring CR, selected collection resources, pod logs and deployment
  readiness; inference request client suitable for the deployed vLLM/EPP/MaaS routes.
- OpenShift Console Observe > Logs with synthetic tenant credentials for request-log
  queries; use a supported Loki query client for full-output exclusion verification
  only when the implementation exposes one. UI layout is not an acceptance gate.
- Request latency and CPU collection tools for the descriptive performance baseline.

---

## 4. Interfaces Under Test

| Interface | Type | Purpose |
|-----------|------|---------|
| Inference request client (vLLM / EPP / MaaS) | CLI | Generate active request-path logs; actual protocol/route must be confirmed per deployment |
| Monitoring CR (`services.platform.opendatahub.io/v1alpha1`) | CRD | Toggle collection and inspect dependency conditions; schema pending RHAISTRAT-2416 |
| Cluster collection resources | CLI | Inspect selected collection resources wherever deployed; kind and location pending design |
| OpenShift Console Observe > Logs | UI | Query Loki-backed request entries as each tenant; tenancy contract pending design |

The CLI row identifies the test client's interaction, not an asserted inference API
protocol. Confirm actual client/route before execution. The Console query UI is described
in the strategy. Do not guess routes, CR field paths, resource kinds or Loki query headers.

---

## 5. Test Cases

**Test Cases Directory**: [test_cases/](test_cases/)
**Complete Test Case Index**: [test_cases/INDEX.md](test_cases/INDEX.md)

### 5.1 Test Case Organization

| Category | Test Cases | Priority Distribution |
|----------|------------|----------------------|
| TC-E2E | 6 | 4 P0, 2 P1 |
| TC-NFR | 1 | 1 P2 |

### 5.2 Test Case Naming Convention

- `TC-E2E-*`: deployed inference request and collection user journeys.
- `TC-NFR-*`: comparative inference overhead with and without log collection.

---

## 6. E2E Test Scenarios

### 6.1 Scenario Summary

| ID | Scenario | Interfaces Covered | Priority |
|----|----------|-------------------|----------|
| TC-E2E-001 | Inference request log reaches Loki | Inference request client, Console | P0 |
| TC-E2E-002 | Missing dependency is surfaced; workloads continue | Monitoring CR, inference request client | P0 |
| TC-E2E-003 | Disabled logging creates no targeting resources | Monitoring CR, collection resources | P0 |
| TC-E2E-004 | Tenants cannot query each other's request logs | Inference request client, Console | P1 |
| TC-E2E-005 | Controller logs excluded during active requests | Inference request client, Console | P1 |
| TC-E2E-006 | Unreachable dependency is surfaced; deployment continues | Monitoring CR, inference request client | P0 |

### 6.2 E2E Coverage Matrix

| Interface (from Section 4) | E2E Scenarios |
|----------------------------|---------------|
| Inference request client (vLLM / EPP / MaaS) | TC-E2E-001, TC-E2E-002, TC-E2E-004, TC-E2E-005, TC-E2E-006 |
| Monitoring CR (`services.platform.opendatahub.io/v1alpha1`) | TC-E2E-002, TC-E2E-003, TC-E2E-006 |
| Cluster collection resources | TC-E2E-003 |
| OpenShift Console Observe > Logs | TC-E2E-001, TC-E2E-004, TC-E2E-005 |

---

## 7. Non-Functional Requirements

### 7.1 Disconnected/Air-Gapped

**Not Applicable** — the current strategy does not establish a disconnected-deployment
acceptance gate. It explicitly leaves Loki and conditional CLO installation to the admin.

### 7.2 Upgrade/Migration

**Not Applicable** — the current strategy introduces no log-format migration or upgrade
acceptance criterion. The Monitoring CR schema change is pending coordination with
RHAISTRAT-2416; assess upgrade compatibility separately once that contract exists.

### 7.3 Performance/Scalability

- Compare request latency and runtime CPU with collection enabled and disabled under
  equivalent synthetic load; record baseline and any differences rather than asserting
  a threshold not specified by the strategy. (Objective: #6)
- Observe collector and Loki backlog/storage under representative load and retain evidence
  of best-effort delivery; do not demand exactly-once ingestion. (Objective: #7)

### 7.4 RBAC/Authorization

- Query each tenant's own and the other tenant's inference namespace as distinct
  namespace-scoped identities; fail if cross-tenant request entries are visible.
  (Objective: #4)

### 7.5 Security

- **Not Applicable** — the strategy defines no new authentication, TLS, credential
  storage, or audit-logging mechanism. Cross-tenant access control is tested in 7.4.

---

## 8. Risks and Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Collector path (OTel vs conditional CLO) is unresolved | High | High | Record sprint-zero choice before path-specific assertions (Objective: #1) |
| Monitoring CR schema conflicts with RHAISTRAT-2416 | High | Medium | Coordinate toggle/condition contract and test disabled reconciliation (Objective: #3) |
| Missing Loki backend disrupts active inference | High | Medium | Inject failure on a dedicated cluster; check status and workload continuity (Objective: #8) |
| Tenant mapping exposes other namespaces' request logs | High | Medium | Test both A→B and B→A queries with synthetic identities (Objective: #4) |
| Collector accidentally includes controller logs | Medium | Medium | Trigger reconciliation in request window and query for exclusion sentinel (Objective: #5) |
| High request volume exhausts Loki storage | Medium | Medium | Record representative load and document sizing; strategy sets no numeric bound (Objective: #7) |

---

## 9. Appendix

### 9.1 Test Case Summary

| Category | Total | P0 | P1 | P2 |
|----------|-------|----|----|----|
| TC-E2E | 6 | 4 | 2 | 0 |
| TC-NFR | 1 | 0 | 0 | 1 |
| **Total** | **7** | **4** | **2** | **1** |

### 9.2 Interface Coverage

| Interface | Test Cases | Coverage |
|-----------|------------|----------|
| Inference request client (vLLM / EPP / MaaS) | TC-E2E-001, TC-E2E-002, TC-E2E-004, TC-E2E-005, TC-E2E-006, TC-NFR-001 | |
| Monitoring CR (`services.platform.opendatahub.io/v1alpha1`) | TC-E2E-002, TC-E2E-003, TC-E2E-006 | |
| Cluster collection resources | TC-E2E-003 | |
| OpenShift Console Observe > Logs | TC-E2E-001, TC-E2E-004, TC-E2E-005 | |

### 9.3 Document Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Original structured-logging plan (superseded) |
| 1.2.0 | 2026-11-03 | Rebased on Aug 26 request-log-only strategy; replaced stale cases |

---

## End of Test Plan
