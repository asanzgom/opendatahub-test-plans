---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
source_type: strat
status: In Review
author: AI Core Platform
components:
- AI Core Platform
- Model and Agent Observability
additional_docs:
- https://redhat.atlassian.net/browse/RHAIRFE-2776
last_updated: '2026-08-17'
version: 1.0.0
reviewers: []
---
# Structured Logging for Inference Components Test Plan
**AI Core Platform / Model and Agent Observability – Structured JSON Logging
and Log Collection Pipeline**

**Strategy**: [RHAISTRAT-2413](https://redhat.atlassian.net/browse/RHAISTRAT-2413)

---

## 1. Executive Summary

### 1.1 Purpose

This test plan validates the structured JSON logging capability for
RHOAI inference components (vLLM, llm-d EPP, MaaS, KServe controllers)
as defined in RHAISTRAT-2413. The strategy standardizes log emission on
the OTel Logs Data Model with `trace_id`/`span_id` correlation, extends
the odh-observability module to collect inference logs and route them to
Loki, and enables operators to query logs by service name, trace ID, pod
name, severity, and time range through the OpenShift Console.

Testing covers the full lifecycle: structured log emission from each
inference component, collection pipeline creation and dependency
management by the odh-observability controller, log delivery to Loki,
multi-tenant log isolation, log-to-trace correlation with Tempo, and
opt-in privacy-sensitive content logging. The strategy affects five teams
and requires sprint-zero evaluation of the collection pipeline approach
(OTel-based vs CLO-based) before controller work begins.

### 1.2 Scope

#### In Scope (AI Core Platform / Observability Responsibilities)

- Structured JSON log emission from vLLM, llm-d EPP, MaaS, and KServe
  controllers conforming to the OTel Logs Data Model
- Log entries with `timestamp`, `severity`, `body`, and resource
  attributes (`service.name`, `k8s.pod.name`, `k8s.namespace.name`)
- `trace_id` and `span_id` inclusion when W3C traceparent context is
  available
- vLLM queue lifecycle logging (enqueue, dequeue, timeout/discard with
  reason and engine identity)
- llm-d EPP routing decision logs (engine selected, selection reason,
  queue depth, KV-cache utilization)
- MaaS routing rejection logs and tenant/token logs with GenAI semantic
  convention fields
- KServe controller lifecycle logs (reconciliation events, status
  transitions, error conditions)
- GenAI semantic convention fields (`gen_ai.request.model`,
  `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`) where
  components have the data
- Logging collection pipeline extension in odh-observability module to
  route inference namespace logs to Loki
- Dependency status surfacing on Monitoring CR when required operators
  (CLO if Path B, Loki) are missing or unhealthy
- Multi-tenant log isolation in Loki aligned with Kubernetes namespace
  RBAC
- Log query via OpenShift Console by `trace_id`, `service.name`,
  `k8s.pod.name`, severity, and time range
- Log-to-trace navigation from OpenShift Console to Tempo via `trace_id`
- Opt-in request/response content logging for vLLM, gated behind
  explicit configuration flag (`VLLM_LOG_REQUESTS_CONTENT`)
- Backwards compatibility for unstructured text log output via
  configuration toggle (`LOG_FORMAT=text`)

#### Out of Scope (Other Teams)

- Vendor gateway logging (Envoy access logs, Authorino auth decision
  logs, Limitador rate-limit logs) -- owned by RHAIRFE-2784
- DCGM hardware error logs -- owned by RHAIRFE-2784
- Custom log retention policies -- delegated to cluster admin via CLO
  configuration
- Real-time log streaming/tailing UIs beyond the OpenShift Console
- Log-based alerting rules -- future phase
- Training workload logging (Ray, training-operator, trainer)
- Notebook server logging
- Auto-installation or lifecycle management of CLO or Loki operators --
  per platform policy (overlay 0008)
- kserve-agent sidecar logging -- incompatible with RHOAI's
  RawDeployment-only mode

### 1.3 Test Objectives

1. Verify structured JSON log emission from all inference components
   (vLLM, llm-d EPP, MaaS, KServe controllers) conforms to OTel Logs
   Data Model schema with required fields (`timestamp`, `severity`,
   `body`, `service.name`, `k8s.pod.name`, `k8s.namespace.name`)
2. Validate `trace_id` and `span_id` correlation between logs and
   distributed traces across the full inference request path
   (Gateway -> EPP -> vLLM), enabling log-to-trace navigation in
   the OpenShift Console
3. Confirm the logging collection pipeline creates resources and routes
   inference namespace logs to Loki when dependencies are installed
   and healthy
4. Verify dependency status surfacing on Monitoring CR status conditions
   when required operators are missing or unhealthy, with clear operator
   identification and installation documentation reference
5. Validate multi-tenant log isolation in Loki aligned with Kubernetes
   namespace RBAC, ensuring tenants cannot access logs from other
   namespaces
6. Confirm opt-in request/response content logging is gated by an
   explicit configuration flag and does not log content by default
7. Verify log queryability by `trace_id`, `service.name`,
   `k8s.pod.name`, severity, and time range in the OpenShift Console
   Observe > Logs interface

---

## 2. Test Strategy

### 2.1 Test Levels

- **API Integration Testing** -- Structured log emission from
  REST/gRPC endpoints in vLLM, EPP, MaaS, and KServe controllers;
  validation that logs include required OTel fields
- **Data Validation Testing** -- Log schema conformance against OTel
  Logs Data Model, GenAI semantic convention field validation, JSON
  parsing correctness
- **Functional Testing** -- Log collection pipeline creation,
  dependency status surfacing, log-to-trace correlation, query
  filtering by `trace_id`/pod name/service name, opt-in content
  logging gate
- **Integration Testing** -- End-to-end log flow from component stdout
  through collection agent (OTel or CLO) to Loki to Console query
  interface; cross-component trace correlation across vLLM, EPP,
  and MaaS
- **Performance Testing** -- Log emission latency (no measurable
  regression from JSON serialization), log volume handling at high
  RPS, Loki ingest capacity under load
- **Security Testing** -- Multi-tenant log isolation via Loki tenant
  mapping, namespace-scoped RBAC enforcement, opt-in content logging
  privacy gate validation

### 2.2 Test Types

- **Positive Testing** -- Valid log emission with all required fields,
  successful collection pipeline creation, correct schema conformance,
  `trace_id` correlation across components
- **Negative Testing** -- Missing dependencies (CLO/Loki unavailable),
  invalid configuration, Loki backend unreachable, missing W3C
  traceparent headers, request failures before token count generation
- **Boundary Testing** -- High RPS inference loads, log volume
  saturation, OOM-kill scenarios during KV cache pressure, large-scale
  multi-tenant deployments
- **Regression Testing** -- Backwards compatibility with unstructured
  text format (`LOG_FORMAT=text`), no breaking changes to existing
  `kubectl logs` workflows

### 2.3 Test Priorities

- **P0 (Critical)** -- Core structured log emission with
  `timestamp`/`severity`/`trace_id`/resource attributes from all
  inference components; logging collection pipeline creation and
  lifecycle management by odh-observability; dependency status
  condition surfacing on Monitoring CR; vLLM queue lifecycle logs;
  EPP routing decision logs
- **P1 (High)** -- GenAI semantic convention fields on completion logs;
  log-to-trace navigation in Console; opt-in content logging gate;
  MaaS tenant/token logs for chargeback; multi-tenant log isolation;
  schema conformance automated tests
- **P2 (Medium)** -- Per-component log schema documentation validation;
  telemetry contract alignment (RHAIRFE-2227); backwards compatibility
  validation (text format toggle)

---

## 3. Test Environment

### 3.1 Test Cluster Configuration

- **OpenShift version**: 4.16+ (minimum supported for RHOAI 3.6-ea.1)
- **RHOAI version**: 3.6
- **Inference components**: vLLM (llm-d), llm-d-inference-scheduler
  (EPP), models-as-a-service (MaaS), KServe controllers,
  odh-model-controller, odh-observability
- **External operators** (admin-installed, not auto-provisioned):
  - Loki Operator with LokiStack (required for all paths)
  - Cluster Logging Operator v6 (conditional -- only if Path B
    chosen during sprint-zero evaluation)
  - Tempo Operator (for trace correlation)
  - OpenTelemetry Operator
- **GPU requirements**: GPU-equipped cluster for KV cache pressure
  testing (AC 6) -- weekly cadence on dedicated QE cluster
- **Runtimes**: Python (vLLM engine), Go 1.25 (EPP, MaaS, KServe
  controllers)

### 3.2 Test Data Requirements

- **Structured log samples**: JSON log entries conforming to OTel Logs
  Data Model with fields: `timestamp`, `severity`, `body`,
  `service.name`, `k8s.pod.name`, `k8s.namespace.name`, `trace_id`,
  `span_id`
- **W3C traceparent headers**: For trace correlation testing across
  Gateway -> EPP -> vLLM request path
- **Mock inference requests**: Known prompts with expected token counts
  for content logging and GenAI semconv field validation
- **Configuration artifacts**:
  - Monitoring CR YAML with logging subsection (schema TBD in
    sprint-zero, coordinated with RHAISTRAT-2416)
  - LokiStack configuration with multi-tenancy (namespace-to-tenant
    mapping)
  - OTel collector configuration (if Path A) or CLO
    ClusterLogForwarder resources (if Path B)
- **Model artifacts**: LLMInferenceService deployments for vLLM runtime
  testing
- **Scenario-specific data**:
  - Quota-exceeded trigger data for MaaS rejection logs (AC 3)
  - KV cache pressure scenario data for vLLM discard logs (AC 6)
  - Multi-namespace tenant configurations for isolation testing
    (AC 13)

### 3.3 Test Users

- **Cluster admin**: For installing external operators (CLO, Loki
  Operator, Tempo Operator, OpenTelemetry Operator) and configuring
  DSCInitialization
- **Namespace admin** (llm-d-as-a-service tenancy): Full visibility to
  all inference logs in their namespaces
- **Platform team user** (MaaS tenancy): Full log access across
  inference namespaces for SLA breach investigation
- **MaaS tenant users**: Two or more tenants in separate namespaces
  for multi-tenant isolation testing (AC 13) -- tenants must NOT see
  each other's logs
- **Service accounts**:
  - OTel collector service account with
    `lokistack-application-logs-writer` ClusterRoleBinding
  - Inference component service accounts for LLMInferenceService
    and InferenceService pods

---

## 4. Components and Methods Under Test

| Component / Method | Type | Purpose | Priority |
|--------------------|------|---------|----------|
| vLLM JSON log formatter | Python logging module | Emit structured JSON logs to stdout with OTel schema | P0 |
| vLLM queue lifecycle event emission | Python method | Emit enqueue/dequeue/timeout/discard logs with reason and engine identity | P0 |
| llm-d EPP OTel SDK integration | Go method | Propagate `trace_id`/`span_id` via W3C traceparent | P0 |
| KServe controller lifecycle log emission | Go/controller-runtime | Emit LLMInferenceService reconciliation, status transition, error logs | P0 |
| odh-observability Monitoring CR reconciliation | Go controller | Reconcile Monitoring CR and create/update logging collection resources | P0 |
| odh-observability dependency status management | Go controller | Surface missing/unhealthy dependency status on Monitoring CR | P0 |
| odh-observability logging resource creation | Go controller | Create OTel collector config or CLO ClusterLogForwarder resources | P0 |
| Monitoring CR logging subsection | CRD field | Configure logging collection pipeline (schema pending sprint-zero) | P0 |
| Loki log ingestion endpoint | REST API | Receive structured logs from OTel collector or CLO | P0 |
| Loki multi-tenant isolation | Config | Map Kubernetes namespaces to Loki tenants for RBAC-aligned isolation | P0 |
| llm-d EPP routing decision log emission | Go/zap | Emit logs with engine selected, reason, queue depth, KV-cache util | P1 |
| MaaS routing rejection log emission | Go/Gin | Emit rejection logs with tenant identity, model name, reason, `trace_id` | P1 |
| MaaS tenant/token log emission | Go/Gin | Emit logs with tenant identity, model name, token counts, `trace_id` | P1 |
| vLLM GenAI semconv field emission | Python method | Include `gen_ai.request.model`, `gen_ai.usage.*_tokens` on completions | P1 |
| vLLM opt-in content logging | Python method | Emit prompt/completion content when `VLLM_LOG_REQUESTS_CONTENT=true` | P1 |
| LLMInferenceServiceConfig content logging flag | CRD field | Surface `VLLM_LOG_REQUESTS_CONTENT` per versioned base config | P1 |
| Loki log query API (via Console) | REST API | Query logs by `trace_id`, `service.name`, `k8s.pod.name`, severity | P1 |
| OpenShift Console Observe > Logs interface | UI | Filter and display logs from Loki | P1 |
| OpenShift Console log-to-trace navigation | UI | Navigate from log entry `trace_id` to Tempo trace view | P1 |
| vLLM `LOG_FORMAT` env var | Config | Toggle between JSON (default) and text log output | P2 |

---

## 5. Test Cases

**51 test cases** generated across 12 categories (24 P0, 24 P1, 3 P2).

**Test Cases Directory**: [test_cases/](test_cases/)
**Complete Test Case Index**: [test_cases/INDEX.md](test_cases/INDEX.md)

### 5.1 Test Case Organization

| Category | Test Cases | Priority Distribution |
|----------|------------|----------------------|
| TC-SCHEMA | 6 | 3 P0, 3 P1 |
| TC-EMIT | 4 | 3 P0, 1 P1 |
| TC-QUEUE | 4 | 4 P0 |
| TC-ROUTE | 5 | 5 P1 |
| TC-PIPE | 5 | 4 P0, 1 P1 |
| TC-DEP | 4 | 3 P0, 1 P1 |
| TC-QUERY | 5 | 5 P1 |
| TC-TRACE | 4 | 1 P0, 3 P1 |
| TC-TENANT | 3 | 1 P0, 2 P1 |
| TC-PRIV | 3 | 3 P1 |
| TC-COMPAT | 3 | 3 P2 |
| TC-E2E | 5 | 5 P0 |

### 5.2 Test Case Naming Convention

Test cases follow the naming pattern: `TC-<CATEGORY>-<NUMBER>`

- `TC-SCHEMA` -- Log schema conformance and OTel field validation
- `TC-EMIT` -- Log emission from individual inference components
- `TC-QUEUE` -- vLLM queue lifecycle event logging
- `TC-ROUTE` -- EPP routing decision and MaaS rejection logging
- `TC-PIPE` -- Collection pipeline creation and lifecycle
- `TC-DEP` -- Dependency status surfacing on Monitoring CR
- `TC-QUERY` -- Log query and filtering via Loki/Console
- `TC-TRACE` -- Log-to-trace correlation and navigation
- `TC-TENANT` -- Multi-tenant log isolation
- `TC-PRIV` -- Opt-in content logging and privacy controls
- `TC-COMPAT` -- Backwards compatibility (text format toggle)
- `TC-E2E` -- End-to-end scenarios across multiple components

---

## 6. E2E Test Scenarios

End-to-end scenarios that validate the user journeys defined in the
strategy. Each scenario maps to one or more TC-E2E-*.md test cases
generated by `/test-plan-create-cases`.

> **Requirement**: At least one E2E scenario MUST be generated for
> each P0 endpoint in Section 4.
> E2E scenarios will be filled by `/test-plan-create-cases`.

### 6.1 Scenario Summary

| ID | Scenario | Endpoints Covered | Priority |
|----|----------|-------------------|----------|
| TC-E2E-001 | End-to-end structured log emission and collection | vLLM log formatter, Loki ingestion, Monitoring CR reconciliation, logging resource creation | P0 |
| TC-E2E-002 | Cross-component trace correlation through inference path | EPP OTel SDK, vLLM log formatter, Console log-to-trace navigation | P0 |
| TC-E2E-003 | Collection pipeline lifecycle and dependency surfacing | Monitoring CR reconciliation, dependency status management, logging resource creation, Monitoring CR logging subsection | P0 |
| TC-E2E-004 | Multi-tenant log isolation in MaaS deployment | Loki multi-tenant isolation, MaaS tenant/token emission, Loki query API | P0 |
| TC-E2E-005 | vLLM queue lifecycle debugging workflow | vLLM queue lifecycle emission, Loki ingestion, Loki query API | P0 |

### 6.2 E2E Coverage Matrix

| Endpoint (from Section 4) | E2E Scenarios |
|----------------------------|---------------|
| vLLM JSON log formatter | TC-E2E-001, TC-E2E-002 |
| vLLM queue lifecycle event emission | TC-E2E-005 |
| llm-d EPP OTel SDK integration | TC-E2E-002 |
| KServe controller lifecycle log emission | TC-E2E-001 |
| odh-observability Monitoring CR reconciliation | TC-E2E-001, TC-E2E-003 |
| odh-observability dependency status management | TC-E2E-003 |
| odh-observability logging resource creation | TC-E2E-001, TC-E2E-003 |
| Monitoring CR logging subsection | TC-E2E-003 |
| Loki log ingestion endpoint | TC-E2E-001, TC-E2E-005 |
| Loki multi-tenant isolation | TC-E2E-004 |

---

## 7. Non-Functional Requirements

Each category below must be explicitly addressed. If a category
does not apply to this feature, state **Not Applicable** with a
brief justification.

### 7.1 Disconnected/Air-Gapped

Validate that external operator dependencies (Loki Operator, and CLO
if Path B collection is chosen) can be installed in disconnected
OpenShift environments with mirrored operator catalogs. Verify that the
logging collection pipeline itself does not introduce runtime
dependencies on external registries or network-accessible resources
beyond the in-cluster Loki backend. Test that structured logs are
emitted and collected successfully when all images are pulled from a
disconnected registry.

### 7.2 Upgrade/Migration

Validate upgrade scenarios where logging is enabled on an existing
RHOAI deployment. Test Monitoring CR schema extensions for backwards
compatibility -- clusters without logging configuration should continue
to operate unchanged. Verify that components can migrate from
unstructured text logs to structured JSON logs without disrupting
existing `kubectl logs` workflows (`LOG_FORMAT` toggle). Test that
Monitoring CR schema changes coordinated with RHAISTRAT-2416 do not
conflict or create duplicate collection resources. Validate rollback
scenarios where logging is disabled after being enabled.

### 7.3 Performance/Scalability

Measure JSON serialization latency added by structured logging to
ensure no measurable regression in inference request latency (strategy
specifies "best effort, no measurable regression" bar per PM decision).
Validate log collection pipeline throughput under realistic RPS loads
(100, 500, 1000 RPS across 4 components generating estimated
~350GB/day at peak). Test Loki ingest capacity and query performance
under sustained high log volume. Verify that log emission does not
degrade inference path performance when content logging is disabled
(default). Conduct Loki sizing proof-of-concept to produce retention
and storage recommendations for production deployments.

### 7.4 RBAC/Authorization

Validate that Loki tenant mapping aligns with Kubernetes namespace
RBAC to prevent cross-tenant log exposure in model-as-a-service
deployments (AC 13). Verify that MaaS tenants querying Loki cannot
access logs from other tenant namespaces or platform-internal logs.
Test that OTel collector or CLO service accounts have correctly scoped
permissions (`lokistack-application-logs-writer` ClusterRoleBinding).
Ensure that namespace-scoped users can only query logs from namespaces
they have access to via standard Kubernetes RBAC.

---

## 8. Risks and Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Collection pipeline approach uncertainty (OTel Path A vs CLO Path B) -- choice determines external dependencies and architecture | High | High | Sprint-zero joint evaluation with logging team and Platform/Observability team. Prefer Path A if viable. If Path B, abstract resource creation behind version-aware factory. |
| RHAISTRAT-2416 Monitoring CR schema conflict -- both strategies extend same CR for logging configuration | High | High | Joint schema design session during sprint-zero. Single coordinated logging subsection in Monitoring CR spec. |
| Trace context propagation gaps -- W3C traceparent may be stripped in Gateway -> EPP -> vLLM chain | High | Medium | Validate end-to-end traceparent propagation during sprint 1. Add forwarding to any component that does not propagate it. |
| RHAII coordination for vLLM queue lifecycle instrumentation -- requires scheduler-level changes in fork-based multiprocessing internals | Medium | Medium | Phase work: (1) JSON formatter wrapper (low risk), (2) queue lifecycle instrumentation (higher risk, scheduler expertise). Engage RHAII early. |
| Log volume overload at high RPS -- ~350GB/day at 1000 RPS could overwhelm Loki ingest capacity | High | Medium | Default log levels to INFO; content logging gated behind flag. Conduct Loki sizing PoC before GA rollout. |
| Multi-tenant log isolation failure -- incorrect Loki tenant mapping could expose cross-tenant logs | High | Low | Align Loki tenancy with Kubernetes namespace RBAC. Include isolation validation in acceptance tests (AC 13). |
| OOM-kill log loss -- kernel-buffered stdout may not flush before process termination | Medium | Medium | Document limitation. Correlate with OOM-kill kubelet events and DCGM metrics. Queue lifecycle logs emit at enqueue time. |
| GenAI semconv field instability -- experimental OTel field names may change | Low | Medium | Centralize field name constants per component for coordinated rename. |
| Console log-to-trace navigation may require new plugin work | Medium | Medium | Validate Console plugin configuration for Tempo datasource in sprint 1. Scope as separate task if needed. |

---

## 9. Test Environment Requirements

### 9.1 Infrastructure

- **Multi-tenant cluster**: Namespace isolation for llm-d-as-a-service
  and MaaS tenancy models
- **GPU-equipped cluster**: Weekly cadence for KV cache pressure tests
  (AC 6); provisioned via Hive, shared across inference test suite
- **Shared QE cluster**: Non-GPU tests with per-test-run namespace
  isolation (multi-tenant, persistent)
- **Inference pool**: Multiple vLLM engine pods for pod-specific log
  filtering (AC 10)
- **LokiStack deployment**: Via Loki Operator (OpenShift-managed mode)
- **Tempo deployment**: For distributed trace storage and log-to-trace
  correlation
- **Collection agent**: OTel collector (Path A) or CLO v6 (Path B) --
  determined by sprint-zero evaluation

### 9.2 Configuration

- **Environment variables**:
  - `LOG_FORMAT=json` / `LOG_FORMAT=text` (vLLM structured vs
    unstructured output)
  - `VLLM_LOG_REQUESTS_CONTENT=true` / `false` (opt-in content
    logging)
  - `VLLM_WORKER_MULTIPROC_METHOD=fork` (vLLM process model)
- **Custom resources**:
  - Monitoring CR (`services.platform.opendatahub.io/v1alpha1`)
    with logging subsection -- schema TBD in sprint-zero,
    coordinated with RHAISTRAT-2416
  - DSCInitialization for odh-observability lifecycle
  - LLMInferenceServiceConfig template for surfacing
    `VLLM_LOG_REQUESTS_CONTENT` flag (overlay 0022)
- **Operator subscriptions**: Loki Operator, Tempo Operator,
  OpenTelemetry Operator, and CLO (if Path B)
- **Loki multi-tenancy**: Namespace-to-tenant ID mapping via logging
  pipeline resources

### 9.3 Test Tools

- **Kubernetes tools**: `kubectl`, `oc` for pod logs, resource
  inspection; `kustomize` for manifest management
- **Log viewing and query**: OpenShift Console Observe > Logs interface
  (Loki query with LogQL filtering)
- **Log parsing and validation**: `jq` for JSON schema conformance
  validation
- **API testing**: `curl` or `grpcurl` for triggering inference
  requests with trace context injection
- **Trace visualization**: Tempo UI for viewing distributed traces and
  validating log-to-trace correlation
- **Stdout inspection**: `kubectl logs` for direct container log access
  (fallback when collection pipeline unavailable)
- **Performance testing**: High-RPS load generation for Loki sizing PoC
  (100, 500, 1000 RPS tiers)
- **Scenario triggers**: Python scripts or Go test harnesses for
  quota-exceeded, KV cache pressure, and multi-tenant isolation
  scenarios
- **Schema validators**: OTel Logs Data Model and telemetry contract
  (RHAIRFE-2227) conformance testing

---

## 10. Appendix

### 10.1 Test Case Summary

| Category | Total | P0 | P1 | P2 |
|----------|-------|----|----|-----|
| TC-SCHEMA | 6 | 3 | 3 | 0 |
| TC-EMIT | 4 | 3 | 1 | 0 |
| TC-QUEUE | 4 | 4 | 0 | 0 |
| TC-ROUTE | 5 | 0 | 5 | 0 |
| TC-PIPE | 5 | 4 | 1 | 0 |
| TC-DEP | 4 | 3 | 1 | 0 |
| TC-QUERY | 5 | 0 | 5 | 0 |
| TC-TRACE | 4 | 1 | 3 | 0 |
| TC-TENANT | 3 | 1 | 2 | 0 |
| TC-PRIV | 3 | 0 | 3 | 0 |
| TC-COMPAT | 3 | 0 | 0 | 3 |
| TC-E2E | 5 | 5 | 0 | 0 |
| **Total** | **51** | **24** | **24** | **3** |

### 10.2 Component Coverage

| Component / Method | Test Cases | Coverage |
|--------------------|------------|----------|
| vLLM JSON log formatter | TC-SCHEMA-001, TC-EMIT-001, TC-E2E-001, TC-E2E-002 | |
| vLLM queue lifecycle event emission | TC-QUEUE-001, TC-QUEUE-002, TC-QUEUE-003, TC-QUEUE-004, TC-E2E-005 | |
| llm-d EPP OTel SDK integration | TC-SCHEMA-002, TC-EMIT-002, TC-TRACE-001, TC-E2E-002 | |
| KServe controller lifecycle log emission | TC-SCHEMA-004, TC-EMIT-004, TC-E2E-001 | |
| odh-observability Monitoring CR reconciliation | TC-PIPE-001, TC-PIPE-004, TC-E2E-003 | |
| odh-observability dependency status management | TC-DEP-001, TC-DEP-002, TC-DEP-003, TC-DEP-004, TC-E2E-003 | |
| odh-observability logging resource creation | TC-PIPE-002, TC-PIPE-003, TC-PIPE-005, TC-E2E-001, TC-E2E-003 | |
| Monitoring CR logging subsection | TC-PIPE-001, TC-PIPE-004, TC-E2E-003 | |
| Loki log ingestion endpoint | TC-E2E-001, TC-E2E-005 | |
| Loki multi-tenant isolation | TC-TENANT-001, TC-TENANT-002, TC-TENANT-003, TC-E2E-004 | |
| llm-d EPP routing decision log emission | TC-ROUTE-001, TC-ROUTE-002 | |
| MaaS routing rejection log emission | TC-ROUTE-003, TC-ROUTE-004 | |
| MaaS tenant/token log emission | TC-EMIT-003, TC-ROUTE-005, TC-E2E-004 | |
| vLLM GenAI semconv field emission | TC-SCHEMA-005, TC-SCHEMA-006 | |
| vLLM opt-in content logging | TC-PRIV-001, TC-PRIV-002 | |
| LLMInferenceServiceConfig content logging flag | TC-PRIV-003 | |
| Loki log query API (via Console) | TC-QUERY-001, TC-QUERY-002, TC-QUERY-003, TC-QUERY-004, TC-QUERY-005 | |
| OpenShift Console Observe > Logs interface | TC-QUERY-001, TC-QUERY-002, TC-QUERY-003, TC-QUERY-004, TC-QUERY-005 | |
| OpenShift Console log-to-trace navigation | TC-TRACE-003, TC-TRACE-004, TC-E2E-002 | |
| vLLM `LOG_FORMAT` env var | TC-COMPAT-001, TC-COMPAT-002, TC-COMPAT-003 | |

### 10.3 Document Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-08-17 | Initial test plan |

---

**End of Test Plan**
