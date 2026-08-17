---
feature: structured_logging_inference
source_key: RHAISTRAT-2413
score: 10
pass: true
verdict: Ready
scores:
  specificity: 2
  grounding: 2
  scope_fidelity: 2
  actionability: 2
  consistency: 2
last_updated: '2026-08-17'
auto_revised: false
before_score: 10
before_scores:
  specificity: 2
  grounding: 2
  scope_fidelity: 2
  actionability: 2
  consistency: 2
error: null
---
## Rubric Scores

| Criterion | Score | Notes |
|-----------|-------|-------|
| Specificity | 2/2 | Priority definitions name feature-specific scenarios (vLLM queue lifecycle, EPP routing decisions, Monitoring CR dependency surfacing). Risks reference failure modes unique to this feature (RHAISTRAT-2416 schema conflict, OOM-kill log loss, GenAI semconv instability, collection pipeline approach uncertainty). Swap test passed for all risks. |
| Grounding | 2/2 | All 20 Section 4 entries traceable to specific strategy sentences. No fabricated endpoint paths or invented API signatures. Unknowns correctly marked as TBD with resolution context (e.g., "schema pending sprint-zero, coordinated with RHAISTRAT-2416"). |
| Scope Fidelity | 2/2 | Every strategy deliverable maps to at least one test objective. Out-of-scope items match strategy exactly (vendor gateway logging, DCGM, kserve-agent sidecar, training workloads, CLO/Loki auto-install). No orphans in either direction. |
| Actionability | 2/2 | OCP 4.16+, RHOAI 3.6, Go 1.25 specified. External operators listed with conditionality. Test users have defined roles with specific permissions (cluster admin, namespace admin, MaaS tenants, service accounts with named ClusterRoleBindings). Test data includes concrete scenario triggers (AC 3, AC 6, AC 13). TBDs include resolution paths. |
| Consistency | 2/2 | All six cross-checks pass. Section 10.2 lists all 20 Section 4 entries. Section 6 has placeholder text (pre-create-cases, acceptable). NFR categories all addressed with feature-specific content. Minor note: llm-d EPP routing decision log emission is P1 in Section 4 but named in P0 definition — borderline, since EPP OTel SDK integration IS P0 in Section 4. |

**Total: 10/10 — Verdict: Ready**

## Grounding Cross-Reference

| Section 4 Entry | Source Match | Status |
|-----------------|-------------|--------|
| vLLM JSON log formatter | "Structured JSON output is configurable via environment variable (e.g., `LOG_FORMAT=json`)" and "The formatter itself is a JSON wrapper around Python's standard logging module" | Grounded |
| vLLM queue lifecycle event emission | "[P0] vLLM emits structured request queue lifecycle logs (enqueue, dequeue, timeout/discard with reason and engine identity)" | Grounded |
| llm-d EPP OTel SDK integration | "The EPP already integrates with OpenTelemetry SDK (>= 1.39.1) and exports traces to an OTLP collector on port 4317" | Grounded |
| KServe controller lifecycle log emission | "Controller-level lifecycle logs are standardized to the OTel schema for operational debugging. These cover LLMInferenceService reconciliation events, status transitions, error conditions" | Grounded |
| odh-observability Monitoring CR reconciliation | "[P0] Platform monitoring stack creates logging collection resources routing inference namespace logs to Loki" | Grounded |
| odh-observability dependency status management | "[P0] Platform surfaces a clear status condition on Monitoring CR when required logging dependencies are not installed or unhealthy" | Grounded |
| odh-observability logging resource creation | "Path A — Extend existing OTel-to-LokiStack pipeline" / "Path B — Add CLO-based collection" | Grounded |
| Monitoring CR logging subsection | "The Monitoring CR must be extended with a logging configuration subsection" | Grounded |
| Loki log ingestion endpoint | "routing logs to Loki for storage and query" and "Forward parsed logs to the configured Loki backend endpoint" | Grounded |
| Loki multi-tenant isolation | "Loki multi-tenancy is configured via tenant ID injection in the logging pipeline resources, mapping Kubernetes namespaces to Loki tenants" | Grounded |
| llm-d EPP routing decision log emission | "[P0] llm-d EPP emits structured routing decision logs (engine selected, selection reason, queue depth, KV-cache utilization)" | Grounded |
| MaaS routing rejection log emission | "[P1] MaaS emits structured routing rejection and tenant/token logs for capacity governance" | Grounded |
| MaaS tenant/token log emission | "per-request token counts... enable both the auth/rate-limit denial debugging use case and the chargeback/showback use case" | Grounded |
| vLLM GenAI semconv field emission | "[P1] Inference log entries include GenAI semconv fields where the component has the information" | Grounded |
| vLLM opt-in content logging | "[P1] Opt-in request/response content logging for vLLM, gated behind explicit configuration flag" | Grounded |
| LLMInferenceServiceConfig content logging flag | "The flag is surfaceable through the LLMInferenceServiceConfig template so it can be set per-model or fleet-wide via the versioned base config mechanism (overlay 0022)" | Grounded |
| Loki log query API (via Console) | "Users query logs through the OpenShift Console's Observe > Logs interface, which supports filtering by service.name... k8s.pod.name... trace_id... severity, time range" | Grounded |
| OpenShift Console Observe > Logs interface | "Users query logs through the OpenShift Console's Observe > Logs interface" | Grounded |
| OpenShift Console log-to-trace navigation | "[P1] Users can query logs by trace_id and navigate to the corresponding trace in Tempo" | Grounded |
| vLLM LOG_FORMAT env var | "Existing unstructured text log output remains available via configuration toggle (e.g., LOG_FORMAT=text) for development" | Grounded |

## Section-by-Section Feedback

All criteria passed — no improvements needed.

## Revision History

Initial assessment
