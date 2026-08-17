---
test_case_id: TC-ROUTE-001
source_key: RHAISTRAT-2413
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-ROUTE-001: EPP routing decision log includes engine selected and selection reason

**Objective**: Verify that llm-d EPP emits a structured JSON routing
decision log entry containing the selected engine pod, the selection
reason, and scheduling metrics when routing an inference request.

**Preconditions**:
- Inference pool deployed with llm-d EPP and at least 2 vLLM engine
  pods
- EPP configured with OTel SDK integration (>= 1.39.1)
- `kubectl logs` access to the EPP pod

**Test Steps**:
1. Identify the EPP pod:
   ```bash
   EPP_POD=$(oc get pods -n $INFERENCE_NS \
     -l app=llm-d-epp -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send an inference request through the EPP:
   ```bash
   curl -H "traceparent: 00-ff0011223344556677889900aabbccdd-aabbccdd00112233-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "granite-3b", "prompt": "List three planets:", "max_tokens": 50}' \
        https://$INFERENCE_ENDPOINT/v1/completions
   ```
3. Retrieve EPP pod logs and filter for routing decision entries:
   ```bash
   oc logs $EPP_POD -n $INFERENCE_NS | \
     jq -c 'select(.body | test("routing|selected|route"; "i"))'
   ```
4. Validate the routing decision log entry contains the required
   fields.

**Expected Results**:
- EPP pod logs contain a JSON routing decision log entry
- The log entry includes the selected engine pod name
- The log entry includes a selection reason (e.g., "lowest queue
  depth", "prefix cache hit", "predicted latency")
- The log entry includes `trace_id` matching
  `ff0011223344556677889900aabbccdd`
- The log entry includes `service.name` identifying the EPP component

**Notes**: To be filled later in the process.
