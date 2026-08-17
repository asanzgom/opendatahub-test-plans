---
test_case_id: TC-EMIT-002
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-EMIT-002: EPP emits structured logs on routing decisions

**Objective**: Verify that the llm-d EPP emits a structured routing
decision log entry when it selects an engine for an inference
request, including the selected engine, selection reason, and queue
depth.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- Inference pool deployed with llm-d EPP in namespace
  `inference-test-ns`
- At least two vLLM engine pods registered with the pool (to
  exercise selection logic)

**Test Steps**:
1. Identify the EPP pod:
   ```bash
   EPP_POD=$(kubectl get pods -n inference-test-ns \
     -l app=epp -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send an inference request through the EPP:
   ```bash
   curl -s -X POST \
     "https://inference-pool-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b-instruct", "prompt": "Hello", "max_tokens": 10}'
   ```
3. Capture EPP logs and filter for routing decision entries:
   ```bash
   kubectl logs -n inference-test-ns "$EPP_POD" --tail=20 | \
     jq 'select(.body | test("rout|select|pick"; "i"))' 2>/dev/null
   ```
4. Validate routing decision fields are present:
   ```bash
   kubectl logs -n inference-test-ns "$EPP_POD" --tail=20 | \
     jq 'select(.body | test("rout|select|pick"; "i")) | {
       engine_selected: (.engine_selected // .selected_engine),
       selection_reason: (.selection_reason // .reason),
       queue_depth: (.queue_depth)
     }' 2>/dev/null
   ```

**Expected Results**:
- At least one routing decision log entry appears after the
  request
- The entry identifies which engine pod was selected (by pod name
  or endpoint)
- The entry includes the selection reason (e.g., lowest queue
  depth, prefix cache hit, predicted latency)
- The entry includes the queue depth at decision time as a numeric
  value
- All standard OTel fields (`timestamp`, `severity`,
  `service.name`, `k8s.pod.name`, `k8s.namespace.name`) are
  present

**Notes**: To be filled later in the process.
