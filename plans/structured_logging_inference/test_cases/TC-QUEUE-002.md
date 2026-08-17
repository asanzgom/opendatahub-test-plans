---
test_case_id: TC-QUEUE-002
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUEUE-002: vLLM emits structured dequeue log when request leaves queue

**Objective**: Verify that vLLM emits a structured JSON log entry with
a dequeue event when an inference request is picked up from the queue
for processing.

**Preconditions**:
- LLMInferenceService deployed with a vLLM runtime and
  `LOG_FORMAT=json`
- W3C traceparent header propagation is active
- `kubectl logs` access to the vLLM engine pod

**Test Steps**:
1. Identify a vLLM engine pod:
   ```bash
   VLLM_POD=$(oc get pods -n $INFERENCE_NS \
     -l app=vllm-engine -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send an inference request with a traceparent header:
   ```bash
   curl -H "traceparent: 00-11223344556677889900aabbccddeeff-1122334455667788-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "granite-3b", "prompt": "Explain quantum computing:", "max_tokens": 100}' \
        https://$INFERENCE_ENDPOINT/v1/completions
   ```
3. Wait for the request to complete successfully (HTTP 200).
4. Retrieve the vLLM engine pod logs and filter for dequeue events:
   ```bash
   oc logs $VLLM_POD -n $INFERENCE_NS | \
     jq -c 'select(.body | test("dequeue"; "i"))'
   ```
5. Validate that the dequeue log entry contains the required fields.

**Expected Results**:
- vLLM pod logs contain a JSON log entry with a `body` field
  referencing a dequeue event
- The log entry includes `timestamp` in ISO 8601 format
- The log entry includes `k8s.pod.name` matching `$VLLM_POD`
- The log entry includes `trace_id` matching
  `11223344556677889900aabbccddeeff`
- The dequeue log entry appears chronologically after the
  corresponding enqueue log entry for the same `trace_id`

**Notes**: To be filled later in the process.
