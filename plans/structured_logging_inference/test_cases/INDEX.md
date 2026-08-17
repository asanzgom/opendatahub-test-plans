# Test Cases Index — Structured Logging for Inference Components

**Test Plan**: [TestPlan.md](../TestPlan.md)
**Source**: [RHAISTRAT-2413](https://redhat.atlassian.net/browse/RHAISTRAT-2413)

## Quick Stats

| Metric | Count |
|--------|-------|
| Total Test Cases | 51 |
| P0 (Critical) | 24 |
| P1 (High) | 24 |
| P2 (Medium) | 3 |

---

## Log Schema Conformance (TC-SCHEMA)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-SCHEMA-001](TC-SCHEMA-001.md) | vLLM structured JSON log schema conformance | P0 |
| [TC-SCHEMA-002](TC-SCHEMA-002.md) | llm-d EPP structured JSON log schema conformance | P0 |
| [TC-SCHEMA-003](TC-SCHEMA-003.md) | MaaS structured JSON log schema conformance | P1 |
| [TC-SCHEMA-004](TC-SCHEMA-004.md) | KServe controller lifecycle log schema conformance | P0 |
| [TC-SCHEMA-005](TC-SCHEMA-005.md) | GenAI semantic convention fields on vLLM completion logs | P1 |
| [TC-SCHEMA-006](TC-SCHEMA-006.md) | GenAI semconv fields absent on failed requests | P1 |

## Log Emission (TC-EMIT)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-EMIT-001](TC-EMIT-001.md) | vLLM emits structured logs on inference request | P0 |
| [TC-EMIT-002](TC-EMIT-002.md) | EPP emits structured logs on routing decisions | P0 |
| [TC-EMIT-003](TC-EMIT-003.md) | MaaS emits structured log on successful tenant request | P1 |
| [TC-EMIT-004](TC-EMIT-004.md) | KServe controller emits lifecycle logs | P0 |

## Queue Lifecycle (TC-QUEUE)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-QUEUE-001](TC-QUEUE-001.md) | vLLM emits structured enqueue log when request enters queue | P0 |
| [TC-QUEUE-002](TC-QUEUE-002.md) | vLLM emits structured dequeue log when request leaves queue | P0 |
| [TC-QUEUE-003](TC-QUEUE-003.md) | vLLM emits discard log with KV cache pressure reason | P0 |
| [TC-QUEUE-004](TC-QUEUE-004.md) | Queue lifecycle logs include engine identity for per-engine debugging | P0 |

## Routing and Rejection (TC-ROUTE)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-ROUTE-001](TC-ROUTE-001.md) | EPP routing decision log includes engine selected and selection reason | P1 |
| [TC-ROUTE-002](TC-ROUTE-002.md) | EPP routing decision log includes queue depth and KV-cache utilization | P1 |
| [TC-ROUTE-003](TC-ROUTE-003.md) | MaaS routing rejection log on quota exceeded | P1 |
| [TC-ROUTE-004](TC-ROUTE-004.md) | MaaS routing rejection log on model unavailable | P1 |
| [TC-ROUTE-005](TC-ROUTE-005.md) | MaaS tenant/token log for chargeback | P1 |

## Collection Pipeline (TC-PIPE)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-PIPE-001](TC-PIPE-001.md) | Logging collection resources created when Monitoring CR enables logging | P0 |
| [TC-PIPE-002](TC-PIPE-002.md) | Collection resources select logs from inference component namespaces | P0 |
| [TC-PIPE-003](TC-PIPE-003.md) | Collection resources apply JSON parsing to container stdout | P0 |
| [TC-PIPE-004](TC-PIPE-004.md) | Logging resources absent when Monitoring CR does not enable logging | P0 |
| [TC-PIPE-005](TC-PIPE-005.md) | Collection pipeline is additive to existing observability | P1 |

## Dependency Status (TC-DEP)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-DEP-001](TC-DEP-001.md) | Monitoring CR status indicates missing Loki backend | P0 |
| [TC-DEP-002](TC-DEP-002.md) | Monitoring CR status indicates unhealthy Loki backend | P0 |
| [TC-DEP-003](TC-DEP-003.md) | Inference workloads continue when logging dependencies missing | P0 |
| [TC-DEP-004](TC-DEP-004.md) | Status condition distinguishes "not installed" from "misconfigured" | P1 |

## Log Query (TC-QUERY)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-QUERY-001](TC-QUERY-001.md) | Query logs by service.name in OpenShift Console | P1 |
| [TC-QUERY-002](TC-QUERY-002.md) | Query logs by k8s.pod.name to isolate engine logs | P1 |
| [TC-QUERY-003](TC-QUERY-003.md) | Query logs by severity level | P1 |
| [TC-QUERY-004](TC-QUERY-004.md) | Query logs by time range | P1 |
| [TC-QUERY-005](TC-QUERY-005.md) | Free text search in logs | P1 |

## Trace Correlation (TC-TRACE)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-TRACE-001](TC-TRACE-001.md) | trace_id present in log entries when traceparent available | P0 |
| [TC-TRACE-002](TC-TRACE-002.md) | trace_id and span_id omitted when no traceparent | P1 |
| [TC-TRACE-003](TC-TRACE-003.md) | Cross-component trace correlation via trace_id | P1 |
| [TC-TRACE-004](TC-TRACE-004.md) | Log-to-trace navigation in OpenShift Console | P1 |

## Multi-Tenant Isolation (TC-TENANT)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-TENANT-001](TC-TENANT-001.md) | Tenant A cannot see Tenant B's logs | P0 |
| [TC-TENANT-002](TC-TENANT-002.md) | Namespace admin sees all inference logs in their namespace | P1 |
| [TC-TENANT-003](TC-TENANT-003.md) | Platform team has cross-namespace log access | P1 |

## Privacy Controls (TC-PRIV)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-PRIV-001](TC-PRIV-001.md) | No prompt or completion content in logs by default | P1 |
| [TC-PRIV-002](TC-PRIV-002.md) | Content logging enabled includes prompt and completion | P1 |
| [TC-PRIV-003](TC-PRIV-003.md) | Content logging flag surfaceable via LLMInferenceServiceConfig | P1 |

## Backwards Compatibility (TC-COMPAT)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-COMPAT-001](TC-COMPAT-001.md) | LOG_FORMAT=text produces unstructured output | P2 |
| [TC-COMPAT-002](TC-COMPAT-002.md) | LOG_FORMAT=json (default) produces structured output | P2 |
| [TC-COMPAT-003](TC-COMPAT-003.md) | kubectl logs works regardless of log format | P2 |

## End-to-End Scenarios (TC-E2E)

| Test Case | Title | Priority |
|-----------|-------|----------|
| [TC-E2E-001](TC-E2E-001.md) | End-to-end structured log emission and collection | P0 |
| [TC-E2E-002](TC-E2E-002.md) | Cross-component trace correlation through inference path | P0 |
| [TC-E2E-003](TC-E2E-003.md) | Collection pipeline lifecycle and dependency surfacing | P0 |
| [TC-E2E-004](TC-E2E-004.md) | Multi-tenant log isolation in MaaS deployment | P0 |
| [TC-E2E-005](TC-E2E-005.md) | vLLM queue lifecycle debugging workflow | P0 |
