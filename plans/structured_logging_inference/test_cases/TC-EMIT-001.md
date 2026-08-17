---
test_case_id: TC-EMIT-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-EMIT-001: vLLM emits structured logs on inference request

**Objective**: Verify that vLLM emits a structured JSON log entry to
stdout when processing an inference request, with all required OTel
resource attributes populated.

**Preconditions**:
- OpenShift 4.16+ cluster with RHOAI 3.6 installed
- LLMInferenceService deployed with vLLM runtime in namespace
  `inference-test-ns` using model `granite-3b-instruct`
- vLLM configured with `LOG_FORMAT=json`
- No other requests in flight (to isolate log entries)

**Test Steps**:
1. Record the current log tail position:
   ```bash
   VLLM_POD=$(kubectl get pods -n inference-test-ns \
     -l app=granite-3b-instruct \
     -o jsonpath='{.items[0].metadata.name}')
   LOG_LINES_BEFORE=$(kubectl logs -n inference-test-ns "$VLLM_POD" \
     | wc -l)
   ```
2. Send an inference request:
   ```bash
   curl -s -X POST \
     "https://granite-3b-instruct-inference-test-ns.apps.cluster.example.com/v1/completions" \
     -H "Content-Type: application/json" \
     -d '{"model": "granite-3b-instruct", "prompt": "List three planets.", "max_tokens": 30}'
   ```
3. Wait briefly for log emission, then capture new log entries:
   ```bash
   sleep 2
   kubectl logs -n inference-test-ns "$VLLM_POD" \
     --tail=$(($(kubectl logs -n inference-test-ns "$VLLM_POD" \
       | wc -l) - LOG_LINES_BEFORE))
   ```
4. Verify at least one new log entry is valid JSON with all
   required fields:
   ```bash
   kubectl logs -n inference-test-ns "$VLLM_POD" \
     --tail=5 | jq -e '
       .timestamp != null and
       .severity != null and
       .body != null and
       .["service.name"] != null and
       .["k8s.pod.name"] != null and
       .["k8s.namespace.name"] != null
     ' 2>/dev/null
   ```

**Expected Results**:
- At least one new JSON log entry appears in vLLM stdout after the
  inference request
- The entry contains `timestamp`, `severity`, `body`,
  `service.name`, `k8s.pod.name`, and `k8s.namespace.name`
- `k8s.pod.name` matches the actual pod name (`$VLLM_POD`)
- `k8s.namespace.name` equals `inference-test-ns`
- The log entry body references the inference request processing

**Notes**: To be filled later in the process.
