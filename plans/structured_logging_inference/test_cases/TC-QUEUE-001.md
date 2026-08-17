---
test_case_id: TC-QUEUE-001
source_key: RHAISTRAT-2413
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-17"
---
# TC-QUEUE-001: vLLM emits structured enqueue log when request enters queue

**Objective**: Verify that vLLM emits a structured JSON log entry with
an enqueue event, engine identity (`k8s.pod.name`), and `trace_id` when
an inference request enters the request queue.

**Preconditions**:
- LLMInferenceService deployed with a vLLM runtime and
  `LOG_FORMAT=json` (default for RHOAI)
- W3C traceparent header propagation is active through the inference
  path
- `kubectl logs` access to the vLLM engine pod

**Test Steps**:
1. Identify a vLLM engine pod in the inference pool:
   ```bash
   VLLM_POD=$(oc get pods -n $INFERENCE_NS \
     -l app=vllm-engine -o jsonpath='{.items[0].metadata.name}')
   ```
2. Send an inference request with a W3C traceparent header:
   ```bash
   curl -H "traceparent: 00-abcdef1234567890abcdef1234567890-fedcba0987654321-01" \
        -H "Content-Type: application/json" \
        -d '{"model": "granite-3b", "prompt": "Summarize:", "max_tokens": 50}' \
        https://$INFERENCE_ENDPOINT/v1/completions
   ```
3. Retrieve the vLLM engine pod logs and filter for enqueue events:
   ```bash
   oc logs $VLLM_POD -n $INFERENCE_NS | \
     jq -c 'select(.body | test("enqueue"; "i"))'
   ```
4. Parse the enqueue log entry and validate required fields.

**Expected Results**:
- vLLM pod logs contain a JSON log entry with a `body` field
  referencing an enqueue event
- The log entry includes `timestamp` in ISO 8601 format
- The log entry includes `severity` set to an OTel severity text value
- The log entry includes `k8s.pod.name` matching the vLLM engine pod
  name (`$VLLM_POD`)
- The log entry includes `trace_id` matching
  `abcdef1234567890abcdef1234567890` from the traceparent header
- The log entry parses as valid JSON (not mixed with unstructured text)

**Notes**: To be filled later in the process.
